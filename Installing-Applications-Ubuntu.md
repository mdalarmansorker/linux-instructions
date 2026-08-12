# Comprehensive Guide: Installing Applications on Ubuntu 26.04 LTS

There are several ways to install software on Ubuntu, depending on the file format you download. This guide covers the exact commands and steps for the most common Linux package formats.

---

## 1. Debian Packages (`.deb` files)
This is the standard installation file for Ubuntu and Debian-based systems, similar to a `.exe` or `.msi` in Windows. You will often download these directly from an app's official website (e.g., Google Chrome, VS Code).

**How to install or upgrade:**
1. Open your terminal.
2. Navigate to where the file is downloaded (usually the Downloads folder):
   ```bash
   cd ~/Downloads
   ```
3. Run the installation using `apt` (which automatically handles missing dependencies):
   ```bash
   sudo apt install ./filename.deb
   ```
   *Important: You MUST include the `./` before the filename so the system knows to look in your current folder instead of the internet.*

---

## 2. AppImages (`.AppImage` files)
AppImages are portable applications. You do not actually "install" them; you just give them permission to run, and they execute directly from wherever you saved them. 

**How to run:**
1. Open your terminal and navigate to the file:
   ```bash
   cd ~/Downloads
   ```
2. Make the file executable (you only have to do this once):
   ```bash
   chmod +x filename.AppImage
   ```
3. Run the app:
   ```bash
   ./filename.AppImage
   ```
   *(Alternatively, after making it executable in the terminal, you can just double-click the file in your file manager to open it).*

---

## 3. Shell Scripts (`.sh` or `.run` files)
Some developers distribute installers as scripts that automate the installation process for you. 

**How to install:**
1. Navigate to the folder containing the script:
   ```bash
   cd ~/Downloads
   ```
2. Make the script executable:
   ```bash
   chmod +x installer-name.sh
   ```
3. Run the script (often requires `sudo` if it installs system-wide):
   ```bash
   sudo ./installer-name.sh
   ```

---

## 4. Compressed Archives (`.tar.gz`, `.tar.xz`, or `.zip`)
A "tarball" is a compressed folder. Sometimes it contains pre-compiled binaries (ready to run), and sometimes it contains source code that you must compile yourself.

**Scenario A: Pre-compiled Binaries (Portable)**
1. Right-click the archive in your file manager and select **Extract Here**.
2. Open the extracted folder.
3. Look for an executable file (usually named after the app, or `run.sh`).
4. Double-click it, or run it from the terminal via `./app-name`.

**Scenario B: Source Code (Requires Compiling)**
If you are compiling from source, you usually need standard build tools:
```bash
sudo apt install build-essential
```
Then, extract the folder, open a terminal inside that folder, and run the standard build sequence:
```bash
./configure
make
sudo make install
```
*(Always read the `README.md` or `INSTALL` file inside the extracted folder first, as build commands can vary heavily depending on the language the app was written in).*

---

## 5. Official APT Repositories (Command Line Store)
This is the most common way to install software that is officially maintained by Ubuntu or community repositories. It downloads and installs the app from a verified online server.

**How to install:**
1. Update your local list of available software:
   ```bash
   sudo apt update
   ```
2. Install the application:
   ```bash
   sudo apt install package-name
   ```

---

## 6. Snap Packages (`snap`)
Snap is a universal package manager created by Canonical (the makers of Ubuntu). It bundles the app and all its dependencies into one secure sandbox. This is the default format used by the "Ubuntu Software" or "App Center" GUI.

**How to install:**
```bash
sudo snap install package-name
```
*(If you are downloading a beta version or an app that needs deep system integration, you might need to add the `--classic` flag at the end of the command).*

---

## 7. Flatpak Packages (`.flatpakref` or Flathub)
Flatpak is another universal, sandboxed package manager popular in the Linux community. Ubuntu doesn't come with it by default, but it is highly recommended for access to Flathub.

**How to set up Flatpak (One-time setup):**
```bash
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

**How to install an app from Flathub:**
```bash
flatpak install flathub com.developer.AppName
```
