# hp-anywhere-pcoip-client-distrobox-usb-passthrough-guide
Guide to enable USB Devices option in PCoIP Client via Ubuntu Distrobox

In this guide we are going to enable Wacom Intuos Pro USB support in **Teradici PCoIP Client** running inside an **Ubuntu 22.04** Distrobox on an **Arch Linux host**

This will allow for pen pressure to work within the PCoIP Client.

---

## 🚀 Prerequisites

- **Arch Linux host** with Podman (or Docker) and Distrobox installed.
- A **Wacom Intuos Pro** tablet connected via USB.
- **Teradici PCoIP Software Client** installed inside the Ubuntu container (v21.01+).

https://anyware.hp.com/find/product/hp-anyware/2021.03/software-client-for-linux

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

## 2. Create a Rootful Container with USB Passthrough


```
distrobox create --name ubuntu-pcoip-usb-root --image ubuntu:22.04 --root
```

---

Enter Container & Install Dependencies

```
distrobox enter ubuntu-pcoip-usb-root

sudo apt update
sudo apt install usbutils libusb-1.0-0 pcscd

lsusb
```

You Should see

```
...Wacom Intuos Pro M...

```
## 4. Launch PCoIP Client & Connect Tablet

Inside container:

    Launch PCoIP client

    Navigate to USB Devices → Connections → Human Interface Devices (HID).

    Connect Wacom Tablet

Your tablet should now work properly with pen pressure and tilt support.
