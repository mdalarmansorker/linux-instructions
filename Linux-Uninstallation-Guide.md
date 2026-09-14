# The Ultimate Linux Uninstallation Guide

Applications in Linux can be installed in several different ways. If you cannot find an app in your standard package manager, it might be a Snap, a Flatpak, or a manually installed binary. 

Follow this guide step-by-step to track down and completely remove any application.

---

## 1. The GUI Method (App Center / Software)
The easiest way to remove standard applications is through your distribution's graphical software manager.

1. Open **App Center** (or **Ubuntu Software** / **GNOME Software**).
2. Navigate to the **Installed** or **Manage** tab.
3. Locate the application in the list.
4. Click **Uninstall** or the trash can icon.

---

## 2. APT (Advanced Package Tool)
APT is the default package manager for Debian and Ubuntu-based systems. Use this for standard system packages and `.deb` files.

**Find the exact package name:**
```bash
dpkg --list | grep -i "app_name"
```

**Uninstall the package:**
```bash
sudo apt remove exact_package_name
```

**Uninstall the package AND remove its configuration files:**
```bash
sudo apt remove --purge exact_package_name
```

**Clean up leftover dependencies:**
```bash
sudo apt autoremove
```

---

## 3. Snap Packages
Snap is heavily used in modern Ubuntu systems for containerized applications.

**List all installed Snaps:**
```bash
snap list
```

**Uninstall a Snap package:**
```bash
sudo snap remove exact_package_name
```

---

## 4. Flatpak Packages
Flatpak is another popular universal package format for Linux. 

**List all installed Flatpaks:**
```bash
flatpak list
```

**Uninstall a Flatpak package:**
```bash
flatpak uninstall exact_package_name
```

*Note: You can also use `flatpak uninstall --unused` to remove leftover runtimes that are no longer needed.*

---

## 5. Standalone Files (.AppImage or Binaries)
AppImages and portable binaries are not "installed" in the traditional sense. They are just executable files.

1. Locate where you downloaded or moved the file (usually `~/Downloads`, `~/Desktop`, or `~/Applications`).
2. Delete the file:
```bash
rm /path/to/the/AppImage_or_Binary
```

---

## 6. Manual Removal (Tracing hidden applications)
If an application appears in your app menu but isn't managed by APT, Snap, or Flatpak, it was likely extracted manually (often by an install script like Thonny, PyCharm, or tarball extracts). 

You can remove it by finding its menu shortcut (`.desktop` file) and tracing it to the source.

**Step A: Find the shortcut file**
Search the standard application directories for the app's name:
```bash
find ~/.local/share/applications /usr/share/applications /usr/local/share/applications -iname "*app_name*"
```

**Step B: Read the shortcut to find the app files**
Using the path found in Step A, look for the `Exec=` line to see where the app is launched from:
```bash
cat /path/to/found_file.desktop | grep Exec
```

**Step C: Delete the application folder**
The `Exec=` output will show you a path (e.g., `/home/user/apps/app_name/bin/app`). Delete the parent folder containing those application files:
```bash
rm -rf /path/to/the/application_folder
```

**Step D: Delete the shortcut**
Finally, remove the `.desktop` file so the icon disappears from your app menu:
```bash
rm /path/to/found_file.desktop
```

*Pro-tip: If the app icon still appears in your menu after deleting the `.desktop` file, simply log out and log back in to refresh the graphical shell.*