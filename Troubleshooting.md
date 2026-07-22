# Troubleshooting

## `No DYNAP-SE2 device found`

If `samna` cannot detect your board, it is usually because Windows has reclaimed the USB device after installing or updating the FrontPanel USB Driver.

### Solution

🔑 **Windows PowerShell (Administrator)**

Reattach the device to WSL:

```powershell
usbipd attach --wsl --busid <BUSID>
```

Replace `<BUSID>` with the BUSID of your DYNAP-SE2 Stack (see **Step 3**).

---

## `configure_opal_kelly()` hangs indefinitely

If the script hangs while calling `configure_opal_kelly()`, check the following:

- The **FrontPanel USB Driver** is installed correctly.
- The DYNAP-SE2 Stack is attached to WSL using `usbipd`.
- The bitfile path is correct (e.g. `bitfiles/Dynapse2Stack.bit`).
- The USB permissions are configured correctly using the udev rule described in **Step 5**.

---

## `lsusb: command not found`

The `usbutils` package is not installed.

🐧 **Ubuntu (WSL Terminal)**

Install it by running:

```bash
sudo apt install usbutils
```

---

## `Permission denied`

If you receive a permission error when accessing the USB device, first check the device permissions.

🐧 **Ubuntu (WSL Terminal)**

Run:

```bash
ls -l /dev/bus/usb/*
```

If the permissions appear incorrect, verify that:

- The udev rule was created correctly.
- The Vendor ID in the rule matches the output of `lsusb`.
- The udev rules have been reloaded.
- The device has been reattached to WSL.

---

## The board was detected before, but is no longer detected

If the board was previously working but suddenly disappears after installing or updating the FrontPanel USB Driver, Windows has likely taken control of the USB device again.

Simply reattach it to WSL:

```powershell
usbipd attach --wsl --busid <BUSID>
```

You do **not** need to reinstall the driver or recreate the udev rule.

---



# Final verification

If you are unsure whether everything is configured correctly, run the following script:

```python
import samna

devices = samna.device.get_unopened_devices()

print(devices)

board = samna.device.open_device(devices[0])

board.configure_opal_kelly("bitfiles/Dynapse2Stack.bit")

board.reset_fpga()

print("Board ready.")
```

If everything has been configured successfully, the output should be similar to:

```text
Found 1 device(s):
  DYNAP-SE2 Stack
Board ready.
```




