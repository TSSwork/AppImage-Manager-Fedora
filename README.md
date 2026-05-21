# appimage-manager

A lightweight, interactive AppImage package manager for Linux.  
Installs, updates, and removes AppImages with proper desktop integration — no root required.

---

## Features

- Organized per-app directory structure under `~/Applications/`
- Automatic `.desktop` entry creation (shows in your app launcher)
- Icon management under `~/.local/share/icons/`
- Metadata file per app for easy maintenance
- Optional terminal command symlink via `~/.local/bin/`
- Update and remove modes included
- Works on Fedora, Ubuntu, Arch, NixOS, and any systemd-based distro
- Wayland-friendly (supports custom launch arguments)

---

## Installation

```bash
# Download the script
curl -o appimage-manager https://raw.githubusercontent.com/yourname/appimage-manager/main/appimage-manager

# Make it executable
chmod +x appimage-manager

# Move to your PATH
mv appimage-manager ~/.local/bin/appimage-manager
```

Make sure `~/.local/bin` is in your `PATH`. Add this to your `~/.bashrc` or `~/.zshrc` if it isn't:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

---

## Usage

```
appimage-manager              Install a new AppImage
appimage-manager --update     Update an existing AppImage
appimage-manager --remove     Remove an installed AppImage
appimage-manager --list       List all installed AppImages
appimage-manager --help       Show help
```

---

## Workflows

### Install a new AppImage

Run:

```bash
appimage-manager
```

You will be prompted for:

| Prompt | Example |
|---|---|
| AppImage path | `~/Downloads/Helium-1.0.AppImage` |
| App ID (unique, lowercase) | `helium` |
| Display name | `Helium Browser` |
| Icon path | `~/Downloads/helium.png` |
| Category | `Network` |
| Extra launch arguments | `--no-sandbox` |
| Create terminal command? | `y` / `N` |
| Move instead of copy? | `y` / `N` |

After completion, the app will appear in your desktop launcher.

---

### Update an existing AppImage

Run:

```bash
appimage-manager --update
```

- Lists your installed apps
- Asks which app to update
- Asks for the path to the new `.AppImage` file
- Optionally replaces the icon
- Keeps your existing desktop entry and metadata intact

---

### Remove an AppImage

Run:

```bash
appimage-manager --remove
```

Cleanly removes:

- The `~/Applications/<app-id>/` folder (AppImage + icon + metadata)
- The `.desktop` entry
- The icon from `~/.local/share/icons/`
- The terminal symlink from `~/.local/bin/` (if it exists)
- Refreshes the desktop database automatically

---

### List installed apps

Run:

```bash
appimage-manager --list
```

Example output:

```
=== Installed AppImages ===

  helium
    Name:      Helium Browser
    Category:  Network
    Installed: 2025-05-21

  obsidian
    Name:      Obsidian
    Category:  Utility
    Installed: 2025-04-10

  Total: 2 app(s) installed.
```

---

## Directory Structure

After installing an app, the layout looks like this:

```
~/Applications/
├── helium/
│   ├── app.AppImage          ← the AppImage binary
│   ├── icon.png              ← app icon
│   └── metadata.conf         ← saved settings for update/remove
│
└── obsidian/
    ├── app.AppImage
    ├── icon.png
    └── metadata.conf

~/.local/share/applications/
├── appimage-helium.desktop
└── appimage-obsidian.desktop

~/.local/share/icons/
├── helium.png
└── obsidian.png

~/.local/bin/
├── helium     ← symlink to ~/Applications/helium/app.AppImage
└── obsidian
```

---

## Metadata File

Each app stores a `metadata.conf` in its directory:

```ini
APP_ID=helium
DISPLAY_NAME=Helium Browser
CATEGORY=Network
ICON_EXT=png
ARGS=--no-sandbox
INSTALLED=2025-05-21
```

This is used by `--update`, `--remove`, and `--list` to manage the app without asking redundant questions.

---

## Desktop Entry Format

The generated `.desktop` file looks like:

```ini
[Desktop Entry]
Type=Application
Name=Helium Browser
Exec=/home/user/Applications/helium/app.AppImage --no-sandbox
Icon=helium
Categories=Network;
Terminal=false
StartupNotify=true
X-AppImage-Integrate=false
```

The `X-AppImage-Integrate=false` line prevents AppImage from trying to self-integrate, which avoids double entries in some environments.

---

## Wayland / Niri Tips

For Electron-based AppImages on Wayland, use these launch arguments when prompted:

| App type | Recommended args |
|---|---|
| Electron apps | `--enable-features=UseOzonePlatform --ozone-platform=wayland` |
| Electron (fallback X11) | `--no-sandbox` |
| JetBrains IDEs | Set `GDK_BACKEND=x11` in the Exec line manually |
| GTK apps | Usually no flags needed |

You can always edit the `.desktop` file after installation:

```bash
nano ~/.local/share/applications/appimage-helium.desktop
```

---

## Requirements

- Bash 4+
- Standard Linux utilities: `cp`, `chmod`, `mkdir`, `ln`, `rm`
- Optional: `update-desktop-database` (from `desktop-file-utils`) — auto-refreshes launcher
- Optional: `gio` (from `glib2`) — auto-trusts AppImage in file managers

Install on Fedora:

```bash
sudo dnf install desktop-file-utils glib2
```

Install on Ubuntu/Debian:

```bash
sudo apt install desktop-file-utils libglib2.0-bin
```

---

## Troubleshooting

**App doesn't appear in launcher**

Run manually:

```bash
update-desktop-database ~/.local/share/applications
```

Then log out and back in, or restart your compositor.

**"Permission denied" when running**

```bash
chmod +x ~/Applications/<app-id>/app.AppImage
```

**AppImage fails to run (FUSE error)**

Some AppImages need FUSE. On Fedora:

```bash
sudo dnf install fuse fuse-libs
```

Alternatively, many AppImages support running without FUSE:

```bash
./app.AppImage --appimage-extract-and-run
```

To make this permanent, edit the desktop entry's `Exec` line and append `--appimage-extract-and-run`.

**Icon not showing in launcher**

Make sure the icon extension in `metadata.conf` matches the actual file in `~/.local/share/icons/`. Then run `update-desktop-database` again.

---

## License

MIT — free to use, modify, and distribute.
