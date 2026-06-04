# Exchange Server Administrative Toolkit

A centralized, interactive PowerShell toolkit designed for Exchange Server administrators to perform routine maintenance, distribution group updates, and rapid incident response (phishing cleanup) through a guide-based console application.

---

## 🚀 Features

The toolkit consolidates six essential administration tasks into a single interactive menu:
1. **Message Tracking:** Deep-dive search by Sender and Recipient with automated CSV exports.
2. **Export Distribution Group Members:** Fast extraction of membership lists to CSV.
3. **Bulk ADD Members to DG:** Streamlined batch provisioning of users from a unified CSV structure.
4. **Bulk REMOVE Members from DG:** Safe batch de-provisioning of users from a unified CSV structure.
5. **Phishing Cleanup (Delete by Subject):** Multi-mailbox evaluation and hard/soft deletion based on Subject lines with audit logs.
6. **Phishing Cleanup (Delete by Sender):** Multi-mailbox evaluation and hard/soft deletion based on Sender address with audit logs.

---

## 📊 Global CSV Prerequisites

To eliminate configuration complexity, **all CSV-driven options (Options 3, 4, 5, and 6) utilize the exact same input file structure.** Your input CSV file must contain a single column with the header strictly named **`Email`**.

### Target Input Format Example:
| Email |
| :--- |
| john.doe@company.com |
| jane.smith@company.com |
| external-user@domain.com |

> ⚠️ **Note:** Do not use headers like `PrimarySmtpAddress`, `UserPrincipalName`, or `Identity`. The toolkit explicitly looks for the **`Email`** column.

---

## 🛠️ Detailed Operational Guide

### Option 1: Message Tracking (Search by Sender/Recipient)
* **Purpose:** Locates and audits mail flow between specific entities across the organization.
* **Inputs Required:** * Sender Email Address (Wildcards accepted depending on your environment).
  * Recipient Email Address.
* **Output:** Generates a structured log at `C:\Temp\TrackingLog_[Timestamp].csv` containing Timestamp, EventId, Source, Sender, Recipients (joined by semicolons), and MessageSubject.

### Option 2: Export Distribution Group Members to CSV
* **Purpose:** Generates a clean baseline list of any Distribution Group’s current membership.
* **Inputs Required:** Distribution Group Identity (Alias, Name, or SMTP Address).
* **Output:** Exports all member SMTP addresses to `C:\Temp\Members_[Group_Identity].csv`.

### Option 3: Bulk ADD Members to Distribution Group
* **Purpose:** Automates the enrollment of hundreds of users into a targeted Distribution Group simultaneously.
* **Inputs Required:** * Target Group Identity.
  * Full file path to your input CSV (e.g., `C:\Scripts\users.csv`) containing the **`Email`** header.
* **Behavior:** Automatically skips existing members gracefully (`-ErrorAction SilentlyContinue`) while providing real-time CLI console processing indicators.

### Option 4: Bulk REMOVE Members from Distribution Group
* **Purpose:** Safely strips group membership in bulk during off-boarding or restructuring cycles.
* **Inputs Required:** * Target Group Identity.
  * Full file path to your input CSV containing the **`Email`** header.
* **Behavior:** Processes removals with internal confirmations suppressed (`-Confirm:$false`) for true automated hands-off execution.

### Option 5: Search and Delete Mail by SUBJECT (Phishing Cleanup)
* **Purpose:** Targeted remediation of widespread phishing campaigns using specific email subjects.
* **Inputs Required:** * Full path to the input CSV containing target mailboxes to scan (Header: **`Email`**).
  * Exact subject query string.
  * Admin/Discovery Mailbox Identity to receive the mandatory Exchange search logs.
* **Execution Flow:** 1. **Phase 1 (LogOnly):** Scans the target list, logs details to the specified Admin Mailbox folder (`PhishingSubjectLogs`), and displays the total match footprint.
  2. **Confirmation Boundary:** Prompts the admin to type **`OK`** to execute destruction.
  3. **Phase 2 (DeleteContent):** Hard-deletes matching items across all verified targets and compiles an automated master report at `C:\Temp\Deletion_Subject_Report_[Timestamp].csv`.

### Option 6: Search and Delete Mail by SENDER (Phishing Cleanup)
* **Purpose:** Rapid incident response blocking and content eradication when an external or internal compromised account blasts malicious emails.
* **Inputs Required:** * Full path to the input CSV containing target mailboxes to scan (Header: **`Email`**).
  * Threat Actor's Sender Address.
  * Admin/Discovery Mailbox Identity for audit logs.
* **Execution Flow:** Mirroring Option 5, this triggers a **LogOnly** verification scan, awaits a literal **`OK`** confirmation bypass, and performs automated content deletion, exporting a completion status summary report to `C:\Temp\Deletion_Sender_Report_[Timestamp].csv`.

---

## 🔒 Requirements & RBAC Permissions

To execute this toolkit successfully, ensure your administrative terminal meets the following specifications:
* **Directory Path:** The script will automatically attempt to initialize `C:\Temp` if it does not exist. Ensure your local account has write permissions to the C drive root.
* **Exchange Management Shell:** Must be executed directly from an Exchange Management Shell (EMS) session or via a remote implicit PSSession with Exchange commandlets imported.
* **RBAC Roles Needed (Critical for Options 5 & 6):**
  * **`Mailbox Search`** role (Typically belongs to the *Discovery Management* role group).
  * **`Mail Import Export`** role (Required to process absolute content purging using `Search-Mailbox -DeleteContent`).

---

## 🚨 Disclaimer
> **CRITICAL WARNING:** Options 5 and 6 use destructive parameters (`-DeleteContent`). This completely purges matching emails from user mailboxes. Always carefully verify your search terms during Phase 1 (LogOnly) before confirming execution with `OK`. Use this toolkit at your own discretion.
