# DYNAP-SE2 Stack on Windows (WSL2) Setup Guide

A complete guide for connecting a SynSense DYNAP-SE2 Stack (Opal Kelly XEM7310-A75) to Windows using WSL2 and running it with samna.

## Why this guide exists

At the time of writing, most DYNAP-SE2 examples assume Linux or macOS. This guide documents the additional steps required to make the hardware work on Windows 10/11 using WSL2 (Ubuntu).

It also includes solutions to several issues that are not documented elsewhere.

## Tested setup
| Component         | Version                |
| ----------------- | ---------------------- |
| Windows           | Windows 11             |
| Ubuntu            | 24.04.4 LTS            |
| WSL               | WSL2                   |
| samna             | 0.38.3.0               |
| FPGA board        | Opal Kelly XEM7310-A75 |
| Device            | DYNAP-SE2 Stack        |
| FrontPanel Driver | 5.2.2                  |
| VS Code           | Latest                 |


## Prerequisites

Before starting, make sure you have:

- **Windows 10 or Windows 11**
- **WSL2** enabled
- **Ubuntu** installed in WSL2 (this guide was tested with Ubuntu 24.04 LTS)
- **Visual Studio Code**
- The **Remote - WSL** extension for VS Code
- **Miniconda** (or Anaconda) installed inside WSL2
- **Python 3.10+** and **pip** (included with Miniconda)
- **Administrator access** on Windows (required to install drivers and use `usbipd`)
- A **SynSense DYNAP-SE2 Stack** (Opal Kelly XEM7310-A75 FPGA)


## Software used in this guide

During the setup you will install:

| Software | Purpose |
|----------|---------|
| **FrontPanel USB Driver** | Allows Windows to recognize the Opal Kelly FPGA on the DYNAP-SE2 Stack. |
| **usbipd-win** | Shares USB devices from Windows with WSL2. |
| **samna** | Python library used to communicate with the DYNAP-SE2 hardware. |
| **VS Code + Remote - WSL** | Provides a development environment inside WSL2. |



## Step 1 – Install the Opal Kelly FrontPanel USB Driver

### 1.1 Download the driver

1. Open your web browser.
2. Download the **Opal Kelly FrontPanel USB Driver** (Version used in this guide: `FrontPanelUSB-DriverOnly-5.2.2.exe`). 
3. Save the installer to your computer.

> **Note:** The driver allows Windows to correctly recognize the FPGA on the DYNAP-SE2 Stack. Without it, Windows may identify the board as an unknown USB device, and communication with the FPGA may fail.

---

### 1.2 Install the driver

1. Double-click the downloaded `FrontPanelUSB-DriverOnly-5.2.2.exe` installer.
2. Follow the installation wizard using the default settings.
3. Restart your computer if prompted.
4. If your DYNAP-SE2 Stack was already connected, unplug it and plug it back in after the installation is complete.

---

### 1.3 Verify that the driver is installed correctly

1. Press **Windows + X** and select **Device Manager**.
2. Locate your connected DYNAP-SE2 Stack (it may appear under **FrontPanel devices** or **Universal Serial Bus devices** or a similar category).

If the installation was successful, Windows should recognize the board as:

```text
Opal Kelly XEM7310-A75
```

instead of Unkown/Other device or a device with a warning icon.

<img width="420" height="377" alt="image" src="https://github.com/user-attachments/assets/93e2f43d-cb70-48ea-ab34-dee9b0655bbe" />

> **Note:** If the XEM7310‑A75 does not appear in the device list, disconnect the board from both power and USB, wait a few seconds, and reconnect it. Windows sometimes needs a fresh enumeration before the device becomes visible.

> > **Note:** The Opal Kelly XEM7310‑A75 module (the green board on top of the Dynapse‑2 stack) has a small “power” indicator LED. It should glow green when the board is powered correctly. If the LED is off or, the issue is likely related to the power supply, cable, or even the board itself.



## Step 2 — Install usbipd-win

### 2.1 Download usbipd-win

1. Open your web browser.
2. Go to the usbipd-win GitHub releases page:
   https://github.com/dorssel/usbipd-win/releases
