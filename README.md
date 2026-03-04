# CRISP GNU/Linux

> **Clean, Reliable, Intelligent, Simple Platform**

---

## System Overview

| Component        | Choice                            |
|------------------|-----------------------------------|
| Base             | LFS (Linux From Scratch)          |
| Init System      | runit                             |
| Bootloader       | Limine                            |
| Kernel           | Slightly Tweaked Linux Zen        |
| Base System      | glibc + GNU coreutils             |
| Filesystem       | btrfs (with snapshots)            |
| Display Server   | Wayland                           |
| Default Shell    | Z Shell (zsh)                     |
| Network Manager  | iwd + dhcpcd                      |
| Package Manager  | SOBI (details below)              |
| Greeter          | ly                                |
| DE/WM            | XFCE (pre-configured/riced)       |
| Privilege Tool   | doas (replaces sudo)              |

---

## Package Manager: SOBI

> ***Source or Binary Installer***

SOBI is CRISP's native package manager. It supports installing from binary or source, with a smart default that prefers binaries and falls back to source automatically when no binary is available.

### Features

- **Smart default** - prefers binary, falls back to source automatically
- **Community repo** - packages submitted by community members
- **Local installs** - supports portable `.sobi` package files for offline/local use
- **Clean builds** - build dependencies are automatically removed after installation

### Commands

| Action                  | Command                    |
|-------------------------|----------------------------|
| Install (smart default) | `sobi ins firefox`         |
| Install binary          | `sobi ins -b firefox`      |
| Install from source     | `sobi ins -s firefox`      |
| Install community pkg   | `sobi ins -c firefox`      |
| Install from .sobi file | `sobi ins -f file.sobi`    |
| Update all packages     | `sobi upd`                 |
| Remove a package        | `sobi rem firefox`         |

### The `.sobi` Format

`.sobi` is SOBI's portable package format, similar to `.deb` or `.rpm`. It allows packages to be distributed and installed locally or offline without needing a network connection to a repo.

### Repositories
The repos are stored in `/etc/sobi.d/repos.yml` and this is what an example would look like:
```yaml
# Example Repos file
Keyring: "https://repo.crisplinux.sh/public.gpg"

REPOS:
  Main:
    Binary: "https://repo.crisplinux.sh/main/binaries/"
    Source: "https://repo.crisplinux.sh/main/source/"
    Enabled: true

  Community:
    Binary: "https://repo.crisplinux.sh/community/binaries/"
    Source: "https://repo.crisplinux.sh/community/source/"
    Enabled: true
```
> ^ *(This is not complete and will probably change with time)*

---

### DOAS Overview
doas will come with a pre-configured & locked down configuration that will look something like this:
```conf
# Allow wheel group to run any command
permit persist :doasperm

# Allow wheel to run specific commands without password
permit nopass :doasperm cmd /sbin/reboot
permit nopass :doasperm cmd /sbin/shutdown
permit nopass :doasperm cmd /sbin/poweroff
```
The config will live in `/etc/doas.conf` and can be configured via a custom GUI made by the CRISP team. *(the GUI can only be used by root or users in the **doasperm** group)*

- **NOTE:** During installation, while making a user the TUI will ask the user **"Should this user have admin privileges?"** This will determine whether that user is in the **doasperm** group or not.

> ***doasperm** is the group CRISP uses instead of the usual **wheel** group*

---

### Installation Overview
On boot, the live ISO drops into a TTY where the user will login with the username `root` *(the password is `crispy`)*. Running install launches a TUI installer that walks through locale, timezone, hostname, user setup, and bootloader configuration. Disk partitioning is handled manually using `cfdisk` (recommended) or `fdisk`, giving the user full control over their layout before installation begins.

---

### Architectures it should support
- `x86_64 (amd64)` - **64 bit**, Most modern systems run this architecture
- `i686 (x86)` - **32 bit**, some older systems run this architecture
- `aarch64 (ARM64)` - **64 bit**, Systems like Raspberry Pis run this architecture
- `PowerPC (ppc64le)` - **64 bit** - IBM POWER servers and some embedded systems run this architecture

---

### The team this project will need

- **Multiple OSDevs** - Building off LFS and maintaining a custom kernel requires significant low-level expertise. At minimum 4 OSDevs are recommended.
- **SOBI Developers** - SOBI is essentially its own standalone project and needs dedicated developers separate from the OSDevs.
- **Infrastructure/Ops** - Manages repo hosting, mirrors, package signing, and CI/CD pipelines for automated builds. Needed before anything is publicly usable.
- **Security** - Handles CVE tracking, vulnerability response, and the package signing process.
- **QA/Testers** - Dedicated people whose job is breaking things across all supported architectures and upgrade paths.
- **Documentation Writers** - Critical for adoption, especially for an LFS-based distro. Needs to be someone's dedicated responsibility.
- **Community/Project Manager** - Keeps momentum going, triages issues, and onboards new contributors. Projects often die from coordination failures, not technical ones.
- **Ricers/Designers** - For configuring and setting up the pre-riced XFCE environment.
- **Artist** - For official wallpapers, the logo, and other visual assets.
- **Web Developer** - For the official website.

#### Recruiting Priority
1. **OSDevs and SOBI Developers** - nothing exists without them
2. **Infrastructure/Ops** - needed before anything is publicly usable
3. **Documentation and QA** - needed before any public release
4. **Everything else** - Once the project has something real to show

---

### Other
- The ISO format will always be: `crisp_live_v[version]_[MM]-[DD]-[YYYY].iso`
    - An example of this would be: `crisp_live_v0.1_03-04-2026.iso`
- I plan on getting the `crisplinux.sh` domain but if thats not available or too pricey:
    - `crisp.org` - (most likely taken but worth a shot cause `.org` is cheap)
    - `crisplinux.org` - Fallback if the one above is taken
    - `crisp-project.org` - Just sounds nice
    - `crisplinux.dev` - `.dev` sounds *fancy*
    - `getcrisp.org` - Like how Fedora has `getfedora.org`
    - `crisplinux.net` - `.net` is also cheap
- I want the logo to be a tube of wasabi (mainly because the package managers name `SOBI` reminds me of wasabi for whatever reason)
- Automatically symlink doas to where sudo would be so that apps that assume sudo work fine (`/usr/bin/sudo -> doas`)
    - I plan on seeing if instead of this I could develop a compatibility layer for this instead

---
### About CRISP & SOBI
CRISP GNU/Linux is a concept I hope to make real one day. The core idea behind SOBI came from a frustration with having to commit to either a source-based distro (like Gentoo) or a binary-based one (like Arch). Why not both? SOBI takes Portage's philosophy of building from source when it matters, and pairs it with the convenience of binaries when it doesn't.

> *If you wanna help work on this contact me via Github messages :)*
