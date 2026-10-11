# Installing the Java Kernel in VS Code
by Engr. EJ S. Ramirez

This guide explains how to use Java in Jupyter Notebooks inside Visual Studio Code on a new Windows computer.

It assumes that the computer currently has:

- Visual Studio Code
- The Extension Pack for Java (or another Java extension from Microsoft)
- No Python
- No Jupyter installation
- No IJava Java kernel

The instructions use the prebuilt **IJava 1.3.0** distribution. This is recommended because building the source project uses Gradle 4.8.1, which is not compatible with modern Java versions such as Java 25.

## 1. Install the Java JDK

The Java kernel requires a **JDK**, not only a JRE. Java 9 or newer is required. This guide uses the **JDK 25 LTS** installer, which provides the Java runtime and JShell required by IJava.

Install the **JDK 25 LTS** package, such as Eclipse Temurin, from:

<https://adoptium.net/temurin/releases/>

Choose:

- Operating system: Windows
- Architecture: x64, unless your computer uses a different architecture
- Version: 25 (LTS)
- Package: JDK

After installation, open a new PowerShell window and verify Java:

```powershell
java -version
```

Verify that JShell is included:

```powershell
java --list-modules | findstr "jdk.jshell"
```

The second command should display a module beginning with `jdk.jshell@`.

If `java` is not recognized, restart PowerShell or add the JDK `bin` directory to the Windows `PATH`.

## 2. Install Python 3.14

Download the **Python 3.14** Windows installer from:

<https://www.python.org/downloads/windows/>

Choose the Python 3.14 installer appropriate for your computer. During installation, enable:

```text
Add python.exe to PATH
```

After installation, close and reopen PowerShell. Verify Python:

```powershell
py --version
```

The output should begin with `Python 3.14`.

If the `py` command is unavailable but `python` works, use `python` instead of `py` in the commands below. Verify that it reports Python 3.14:

```powershell
python --version
```

## 3. Install Jupyter and its kernel tools

Install Jupyter, Jupyter Notebook, and `jupyter-client`:

```powershell
py -m pip install --upgrade pip
py -m pip install --upgrade jupyter jupyter-client notebook
```

The `jupyter-client` package is important because it provides the `kernelspec` command used to install and inspect Java kernels.

Verify the installation:

```powershell
py -m jupyter --version
py -m jupyter kernelspec list
```

If the second command reports that `jupyter-kernelspec` is not found, install the missing package explicitly:

```powershell
py -m pip install --upgrade jupyter-client
```

Then retry:

```powershell
py -m jupyter kernelspec list
```

Using `py -m jupyter` is recommended instead of calling `jupyter` directly. It ensures that Jupyter is run from the same Python installation where it was installed.

## 4. Download the prebuilt IJava distribution

Download the `ijava-1.3.0.zip` artifact from the IJava releases page:

<https://github.com/SpencerPark/IJava/releases>

Extract the ZIP file to a convenient directory. For example:

```text
C:\ijava-1.3.0
```

The extracted directory must contain at least:

```text
install.py
java\
```

The `java` directory should contain the IJava JAR and `kernel.json`.

## 5. Install the Java kernel

Open PowerShell and change to the extracted IJava directory. Adjust the path if you extracted it elsewhere:

```powershell
Set-Location "C:\ijava-1.3.0"
```

Install the kernel for the current Windows user:

```powershell
py .\install.py --user --replace
```

The `--user` option avoids requiring administrator privileges. The `--replace` option replaces an existing Java kernel registration if one is already present.

Expected output is similar to:

```text
Installed java kernel into "C:\Users\<username>\AppData\Roaming\jupyter\kernels\java"
```

## 6. Verify the Java kernel

Run:

```powershell
py -m jupyter kernelspec list
```

The output should contain an entry similar to:

```text
java    C:\Users\<username>\AppData\Roaming\jupyter\kernels\java
```

If an old or broken Java kernel is already registered, remove it and install again:

```powershell
py -m jupyter kernelspec remove java
py .\install.py --user --replace
```

Confirm the removal when prompted by entering `y`.

## 7. Install the Coding Pack for Java

If Visual Studio Code is not already installed, download and install it from:

<https://code.visualstudio.com/>

For Java-specific setup instructions, follow the official Visual Studio Code documentation:

<https://code.visualstudio.com/docs/languages/java>

That guide covers the Java development prerequisites and the Java extensions for VS Code. After completing it, continue with the extensions required for Jupyter notebooks below.

## 8. Install the VS Code extensions

The Java Pack for VS Code provides Java language and project support, but it does not install Python, Jupyter, or the Java Jupyter kernel.

In VS Code, install or enable:

1. **Extension Pack for Java**
2. **Python**
3. **Jupyter**

The Python and Jupyter extensions are required to open and run `.ipynb` files in VS Code.

## 9. Open a Java notebook in VS Code

Start Jupyter once to verify the installation:

```powershell
py -m jupyter notebook
```

You can close the browser window after confirming that Jupyter starts.

Then, in VS Code:

1. Create or open a file with the `.ipynb` extension.
2. Click **Select Kernel** in the upper-right corner.
3. Select **Java**.
4. Create a code cell.
5. Run the cell.

Test the kernel with:

```java
System.out.println("Hello from Java " + System.getProperty("java.version"));
```

A second test:

```java
int a = 10;
int b = 20;
a + b
```

The result should be:

```text
30
```

## 10. Troubleshooting

### `python` or `py` is not recognized

Python 3.14 is not installed or is not on `PATH`. Reinstall Python 3.14 and enable **Add python.exe to PATH**, then restart PowerShell.

### `jupyter` is not recognized

Use the module form:

```powershell
py -m jupyter notebook
```

If that fails, install Jupyter:

```powershell
py -m pip install --upgrade jupyter jupyter-client notebook
```

### `Jupyter command "jupyter-kernelspec" not found`

Install `jupyter-client`:

```powershell
py -m pip install --upgrade jupyter-client
```

Then check:

```powershell
py -m jupyter kernelspec list
```

### Java is not listed in VS Code

First verify that Jupyter can see it:

```powershell
py -m jupyter kernelspec list
```

If `java` is not listed, reinstall the kernel:

```powershell
Set-Location "C:\Tools\ijava-1.3.0"
py .\install.py --user --replace
```

If `java` is listed but VS Code does not show it:

1. Confirm that the **Python** and **Jupyter** extensions are installed.
2. Reload or restart VS Code.
3. Open the notebook again.
4. Click **Select Kernel** and search for `Java`.

### The Java kernel starts and immediately stops

Verify that the active Java command is a JDK containing JShell:

```powershell
Get-Command java
java -version
java --list-modules | findstr "jdk.jshell"
```

If multiple Java installations exist, make sure the desired JDK appears first in the Windows `PATH`.

### `gradlew installKernel` fails

Do not use the source build for this installation. The IJava source project uses Gradle 4.8.1, which is too old for JDK 25 and can fail with errors involving `ScriptPluginFactory`, `PluginResolutionStrategyInternal`, or Java module access.

Use the prebuilt installer instead:

```powershell
Set-Location "C:\Tools\ijava-1.3.0"
py .\install.py --user --replace
```

## Complete installation commands

After Python and the JDK are installed, the main command sequence is:

```powershell
py -m pip install --upgrade pip
py -m pip install --upgrade jupyter jupyter-client notebook

Set-Location "C:\Tools\ijava-1.3.0"
py .\install.py --user --replace

py -m jupyter kernelspec list
py -m jupyter notebook
```

After these steps, VS Code should be able to use **Java** as a Jupyter notebook kernel while JDK 25 remains the runtime used by the kernel.
