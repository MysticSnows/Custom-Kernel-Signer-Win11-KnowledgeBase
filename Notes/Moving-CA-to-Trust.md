# Store Root CA-Certificate to Windows Trust

Windows must have the custom root CA certificate installed in the "Trusted Root Certification Authorities" store;
Otherwise, it will not acknowledge the drivers signed by the localhost-km.pfx kernel certificate despite it being issued by root CA (as it was not in the trusted vault).

---

## Steps to add root CA into Trust

1. **Open Microsoft Management Console (MMC)**:
   - Press `Win + R`, type `mmc`, and hit Enter.
2. **Add Certificates snap-in**:
   - Go to `File > Add/Remove Snap-in....`
   - Select `Certificates`
   - Click `Add >` Button
   - Choose `Computer` account, and click `Finish`.
3. **Import the root certificate**:
   - In the MMC, navigate to `Certificates (Local Computer) > Trusted Root Certification Authorities`.
   - Right-click on Certificates, go to `All Tasks > Import....`
   - **Follow the wizard**: Import your custom root CA certificate (`localhost-root-ca.cer`) into this store. You should see a warning about importing a non-trusted CA; confirm that you want to proceed.
4. **Reboot**: Reboot your computer for the change to take effect.

---
