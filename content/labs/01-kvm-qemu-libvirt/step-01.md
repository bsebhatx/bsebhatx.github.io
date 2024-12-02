---
title: "Step 01: Enable Virtualization"
type: docs
breadcrumbs: true
---

Before setting up KVM, ensure your system supports hardware virtualization.

## Check for Virtualization Support

Open a terminal and check the contents of `/proc/cpuinfo` for virtualization flags:

```bash
grep -E 'vmx|svm' /proc/cpuinfo
```

- **vmx**: Indicates Intel Virtualization Technology is supported.
- **svm**: Indicates AMD Secure Virtual Machine is supported.

If you see one of these flags, your system supports hardware virtualization.

## Enable Virtualization in the BIOS/UEFI

If no flags appear, virtualization might not be enabled in your BIOS/UEFI settings. To enable it:

1. Reboot your system and enter the BIOS/UEFI setup (commonly accessed by pressing a key like `F2`, `DEL`, or `ESC` during boot).
2. Look for an option related to **Virtualization Technology** (Intel VT-x) or **SVM Mode** (for AMD CPUs).
3. Enable the setting, save changes, and reboot.

If you're unsure how to access or modify these settings, consult your computer's documentation or search online for steps specific to your system's make and model.