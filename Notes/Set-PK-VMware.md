# [Set PK in VMware](https://github.com/HyperSine/Windows10-CustomKernelSigners?tab=readme-ov-file#221-set-pk-in-vmware)

If your VMware virtual machine's name is `TestVM` and your VM has SecureBoot, there would be two files under your VM's folder: `TestVM.nvram` and `TestVM.vmx`. You can set PK by the following:

1. Close your VM.

2. Delete `TestVM.nvram`. This would reset your VM's UEFI settings next time your VM starts.

3. Open `TestVM.vmx` by a text editor and append the following two lines:

   ```powershell
   uefi.allowAuthBypass = "TRUE"
   uefi.secureBoot.PKDefault.file0 = "localhost-pk.der"
   ```

   The first line allows you to manage SecureBoot keys in UEFI firmware.

   The second line will make `localhost-pk.der` in the VM's folder as default UEFI PK. If `localhost-pk.der` is not in the VM's folder, please specify a full path.

Then start `TestVM` and your PK has been set.

---
