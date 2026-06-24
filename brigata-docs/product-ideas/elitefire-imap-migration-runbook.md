---
brigata_id: c2bf08d6-ba9c-4e2c-aea7-09f687958a38
title: "EliteFire_IMAP_Migration_Runbook"
---

# Elite Fire Services - Native IMAP Migration Test Runbook

## Overview

This runbook documents the process for testing Microsoft's native IMAP migration from **Turbify (Yahoo Business)** to **Guardian's Microsoft 365 tenant**. We're using the Exchange Admin Center migration wizard instead of MigrationWiz due to authentication compatibility concerns with Yahoo Business app passwords.

---

## Pre-Migration Requirements

### 1. Guardian M365 Tenant (Destination)
- [x] User accounts created and licensed (Exchange Online)
- [x] Mailboxes provisioned for test users
- [x] Admin account with Exchange Administrator role (or Global Admin)

### 2. Turbify/Yahoo Business (Source)
- [ ] Confirm IMAP access is enabled for accounts
- [ ] Generate **app passwords** for each user (required - standard passwords won't work)
- [ ] Document primary email addresses (not aliases)

---

## Turbify IMAP Server Settings

| Setting | Value |
|---------|-------|
| **IMAP Server** | `imap.mail.yahoo.com` |
| **Port** | `993` |
| **Encryption** | SSL/TLS |
| **Authentication** | Basic (app password required) |

> **Important:** Yahoo Business does NOT support admin/super-user credentials for IMAP. Each user must have their own app password.

---

## Phase 1: Generate App Passwords (Per User)

Each Elite Fire user must generate an app password. This is a **blocking prerequisite**.

### Steps for Each User:

1. Sign in to your business webmail at `https://mail.turbify.com` using business credentials
2. Click profile icon → **Account Info**
3. Navigate to **Account Security & Privacy**
4. Click **Manage app passwords**
5. Select **Other** app type and enter custom name (e.g., "Migration")
6. Copy the 16-character password generated
8. Email the password to chris.hager@ferox-consulting.com

### For Test Migration:
- Select **1-2 pilot users** willing to generate app passwords
- Collect: Full email address + app password

---

## Phase 2: Prepare the CSV Migration File

### CSV Format (Required Columns)

```csv
EmailAddress,UserName,Password
user@guardiandomain.com,user@elitefire.com,xxxx-xxxx-xxxx-xxxx
```

| Column | Description |
|--------|-------------|
| `EmailAddress` | Destination M365 email address |
| `UserName` | Source Turbify/Yahoo email address (full address) |
| `Password` | App password (16-char, no spaces) |

### Example for Test Migration:

```csv
EmailAddress,UserName,Password
jsmith@guardianfire.com,jsmith@elitefire.com,abcd-efgh-ijkl-mnop
```

### CSV Requirements:
- No spaces in column headers
- Save as `.csv` (UTF-8 encoding)
- Maximum 50,000 rows per batch
- One mailbox per row

---

## Phase 3: Create Migration Endpoint

1. Sign in to **Exchange Admin Center**: `https://admin.exchange.microsoft.com`
2. Navigate to **Migration** (under Recipients or Data migration)
3. Click **Add migration batch** → Select **Migrate to Exchange Online**
4. Choose **IMAP migration**
5. On the endpoint screen, click **Create new endpoint** or select existing
6. Configure endpoint:

| Field | Value |
|-------|-------|
| **Endpoint Name** | `Turbify-EliteFire` |
| **IMAP Server** | `imap.mail.yahoo.com` |
| **Port** | `993` |
| **Security** | SSL |
| **Authentication** | Basic |
| **Max concurrent migrations** | `10` (start conservative) |
| **Max concurrent incremental syncs** | `10` |

7. Click **Save**

---

## Phase 4: Create and Run Test Migration Batch

### Create Batch:

1. From Migration dashboard, click **Add migration batch**
2. Select **Migrate to Exchange Online** → **IMAP migration**
3. Name the batch: `EliteFire-Test-Batch1`
4. Select the `Turbify-EliteFire` endpoint
5. Upload the CSV file
6. Review validation results - fix any errors
7. Configure options:
   - **Automatically start batch**: No (manual start for testing)
   - **Automatically complete batch**: No
   - **Notification email**: Your email address

### Folder Exclusions (Optional):
If Yahoo creates duplicate folders, you may want to exclude:
- `[Yahoo]`
- `Bulk Mail` (if you want to skip spam)

### Start Batch:

1. Select the batch from the Migration dashboard
2. Click **Start** (play button)
3. Monitor status - initial sync may take time depending on mailbox size

---

## Phase 5: Monitor and Validate

### Status Codes:
| Status | Meaning |
|--------|---------|
| **Syncing** | Initial migration in progress |
| **Synced** | Initial sync complete, incremental sync active |
| **Completed** | Migration finished, batch manually completed |
| **Failed** | Check error report |

### Validation Steps:

1. **Check folder structure** - Verify folders migrated correctly
2. **Spot-check emails** - Compare recent emails between source and destination
3. **Check sent items** - Verify sent mail migrated
4. **Check date range** - Confirm oldest emails present
5. **Note any missing items** - IMAP only migrates mail (no contacts/calendar)

### Common Issues:

| Issue | Likely Cause | Resolution |
|-------|--------------|------------|
| `Incorrect username or password` | Wrong app password or using regular password | Regenerate app password |
| `Connection failed` | IMAP port blocked or wrong server | Verify `imap.mail.yahoo.com:993` |
| `Mailbox not found` | Email address mismatch | Use primary address, not alias |
| Duplicate folders | Yahoo folder structure quirk | Add `[Yahoo]` to excluded folders |

---

## Phase 6: Post-Test Decision

### If Test Succeeds:
- [ ] Document any adjustments needed
- [ ] Plan full migration in batches (10-20 users per batch recommended)
- [ ] Coordinate app password generation with all Elite users
- [ ] Schedule cutover date

### If Test Fails:
- [ ] Review error logs
- [ ] Consider MigrationWiz fallback
- [ ] Consider PST export/import as alternative

---

## IMAP Migration Limitations

**What Migrates:**
- Email messages
- Folder structure

**What Does NOT Migrate:**
- Contacts
- Calendar events
- Tasks
- Rules/Filters
- Signatures

### Workarounds for Non-Email Data:
- **Contacts**: Export from Yahoo as CSV → Import to Outlook/M365
- **Calendar**: Export as ICS → Import to M365 calendar
- **Rules**: Recreate manually in Outlook/OWA

---

## Timeline Considerations

| Phase | Duration |
|-------|----------|
| App password generation (per user) | 5-10 min |
| CSV preparation | 30 min |
| Endpoint setup | 15 min |
| Test batch creation | 15 min |
| Initial sync (depends on mailbox size) | Hours to days |
| Validation | 30-60 min |

---

## Contact Information

**Turbify Support:** 1-833-689-8585  
**Microsoft IMAP Migration Docs:** https://learn.microsoft.com/en-us/exchange/mailbox-migration/migrating-imap-mailboxes/migrating-imap-mailboxes

---

## Appendix: PowerShell Alternative

If GUI has issues, you can create the endpoint and batch via PowerShell:

```powershell
# Connect to Exchange Online
Connect-ExchangeOnline

# Create migration endpoint
New-MigrationEndpoint -IMAP `
    -Name "Turbify-EliteFire" `
    -RemoteServer "imap.mail.yahoo.com" `
    -Port 993 `
    -Security Ssl

# Create migration batch from CSV
New-MigrationBatch -Name "EliteFire-Test-Batch1" `
    -SourceEndpoint "Turbify-EliteFire" `
    -CSVData ([System.IO.File]::ReadAllBytes("C:\Migration\EliteFire_test.csv"))

# Start the batch
Start-MigrationBatch -Identity "EliteFire-Test-Batch1"

# Check status
Get-MigrationBatch -Identity "EliteFire-Test-Batch1" | Format-List
Get-MigrationUser -BatchId "EliteFire-Test-Batch1" | Format-Table
```

---

*Last Updated: January 2025*
