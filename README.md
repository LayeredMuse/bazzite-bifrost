# Bazzite Bifrost 🌈⚡

**bazzite-bifrost** is a custom atomic Linux operating system image built using [BlueBuild](https://blue-build.org/) and `bootc`, based on [`bazzite-nvidia`](https://github.com/ublue-os/bazzite). 

Designed for headless rack-mounted gaming servers equipped with Nvidia GPUs, **bazzite-bifrost** acts as a bridge to seamlessly switch between two distinct user sessions on demand via remote SSH commands (e.g. from Home Assistant or shell scripts):

1. **Workstation Session**: Niri Wayland compositor + Noctalia shell.
2. **Big Picture Gaming Session**: `gamescope-session` with 4K 120Hz HDR enabled for TV playback.

---

## 🌟 Key Architecture Features

- **Direct TTY1 Boot (Bypass GDM/SDDM)**: `gdm.service` and `sddm.service` are masked. The system boots straight to TTY1, bypassing display manager overhead on headless boots.
- **Dynamic TTY1 Auto-Login**: Automatically resolves the primary user account created during OS installation (`UID 1000` / `wheel` group) without hardcoding credentials into the image.
- **Nvidia DRM Modesetting Enforced**: Kernel arguments (`nvidia_drm.modeset=1` and `nvidia_drm.fbdev=1`) pre-configured for full Wayland & HDR support.
- **User-Level Session Lifecycle**: Graphical sessions are managed as user-level systemd units (`systemctl --user`), enabling instant, remote session toggling.

---

## 📦 Installed Packages

The following RPM packages are pre-installed on top of the base Bazzite Nvidia image:
- `niri` - Scrollable-tiling Wayland compositor
- `noctalia-shell` - Desktop shell interface
- `gamescope-session` - Steam Big Picture session manager
- `wlr-randr` - Wayland output configuration utility
- `pipewire` & `wireplumber` - Audio and media routing
- `alacritty` - GPU-accelerated terminal emulator

---

## ⚙️ Environment Variables (HDR & Gamescope)

Pre-configured in `/usr/share/user-style/environment.d/gamescope-session.conf` (and `/etc/environment.d/gamescope-session.conf`):

```env
GAMESCOPE_OUTPUT_NAME="HDMI-A-1"
DXVK_HDR=1
ENABLE_HDR_WSI=1
PROTON_ENABLE_HDR=1
VKD3D_DISABLE_EXTENSIONS="VK_KHR_present_wait"
GAMESCOPECMD="/usr/bin/gamescope -W 3840 -H 2160 -r 120 --hdr-enabled --hdr-itm-enabled --prefer-output HDMI-A-1 --"
```

---

## 🎮 Remote Session Control (Home Assistant / SSH)

Since display managers are disabled, sessions can be started, stopped, or swapped remotely via SSH.

### 1. Launch Workstation Desktop (Niri + Noctalia)
```bash
ssh <user>@<host-ip> "systemctl --user stop gamescope-session.service; systemctl --user start niri.service"
```

### 2. Launch 4K 120Hz HDR TV Gaming (Gamescope)
```bash
ssh <user>@<host-ip> "systemctl --user stop niri.service; systemctl --user start gamescope-session.service"
```

### 3. Stop All Graphical Sessions (Return to bare TTY1)
```bash
ssh <user>@<host-ip> "systemctl --user stop niri.service gamescope-session.service"
```

---

## 🔐 Setting Up Cosign Image Signing

To sign your built images with [Cosign](https://github.com/sigstore/cosign) for verified `rpm-ostree` rebases:

### Step 1: Generate Key Pair
Run Cosign locally (or via container):
```bash
cosign generate-key-pair
```
This produces `cosign.key` (private key) and `cosign.pub` (public key).

### Step 2: Add Secret to GitHub
1. Navigate to your GitHub repository -> **Settings** -> **Secrets and variables** -> **Actions**.
2. Click **New repository secret**.
3. Set **Name**: `SIGNING_SECRET`.
4. Set **Value**: Paste the full contents of your `cosign.key` file.
5. (Optional) If you encrypted `cosign.key` with a passphrase, add another secret named `COSIGN_PASSWORD`.

### Step 3: Add Public Key to Repository
Copy `cosign.pub` into `config/files/etc/pki/containers/cosign.pub`.

---

## 🚀 Deployment & Installation

### Option A: Install from ISO
1. Trigger the **Build Bootable Install ISO** GitHub Action (or download the artifact generated automatically after a successful main image build).
2. Flash the generated `bazzite-bifrost-latest.iso` to a USB drive using Rufus, Ventoy, or `dd`.
3. Boot the target machine from USB and follow the Anaconda installer instructions.

### Option B: Rebase an Existing Atomic System

#### Unverified (Before Cosign setup):
```bash
rpm-ostree rebase ostree-unverified-registry:ghcr.io/<YOUR_GITHUB_USER>/bazzite-bifrost:latest
```

#### Signed (With Cosign enabled):
```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/<YOUR_GITHUB_USER>/bazzite-bifrost:latest
```

After the rebase completes, reboot the system:
```bash
systemctl reboot
```

---

## 🔨 Local Building & ISO Generation

### 1. Build Image Locally
```bash
bluebuild build recipes/recipe.yml
```

### 2. Generate ISO Locally
```bash
sudo bluebuild generate-iso --iso-name bazzite-bifrost.iso recipe recipes/recipe.yml
```