3. Download the latest `.msi` installer (for example, `usbipd-win-x.x.x.msi`).
4. Run the installer and complete the installation using the default settings.


### 2.2 Verify the installation

Open **PowerShell as Administrator** :
  i.	  Click Start menu
  ii.	  Type PowerShell
  iii.	Right-click Windows PowerShell
  iv.	  Choose Run as administrator (will open a blue window)
  v.	Alternatively, to open PowerShell as administrator, in normal Power Shell run: `Start-Process powershell -Verb RunAs`
 
In Administrator PowerShell run:

```powershell
usbipd list
```

If the installation was successful, you should see a list of USB devices (the list may be empty if no USB devices are connected).




## Step 3 – Share the USB device with WSL

After installing the FrontPanel USB driver, Windows can communicate with the DYNAP-SE2 Stack. However, since the Python code will run inside **Ubuntu (WSL2)**, the USB device must be shared with WSL before it can be accessed.

---

### 3.1 Find the USB device

In **PowerShell as Administrator** run:

```powershell
usbipd list
```

Locate the entry corresponding to your DYNAP-SE2 Stack. It should appear similar to:

```text
BUSID  VIP:PID    DEVICE                           STATE
3-1    152f:0130  FrontPanel Device XEM7310-A75    Not Shared
```

> **Note:** The **BUSID** uniquely identifies your USB device. Your BUSID may be different from the example shown above (for example, `2-4` or `4-1`). Make sure to use **your own BUSID** in the following commands.

---

### 3.2 Bind the device

Run:

```powershell
usbipd bind --busid 3-1
```

> **Note:** Replace `3-1` with your own BUSID.

Binding tells Windows that this USB device can be shared with WSL.

---

### 3.3 Attach the device to WSL

Run:

```powershell
usbipd attach --wsl --busid 3-1
```

> **Note:** Again, replace `3-1` with your own BUSID.

This command disconnects the device from Windows and makes it available inside your Ubuntu (WSL2) environment.

If the command completes successfully, your DYNAP-SE2 Stack is now connected to WSL and can be accessed by `samna`.

> **Important:** If you unplug and reconnect the board, or if Windows reclaims the device (for example after reinstalling the FrontPanel driver), you need to run the `attach` command again (so every time you connect the chip to you computer).

> **Note:** To disconnect from WSL, run:
  ```powershell
  usbipd detach --busid 3-1
  ```



## Step 4 — Verify Ubuntu sees it

Inside **Ubuntu** run:

```ubuntu
lsusb
```
If you get

```Command 'lsusb' not found```

install it:

```ubuntu
sudo apt install usbutils
```

Expected output:
```ubuntu
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 152f:0200 Opal Kelly Inc. XEM7310-A75
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```





## Step 5 – Configure USB permissions (udev rules)

By default, Ubuntu may not allow your user account to access USB devices directly. To allow `samna` to communicate with the DYNAP-SE2 Stack without requiring `sudo`, you need to create a **udev rule**.

---

### 5.1 Find your USB Vendor ID

After running in Ubuntu (WSL Terminal),

```bash
lsusb
```

locate the line corresponding to your DYNAP-SE2 Stack. For example:

```text
Bus 001 Device 002: ID 152f:XXXX FrontPanel Device
```

The first four hexadecimal digits (`152f` in this example) are the **Vendor ID**.

> **Note:** Make sure to check the Vendor ID value reported on **your** system.

---

### 5.2 Create a udev rule

Open a new udev rules file:

```bash
sudo nano /etc/udev/rules.d/99-opalkelly.rules
```
This will open the Nano editor.

Add the following line, replacing `152f` with your own Vendor ID if necessary:

```text
SUBSYSTEM=="usb", ATTR{idVendor}=="151f", MODE="0666"
```

This rule grants read and write access to the USB device for all users, allowing `samna` to communicate with the board.

Save the file:

- Press **Ctrl + O** to save.
- Press **Enter** to confirm the filename.
- Press **Ctrl + X** to exit Nano.

---

### 5.3 Reload the udev rules

In Ubuntu, run:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

These commands tell Ubuntu to reload the updated udev configuration without requiring a restart.


### 5.4 Verify the udev rule

After reloading the rules, check that the USB device has read and write permissions by running:

