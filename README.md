<div align="center">

<img width="160" alt="arc-flash" src=".github/arc-flash.png">

### Arc/Arx Flash — the USB writer for Arc and Arx

Write an Arc or Arx loader to a USB stick on Windows, Linux and macOS,<br>
and update a loader stick to a newer release without losing its settings.

<a href="https://github.com/AuxXxilium/arc-flash/releases/latest"><img alt="Download" src="https://img.shields.io/badge/download-red?style=for-the-badge&label=latest&color=%23FF0000"></a>
<a href="https://xpenology.tech/wiki"><img alt="Wiki" src="https://img.shields.io/badge/read_first-blue?style=for-the-badge&label=wiki&color=%230066CC"></a>
<a href="https://discord.auxxxilium.tech"><img alt="Discord" src="https://img.shields.io/badge/discord-5865F2?style=for-the-badge&label=chat&color=%235865F2"></a>

</div>

---

> [!IMPORTANT]
> * Arc/Arx Flash writes [Arc](https://github.com/AuxXxilium/arc) and [Arx](https://github.com/AuxXxilium/arx) loader images. It is not a loader.
> * **Commercial use is not permitted and strictly forbidden.**
> * DSM and all parts of it are under copyright / ownership by Synology Inc.
> * The app is free and will stay free forever. If you paid a suspicious person for it, I can't help you — I'm not connected to them.

> [!WARNING]
> **Flash** replaces everything on the chosen stick. Check the device before you press **Start**, and back up anything on it first. I'm not liable for damage or loss of any kind.

---

## ✨ What Arc/Arx Flash does

* **Flash** — writes a release image to a stick: downloaded from GitHub, or a local `.img` / `.img.zip`. Everything on the stick is replaced
* **Update** — puts a newer release on a stick that already holds the same loader. The settings, the DSM partition, the password and the DSM files the loader downloaded are kept
* **Knows your sticks** — the device list shows which loader and version each stick holds. Only USB, SD and other removable disks are offered, never the disk the system runs from
* **Releases built in** — choose from the newest Arc and Arx releases, or pick an image file you already have
* **Checked downloads** — images are checked against the sha256 digest GitHub publishes; **Verify** reads the stick back after writing and compares it
* **Safe to interrupt** — the partition table and boot code are written last, so a stick that was pulled halfway never looks like a loader
* **Backups** — the files Update saves stay on this computer, the last 3 per loader
* **One file, nothing to install** — a native app with no browser, web view or runtime, styled like Arc Control, in light and dark
* **Command line** — every action also works without the window

---

## 🧭 How it works

```
Flash:   wipe old partition tables (backup GPT included) → write the image
         → partition table and boot code last → Verify (optional)

Update:  save     user files from partition 1 (FAT32) and partition 3 (ext4) to this computer
         write    boot code, partition 1 and partition 3 from the new release
                  partition 2 (DSM's files, machine.key) is not touched
         restore  the saved files → the loader builds DSM again on the next boot
```

Update keeps, from **partition 1**, the config (`user-config.yml` / `loader.json`), `grubenv`, `rsysenv`, logs and anything else that is not part of the loader; from **partition 3**, `initrd-user` (password), `users/` (DSM boot-file cache), `backup/`, `extractor/` and anything else that is not part of the loader. The built DSM payload is left out, so the loader builds it again. GRUB is updated too, which the loaders' own **Update Loader** does not do.

For Arc, it also does what Arc's updater does: `arc.builddone` is set to `false`, and the next boot goes to Automated Mode (or Config Mode if the loader was not configured yet).

---

## 🖥️ Supported

| System | Download | |
| :-- | :-- | :-- |
| Windows | `arc-flash-windows-amd64.exe` | most PCs |
| | `arc-flash-windows-arm64.exe` | Windows on ARM |
| macOS | `arc-flash-macos.app.zip` | Apple silicon and Intel, macOS 11 or later |
| Linux | `arc-flash-linux-amd64` | x86-64, X11 or Wayland |
| | `arc-flash-linux-arm64` | ARM64, X11 or Wayland |

| Loader | |
| :-- | :-- |
| [Arc](https://github.com/AuxXxilium/arc) | Flash and Update |
| [Arx](https://github.com/AuxXxilium/arx) | Flash and Update |

---

## 📋 What you need

* Windows, Linux or macOS, and an administrator account
* A USB stick — 4 GB or more for Arc, about 1 GB for Arx
* Internet, to download a release — or an image file you already have

For **Update**, the stick must hold the same loader with the same partition layout as the new release, and its ext4 partition must have been shut down cleanly. If the machine was switched off while the loader ran, boot the loader once and shut it down, or use **Flash**.

---

## 📦 Installation

1. Download the file for your system from the [releases](https://github.com/AuxXxilium/arc-flash/releases/latest)
2. Start it:

   | System | How |
   | :-- | :-- |
   | Windows | Double-click `arc-flash-windows-amd64.exe` and accept the administrator prompt |
   | Linux | Make it executable (`chmod +x arc-flash-linux-amd64`) and start it. The system asks for your password (polkit) |
   | macOS | Unzip and open `arc-flash.app`. macOS asks for an administrator password |

3. Choose **Flash** or **Update**, pick the USB device, pick a release or an image file, and press **Start**

Writing to a USB stick needs administrator rights. On Windows the app restarts itself elevated. On Linux and macOS the window runs as you, and only a small helper (the same program, `arc-flash serve`) runs as root; the two talk over `127.0.0.1` with a per-run token.

> [!TIP]
> [Rufus](https://rufus.ie) or [balenaEtcher](https://etcher.balena.io) can also write a loader image to a stick. Arc/Arx Flash is the recommended way: it fetches and checks the release for you, and it is the only one that can **Update** a loader stick and keep its settings.

---

## 💾 Backups and downloads

| System | Folder |
| :-- | :-- |
| Windows | `%LOCALAPPDATA%\arc-flash` |
| Linux | `~/.cache/arc-flash` (root's home under sudo) |
| macOS | `~/Library/Caches/arc-flash` (root's home under sudo) |

The files Update saves stay here as a backup; the last 3 per loader are kept. Only the newest downloaded image per loader is kept.

---

## ⌨️ Command line

```
arc-flash list [-all]                       USB disks and the loader on each
arc-flash info -disk ID                     the loader on one stick
arc-flash releases -loader arc|arx          available releases
arc-flash flash  -disk ID -loader arc|arx [-version TAG | -file IMG] [-verify] [-yes]
arc-flash update -disk ID [-version TAG | -file IMG] [-yes]
arc-flash [-all] [-no-elevate]                                the app (default)
```

`-disk` takes the ID from `list`: `PhysicalDrive2` on Windows, `sdb` on Linux, `disk4` on macOS.

---

## 📚 Documentation

* [Documentation](https://xpenology.tech/wiki) — **read this first**

---

## 🧰 More from the Arc Project

| Project | Description |
| :-- | :-- |
| [Arc Loader](https://github.com/AuxXxilium/arc) | Redpill loader for DSM 7.x, with guided setup for many platforms and models |
| [Arx](https://github.com/AuxXxilium/arx) | The evolution of Arc — set up entirely from your browser |
| [Arc Control](https://github.com/AuxXxilium/arc-control) | DSM app for loader settings, monitoring and hardware tuning |
| [AuxXxilium](https://github.com/AuxXxilium) | Everything else from the Arc Project |

---

### Developer

- <a href="https://github.com/AuxXxilium">AuxXxilium</a>

### Thanks

* [Gio](https://gioui.org), which draws the app window
* [go-diskfs](https://github.com/diskfs/go-diskfs), for the partition tables and FAT32

<div align="center">

[![Stars](https://img.shields.io/github/stars/AuxXxilium/arc-flash?style=for-the-badge&logo=github)](https://github.com/AuxXxilium/arc-flash)
[![Discord](https://img.shields.io/discord/639072565155069962?style=for-the-badge&logo=discord&label=Discord)](https://discord.auxxxilium.tech)

</div>
