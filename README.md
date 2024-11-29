If you want to preserve secureboot and run vmware on Fedora 41, here's what you have to do.

Download bundle package from here and install it.

`aHR0cHM6Ly9zb2Z0d2FyZXVwZGF0ZS52bXdhcmUuY29tL2Nkcy92bXctZGVza3RvcC93cy8xNy42LjEvMjQzMTkwMjMvbGludXgvY29yZS8=`

Workstation does not ship vmmon.ko and vmnet.ko in the bundle currently. The two modules are built during the installation or the first launch of workstation.

During the installation if the host provides the proper kernel headers and gcc, these two modules will be built silently. The progress is logged into /tmp/vmware-root/vmware-PID.log.

When workstation is first launched, a dialog will pop up to ask you for the usable kernel headers and/or gcc. These two modules will be built with window showing the progress and log printed on terminal.

Ensure to have the Host OS updated with the latest patches.

```bash
sudo dnf install kernel-devel
```

To correct the issue with secure boot enabled:

- Generate a key pair using the openssl to sign vmmon and vmnet modules:

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=VMware/"
```
- Replace MOK with the name of the file you want for the key.

- Sign the modules using the generated key by running these commands:
```bash
sudo /usr/src/kernels/`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)
```
```bash
sudo /usr/src/kernels/`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)
```

- Import the public key to the system's MOK list by running this command:
```bash
sudo mokutil --import MOK.der
```
- Confirm a password for this MOK enrollment request.
- Reboot your machine. Follow the instructions to complete the enrollment from the UEFI consol.

In case Vmtools install is greyed out, download the iso for tools from here and mount it after instalation

`aHR0cHM6Ly9zb2Z0d2FyZXVwZGF0ZS52bXdhcmUuY29tL2Nkcy92bXctZGVza3RvcC93cy8xNy42LjEvMjQzMTkwMjMvd2luZG93cy9wYWNrYWdlcy8=`

*Note: Preceding commands are verified to work on* ***Fedora 41 hosts.*** *The general steps are applicable to all Linux distributions, but specific Linux distributions might differ in commands.*

