# CRISP GNU/Linux

> **Clean, Reliable, Intelligent, Simple Platform**

---

## System Overview

| Component        | Choice                            |
|:-----------------|----------------------------------:|
| Base             | LFS (Linux From Scratch)          |
| Kernel           | Slightly Tweaked Linux Zen        |
| Init System      | runit                             |
| Base System      | glibc + GNU coreutils             |
| C Compiler       | GCC                               |
| Filesystem       | btrfs (with snapshots)            |
| Bootloader       | Limine                            |
| Network Manager  | iwd + dhcpcd                      |
| Audio Server     | PipeWire                          |
| Boot Splash      | Plymouth (custom CRISP theme)     |
| Privilege Tool   | doas (replaces sudo)              |
| Default Shell    | Z Shell (zsh)                     |
| Text Editor      | nano (base install)               |
| Package Manager  | SOBI (details below)              |
| Display Server   | Wayland                           |
| Greeter          | ly or emptty (user choice)        |
| DE/WM            | XFCE (pre-configured/riced)       |
| Terminal App     | Kitty (pre-configured)            |
| File Manager     | Thunar (pre-configured)           |
| Browser          | Firefox ESR                       |

---

## Package Manager: SOBI

> ***S**ource **o**r **B**inary **I**nstaller*

SOBI is CRISP's native package manager. It supports installing from binary or source, with a smart default that prefers binaries and falls back to source automatically when no binary is available.

### Features

- **Smart default** - prefers binary, falls back to source automatically
- **Community repo** - packages submitted by community members (disabled by default)
- **Local installs** - supports portable `.sobi` package files for offline/local use
- **Clean builds** - build dependencies are automatically removed after installation
- **Kernel management** - tracks installed kernels and can remove old ones with `sobi clean -k`
- **Repo name formatting** - repo names like `MyCustomRepo` are automatically formatted as `My Custom Repo` when displayed

### Commands

| Action                  | Command                    |
|:------------------------|---------------------------:|
| Install (smart default) | `sobi ins firefox`         |
| Install binary          | `sobi ins -b firefox`      |
| Install from source     | `sobi ins -s firefox`      |
| Install community pkg   | `sobi ins -c firefox`      |
| Install from .sobi file | `sobi ins -f file.sobi`    |
| Update all packages     | `sobi upd`                 |
| Remove a package        | `sobi rem firefox`         |
| List active repos       | `sobi lst repos`           |
| List installed packages | `sobi lst pkgs`            |
| Clean unused deps       | `sobi cln d`               |
| Clean old kernels       | `sobi cln k`               |

### The `.sobi` Format

`.sobi` is SOBI's portable package format, similar to `.deb` or `.rpm`. It allows packages to be distributed and installed locally or offline without needing a network connection to a repo.

### Repositories
The repos are stored in `/etc/sobi.d/repos.yml` and this is what an example would look like:
```yaml
# Example Repos file

REPOS:
  Main:
    Binary: "https://repo.crisplinux.sh/main/binaries/"
    Source: "https://repo.crisplinux.sh/main/source/"
    Keyring: "https://repo.crisplinux.sh/public.gpg"
    Enabled: true

  Community:
    # Community repo is disabled by default for security.
    Binary: "https://repo.crisplinux.sh/community/binaries/"
    Source: "https://repo.crisplinux.sh/community/source/"
    Keyring: "https://repo.crisplinux.sh/public.gpg"
    Enabled: false
  
  MyCustomRepo:
    Binary: "https://mywebsite.org/repos/crisp-linux/bin/"
    Source: null # Note: if this is set to null it will ignore it
    Keyring: "https://mywebsite.org/repos/crisp-linux/pub.gpg"
    Enabled: true
```
> ^ *(This is not complete and will probably change with time)*

---

## Boot Splash

CRISP uses a custom boot splash screen to cover the runit startup output, giving the system a clean and polished first impression. This is handled via **Plymouth**, which is started early in the boot process before runit brings up services.

### Behavior
- On boot, the Plymouth splash appears immediately after the bootloader hands off to the kernel
- The splash displays the CRISP logo (wasabi tube) with a minimal loading indicator
- Any runit service output is hidden behind the splash by default
- If the boot fails or takes unusually long, the splash will drop to the TTY automatically so the user can see what went wrong
- Pressing `ESC` during boot will dismiss the splash and show the raw runit output for debugging

