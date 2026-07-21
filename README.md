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

