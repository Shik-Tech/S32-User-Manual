# Install

s32 **2.0.1** speaks protocol 4. One line installs it; no download page, and no `sudo` for the CLI itself.

## macOS

```bash
curl --proto '=https' --tlsv1.2 -fsSL https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.sh | sh
```

The script picks the right build (Apple Silicon or Intel), checks the download against `SHA256SUMS` from the same release, and installs `s32` into `~/.local/bin`. libusb is built in; there is nothing else to install.

## Linux

```bash
curl --proto '=https' --tlsv1.2 -fsSL https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.sh | sh
```

The script installs `s32` into `~/.local/bin` and needs `libusb-1.0` from your distribution. If it is missing, the script stops before it downloads anything and prints the command for your system (`sudo apt-get install libusb-1.0-0` on Debian and Ubuntu, `sudo dnf install libusb1` on Fedora).

It then asks, at your terminal, whether to install the USB rule that lets `s32` reach the S32 without root. Answer `y` and it runs `sudo` for exactly three commands: it copies the rule to `/etc/udev/rules.d/99-s32-editor.rules`, reloads the rules and triggers udev. Any other answer, or no terminal (a script or an agent), runs no `sudo` at all: the script prints the same three commands for you to run. It keeps its copy of the rule in `~/.local/share/s32/99-s32-editor.rules`, and the printed commands point there.

Unplug the S32 and plug it back in after the rule is installed. Without the rule, `--usb` needs root.

The rule gives the logged-in desktop user access to VID/PID `0483:a56e`:

```
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=="a56e", TAG+="uaccess"
```

Don't use `MODE="0666"` on a shared machine: whoever opens the interface controls the S32.

## Windows

In PowerShell:

```powershell
irm https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.ps1 | iex
```

On an older Windows PowerShell 5.1 (Windows 10 1607 or earlier, Windows 8.1), if the line fails with a TLS or "could not create SSL/TLS secure channel" error, run `[Net.ServicePointManager]::SecurityProtocol = 'Tls12'` first and then the `irm` line again.

From a script or an AI agent (no one at the window), run it as one command that returns the installer's result, so a failed install is not reported as success:

```powershell
powershell -NoProfile -Command 'irm https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.ps1 | iex; exit $LASTEXITCODE'
```

The single quotes matter: they stop the calling shell (PowerShell 7, Git Bash, a CI step) from expanding `$LASTEXITCODE` before `powershell` runs, which would turn a failed install into exit code 0. From `cmd.exe`, which does not treat single quotes as quotes, use double quotes instead:

```bat
powershell -NoProfile -Command "irm https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.ps1 | iex; exit $LASTEXITCODE"
```

The script installs `s32.exe` and its DLLs into `%LOCALAPPDATA%\Programs\s32` and adds that folder to your user `PATH` (no administrator needed). Open a new terminal, or use the same window, and run `s32 --version`. If `s32.exe` is running, close it and run the line again.

## PATH, upgrade and versions

- On macOS and Linux the script never edits a shell file. If `~/.local/bin` is not on your `PATH`, it prints the line to add and the file it belongs in. If another `s32` comes first on your `PATH`, it warns you with both locations and the other version.
- Run the same line again to upgrade. It prints `s32 <old> -> <new> installed in <folder>`, or `s32 <version> reinstalled` when you already have the latest.
- To install a specific version, set `S32_VERSION` (`S32_VERSION=1.3.45` before `sh`, or `sh -s -- --version 1.3.45` after the pipe; on Windows `$env:S32_VERSION='1.3.45'` before the line). A pin may go back to an older release.
- `S32_INSTALL_DIR` changes the install folder. On Windows it must be a full path and a folder that holds nothing else: the installer replaces the whole folder, so it stops with `has other files` instead of touching a folder that has files of yours.
- Supported: macOS (Apple Silicon and Intel), Linux x86_64, Windows x64. On anything else the script stops with a link to the manual install below.

Each script is a release asset with its SHA-256 listed in `SHA256SUMS`: download it and read it first if you prefer.

## Manual install

Download the archive for your machine from [S32-CLI Releases](https://github.com/Shik-Tech/S32-CLI/releases).

macOS: `s32-*-darwin-arm64.tar.gz` (Apple Silicon) or `s32-*-darwin-x64.tar.gz` (Intel). libusb is built in. Download only the one for your Mac, so the wildcard below matches a single file.

```bash
tar xf s32-*-darwin-*.tar.gz
sudo install -m 755 s32 /usr/local/bin/s32
s32 --version
```

Linux: `s32-*-linux-x64.tar.gz` holds the binary and the udev rule.

```bash
tar xf s32-*-linux-x64.tar.gz
sudo install -m 755 s32 /usr/local/bin/s32
sudo apt-get install libusb-1.0-0          # Debian / Ubuntu
# sudo dnf install libusb1                 # Fedora
sudo install -m 644 99-s32-editor.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Windows: unzip `s32-*-windows-x64.zip` into a folder of its own and put that folder on your `PATH`. Keep the DLLs from the zip next to `s32.exe`.

The install folder belongs to the s32 CLI. The one-line installer replaces the whole folder on an upgrade, so it refuses (`has other files`, changing nothing) when the folder holds anything that is not part of the s32 CLI: a file from the release zip, or a file listed in the `.s32-install` list that an earlier one-line install wrote. A manual install does not write that list, so do not unzip into a shared folder such as your `bin` folder. Move the s32 CLI to an empty folder, or to the default `%LOCALAPPDATA%\Programs\s32`, before you run the one-line installer.

## First commands

```bash
s32 hello
s32 ping
```

Without an S32 plugged in, the s32 CLI uses its built-in simulator ([Connect](./connecting.md)). `hello` prints the protocol and firmware versions, the serial number and the S32's limits.

## Help and completion

```bash
s32 --help                 # the overview
s32 macro set --help       # arguments, keys and examples of one command
source <(s32 completion bash)   # Tab completion (zsh: s32 completion zsh)
```

See [Session](./commands/session.md#help) for the help pages, [exit codes](./commands/session.md#exit-codes) and `-q`.

## Interactive session

Run `s32` with no command (or `s32 repl`) to keep the connection open:

```text
s32 --usb repl
s32> list
s32> pull internal 1 /tmp/p.s32p --force
s32> quit
```

`pull` does not replace a file unless you add `--force`.
