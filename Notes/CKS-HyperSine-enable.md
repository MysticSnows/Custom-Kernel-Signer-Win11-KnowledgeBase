# [Enable CKS on Win10](https://github.com/HyperSine/Windows10-CustomKernelSigners?tab=readme-ov-file#25-enable-customkernelsigners)

Double click `EnableCKS.exe` and you will see

```powershell
[+] Succeeded to open "HKLM\SYSTEM\Setup".
[+] Succeeded to set "CmdLine" value.
[+] Succeeded to set "SetupType" value.

Reboot is required. Are you ready to reboot? [y/N]
```

Type `y` to reboot. Then the system will enter Setup Mode. `EnableCKS.exe` will run automatically and enable the following two policies

```powershell
CodeIntegrity-AllowConfigurablePolicy
CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners
```

Finally, the system will reboot again and go back to normal mode.

---
