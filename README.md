🐧 Part 1: Setup on Linux (Ubuntu / Debian / Oracle Cloud)
1. Install System Packages & OpenVPN
Run this single command in terminal to install OpenVPN, curl, and window/desktop management tools:

bash
sudo apt-get update && sudo apt-get install -y openvpn curl wmctrl xdotool tmux python3-venv python3-pip
2. Configure Passwordless Sudo for OpenVPN
The Python auto-rotator (rotate_and_run_linux.py) runs OpenVPN as a daemon and terminates it when switching IPs. To allow it to do this automatically without prompting you for a sudo password every cycle, run:

bash
echo "$USER ALL=(ALL) NOPASSWD: /usr/sbin/openvpn, /usr/bin/pkill" | sudo tee /etc/sudoers.d/openvpn
sudo chmod 0440 /etc/sudoers.d/openvpn
3. Setup Python Virtual Environment & Playwright
From the project folder:

bash
cd CloudflareAuto
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
playwright install --with-deps chromium
4. NordVPN Credentials
The script 

src/automation/nordvpn_rotator.py
 will automatically generate .nord_auth.txt with your Nord service credentials if it does not already exist. If you want to verify or create it manually:

bash
cat << 'EOF' > .nord_auth.txt
NnZb1qmmvz4iJf7y8EjWDjo8
uwXH67Ynz64N3M85r9EuwQxa
EOF
chmod 600 .nord_auth.txt
5. Run the 10-Worker Auto-Rotator & Dashboard
Make scripts executable and run:

bash
chmod +x launch_multi_linux.sh run_linux_auto_rotate.sh rotate_and_run_linux.py
./run_linux_auto_rotate.sh
What happens:
The runner connects OpenVPN to a random NordVPN country server.
Opens 10 visible terminal windows on your desktop (or arranged in tmux if on headless SSH).
Chromium browser runs headlessly.
The main terminal displays the live real-time dashboard (Successes, Failures, IP, Speed, Active Workers).
When the batch completes or a rate limit occurs, it rotates to a new NordVPN location and launches 10 fresh workers.
🪟 Part 2: Setup on Windows (Windows 10 / 11)
1. Install OpenVPN Community Edition
OpenVPN needs to be installed on your Windows system:

Option A (Via PowerShell / Winget) — Recommended: Open PowerShell as Administrator and run:
powershell
winget install OpenVPNTechnologies.OpenVPN
Option B (Direct Download): Download the Windows 64-bit MSI installer from official OpenVPN site: 👉 https://openvpn.net/community-downloads/ (Install using default settings into C:\Program Files\OpenVPN).
2. Setup Python Environment & Dependencies
You can simply double-click 

setup_windows.bat
, or run manually in Command Prompt / PowerShell:

cmd
python -m venv .venv
call .venv\Scripts\activate.bat
pip install --upgrade pip
pip install -r requirements.txt
playwright install chromium
3. NordVPN Credentials
Like Linux, .nord_auth.txt is automatically created on first launch if absent. To manually create it in the project root:

cmd
(echo NnZb1qmmvz4iJf7y8EjWDjo8& echo uwXH67Ynz64N3M85r9EuwQxa) > .nord_auth.txt
4. Run the 10-Worker Auto-Rotator & Dashboard
IMPORTANT

Open Command Prompt or PowerShell as Administrator on Windows so OpenVPN has permission to route network traffic through the virtual adapter.

In your Admin console, navigate to the folder and run:

cmd
run_windows_auto_rotate.bat
(Or in PowerShell: .\run_windows_auto_rotate.ps1)

What happens:
Connects to a NordVPN server via OpenVPN and verifies the new external IP.
Automatically launches and tiles 10 visible CMD terminal windows in a 5×2 grid across your desktop.
Headless Chromium runs in background.
The supervisor terminal displays the live dashboard.
Upon batch completion or rate-limit detection, it closes the worker windows, rotates NordVPN, and relaunches 10 fresh workers.



Viewed .nord_auth.txt:1-3
Ran command: `head -n 40 $(ls ovpn_configs/*.ovpn | head -n 1)`
Viewed nordvpn_rotator.py:40-75
Viewed nordvpn_rotator.py:180-250
Edited nordvpn_rotator.py
Ran command: `python3 -m py_compile src/automation/nordvpn_rotator.py`
Ran command: `git add src/automation/nordvpn_rotator.py && git commit -m "Add privilege checks and OpenVPN log diagnostic output on Windows and Linux" && git push origin main`

### 🔑 NordVPN Manual Service Credentials

Here are the NordVPN service credentials:

```text
Username:  NnZb1qmmvz4iJf7y8EjWDjo8
Password:  uwXH67Ynz64N3M85r9EuwQxa
```

---

### 📝 Where & How to Enter Them

#### 1. For the Automatic Runner (`.nord_auth.txt`)
Because `.nord_auth.txt` is deliberately ignored by Git for security, create the file manually inside your project directory on Linux or Windows:

* **On Linux**:
  ```bash
  cat << 'EOF' > .nord_auth.txt
  NnZb1qmmvz4iJf7y8EjWDjo8
  uwXH67Ynz64N3M85r9EuwQxa
  EOF
  chmod 600 .nord_auth.txt
  ```

* **On Windows (Command Prompt)**:
  ```cmd
  (echo NnZb1qmmvz4iJf7y8EjWDjo8& echo uwXH67Ynz64N3M85r9EuwQxa) > .nord_auth.txt
  ```

* **On Windows (PowerShell)**:
  ```powershell
  Set-Content -Path .nord_auth.txt -Value @("NnZb1qmmvz4iJf7y8EjWDjo8", "uwXH67Ynz64N3M85r9EuwQxa")
  ```

---

### 🛠️ Why the Connection Was Failing on Linux / Windows & How to Fix

#### 1. On Windows
* **Root Cause**: On Windows, OpenVPN **cannot add routing table entries** or bind to the network adapter unless PowerShell / Command Prompt is opened with **Administrator privileges**.
* **Fix**:
  1. Close any regular terminal.
  2. Search for **PowerShell** or **Command Prompt** in the Start Menu, **Right-Click** and select **"Run as Administrator"**.
  3. Navigate to the project and run:
     ```cmd
     run_windows_auto_rotate.bat
     ```
  4. Ensure OpenVPN is installed:
     ```powershell
     winget install OpenVPNTechnologies.OpenVPN
     ```

#### 2. On Linux (Ubuntu / Debian)
* **Root Cause**: OpenVPN requires root permissions to configure network interfaces (`tun`). If passwordless `sudo` is not configured, OpenVPN is denied silently with `sudo: a password is required`.
* **Fix**:
  Run this one-time command on your Linux machine to allow the user to start OpenVPN without a password prompt:
  ```bash
  echo "$USER ALL=(ALL) NOPASSWD: /usr/sbin/openvpn, /usr/bin/pkill" | sudo tee /etc/sudoers.d/openvpn
  sudo chmod 0440 /etc/sudoers.d/openvpn
  ```
  Then run:
  ```bash
  git pull
  ./run_linux_auto_rotate.sh
  ```

---

### 🚀 Improved Error Reporting
I have updated [src/automation/nordvpn_rotator.py](file:///Users/pynshainongsiej/Desktop/CloudflareAuto/src/automation/nordvpn_rotator.py) and pushed it to Git:
* It now actively checks if OpenVPN is installed and if you have Administrator / Sudo rights before attempting to connect.
* If a connection fails, it will now print the **exact OpenVPN log tail** to the console, showing the exact reason (e.g. invalid credentials, missing driver, or permission denial).
