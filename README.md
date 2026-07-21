# Connecting a SynSense DYNAP-SE2 Stack to Windows 11 using WSL2

This guide explains how to connect a SynSense DYNAP-SE2 Stack (using the Opal Kelly XEM7310-A75) to Windows 11 with WSL2.
Unlike Linux, Windows requires a few extra steps to allow the board to be accessed from WSL.


## Requirements
Before starting, make sure you have:
- Windows 11
- WSL2 installed
- Ubuntu installed in WSL
- Conda/Miniconda installed
- A working Python environment containing samna
- usbipd-win installed
- Administrator privileges (at least once)


## Step 1 – Verify Windows detects the board
Plug in the DYNAP-SE2 Stack. (First to PC then to power.)

Open PowerShell (open Start window and search for PowerShell) and run:

```usbipd list```

Expected output:

```
BUSID  VID:PID     DEVICE        STATE 
3-1    151f:0130   XEM7310-A75   Not Shared
```

If the board appears, Windows can see it.

If "STATE" is "Shared", skip to step 4.