### Theming
The splash will ship with a custom CRISP theme built around the wasabi tube logo and the distro's color palette. The theme files will live in `/usr/share/plymouth/themes/crisp/`.

> *Verbose boot (no splash) can be enabled by passing `plymouth.enable=0` to the kernel at boot via Limine*

---

## DOAS Overview
doas will come with a pre-configured & locked down configuration that will look something like this:
```conf
# Note: CRISP uses 'doasperm' instead of the conventional 'wheel' group

# Allow doasperm group to run any command
permit persist :doasperm

# Allow doasperm to run specific commands without password (these are not the default, just examples the user might add)
permit nopass :doasperm cmd /sbin/reboot
permit nopass :doasperm cmd /sbin/shutdown
permit nopass :doasperm cmd /sbin/poweroff
permit nopass :doasperm cmd /bin/sobi
```
The config will live in `/etc/doas.conf` and can be configured via a planned custom GUI made by the CRISP team. *(the GUI can only be used by root or users in the **doasperm** group)*

- **NOTE:** During installation, while making a user the TUI will ask the user **"Should this user have admin privileges?"** This will determine whether that user is in the **doasperm** group or not.

---

## Installation Overview

On boot, the live ISO drops into a TTY where the user will login with the username `root` *(the password is `crispy` and will be changed during installation)*. Running `install` launches a TUI installer that walks through the following steps:

1. **Locale & timezone** selection
2. **Disk partitioning** - handled manually using `cfdisk` (recommended) or `fdisk`, giving the user full control over their layout
3. **Filesystem setup** - btrfs with snapshots configured automatically
4. **Bootloader configuration** - Limine, with both UEFI and BIOS/Legacy supported
5. **Hostname & network** setup
6. **User creation** - including the admin privileges prompt (see DOAS Overview)
7. **Package selection** - minimal or full install

> *UEFI is recommended on modern hardware. BIOS/Legacy mode is supported for older systems.*

---

## Supported Architectures
- `x86_64 (amd64)` - **64 bit**, Most modern systems run this architecture
- `i686 (x86)` - **32 bit**, some older systems run this architecture
- `aarch64 (ARM64)` - **64 bit**, Systems like Raspberry Pis run this architecture
> *Support for PowerPC might come soon*

---

## The Team This Project Will Need

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

### Recruiting Priority
1. **OSDevs and SOBI Developers** - nothing exists without them
2. **Infrastructure/Ops** - needed before anything is publicly usable
3. **Documentation and QA** - needed before any public release
4. **Everything else** - Once the project has something real to show

---

## Other

### Releases
- The ISO format will always be: `crisp_live_v[version]_[MM]-[DD]-[YYYY].iso`
    - An example of this would be: `crisp_live_v0.1_03-04-2026.iso`

### Domain
- I plan on getting the `crisplinux.sh` domain but if that's not available or too pricey:
    - `crisp.org` - (most likely taken but worth a shot cause `.org` is cheap)
    - `crisplinux.org` - Fallback if the one above is taken
    - `crisp-project.org` - Just sounds nice
    - `crisplinux.dev` - `.dev` sounds *fancy*
    - `getcrisp.org` - Like how Fedora has `getfedora.org`
    - `crisplinux.net` - `.net` is also cheap

### Branding
- The logo will be a tube of wasabi *(mainly because the package manager's name `SOBI` reminds me of wasabi for whatever reason)*
- These are the colors I want in the color pallete:
    - `#6fe897` - Primary
    - `#45852f` - Secondary
    - `#edfee7` - Tertiary

### Compatibility
- CRISP ships a compatibility layer so that apps assuming `sudo` or `pkexec` work out of the box:
    - `/usr/bin/sudo` → symlink or wrapper pointing to `doas`
    - `/usr/bin/pkexec` → thin wrapper script, since `pkexec` uses a different calling convention than `doas`, a raw symlink won't work here
- The goal is to eventually replace these with a proper compatibility layer developed by the CRISP team

---

## About CRISP & SOBI
CRISP GNU/Linux is a concept I hope to make real one day. The core idea behind SOBI came from a frustration with having to commit to either a source-based distro (like Gentoo) or a binary-based one (like Arch). Why not both? SOBI takes Portage's philosophy of building from source when it matters, and pairs it with the convenience of binaries when it doesn't.

> *If you wanna help work on this contact me via Github messages :)*
