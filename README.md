# HP Anywhere PCoIP Client Distrobox + USB Passthrough Guide
Guide to enable USB Devices option in PCoIP Client via Ubuntu Distrobox

In this guide we are going install **Teradici PCoIP Client** running inside an **Ubuntu 22.04** Distrobox on an **Arch Linux host** with USB Devices (passthrough) working.

This will allow USB Devices such as Wacom Tablet to work with pen pressure to work within the PCoIP Client.

---

## 🚀 Prerequisites

- **Arch Linux host** with Podman (or Docker) and Distrobox installed.
- A **Wacom Intuos Pro** tablet connected via USB.
- **Teradici PCoIP Software Client** installed inside the Ubuntu container (v21.01+).

---

## 1. Why `--root` Is Needed for USB Passthrough, Specifically for Wacom Tablets

By default, Distrobox creates **rootless containers** where the container process runs under a user namespace mapping. Such containers:
- Cannot create new device nodes in `/dev/` (USB device files).
- Are functionally isolated from host device groups and permissions.  
  Rootless containers typically **bind-mount** device files from the host, but **cannot modify permissions or group ownership**, making access unreliable.

In contrast, a **rootful container** (created using `--root`) runs with real root privileges:
- Device nodes appear inside the container as actual devices—not just bind-mounted.
- Full access to `/dev/bus/usb` is granted, which is required for USB bridging via Teradici’s `usb-helper`.

Therefore, using `--root` ensures the PCoIP client inside the container can truly interface with host USB devices.

### Security Risks of `--root`

Running with `--root` significantly reduces container isolation:

- The container has **full root privileges** on the host—any vulnerability inside could escalate to host root compromise.
- Applications inside can modify host system files, your home directory, or create new containers with full privileges.
- Distrobox is **not designed as a sandbox**; it intentionally integrates tightly with the host.

> ⚠️ Use `--root` only when necessary (such as for USB passthrough). Do **not** run untrusted code inside this container. For day-to-day tasks, prefer rootless containers.

---

## 2. Create a Rootful Container


```
distrobox create --name ubuntu-pcoip-root --image ubuntu:22.04 --root
```

---

Enter Container & Install Dependencies

```
distrobox enter ubuntu-pcoip-root

sudo apt update
sudo apt install usbutils libusb-1.0-0 pcscd

lsusb
```

You Should see

```
...Wacom Intuos Pro M...

```
## 4. Install and Launch PCoIP Client

Find the the latest version of PCoIP Client ```https://anyware.hp.com/find/product/hp-anyware/2021.03/software-client-for-linux```


Inside container:

```
e.g. curl -1sLf https://dl.anyware.hp.com/DeAdBCiUYInHcSTy/pcoip-client/cfg/setup/bash.deb.sh | sudo -E distro=ubuntu codename=jammy bash

sudo apt update

sudo apt install pcoip-client
```

Launch PCoIP client

---

To check if PCoIP Client detects the USB Devices beforehand, for example the Wacom Tablet:
```
Launch PCoIP Client

Go to settings gear icon

Logs

Log Level set to Level 3 Debug

Show in Folder

Open log file

Ctrl+F (to find text)

Search Wacom
```
It should pick up your Wacom Tablet name or any other USB Devices ```e.g. MGMT_USB :Device 0x00040003 Wacom Intuos Pro M```

If nothing picks up and you see multple rows along the lines of ```MGMT_USB : class 09 is not supported``` that means the USB Devices are not being passed through


---

Log Into your PCoIP Client session

Navigate to USB Devices → Connections → Human Interface Devices (HID).

Connect Wacom Tablet

Your tablet should now work properly with pen pressure.
