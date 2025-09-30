# Custom Kernel Signer

1. This repo is more like a gist that outlines the steps I followed to get the CKS working on Win 11 Pro 24H2.
2. I have directly included most parts of the instructions as is from the Original source. All the original sources are linked in the References and within the article in some instances.
3. The goal of this repo for me is to maintain the Knowledge Base and include a few other steps with more emphasis on understanding how to get CKS working alongside how to sign your own drivers.

## 1. What is Custom Kernel Signers?

[Custom Kernel Signers(CKS)](./Notes/CKS.md) is a product policy supported by Windows10 (since 1703). The full product policy name is `CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`. It allows users to decide which certificate is trusted or denied in kernel.

## 2. How to enable this feature?

### 2.1 Prerequisites

1. You must have administrator privilege.
2. You need a temporary environment whose OS is Windows10 Enterprise or Education.
   Why? Because you need it to execute [`ConvertFrom-CIPolicy`](#23-build-kernel-code-sign-certificate-rules) in Powershell which cannot be done in other editions of Windows10.
3. You are able to set UEFI Platform Key.
4. You have WindowsKit installed which provides the signtool. Located in `C:\Program Files (x86)\Windows Kits\10\bin\<version-number>\x64\signtool.exe`

#### 2.1.1 NOTE Regarding Sign Tool

If the `signtool` command is not working directly in the powershell, install WindowsKit and use the commands as follows:

```powershell
& "C:\Program Files (x86)\Windows Kits\10\bin\<version-number>\x64\signtool.exe" sign ...
```

`<version-number>` is the version you have installed on your system so please change path accordingly.

### 2.2 Create certificates and set Platform Key(PK)

[Please follow this link to **Create** the following **Certificates**](./Notes/Build-your-own-pki.md).

```js
// self-signed root CA certificate
localhost-root-ca.cer
localhost-root-ca.pfx

// kernel mode certificate issued by self-signed root CA
localhost-km.cer
localhost-km.pfx

// UEFI Platform Key certificate issued by self-signed root CA
localhost-pk.cer
localhost-pk.pfx
```

#### 2.2.1 Set PK in UEFI Firmware

1. If you have `.der` instead of `.cer` it's still valid and useable.
2. Next step is to set the PK i.e. `localhost-pk.cer` in your UEFI Firmware. This step could vary from system to system but generally it's in the same menu as the Secure Boot option.

The original source[^1] tells how to [set the PK in VMWare](./Notes/Set-PK-VMware.md).

#### 2.2.2 Store the CA in Trust

This step was already included within [Certificate Creation](./Notes/Build-your-own-pki.md) guide but if you missed it, [here's the guide](./Notes/Moving-CA-to-Trust.md).

### 2.3 Build kernel code-sign certificate rules

Run Powershell as administrator.
**NOTE:** Make sure your *Win10/11* is *Pro/Enterprise/Education edition* otherwise do this process in the respective temporary environment.
For example, if you use Win11 then use its Enterprise or Education environment.

1. Use `New-CIPolicy` to create new CI (Code Integrity) policy. Please make sure that the OS is not affected with any malware.

   ```powershell
   New-CIPolicy -FilePath SiPolicy.xml -Level RootCertificate -ScanPath C:\windows\System32\
   ```

   It will scan the entire `System32` folder and take some time. If you do not want to scan, you can use SiPolicy.xml provided by Original source[^1]. I suggest doing this step in your own environment.

2. Use `Add-SignerRule` to add our own kernel code-sign certificate to `SiPolicy.xml`.

   ```powershell
   Add-SignerRule -FilePath .\SiPolicy.xml -CertificatePath .\localhost-km.der -Kernel
   ```

3. Use `ConvertFrom-CIPolicy` to serialize `SiPolicy.xml` and get binary file `SiPolicy.bin`

   ```powershell
   ConvertFrom-CIPolicy -XmlFilePath .\SiPolicy.xml -BinaryFilePath .\SiPolicy.bin
   ```

Now our policy rules have been built. The newly-generated file can be applied to any editions of Windows once it is signed by PK certificate. From now on, we don't need Windows Enterprise/Education edition for next steps.

### 2.4 Sign policy rules and apply policy rules

Check [2.1.1](#211-note-regarding-sign-tool) if `signtool` command is not working directly.

1. For `SiPolicy.bin`, we should use PK certificate to sign it. If you have Windows SDK, you can sign it by `signtool`.

   ```powershell
   signtool sign /fd sha256 /p7co 1.3.6.1.4.1.311.79.1 /p7 . /f .\localhost-pk.pfx /p <password of localhost-pk.pfx> SiPolicy.bin
   ```

   **Please fill `<password of localhost-pk.pfx>` with password of your `localhost-pk.pfx`.**

   Then you will get `SiPolicy.bin.p7` at current directory.

2. Rename `SiPolicy.bin.p7` to `SiPolicy.p7b` and copy `SiPolicy.p7b` to `EFI\Microsoft\Boot\`

   ```powershell
   # run powershell as administrator
   mv .\SiPolicy.bin.p7 .\SiPolicy.p7b
   mountvol x: /s
   cp .\SiPolicy.p7b X:\EFI\Microsoft\Boot\
   ```

### 2.5 Run SSDE

SSDE[^2] is the modified version of CKS-HyperSine[^1] repo developed by valinet.

**NOTE:** If you are on **Windows 11** you should use the **SSDE** repo tools otherwise use the original one by HyperSine[^1] for **Windows 10**.

However, note that I tested SSDE on my Win11 24H2 and CKS-Hypersine only seems to work fine on Win10. Both the tools are present in this repo as is, without any modifications, Please do visit original sources for any potential updates or if you want to download from there.

The steps in either case are straight forward:

- [Steps to enable **CKS** for **Win10** are *listed here*](./Notes/CKS-HyperSine-enable.md)
- [Steps to enable **SSDE** for **Win11** are *listed here*](./Notes/SSDE-valinet-enable.md)

This will set the policy during this boot session.

**NOTE:** Do not shut-down or reboot your PC 2nd time after this step as the changes are not persistent yet.

### 2.6 Persist CustomKernelSigners

#### 2.6.1 Verify CKS

After the reboot, if you have used SSDE you can verify if it was successful by running `.\ssde_query.exe` in powershell or cmd prompt.
It reads the policy present in `SYSTEM\CurrentControlSet\Control\ProductOptions\ProductPolicy` and returns the evaluation as 1 if everything we did was successful otherwise 0.

#### 2.6.2 Signing and Installing the Service

Check [2.1.1](#211-note-regarding-sign-tool) if `signtool` command is not working directly.

Following instructions are for SSDE. In case you used CKS-HyperSine on your Win10, follow the same but for `ckspdrv.sys` file.

Run "Windows PowerShell" as Administrator:

```powershell
# 1. Copy the ssde.sys into System Services
cp .\ssde.sys C:\Windows\System32\drivers\
# 2. Sign the copied ssde.sys with signtool
## Replace <password-of-localhost-km> field with the password you set
signtool sign /f ".\KEYS\localhost-km.pfx" /p <password-of-localhost-km> /fd sha256 /tr http://timestamp.digicert.com /td sha256 "C:\Windows\System32\drivers\ssde.sys"
# 3. Verify the signature
## It should not contain any errors or warnings caused by failed checks
## Make sure you followed section 2.2.2 carefully
signtool verify /pa /v "C:\Windows\System32\drivers\ssde.sys" 
# 4. Create Service Entry
## sc.exe create ckspdrv binpath="C:\Windows\System32\drivers\ckspdrv.sys" type=kernel start=boot error=normal
sc.exe create ssde binpath="C:\Windows\System32\drivers\ssde.sys" type=kernel start=boot error=normal
# 5. Start the created Service
sc.exe start ssde
```

##### NOTE

At the end of this section, I was not able to start the ssde service despite it being signed correctly. It stated the following error:

```log
[SC] StartService FAILED 577:
Windows cannot verify the digital signature for this file. A recent hardware or software change might have installed a file that is signed incorrectly or damaged, or that might be malicious software from an unknown source.
```

This issue[^3] feedback on SSDE repo helped me. If you face the same issue, you might need to disable [Code Integrity](https://learn.microsoft.com/en-us/windows/security/hardware-security/enable-virtualization-based-protection-of-code-integrity?tabs=security) with the following command:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" -Name "Enabled" -Value 0
```

Make sure to run the start service command again, ensuring everything is working.

#### 2.6.3 Check Working Status

1. Reboot your system after this and [verify](#261-verify-cks) if the status is 1.
2. To check if the ssde-service is running, open the Powershell and run `.\ssde_info.exe`.
   - another way of checking the service status based on the one you installed:
   - `sc.exe query ssde` for ssde service
   - `sc.exe query ckspdrv` for ckspdrv service
3. A **Status** of *RUNNING* means you are ready to sign any other drivers the same way we did in step 2 of [2.6.2](#262-signing-and-installing-the-service) and get rolling! ~

### 2.7 Fix Boot Issue on 2nd Reboot

After the 2nd boot performed at end of section 2.6, you might encounter Windows Booting up itself to **Automatic Repair**.

This is caused only if you missed to ensure the service was started successfully before the 2nd reboot.

1. Go to `Troubleshoot > Advanced options > Command Prompt`
2. Identify the C Drive:
`diskpart`
`list vol`
3. Check the size of drive to estimate your C drive. Its label could be I, K, L, etc. Exit diskpart after confirming.
`exit`
4. Delete the service which is causing the problem:
`del "I:\Windows\System32\drivers\ssde.sys"`

Use tab for auto-completion here. If `ssde.sys` is missing then it must be some other volume label for your Windows partition.

Exit the console and reboot your PC.

**NOTE:** Deleting the service will fix the boot issue but you need to repeat section [2.5](#25-run-ssde) and [2.6](#26-persist-customkernelsigners) again after this boot.

### 2.8 Sign your own drivers

To sign your own drivers or modified drivers, you can use the same `signtool` command as in step 2 of [2.6.2](#262-signing-and-installing-the-service). The command is as follows:

```powershell
# 1. [OPTIONAL] Copy your driver into System Services
cp .\myDriver.sys C:\Windows\System32\drivers\
# 2. Sign the copied driver with signtool
## Replace <password-of-localhost-km> field with the password you set
signtool sign /f ".\KEYS\localhost-km.pfx" /p <password-of-localhost-km> /fd sha256 /tr http://timestamp.digicert.com /td sha256 "C:\Windows\System32\drivers\myDriver.sys"
# 3. Verify the signature
## If it's signed correctly, it won't have any errors or warnings caused by failed checks
signtool verify /pa /v "C:\Windows\System32\drivers\myDriver.sys" 
```

**NOTE:** If you are trying to modify the drivers of some other software which has an executable GUI, just follow the above three steps and let the executable create the service entry for you.

## References

I. [Original Theory Article by *Geoff Chappell*](https://www.geoffchappell.com/notes/windows/license/customkernelsigners.htm)
[^1]: [Win10-CustomKernelSigners by HyperSine for Win10](https://github.com/HyperSine/Windows10-CustomKernelSigners)
[^2]: [SSDE by valinet for Win11](https://github.com/valinet/ssde)
[^3]: [Disable Code Integrity for SSDE Service](https://github.com/valinet/ssde/issues/14)
