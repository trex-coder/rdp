# Windows RDP via GitHub Actions
Educational-use only. Do NOT misuse this project.
I am not responsible for any damages caused by misuse.

This GitHub Actions workflow creates a temporary Windows RDP session (7 hours max per run) using ngrok.

--------------------------------------------------------------
RDP Login Credentials
--------------------------------------------------------------
USERNAME: runneradmin
PASSWORD: P@ssw0rd!

You MUST set your ngrok authtoken:
GitHub → Repository → Settings → Secrets and Variables → Actions → NGROK_AUTH_TOKEN

--------------------------------------------------------------
Start / Stop RDP Session
--------------------------------------------------------------

Start RDP (Triggers the workflow)
https://api.github.com/repos/trex-coder/rdp/actions/workflows/CI.yml/dispatches

Stop RDP (Cancels running workflow)
https://api.github.com/repos/trex-coder/rdp/actions/runs/active/cancel


--------------------------------------------------------------
How These Buttons Work
--------------------------------------------------------------
GitHub requires an authenticated request. If you are logged into GitHub,
clicking the links automatically includes your credentials.

Start Workflow:
POST /repos/{owner}/{repo}/actions/workflows/{workflow_id}/dispatches
{
  "ref": "main"
}

Stop Workflow:
POST /repos/{owner}/{repo}/actions/runs/{run_id}/cancel


--------------------------------------------------------------
Setup (Required)
--------------------------------------------------------------

1. Create a Fine-Grained Token
Go to: Settings → Developer settings → Fine-grained tokens
Permissions required:
- Actions → Read & Write
- Workflows → Read & Write

2. Save the token in repository secrets:
ACTION_TRIGGER_TOKEN

3. Ensure workflow name matches:
.github/workflows/CI.yml


--------------------------------------------------------------
Workflow File (Reference)
--------------------------------------------------------------

name: CI

on: [push, workflow_dispatch]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Download
      run: Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip -OutFile ngrok.zip

    - name: Extract
      run: Expand-Archive ngrok.zip

    - name: Auth
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}

    - name: Enable TS
      run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0

    - run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

    - run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1

    - run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)

    - name: Create Tunnel
      run: .\ngrok\ngrok.exe tcp 3389


--------------------------------------------------------------
Optional: Real HTML Buttons for GitHub (Commented Out)
--------------------------------------------------------------

<!--
<h2>Start RDP</h2>
<button onclick="startRDP()" style="padding:10px 20px;font-size:16px;">Start</button>

<h2>Stop RDP</h2>
<button onclick="stopRDP()" style="padding:10px 20px;font-size:16px;">Stop</button>

<script>
async function startRDP() {
  await fetch('https://api.github.com/repos/trex-coder/rdp/actions/workflows/CI.yml/dispatches', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer YOUR_TOKEN_HERE',
      'Accept': 'application/vnd.github+json'
    },
    body: JSON.stringify({ ref: "main" })
  });
  alert("RDP workflow started!");
}

async function stopRDP() {
  await fetch('https://api.github.com/repos/trex-coder/rdp/actions/runs/active/cancel', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer YOUR_TOKEN_HERE',
      'Accept': 'application/vnd.github+json'
    }
  });
  alert("RDP workflow stopped!");
}
</script>
-->
