# NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP
# 🛡️ Mediroza General Hospital — Penetration Testing Project

**Batch:** B082
**Week:** 4
**Project Model:** PM1
**Target:** `https://medirozahospital.com`
**Status:** Confidential — Authorized Lab Environment

---

## 📌 Project Overview

This project involved performing a structured penetration test against the **Mediroza General Hospital** lab environment.

The assessment followed four milestones covering:

* 🔎 Initial reconnaissance and access
* 🔐 PDF password and encryption testing
* 🕵️ Deep reconnaissance and information discovery
* 📋 Vulnerability analysis and remediation recommendations

The assessment demonstrated how multiple security weaknesses can be connected together, starting from reconnaissance and eventually leading to the discovery of exposed backup data and sensitive information.

---

# 🔎 Milestone 1 — Initial Access

## Step 1 — Reconnaissance

I started by checking the target's `robots.txt` file to identify directories that were not intended to be indexed.

```bash
curl https://medirozahospital.com/robots.txt
```

The file revealed:

```text
Disallow: /patient/
Disallow: /staff/
Disallow: /old/
```

The `/patient/` directory was used for the initial access phase, while `/old/` became relevant during Milestone 3.

### 📸 Screenshot

![Milestone 1 Step 1](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone1-step1-robotstxt.png)

---

## Step 2 — Find the Login Page

The `/patient/` directory revealed a patient login page:

```text
https://medirozahospital.com/patient/login.php
```

### 📸 Screenshot

![Milestone 1 Step 2](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone1-step2-usernamenotfound.png)

---

## Step 3 — Username Enumeration

I tested an invalid username and then tested `admin`.

The application returned different messages:

```text
Username not found
```

and:

```text
Incorrect password
```

This confirmed that the application could reveal whether a username existed.

### 📸 Screenshot

![Milestone 1 Step 3](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone1-step3-incorrectpassword.png)

---

## Step 4 — SQL Injection Testing

I tested the username field with a single quote:

```text
admin'
```

The application returned a MySQL syntax error, indicating that user input was being incorporated into the database query.

### 📸 Screenshot

![Milestone 1 Step 4](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone1-step4-sqlsyntax.png)

---

## Step 5 — Login Bypass

Using the lab-provided SQL injection test, the login was bypassed and the patient portal became accessible.