```bash
ls -l /dev/bus/usb/*/*
```

Locate the entry corresponding to your DYNAP-SE2 Stack.

If the rule was applied successfully, you should see permissions similar to:

```text
crw-rw-rw- ...
```

or at least that the device is no longer restricted to the root user.

If the permissions have not changed:

1. Disconnect and reconnect the board.
2. Reattach it to WSL:

```powershell
usbipd attach --wsl --busid <BUSID>
```

3. Check the permissions again.




## Step 6 – Create the Conda environment

The DYNAP-SE2 project uses a Conda environment to manage all required Python packages and dependencies.

### 6.1 Create the environment

In Ubuntu, navigate to the project directory.

The repository contains an environment file ( `dynapse2_env_py310.yml`), create the Conda environment by running:

```bash
conda env create -f dynapse2_env_py310.yml
```

### 6.2 Activate the environment

```bash
conda activate <environment_name>
```

Replace `<environment_name>` with the name defined in the `environment.yml` file (i.e., `dynapse_env_py310`).

---

### 6.3 Verify the installation

Verify that the correct Conda environment is active:

```bash
conda env list
```

The active environment is marked with an asterisk (`*`).




## Step 7 – Install `samna`

`samna` is the Python library used to communicate with the DYNAP-SE2 Stack. It provides the interface for detecting the board, configuring the FPGA, and interacting with the neuromorphic hardware.

---

### 7.1 Install `samna`

In **Ubuntu (WSL Terminal)**:

Make sure you are still using the activated Conda environment (`dynapse_env_py310`).

Then install the required version of `samna`:

```bash
pip install samna==0.38.3.0
```

> **Note:** This guide was tested using `samna` version **0.38.3.0**. Other versions may behave differently.

---

### 7.2 Verify the installation

Run:

```bash
python -c "import samna; print(samna.__version__)"
```

If the installation was successful, you should see:

```text
0.38.3.0
```






## Step 8 – Verify that `samna` detects the DYNAP-SE2 Stack

Before programming the FPGA, verify that `samna` can detect the connected board.

---

### 8.1 Open the project in Visual Studio Code

In Ubuntu, navigate to your project directory and open it in VS Code:

```bash
cd /path/to/your/project
code .
```

This will open the project inside VS Code using the WSL environment.

In VS Code, create a new Python file and copy and paste the following code there:

```python
import samna

devices = samna.device.get_unopened_devices()

if not devices:
    print("No DYNAP-SE2 device detected.")
else:
    print(f"Found {len(devices)} device(s):")
    for device in devices:
        print(device.device_type_name)
```

Save the file.

> **Note:** Make sure the correct Conda environment is selected as the Python interpreter in VS Code for `.py` scripts. Select the kernel associated with your Conda environment (for example, `dynapse_env_py310`) for `.ipynb` notebooks.


---

### 8.2 Run the script

Run the script in VS Code.

If everything is configured correctly, you should see:

```text
Found 1 device(s):
DYNAP-SE2 Stack
```

---

### Troubleshooting

If no devices are found:

- Verify that the DYNAP-SE2 Stack is connected.
- Make sure the board is attached to WSL:

```powershell
usbipd attach --wsl --busid <BUSID>
```

- Check that the FrontPanel USB Driver is installed correctly.
- Verify that the udev rule is configured correctly and has been reloaded.




## Step 9 – Configure the FPGA

Once `samna` successfully detects the DYNAP-SE2 Stack, you can configure the FPGA by replacing the code in your Python file with the following:

```python
BITFILE = "bitfiles/Dynapse2Stack.bit"

device_infos = samna.device.get_unopened_devices()

if not device_infos:
    raise RuntimeError("No DYNAP-SE2 device found.")

board = samna.device.open_device(device_infos[0])

if not board.configure_opal_kelly(BITFILE):
    raise RuntimeError("Failed to configure FPGA.")

board.reset_fpga()

print("Board ready.")
```

Run the script again.

If the configuration is successful, you should see:

```text
Found 1 device(s):
  DYNAP-SE2 Stack
Board ready.
```

Congratulations! Your DYNAP-SE2 Stack is now configured and ready for use.
