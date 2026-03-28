# OpenWrt for Banana Pi BPI-R4 (kernel 6.12)

This repository builds OpenWrt 25.12 for Banana Pi BPI-R4 (MT7988, Wi-Fi 7) using the MediaTek SDK and includes a rescue + install system that lets you install OpenWrt directly to eMMC — no Linux machine needed, everything runs on GitHub.

You have two options:

- **Use the ready-made release** from this repository — no setup needed, just follow the install steps below.
- **Fork this repository** and build your own customized release — add or remove packages, then install from your own release.

---

## What this repository offers

- **SD card image** — standard OpenWrt image, flash with Etcher and boot directly from SD card.
- **eMMC install** — install OpenWrt permanently to the internal eMMC storage using a guided rescue system. SD card is only used temporarily during installation.
- **NVMe install** — planned for a future release.

---

## DIP switch reference

| Boot medium | SW3-A | SW3-B |
|-------------|-------|-------|
| SD card     | 0     | 0     |
| NAND rescue | 0     | 1     |
| eMMC        | 1     | 0     |

---

## Quick start — SD card

1. Open the [Releases](https://github.com/woziwrt/bpi-r4-rescue/releases/tag/rescue-latest) page and download `openwrt-mediatek-filogic-bananapi_bpi-r4-sdcard.img.gz`.
2. Flash it to your SD card using [Balena Etcher](https://etcher.balena.io/).
3. Insert the SD card into BPI-R4, set DIP switches **SW3-A=0, SW3-B=0**, and power on.

---

## Quick start — eMMC install

This installs OpenWrt permanently to the internal eMMC storage. The SD card is only used during installation and can be reused afterward.

### What you need
- A microSD card (any size, 1 GB is enough).
- A network cable connected to BPI-R4 during installation (for downloading the eMMC image, ~103 MB).

### Step 1 — Flash the rescue SD card

1. Open the [Releases](https://github.com/woziwrt/bpi-r4-rescue/releases/tag/rescue-latest) page and download `bpi-r4-rescue-sdcard.img.gz`.
2. Flash it to your SD card using [Balena Etcher](https://etcher.balena.io/).
3. Insert the SD card into BPI-R4.
4. Set DIP switches **SW3-A=0, SW3-B=0** (SD boot) and power on.

### Step 2 — Install NAND rescue system

1. Connect to BPI-R4 via SSH: `ssh root@192.168.1.1` (no password by default).
2. Run:
   ```
   /root/bpi-r4-install/install-nand.sh
   ```
3. Wait for the script to finish — it will flash the rescue system to NAND.
4. Power off BPI-R4.
5. Set DIP switches **SW3-A=0, SW3-B=1** (NAND boot) and power on.

### Step 3 — Install OpenWrt to eMMC

1. Connect to BPI-R4 via SSH again: `ssh root@192.168.1.1`
2. Make sure a network cable is connected.
3. Run:
   ```
   /root/bpi-r4-install/install-emmc.sh
   ```
4. The script will ask if you are using your own fork. If not, just answer `n` — it will download from this repository.
5. Wait for the script to finish.
6. Power off BPI-R4.
7. Set DIP switches **SW3-A=1, SW3-B=0** (eMMC boot) and power on.

The SD card is no longer needed and can be reused for anything else.

---

## Fork and customize (advanced users)

You can fork this repository, customize the package list, trigger your own build, and use the resulting release for your eMMC installation.

### Step 1 — Fork the repository

Fork this repository on GitHub. **Do not rename the fork** — it must stay named `bpi-r4-rescue`, otherwise the install script will not be able to find your release.

### Step 2 — Enable workflows and set permissions

1. Go to the **Actions** tab in your forked repository and click the button to **enable workflows**.
2. Open **Settings → Actions → General** and configure:
   - **Actions permissions**: set to *Allow all actions and reusable workflows*.
   - **Workflow permissions**: set to **Read and write permissions** — this is required to create releases and upload artifacts. The default is read-only, so you must change this manually.

> ⚠️ Without **Read and write permissions**, the workflow will fail when trying to create a release.

### Step 3 — Customize packages

1. In your fork, open `configs/my_final_defconfig`.
2. Click the pencil icon to edit the file directly on GitHub.
3. You will see lines like:
   ```
   CONFIG_PACKAGE_iperf3=y
   CONFIG_PACKAGE_htop is not set
   ```
   - `CONFIG_PACKAGE_xyz=y` → package enabled
   - `CONFIG_PACKAGE_xyz is not set` → package disabled
4. To enable a package, change `is not set` to `=y`. To disable, change `=y` to `is not set`.
5. **Only change lines starting with `CONFIG_PACKAGE_`.** Do not touch kernel, target, or MTK SDK options unless you know what you are doing.
6. Click **Commit changes** to save.

### Step 4 — Trigger a build

1. Go to the **Actions** tab in your fork.
2. Select the workflow **Build BPI-R4**.
3. Click **Run workflow** and confirm.
4. After the workflow finishes (approx. 2 hours), a release tagged `rescue-latest` will be created in your fork.

### Step 5 — Install using your fork

When running `install-emmc.sh` on BPI-R4, answer `y` to the fork question and enter your GitHub username:

```
Are you using your own fork? [y/n]: y
    INFO: Fork repository name must remain 'bpi-r4-rescue' (do not rename it)
    INFO: Example username: johndoe
Enter your GitHub username: johndoe
    Using release URL: https://github.com/johndoe/bpi-r4-rescue/releases/download/rescue-latest/...
```

The URL is displayed so you can verify it looks correct before the download starts.

---

## Repository contents

- `bpi-r4-openwrt-builder.sh` — main build script (clones OpenWrt + MTK SDK, prepares, builds, applies config).
- `configs/my_final_defconfig` — package config, edit this to customize your build.
- `my_files/` — patches, custom packages, LuCI applications.
- `rescue/bpi-r4-rescue-sdcard.img.gz` — static rescue SD card image used for eMMC installation.
- `.github/workflows/build.yml` — GitHub Actions workflow.

---

## Notes

- This build is for Banana Pi BPI-R4 only.
- OpenWrt and MTK SDK commits are pinned in the build script; updating them requires manual editing.

### Notes about GitHub runners

This workflow runs on GitHub-hosted runners where free disk space is not guaranteed. If a build fails with a disk-related error, simply re-run the workflow — runners with sufficient space (~100 GB free) are usually available within a short time.

External mirrors used during the build can also be temporarily slow or unavailable. Re-running the workflow later usually resolves such issues.
