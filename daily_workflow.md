# Daily Workflow

Once the initial setup is complete, you **do not** need to repeat the installation steps every time you use the DYNAP-SE2 Stack.

Follow these steps whenever you want to start working with the board.

---

## 1. Connect the DYNAP-SE2 Stack

After connecting the board to a power supply, connect the DYNAP-SE2 Stack to your computer via USB.

---


## 2. Attach the board to WSL

**Windows PowerShell (Administrator)**

(To get to the Admin., right click or run in regular PowerShell: `Start-Process powershell -Verb RunAs`)

Find the BUSID of the board:

```powershell
usbipd list
```

Attach it to WSL:

```powershell
usbipd attach --wsl --busid <BUSID>
```

> **Note:** You only need to run `usbipd bind` once when setting up the device for the first time. For daily use, only the `attach` command is required.

---

## 3. Open the project

**Ubuntu (WSL Terminal)**

Navigate to your project directory:

```bash
cd /path/to/your/project
```

Activate the Conda environment (if needed):

```bash
conda activate dynapse_env_py310
```

Open the project in VS Code:

```bash
code .
```

---

## 4. Select the correct Python interpreter or Jupyter kernel

**Visual Studio Code**

- For **Python scripts (`.py`)**, select the Python interpreter from the `dynapse_env_py310` Conda environment.
- For **Jupyter notebooks (`.ipynb`)**, select the Jupyter kernel associated with the same environment.

---

## 5. Verify the connection

Run your connection test or FPGA configuration script.

If everything is configured correctly, you should see:

```text
Found 1 device(s):
  DYNAP-SE2 Stack
Board ready.
```

Your DYNAP-SE2 Stack is now ready to use.
