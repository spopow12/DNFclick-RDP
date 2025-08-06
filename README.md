
# 🖥️ Create RDP Using GitHub Codespaces & Playit

🎥 **Watch the full tutorial on YouTube**  
📎 **Repository Link:** [DNFclick RDP](https://github.com/spopow12/DNFclick-rdp)

---

## ⚙️ Step 1: Create a New Codespace

1. Go to [GitHub Codespaces](https://github.com/codespaces)
2. Choose your repository `DNFclick-rdp`
3. Click **"Create codespace"**

---

## 🧩 Step 2: Install Playit Inside the Codespace

Open the terminal and run the following commands:

```bash
# Add Playit GPG key
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | \
sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null

# Add Playit APT repository
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | \
sudo tee /etc/apt/sources.list.d/playit-cloud.list

# Update package list
sudo apt update

# Install Playit
sudo apt install playit
```

---

## 🛠️ Step 3: Run Playit

```bash
playit
```

Then:

1. Create an account on [playit.gg](https://playit.gg)
2. Add port `3389` (RDP)
3. Copy the displayed IP and Port
4. Terminal : playit secret-path 'Copy the **secret path** displayed by Playit'

---

## 📁 Step 4: Create GitHub Action for RDP

1. Create this path in your repo:  
   `CustomRepositories/.github/workflows/rdp.yml`

2. Paste the following code in the `rdp.yml` file:

```yaml
name: DNFCLICK

on:
  workflow_dispatch:

jobs:
  setup-rdp-tunnel:
    runs-on: windows-latest

    steps:
    - name: Check out the repository
      uses: actions/checkout@v2

    - name: Download and Install Playit
      run: |
        Invoke-WebRequest -Uri "https://github.com/playit-cloud/playit-agent/releases/download/v0.15.26/playit-windows-x86_64-signed.exe" -OutFile "$env:USERPROFILE\playit.exe"
        Start-Sleep -Seconds 5

    - name: Enable Terminal Services
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0

    - run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    - run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
    - run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "Password@123" -Force)

    - name: Run Remote Desktop with Playit
      env:
        PLAYIT_AUTH_KEY: ${{ secrets.PL }}
      run: |
        Start-Process -FilePath "$env:USERPROFILE\playit.exe" -ArgumentList "--secret $env:PLAYIT_AUTH_KEY" -NoNewWindow -Wait
        Start-Process -FilePath "$env:USERPROFILE\playit.exe" -NoNewWindow

    - name: Keep GitHub Action Runner Alive
      run: |
          Start-Sleep -Seconds 21600
```

---

## 🔐 Step 5: Add Your Playit Secret

1. Go to your GitHub repository
2. Navigate to **Settings > Secrets > Actions**
3. Add a new secret:
   - Name: `PL`
   - Value: your `secret-path` from Playit

---

## 🚀 Step 6: Run the Workflow

1. Go to the **Actions** tab
2. Select the **DNFCLICK** workflow
3. Click **"Run workflow"**

---

## 🖥️ RDP Login Info

- **Username:** `runneradmin`  
- **Password:** `Password@123`  
- **IP + Port:** From Playit dashboard

---

> ⚠️ **Note:** The session will stay active for up to 6 hours unless manually stopped.
