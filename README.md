# 🖥️ Generate a Domain Computer Status Report Using PowerShell

## 📘 Overview
In Active Directory environments, administrators often need to **monitor all domain computers** — checking which ones are **online or offline**, and identifying the **currently logged-in user**.  

This guide shows you how to accomplish all of this using a single **PowerShell script** that:

- 🔍 Lists all computers in the Active Directory domain  
- 🌐 Tests whether each computer is online or offline  
- 👤 Retrieves the currently logged-in username (if available)  
- 📊 Exports all results to a clean **CSV report**  

The output can be easily opened in **Excel**, **Power BI**, or any text editor for quick auditing and reporting.

---

## 🧰 Prerequisites

| Requirement | Description |
|--------------|-------------|
| 🪟 **Windows Version** | Windows Server / Windows 10 / Windows 11 with RSAT or AD PowerShell Module installed |
| ⚙️ **Permissions** | Domain Administrator or sufficient AD read access |
| 🧩 **Module Needed** | Active Directory PowerShell Module |
| 💾 **Output Location** | Default export path: `C:\ComputerListStatus.csv` |

---

## ⚙️ Step 1 — Run PowerShell as Administrator

1. Press **`Win + X`** → choose **Windows PowerShell (Admin)** or **Terminal (Admin)**.  
2. Ensure you have access to your domain environment.  
3. Confirm the **Active Directory module** is available:
   ```
   Import-Module ActiveDirectory
   ```
 

If you receive an error, install RSAT:

```powershell
Add-WindowsCapability -Online -Name "Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"
```

---

## 🧾 Step 2 — PowerShell Script

Copy and paste the following script into your PowerShell console or save it as a `.ps1` file:

``` 
Import-Module ActiveDirectory

$computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

$results = @()

foreach ($computer in $computers) { 
    if (Test-Connection -ComputerName $computer -Count 2 -Quiet) { 
        try { 
            $user = Invoke-Command -ComputerName $computer -ScriptBlock { 
                (Get-WMIObject -Class Win32_ComputerSystem).UserName 
            } -ErrorAction Stop 
        } catch { 
            $user = "Access Denied" 
        } 
        $status = "Online" 
    } else { 
        $user = "N/A" 
        $status = "Offline" 
    } 

    $results += [PSCustomObject]@{ 
        ComputerName = $computer 
        Status = $status 
        LoggedInUser = $user 
    }
}

$results | Export-Csv -Path "C:\ComputerListStatus.csv" -NoTypeInformation

Write-Host "Results saved to C:\ComputerListStatus.csv" -ForegroundColor Cyan
```

---

## 🧠 Step 3 — How the Script Works

| Function                        | Explanation                                             |
| ------------------------------- | ------------------------------------------------------- |
| `Import-Module ActiveDirectory` | Loads AD cmdlets for PowerShell                         |
| `Get-ADComputer -Filter *`      | Fetches all computer accounts in the domain             |
| `Test-Connection`               | Pings each computer to check network availability       |
| `Invoke-Command`                | Queries remote systems for the currently logged-in user |
| `Export-Csv`                    | Saves results into a CSV file for reporting             |

---

## 🧩 Step 4 — Output Example

After execution, PowerShell will save a file named:

```
C:\ComputerListStatus.csv
```

🧾 **Sample CSV Output:**

| ComputerName | Status  | LoggedInUser    |
| ------------ | ------- | --------------- |
| DC01         | Online  | DOMAIN\Admin    |
| PC-001       | Offline | N/A             |
| PC-002       | Online  | DOMAIN\John.Doe |
| LAPTOP-05    | Online  | Access Denied   |

---

## ⚙️ Step 5 — Customizing the Script

* **Change Export Path:**
  Modify the `Export-Csv` path to any directory you prefer:

  ```powershell
  $results | Export-Csv -Path "D:\Reports\ComputerStatus.csv" -NoTypeInformation
  ```

* **Filter by Organizational Unit (OU):**
  To target only specific OUs:

  ```powershell
  Get-ADComputer -SearchBase "OU=Workstations,DC=YourDomain,DC=com" -Filter *
  ```

* **Add Timestamp:**
  Include execution date/time in the CSV file name:

  ```powershell
  $date = Get-Date -Format "yyyyMMdd_HHmmss"
  $results | Export-Csv -Path "C:\ComputerListStatus_$date.csv" -NoTypeInformation
  ```

---

## 🔐 Notes & Best Practices

* 🧾 Run this script from a **domain-joined system** with AD PowerShell module installed.
* 💡 “Access Denied” indicates the account running the script lacks admin rights on that machine.
* 🔁 For recurring monitoring, you can **schedule** this script via **Task Scheduler** or **Group Policy logon script**.
* 🧱 This script uses **WMI** and **WinRM** — ensure remote management is enabled on target devices.

---

## ✅ Summary

| Step | Action                         | Description                             |
| ---- | ------------------------------ | --------------------------------------- |
| 1️⃣  | Import Active Directory Module | Enables PowerShell to access AD objects |
| 2️⃣  | Run the Script                 | Scans all domain computers              |
| 3️⃣  | Test Connectivity              | Checks if each computer is online       |
| 4️⃣  | Get Logged-in User             | Retrieves username via WMI              |
| 5️⃣  | Export Results                 | Creates CSV report in `C:\`             |

---

## 🧑‍💻 Document Metadata

| Field            | Detail                                                            |
| ---------------- | ----------------------------------------------------------------- |
| **Author**       | Xitiz Basnet                                                      |
| **Category**     | Active Directory / PowerShell                                     |
| **Tags**         | PowerShell, Active Directory, CSV, Network Monitoring, Automation |

---

> 🏁 **End of Document**
> *Easily monitor your Active Directory environment, track online/offline devices, and export user session data using this automated PowerShell script.*

---

