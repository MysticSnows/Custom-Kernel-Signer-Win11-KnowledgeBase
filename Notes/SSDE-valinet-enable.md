# Enable SSDE on Win11

Double click `ssde_enable.exe` and you will see

```powershell
[+] Succeeded to open "HKLM\SYSTEM\Setup".
[+] Succeeded to set "CmdLine" value.
[+] Succeeded to set "SetupType" value.

Reboot is required. Are you ready to reboot? [y/N]
```

Type `y` to reboot. Then the system will enter Setup Mode. `ssde_enable.exe` will run automatically and enable the following policy

```powershell
CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners
```

Finally, the system will reboot again and go back to normal mode.

---
