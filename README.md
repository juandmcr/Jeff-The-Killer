# Jeff The Killer – Sleep Manager

> Important Notice (Audio Bug After Installation):
There is a known issue that occurs **immediately after installation**:
audio may stop working correctly until you **log out and log back in**.
This only happens once, right after installing.
After a logout
(for example using `loginctl terminate-user $USER`)
and a new session start, audio works normally and the problem does not reappear.
The underlying cause has not yet been identified D:
It seems related to how the new user-level systemd service is registered during its first activation. Until this is resolved, please ensure you perform a logout after installation to restore audio functionality.
Spoiler: idk if I am going to fix it anytime soon


# Overview

Jeff The Killer is a sleep manager for Linux desktop environments written entirely in Bash.
It prevents system sleep while you're actively using your computer or while audio is playing, and suspends the system after a configurable period of inactivity when no audio is detected. If you wish, your screen will be locked after you wake it up (f.e. using i3lock) like in Windows.

The name is inspired by the *Jeff The Killer* creepypasta and its phrase *“Go to sleep”*.

This is a Bash script + user-level systemd service that operates *without root* privileges.

**Important:** If you're already using another sleep inhibitor (for example Caffeine Daemon), running this alongside it may cause conflicts and unpredictable behaviour.

---

# Dependencies
- xprintidle (sudo apt install xprintidle)
- X11 Environment (Wayland shouldn't work)
- PulseAudio (usually pre-installed)

---

# How It Works

The script runs in a continuous loop checking two conditions:

1. **Audio activity** — checked using `pactl`
2. **User activity** — checked using `xprintidle`

If no audio is detected **and** the user is idle for longer than the configured time, the system is suspended.

---

# Quick Start

```bash
git clone https://github.com/juandmcr/Jeff-The-Killer.git
cd Jeff-The-Killer
chmod +x jeffthekiller
./jeffthekiller
````

Follow the installation wizard when prompted.

---

# Installation

## Automatic Installation (Recommended)

On first run, the script will detect that this is your first launch and ask whether to install automatically.
If you answer “y”, it will:

1. Copy the script to `~/.local/bin/jeffthekiller`
2. Create a systemd user service at `~/.config/systemd/user/gotosleep.service`
3. Save default settings to `~/.config/JeffTheKiller/settings.conf`
4. Enable and start the service

## Manual Installation

If you prefer manual installation, run:

```bash
# Create necessary directories
mkdir -p ~/.config/systemd/user
mkdir -p ~/.config/JeffTheKiller
mkdir -p ~/.local/bin

# Copy the script
cp jeffthekiller ~/.local/bin/
chmod +x ~/.local/bin/jeffthekiller

# Copy the service file (if it exists in the repository)
cp gotosleep.service ~/.config/systemd/user/

# If you intend to use systemctl mode (not dbus), install the polkit rule
sudo cp 85-suspend-no-auth.rules /etc/polkit-1/rules.d/
sudo systemctl restart polkit

# Enable the service
systemctl --user daemon-reload
systemctl --user enable gotosleep.service
systemctl --user start gotosleep.service
```

**Note regarding the polkit rule:**
This is not required unless you choose `systemctl` as your suspension method.
The default `dbus` mode does not need any polkit rule.

After manual installation, run `jeffthekiller` again, answer “yes” to the first-use question, and “no” to automatic installation.
The script will detect the manual setup and just create the settings file automatically.

---

# Verification

After installation, check that the service is running:

```bash
jeffthekiller -s
```

You should see output similar to the screenshot below, and in particular:

```
Active: active (running)
```
![Service Status](https://i.imgur.com/CVCO2Lo.png)

If not, check the logs:

```bash
jeffthekiller -j
```
![Service Logs](https://i.imgur.com/8xqbazy.png)

---

# Configuration

Run `jeffthekiller` to access the interactive menu and edit all settings.
Settings are stored at:

```
~/.config/JeffTheKiller/settings.conf
```

Key settings:

* `MAX_IDLE_TIME`: Minutes of inactivity before considering the system idle and (if no audio) going to sleep.
* `CHECK_AGAIN_EVERY`: Check if audio & user are inactive every X seconds until the MAX_IDLE_TIME is reached.
* `ENABLE_SCREEN_LOCK`: Use a screenlock such as i3lock or i3lock-fancy (you can specify yours under SCREEN_LOCK_COMMAND) after the sleep.

After changing settings, you'll be asked if you want to restart the service with the new settings or not. If you decide to not, you can always restart the service later using:

```bash
jeffthekiller -r
```

---

# Command Line Usage

Once installed, you can run `jeffthekiller` from anywhere as long as `~/.local/bin` is in your PATH.

### Service Management

```bash
jeffthekiller -s        # Check service status
jeffthekiller -j        # View service logs
jeffthekiller -r        # Restart service
```

### Temporary Disable

```bash
jeffthekiller -d X     # Disable for X minutes
jeffthekiller -d        # Disable indefinitely until restarted
```

### Debugging

```bash
jeffthekiller -j        # View live service logs
```

### Help

```bash
jeffthekiller -h        # Display help information
jeffthekiller           # Start interactive menu
```

---

# Polkit Rule

The `85-suspend-no-auth.rules` rule is required only if:

1. You choose `systemctl` as your `SUSPEND_MODE`, and
2. Your system does not allow passwordless suspension via systemctl by default.

The rule grants permission for your user to suspend the machine without authentication prompts when using `systemctl suspend`.

Most users will not need this rule, as the `dbus` method works without it.

---

# Cleanup

After installation, the cloned repository can be removed:

```bash
cd ..
rm -rf Jeff-The-Killer
```

The script remains installed at `~/.local/bin/jeffthekiller`.
The only file you may wish to keep is the polkit rule if you intend to use systemctl mode.

---

# Troubleshooting

If the service is not working:

1. Check status: `jeffthekiller -s`
2. View logs: `jeffthekiller -j`
3. Ensure `xprintidle` is installed:

   ```bash
   sudo apt install xprintidle
   ```
4. Confirm you are using X11 (not Wayland)
5. Ensure PulseAudio is running:

   ```bash
   pactl list sinks short
   ```

---

# Uninstallation

From the interactive menu, choose option 6 (Uninstall).
This removes:

* `~/.local/bin/jeffthekiller`
* `~/.config/JeffTheKiller/settings.conf`
* `~/.config/systemd/user/gotosleep.service`

Note: If you installed the polkit rule, it must be removed manually.

---

# License

Free to use, modify, for, and distribute.
No warranty provided.

```
```