The portal contained three PDF reports:

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
```

### 📸 Screenshot

![Milestone 1 Step 5](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone1-step5-labreports.png)

---

# 🔐 Milestone 2 — Crack the Encryption

## Step 1 — Extract PDF Hashes

I uploaded each encrypted PDF to the Networkwalks Hash Calculator and extracted the corresponding PDF hashes.

The extracted hashes started with:

```text
$pdf$
```

### 📸 Screenshot

![Milestone 2 Step 1](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step1-usedhashcalculator.png)

---

## Step 2 — Crack Report 1

Using the provided password-cracking tool and built-in wordlist:

```text
patient_report_1.pdf → 123456
```

### 📸 Screenshot

![Milestone 2 Step 2](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step2-crackedlabreport1.png)

---

## Step 3 — Crack Report 2

The second report was successfully cracked using the built-in wordlist:

```text
patient_report_2.pdf → password
```

### 📸 Screenshot

![Milestone 2 Step 3](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step3-crackedlabreport2.png)

---

## Step 4 — Crack Report 3

The built-in wordlist did not find a match:

```text
Exhausted wordlist. No match. ACCESS DENIED.
```

I then used the instructor-provided JTR wordlist, which successfully identified the password.

```text
patient_report_3.pdf → !@#$%^&
```

### 📸 Screenshot

![Milestone 2 Step 4](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step4-crackedlabreport3.png)

---

## Step 5 — Decrypt Report 3

I used `qpdf` to create an unlocked copy of the third report:

```bash
qpdf --password='!@#$%^&' --decrypt patient_report_3.pdf report3_open.pdf
```

### 📸 Screenshot

![Milestone 2 Step 5](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step5-bashedqpdf.png)

---

## Step 6 — Verify the Unlocked File

The decrypted PDF was successfully created and opened in the Linux environment.

### 📸 Screenshot

![Milestone 2 Step 6](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone2-step6-openedfileonlinux.png)

---

# 🕵️ Milestone 3 — Deep Reconnaissance

## Step 1 — Analyze PDF Metadata

I used `exiftool` to inspect the metadata of the unlocked PDF:

```bash
exiftool report3_open.pdf
```

Important fields included:

```text
Author   : j.malik
Comments : DB backup moved to /old before site migration, do not delete
```

This provided a clue pointing toward the `/old/` directory.

### 📸 Screenshot

![Milestone 3 Step 1](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone3-step1-foundthemetadata-exiftool.png)

---

## Step 2 — Discover the Database Backup

The `/old/` directory had directory listing enabled and exposed a database backup:

```text
mediroza_db_backup_2019.sql
```

The backup contained staff and shareholder information. The staff records also contained **Jameel Malik**, whose identifier matched the `j.malik` author information found in the PDF metadata.

This connected the metadata clue to the exposed database backup.

### 📸 Screenshot

![Milestone 3 Step 2](https://github.com/malakkashraf/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-INTERNSHIP/blob/main/milestone3-step2-gotsqldata.png)

---

# 📋 Milestone 4 — Penetration Testing Report

## Findings Summary

| # | Finding                                      | Location                          | Risk     |
| - | -------------------------------------------- | --------------------------------- | -------- |
| 1 | Username enumeration                         | `patient/login.php`               | Medium   |
| 2 | SQL injection login bypass                   | `patient/login.php`               | Critical |
| 3 | Encrypted PDFs accessible after login bypass | `patient/reports/`                | High     |
| 4 | Weak PDF passwords                           | `patient_report_*.pdf`            | High     |
| 5 | Sensitive PDF metadata                       | `patient_report_3.pdf`            | Medium   |
| 6 | Exposed backup directory                     | `old/`                            | Critical |
| 7 | Confidential database information exposed    | `old/mediroza_db_backup_2019.sql` | Critical |

---

## 🔧 Recommendations & Remediation

### 1. Username Enumeration

Use the same generic error message for invalid usernames and passwords so that valid accounts cannot be identified.

### 2. SQL Injection

Use **parameterized queries / prepared statements** and never place raw user input directly into SQL queries.

### 3. PDF Access Control

Store sensitive PDFs outside the public web root or protect them using proper authorization controls.

### 4. PDF Password Security

Use strong, unique passwords instead of common passwords that can be discovered using wordlists.

### 5. PDF Metadata

Remove unnecessary metadata before distributing sensitive documents.

Example:

```bash
exiftool -all= filename.pdf
```

### 6. Backup Directory Exposure

Disable directory listing and remove old backups from publicly accessible web directories.

### 7. Database Backup Protection

Database backups should never be stored inside a publicly accessible web directory. They should be securely stored outside the web root with appropriate access controls.

---

# 🔗 Attack Chain

The assessment demonstrated the following vulnerability chain:

```text
Reconnaissance
      ↓
robots.txt discovery
      ↓
Patient login page
      ↓
Username enumeration
      ↓
SQL injection
      ↓
Login bypass
      ↓
PDF reports discovered
      ↓
Weak PDF passwords
      ↓
PDF decryption
      ↓
Metadata discovery
      ↓
/old/ backup directory
      ↓
Database backup exposure
      ↓
Sensitive information discovered
```

---

# 🎯 Key Lessons Learned

Through this project, I gained practical experience with:

* 🔎 Reconnaissance and information gathering
* 🌐 `robots.txt` analysis
* 🔐 Username enumeration
* 💉 SQL injection identification
* 🔑 Authentication bypass testing
* 🔓 PDF hash extraction and password testing
* 🛠️ PDF decryption using `qpdf`
* 🕵️ Metadata analysis using `exiftool`
* 📂 Directory listing misconfiguration
* 🗄️ Database backup exposure
* 📋 Vulnerability assessment and remediation

---

# ✅ Conclusion

The Mediroza General Hospital assessment demonstrated how multiple seemingly separate weaknesses can be connected into a complete attack chain.

The project reinforced the importance of secure authentication, parameterized database queries, strong password policies, proper file access controls, metadata sanitization, and secure storage of database backups.

> ⚠️ **Note:** Sensitive personal and confidential database information has intentionally not been included in this public GitHub report.
