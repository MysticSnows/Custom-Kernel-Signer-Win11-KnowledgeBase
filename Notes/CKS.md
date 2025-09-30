# What is CKS?

We know that Windows10 has strict requirements for kernel mode drivers. One of the requirements is that drivers must be signed by an EV certificate that Microsoft trusts. What's more, starting from 1607, new drivers must be submitted to Windows Hardware Portal to get signed by Microsoft. For a driver signed by a self-signed certificate, without enabling TestSigning mode, Windows10 still refuses to load it even if the self-signed certificate was installed into Windows Certificate Store(`certlm.msc` or `certmgr.msc`). That means Windows10 has an independent certificate store for kernel mode drivers.

__Custom Kernel Signers(CKS)__ is a product policy supported by Windows10 (may be from 1703). The full product policy name is `CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`. It allows users to decide which certificates are trusted or denied in kernel. By the way, this policy may require another policy, `CodeIntegrity-AllowConfigurablePolicy`, to be enabled.

Generally, __CKS__ is disabled by default on any editions of Windows10 except __Windows10 China Government Edition__.

If a Windows10 PC meets the following conditions:

1. The product policy `CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners` is enabled.
  (May be `CodeIntegrity-AllowConfigurablePolicy` is also required.)

2. SecureBoot is enabled.

one can add a certificate to kernel certificate store if he owns the PC's UEFI Platform Key so that he can launch any drivers signed by the certificate on that PC.

If you are interested in looking for other product policies, you can see [this](https://www.geoffchappell.com/notes/windows/license/install.htm).

---
