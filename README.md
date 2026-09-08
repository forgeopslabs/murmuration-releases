# Murmuration

**A native macOS and Linux BitTorrent client built around swarm visibility,
durable downloads, and explicit network control.**

Murmuration combines a responsive desktop interface with a persistent daemon.
You can close the window without stopping active transfers, inspect what each
torrent is doing, and return later without losing state.

**Recommended macOS installation:**

```sh
brew install --cask forgeopslabs/tap/murmuration
```

[Homebrew cask](https://github.com/forgeopslabs/homebrew-tap/blob/main/Casks/murmuration.rb)
· [Linux packages](#install-and-open)
· [Manual downloads](https://github.com/forgeopslabs/murmuration-releases/releases)

![Murmuration torrent workspace](docs/images/murmuration-overview.png)

> [!WARNING]
> Current macOS builds are previews. They are ad-hoc signed but are not yet
> authenticated or notarized by Apple. First launch therefore requires a
> one-time approval in **System Settings > Privacy & Security**. Do not disable
> Gatekeeper or remove quarantine attributes.
>
> Linux is a signed ARM64-only preview. Linux x86_64 and GNOME/X11 are parked,
> and the complete distribution, renderer, and lifecycle matrix is still in
> progress. Do not treat the preview repository as a stable channel.

## User guide

- [Requirements](#requirements)
- [Install and open](#install-and-open)
- [Linux trust and preview scope](#linux-trust-and-preview-scope)
- [macOS approvals](#macos-approvals)
- [First run](#first-run)
- [Add a download](#add-a-download)
- [Manage torrents](#manage-torrents)
- [Dashboard, settings, and preferences](#dashboard-settings-and-preferences)
- [Background daemon](#background-daemon)
- [Terminal clients](#terminal-clients)
- [Upgrade](#upgrade)
- [Troubleshooting](#troubleshooting)
- [Uninstall](#uninstall)
- [Releases, source, and licensing](#releases-source-and-licensing)

## What Murmuration provides

- Native desktop GUI for Apple Silicon Macs and Linux ARM64.
- BitTorrent v1, v2, and hybrid torrent support.
- `.torrent` files and magnet links, with a native destination-folder picker.
- Persistent background daemon: transfers continue after the GUI closes.
- Download, upload, queue, pause, resume, remove, and per-file controls.
- Detailed torrent views for files, peers, trackers, piece availability, and
  live transfer behavior.
- DHT, tracker, PEX, local-peer discovery, web-seed, and incoming-peer support.
- Private-torrent discovery restrictions and network-scope leak diagnostics.
- Dark, light, and system appearance modes.
- Optional `magnet:` and `.torrent` default-app registration.
- Bundled command-line and terminal clients.

Murmuration does not provide anonymity. BitTorrent peers can see one another's
IP addresses. Protocol encryption can obscure traffic from casual inspection,
but it does not hide your network identity.

## Requirements

| Requirement | Details |
| --- | --- |
| macOS | macOS 11 Big Sur or later |
| Homebrew | Recommended installation method |
| Apple Silicon | Supported; Homebrew selects the `arm64` build automatically |
| Intel | New macOS releases no longer target Intel; older artifacts remain archived |
| Linux architecture | ARM64 (`aarch64`/`arm64`) preview only |
| Linux packages | Signed APT for Debian/Ubuntu; signed DNF for Fedora |
| Linux sessions | Preview.10 core checks on Ubuntu/Fedora GNOME Wayland; wider matrix incomplete |
| Network | Internet access; incoming connections recommended |
| Storage | Space for the app, state, and selected torrent payloads |

To identify your Mac, choose **Apple menu > About This Mac**. A Mac showing an
Apple chip uses the `arm64` build. Preview.13 and newer macOS releases target
Apple Silicon only.

On Linux, run `uname -m` and continue only when it prints `aarch64` or
`arm64`. Package availability does not yet mean that every compatible ARM64
distribution or graphics stack has completed qualification.

## Linux trust and preview scope

Murmuration's only Linux package origin is
`https://packages.forgeopslabs.com`. Verify the complete release-key
fingerprint before installing:

```text
D4D910B4D597AAB45541F3ED9D80207E60053983
```

Use repository-scoped trust. Never use `apt-key`, `trusted=yes`,
`--nogpgcheck`, disabled repository metadata checks, or `curl | sh`. Stop and
contact `hello@forgeopslabs.com` if the complete fingerprint differs.

The current Linux release is the signed ARM64-only
[0.1.0-preview.10](https://github.com/forgeopslabs/murmuration-releases/releases/tag/v0.1.0-preview.10).
It includes signed packages, APT/DNF metadata, Corresponding Source, checksums,
SBOM, and provenance. Linux x86_64 delivery remains deferred.

Current preview.10 ARM64 core checks cover the Ubuntu 24.04.4 APT and Fedora 44
DNF transitions from preview.9, daemon restart and settings retention, and
normal GUI/TUI exits. Ubuntu also passed portable isolation and CLI detached
fallback checks. The Fedora combined graphical upgrade sequence was interrupted
and remains inconclusive; its client-exit checks passed separately.

Earlier preview.9 evidence includes Ubuntu/Fedora uninstall/reinstall with
retained data and Ubuntu GNOME Wayland activation with confirmation. Older
Plasma Wayland/X11 activation results remain historical. These bounded results
do not establish the complete preview.10 distribution or desktop matrix.
Remaining distribution, hardware Vulkan, daemon-backend, transfer, and other
release-matrix coverage is incomplete; GNOME/X11 qualification is incomplete.

## Install and open

### Linux ARM64: signed APT repository

On Debian or Ubuntu, install the trust tools, download the public key as data,
and display its complete primary fingerprint:

```sh
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

curl --fail --location \
  --output /tmp/murmuration-linux-release.asc \
  https://packages.forgeopslabs.com/keys/murmuration-linux-release.asc
gpg --show-keys --with-colons /tmp/murmuration-linux-release.asc \
  | awk -F: '$1 == "fpr" { print $10; exit }'
```

Continue only when the output exactly matches the fingerprint above. Install
the verified key and generated ARM64 preview source, then install Murmuration:

```sh
sudo install -d -m 0755 /etc/apt/keyrings
gpg --dearmor --yes \
  --output /tmp/murmuration-linux-release.gpg \
  /tmp/murmuration-linux-release.asc
sudo install -m 0644 \
  /tmp/murmuration-linux-release.gpg \
  /etc/apt/keyrings/murmuration.gpg

curl --fail --location \
  --output /tmp/murmuration-preview.sources \
  https://packages.forgeopslabs.com/config/murmuration-preview.sources
sudo install -m 0644 \
  /tmp/murmuration-preview.sources \
  /etc/apt/sources.list.d/murmuration-preview.sources

sudo apt-get update
sudo apt-get install murmuration
```

Verify the selected origin, architecture, and shared version:

```sh
apt-cache policy murmuration
murmur --version
murmur-gui --version
murmur-tui --version
murmurd --version
```

### Linux ARM64: signed DNF repository

On Fedora, install `ca-certificates`, `curl`, and `gnupg2`, then download and
verify the public key. After the full fingerprint matches, install the key and
repository configuration:

```sh
sudo dnf install ca-certificates curl gnupg2
curl --fail --location \
  --output /tmp/murmuration-linux-release.asc \
  https://packages.forgeopslabs.com/keys/murmuration-linux-release.asc
gpg --show-keys --with-colons /tmp/murmuration-linux-release.asc \
  | awk -F: '$1 == "fpr" { print $10; exit }'

sudo install -D -m 0644 \
  /tmp/murmuration-linux-release.asc \
  /etc/pki/rpm-gpg/RPM-GPG-KEY-murmuration

curl --fail --location \
  --output /tmp/murmuration-preview.repo \
  https://packages.forgeopslabs.com/config/murmuration-preview.repo
sudo install -m 0644 \
  /tmp/murmuration-preview.repo \
  /etc/yum.repos.d/murmuration-preview.repo

sudo dnf install murmuration
```

DNF may ask you to confirm the imported key. Compare the full fingerprint
again. The repository keeps both package and repository-metadata verification
enabled. Check the installed identity with:

```sh
dnf --cacheonly info --installed murmuration
rpm -q --qf '%{NAME} %{VERSION}-%{RELEASE} %{ARCH}\n' murmuration
```

### Linux direct-download fallback

From the newest preview on the
[Releases](https://github.com/forgeopslabs/murmuration-releases/releases)
page, download the ARM64 `.deb`, `.rpm`, or portable `.tar.zst` together with
`SHA256SUMS`, `SHA256SUMS.asc`, and
`murmuration-linux-release-public-key.asc`.

Verify the key fingerprint first, then the signed manifest and selected file:

```sh
gpg --show-keys --with-colons murmuration-linux-release-public-key.asc \
  | awk -F: '$1 == "fpr" { print $10; exit }'
gpg --dearmor --yes \
  --output /tmp/murmuration-release-verification.gpg \
  murmuration-linux-release-public-key.asc
gpgv --keyring /tmp/murmuration-release-verification.gpg \
  SHA256SUMS.asc SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS
```

Install only the package for your distribution:

```sh
sudo apt-get install ./murmuration_VERSION_arm64.deb
sudo dnf install ./murmuration-VERSION-1.aarch64.rpm
```

The `./` prefix is required for a local file. Prefer APT over `dpkg -i` so
runtime dependencies are resolved. The portable archive can run without
installing desktop integration:

```sh
tar --zstd -xf murmuration-VERSION-linux-aarch64.tar.zst
./murmuration-VERSION-linux-aarch64/bin/murmur-gui
```

### macOS: recommended Homebrew installation

Install the official cask from the ForgeOps Labs tap:

```sh
brew install --cask forgeopslabs/tap/murmuration
```

Homebrew installs the Apple Silicon build,
verifies its checksum, installs **Murmuration.app**, and exposes `murmur`,
`murmur-tui`, and `murmurd` in your command path.

Upgrade later with:

```sh
brew upgrade --cask forgeopslabs/tap/murmuration
```

If Homebrew is not installed, use the manual method below or follow the
[official Homebrew installation guide](https://brew.sh/).

### Manual ZIP fallback

1. Open the [Releases](https://github.com/forgeopslabs/murmuration-releases/releases)
   page and select the newest preview.
2. Download the Apple Silicon ZIP: `murmuration-VERSION-macos-arm64.zip`.

Each release includes `SHA256SUMS`. Download it beside the app ZIP, then
calculate the ZIP's checksum before opening the archive:

```sh
shasum -a 256 murmuration-VERSION-macos-arm64.zip
```

Compare the printed hash with the matching line in `SHA256SUMS`. Continue
only when they match.

Open the verified ZIP, then drag **Murmuration.app** into **Applications**.

### First-launch approval

Homebrew and manual installations use the same first-launch process:

1. Open **Applications** and double-click **Murmuration** once. macOS will block
   this first attempt because the preview is not notarized.
2. Open **System Settings > Privacy & Security**. Scroll to **Security**, find
   the Murmuration message, then choose **Open Anyway**.
3. Authenticate if macOS asks, then confirm **Open**.

This creates a per-app exception. Future launches do not require repeating the
Gatekeeper approval unless you install a different build that macOS treats as a
new app.

Apple documents this process in
[Open a Mac app from an unknown developer](https://support.apple.com/guide/mac-help/open-a-mac-app-from-an-unknown-developer-mh40616/mac).

## macOS approvals

Murmuration asks only for access connected to downloading and peer networking.
Prompts vary by macOS version, firewall configuration, and chosen destination.
The embedded daemon may appear as `murmurd` in some system prompts.

- **Gatekeeper Open Anyway:** required for the current preview. It allows the
  non-notarized build to launch; without it, the app cannot open.
- **Downloads or selected-folder access:** allow for folders you choose. It is
  used to read `.torrent` files and write payloads; otherwise choose another
  folder or enable access later.
- **Local Network:** allow for full peer discovery. Denying it disables
  local-peer discovery.
- **Incoming network connections:** allow for better peer reachability and
  seeding. Outgoing transfers may still work if it is denied.
- **Make Murmuration the default:** optional. It opens `magnet:` links and
  `.torrent` files in Murmuration; otherwise the existing default remains.

Murmuration does **not** require Accessibility, Full Disk Access, Screen
Recording, Camera, Microphone, Contacts, or Location access.

You can review access later:

- **System Settings > Privacy & Security > Files and Folders** — see
  [Apple's Files and Folders guide](https://support.apple.com/guide/mac-help/control-access-to-files-and-folders-on-mac-mchld5a35146/mac).
- **System Settings > Privacy & Security > Local Network** — see
  [Apple's Local Network guide](https://support.apple.com/guide/mac-help/control-access-to-your-local-network-on-mac-mchla4f49138/mac).
- **System Settings > Network > Firewall > Options** — see
  [Apple's Firewall guide](https://support.apple.com/guide/mac-help/change-firewall-settings-on-mac-mh11783/mac).

## First run

Opening Murmuration automatically starts the bundled `murmurd` daemon if a
compatible daemon is not already running. The green **Daemon** indicator means
the GUI is connected.

Default locations on macOS:

| Data | Location |
| --- | --- |
| Downloaded payloads | `~/Downloads/Murmuration` |
| Persistent state | `~/Library/Application Support/Murmuration` |
| Logs | `~/Library/Logs/Murmuration` |
| Local daemon endpoint | `http://127.0.0.1:6899` |

Default locations on Linux:

- Downloaded payloads: `Murmuration` inside your configured desktop Downloads
  directory, normally `~/Downloads/Murmuration`.
- Persistent state: `$XDG_DATA_HOME/murmuration`, defaulting to
  `~/.local/share/murmuration`.
- Logs: `$XDG_STATE_HOME/murmuration/logs`, defaulting to
  `~/.local/state/murmuration/logs`.
- Local daemon endpoint: `http://127.0.0.1:6899`.

The RPC endpoint is loopback-only by default. It is for local GUI, TUI, and CLI
communication; do not expose it to another network.

If the daemon cannot start, Murmuration offers **Retry**, **Open Logs**, and
**Close**. Open the logs first when reporting or investigating a startup error.

## Add a download

Only download or share content you are permitted to use.

### Add a `.torrent` file

1. Choose **Add Torrent** in the toolbar.
2. Choose the `.torrent` file.
3. Keep the daemon default download location or choose **Change folder…**.
4. Review the paths, then choose **Add torrent**.

Murmuration validates the selected file before submitting it. If the source
file changes or disappears while being read, select it again.

### Add a magnet link

1. Choose **Add Magnet**.
2. Paste a complete URI beginning with `magnet:?`.
3. Keep the default destination or choose **Change folder…**.
4. Choose **Add magnet**.

![Adding a magnet link and selecting its download location](docs/images/murmuration-add-magnet.png)

Canceling the folder picker leaves the current destination unchanged. Magnet
metadata can take time to arrive when trackers or peers are slow; the download
appears after the daemon has enough metadata to identify it.

You can also open a `.torrent` file or `magnet:` link from another app after
making Murmuration the default in **Preferences**.

## Manage torrents

The left sidebar filters the workspace by **Active**, **Downloading**,
**Seeding**, **Completed**, and **Queue**. Use the toolbar search field to
narrow the current list.

Select a torrent, then use:

- **Pause** — stop transfer activity while retaining state.
- **Resume** — continue a paused torrent.
- **Remove** — choose whether to keep or delete downloaded data.
- **Overview** — progress, destination, ratio, swarm, and tracker health.
- **Files** — file tree, completion, and priorities.
- **Peers** — connected peers and transfer accounting.
- **Trackers** — tracker state, peer counts, and errors.
- **Insights** — piece availability, transfer history, upload slots, and disk
  activity.

> [!CAUTION]
> **Remove > Delete downloaded data** deletes payload files owned by that
> torrent. Choose **Keep downloaded data** when you only want to remove the
> torrent from Murmuration.

## Dashboard, settings, and preferences

**Dashboard** summarizes active torrents, connected peers, and aggregate
transfer rates. **Daemon** shows version, uptime, DHT state, free space, and
snapshot revision.

**Settings** lets you edit daemon-owned transfer limits and displays the
daemon-owned queue limits. Transfer changes apply to every connected client and
remain active after the GUI closes. A value of `0` means unlimited where shown.

**Preferences** contains client-local options:

- Make Murmuration the default for `magnet:` and `.torrent` opens.
- Change local daemon endpoint.
- Adjust GUI refresh interval.
- Select dark, light, or system appearance.

![Murmuration preferences](docs/images/murmuration-preferences.png)

Installation never changes the current `magnet:` or `.torrent` defaults. On
Linux, inspect them with `xdg-mime query default x-scheme-handler/magnet` and
`xdg-mime query default application/x-bittorrent`; desktop settings can restore
another handler. On macOS, registration depends on the macOS version and Launch
Services. If macOS rejects a `.torrent` change, select a `.torrent` file in
Finder, choose **Get Info > Open with**, select the preferred app, then choose
**Change All…**. If a `magnet:` change is rejected, keep the current handler;
no qualified manual recovery flow is currently provided.

## Background daemon

Closing or quitting the GUI does not stop active transfers. Reopen Murmuration
to reconnect to the existing daemon.

To stop it gracefully from Terminal:

```sh
murmur shutdown
```

For a manual ZIP installation, use
`/Applications/Murmuration.app/Contents/Resources/bin/murmur shutdown`.

After an update, the app may ask to restart an older daemon. Approve the
restart, or run:

```sh
murmur daemon restart
```

For a manual ZIP installation, use the full `murmur` path shown above.

## Terminal clients

APT, DNF, and Homebrew expose the bundled terminal UI and command-line clients
in your command path. Both start the bundled daemon on demand when possible.
Manual macOS ZIP and Linux portable installations include the same binaries in
their extracted application layout.

### Terminal UI

```sh
murmur-tui
```

Manual ZIP path:
`/Applications/Murmuration.app/Contents/Resources/bin/murmur-tui`.

Linux portable path:
`./murmuration-VERSION-linux-aarch64/bin/murmur-tui`.

For Linux preview.10, use a terminal at least 100 columns by 30 rows. Press `?`
inside the TUI for its current key guide.

### CLI examples

```sh
murmur list
murmur stats
murmur add /path/to/file.torrent
murmur status 1
murmur remove 1
murmur leakcheck
```

For a manual ZIP installation, replace `murmur` with
`/Applications/Murmuration.app/Contents/Resources/bin/murmur`.

For a Linux portable archive, replace it with
`./murmuration-VERSION-linux-aarch64/bin/murmur`.

CLI `remove` keeps downloaded data. Use the GUI when you need the explicit
keep-data/delete-data choice.

## Upgrade

APT:

```sh
sudo apt-get update
sudo apt-get install --only-upgrade murmuration
```

DNF:

```sh
sudo dnf upgrade murmuration
```

Homebrew:

```sh
brew upgrade --cask forgeopslabs/tap/murmuration
```

GUI, CLI, TUI, and daemon ship as one version. A daemon already in memory can
continue running the previous executable after the package is replaced. Use
the GUI restart prompt or run `murmur daemon restart`; state, settings, and
downloaded payloads remain in place.

## Troubleshooting

### Linux signature or fingerprint differs

Stop. Do not import an unexpected key or disable verification. Save the full
command output and contact `hello@forgeopslabs.com`.

### APT says “Unsupported file” or `dpkg` reports missing dependencies

Run the installation from the package directory with a required `./` prefix.
APT resolves dependencies that a direct `dpkg -i` invocation does not:

```sh
sudo apt-get install ./murmuration_VERSION_arm64.deb
```

### Linux GUI does not open

Record the session and renderer, then run `murmur-gui` from the same graphical
terminal and retain its output:

```sh
printf 'desktop=%s session=%s display=%s\n' \
  "$XDG_CURRENT_DESKTOP" "$XDG_SESSION_TYPE" "$DISPLAY"
vulkaninfo --summary
murmur-gui
```

Current preview.10 desktop checks cover Ubuntu and Fedora GNOME Wayland.
Earlier Plasma Wayland/X11 activation results are version-specific; GNOME X11
and the complete hardware Vulkan matrix are not qualified.

### “Murmuration cannot be opened”

Use the one-time **Privacy & Security > Open Anyway** process above. Do not run
commands that disable Gatekeeper or strip quarantine metadata.

### Daemon does not connect

1. Choose **Open Logs** from the startup error. Linux logs are under
   `$XDG_STATE_HOME/murmuration/logs`, defaulting to
   `~/.local/state/murmuration/logs`; macOS logs are in
   `~/Library/Logs/Murmuration`.
2. Check whether another Murmuration version is running.
3. Run `murmur daemon restart`. Linux users can also inspect
   `systemctl --user status murmurd.service`. The installed Linux unit is
   static, not enabled at login, and clients use a detached fallback when a
   usable user manager is unavailable.
4. Reopen the app.

### Downloads do not start

- Confirm the destination exists and has free space.
- Review **Privacy & Security > Files and Folders**.
- Allow Local Network and incoming connections if macOS prompted.
- Open **Trackers**, **Peers**, and **Insights** for source or availability
  errors.
- Some magnet links have no reachable metadata peers and cannot start.

### Magnet or `.torrent` links open in another app

Open **Preferences**, choose **Make Murmuration the default**, and confirm.
Linux users can inspect or restore both associations through desktop default-app
settings. If macOS rejects a `.torrent` change, use Finder's **Get Info > Open
with > Change All…** flow. If it rejects a `magnet:` change, keep the current
handler.

### Where are logs?

Linux logs are under `$XDG_STATE_HOME/murmuration/logs`, defaulting to
`~/.local/state/murmuration/logs`. On macOS, open
`~/Library/Logs/Murmuration` in Finder. The daemon log is `murmurd.log`.

## Uninstall

1. Quit the GUI.
2. If Murmuration is your default app, choose another BitTorrent app as the
   default for `.torrent` files and `magnet:` links using that app's controls.

For an APT installation:

```sh
sudo apt-get remove murmuration
```

For a DNF installation:

```sh
sudo dnf remove murmuration
```

Normal Linux package removal removes package-owned application files only. It
leaves state, settings, and downloaded payloads in place; reinstalling
reconnects to retained state. Review the Linux paths under [First
run](#first-run) and delete them only when you explicitly intend to erase that
data. This guide deliberately provides no recursive deletion command.

For a Homebrew installation:

```sh
brew uninstall --cask forgeopslabs/tap/murmuration
```

Homebrew stops the app and daemon during removal. To also remove Murmuration's
support data, preferences, and logs, use `--zap` instead:

```sh
brew uninstall --cask --zap forgeopslabs/tap/murmuration
```

For a manual ZIP installation, stop the daemon with
`/Applications/Murmuration.app/Contents/Resources/bin/murmur shutdown`, then
move **Murmuration.app** from **Applications** to the Trash.

Normal removal leaves support data and downloaded payloads in place. Homebrew
`--zap` removes the following support items; manual-install users can review and
move only these Murmuration-owned items to the Trash:

- `~/Library/Application Support/Murmuration`
- `~/Library/Caches/org.murmuration-bt.murmuration`
- `~/Library/Logs/Murmuration`
- `~/Library/Preferences/org.murmuration-bt.murmuration.plist`
- `~/Library/Saved Application State/org.murmuration-bt.murmuration.savedState`

Downloaded payloads are separate. Delete `~/Downloads/Murmuration` or another
chosen destination only when you intend to erase those files too.

## Releases, source, and licensing

This public repository contains distribution artifacts, checksums, release
notes, and the GPL Corresponding Source archive for each published version.
Signed Linux previews additionally include the public release key, detached
checksum signature, embedded RPM signature, SPDX SBOM, in-toto provenance, and
a deterministic signed APT/DNF repository evidence bundle.

- Applications are licensed under GPL-3.0-or-later.
- Engine libraries are licensed under MIT OR Apache-2.0.
- Exact Corresponding Source is attached to each release.

Preview limitations and first-launch instructions are also repeated in each
release's notes.
