# SNMP-for-LibreNMS

That's great\! Since you've successfully completed the service configuration, here is a concise, step-by-step guide summarizing all the commands needed to set up and start the **SNMP Agent (Server)** on your Windows machine via SSH/PowerShell.

This guide assumes you are already logged into the Windows machine via SSH:

-----

## 💻 SNMP Agent Setup Guide (Windows)

### 1\. 📂 Install the SNMP Feature

This command installs the necessary files for both the SNMP Agent and the Trap Client.

```powershell
Add-WindowsCapability -Online -Name "SNMP.Client~~~~0.0.1.0"
```

-----

### 2\. ⚙️ Set Services to Autostart and Start Now

These commands ensure both the SNMP Agent (`snmp`) and the Trap Client (`snmptrap`) start automatically and are running immediately.

```powershell
# Set both services to start automatically
Set-Service -Name snmp -StartupType Automatic
Set-Service -Name snmptrap -StartupType Automatic

# Start both services immediately
Start-Service snmp
Start-Service snmptrap
```

-----

### 3\. 🔑 Configure Community String (Security)

This sets the **`public`** community string with **Read-Only** access, allowing managers to query the machine.

```powershell
$CommunityPath = "HKLM:\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities"
New-ItemProperty -Path $CommunityPath -Name "public" -Value 4 -PropertyType DWORD -Force
```

-----

### 4\. 🔒 Allow Specific Manager IP

This restricts SNMP access to only the specified manager IP address, **`172.99.0.80`**, improving security.

```powershell
$ManagersPath = "HKLM:\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\PermittedManagers"
New-ItemProperty -Path $ManagersPath -Name "4" -Value "172.99.0.80" -PropertyType String -Force
```

-----

### 5\. 🔄 Restart the Service

The service must be restarted for the registry changes (Community String and Allowed Managers) to take effect.

```powershell
Restart-Service snmp
```

-----

### 6\. 🔥 Configure Windows Firewall

This is the most critical networking step: it creates a firewall rule to allow inbound SNMP traffic (UDP port 161).

```powershell
New-NetFirewallRule -DisplayName "SNMP UDP In" -Direction Inbound -Protocol UDP -LocalPort 161 -Action Allow
```

After completing these six steps, your Windows machine will be fully configured to respond to SNMP queries from your manager machine at `172.99.0.80` using the `public` community string.
