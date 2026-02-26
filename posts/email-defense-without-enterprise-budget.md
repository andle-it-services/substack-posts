# Email Defense Without Enterprise Budget

Practical email filtering strategies for small organizations using keyword rules and sender whitelists to block phishing attempts.

---

## Why Small Orgs Get Phished

### The Problem

Enterprise companies have expensive email security gateways, dedicated SOC teams, and security awareness training budgets. Small organizations—dental offices, local government, small law firms—have none of that. They're running Microsoft 365 or Google Workspace with default settings, and attackers know it.

### The Attack Surface

Most phishing attempts targeting small organizations follow predictable patterns:

- **Impersonation:** Fake emails from "your bank" or "Microsoft Security"
- **Invoice Fraud:** Fake invoices from unknown vendors
- **Credential Harvesting:** Links to fake login pages
- **BEC (Business Email Compromise):** "Urgent" requests from spoofed executives

### Why Default Filters Fail

Built-in spam filters catch obvious garbage, but sophisticated phishing slips through because:

- The sender's domain is legitimate (compromised accounts)
- The email contains no malware (just a link)
- The message is well-written (no "Dear Valued Customer")

You need a strategy that doesn't rely solely on the email provider's AI catching everything.

---

## The Whitelist Strategy

### Flip the Model

Instead of trying to block every bad sender (impossible), flip it: only allow known-good senders to reach the inbox.

This is the whitelist approach. Everything not on your approved list goes to a quarantine folder for manual review.

### Building Your Whitelist

Start by identifying who legitimately emails your organization:

- **Clients/Patients:** If you're a dental office, your patient base sends appointment confirmations, questions, etc.
- **Vendors:** Your lab, supply companies, software providers
- **Internal:** Staff email addresses, shared mailboxes
- **Services:** Bank notifications, insurance portals, government agencies

### Implementation (Google Workspace)

In Google Admin Console:

1. Go to Apps → Google Workspace → Gmail → Spam, phishing, and malware
2. Under Approved senders, add domains and addresses you trust
3. Create a routing rule that quarantines email from non-approved senders

### Implementation (Microsoft 365)

In Exchange Admin Center:

1. Go to Mail flow → Rules
2. Create a rule: "If sender is NOT in [approved list], move to quarantine"
3. Add your approved senders/domains to the exception list

### The Catch

Pure whitelisting is too restrictive for most orgs — you'll miss legitimate first-contact emails. That's where keywords come in.

---

## Keyword Filtering

### Catching What Whitelists Miss

Keywords let you flag or quarantine emails containing suspicious phrases — even from trusted senders whose accounts might be compromised.

### High-Risk Keywords to Filter

These phrases appear in most phishing attempts:

- **Urgency triggers:** "immediate action required", "your account will be suspended", "within 24 hours"
- **Financial bait:** "wire transfer", "update payment method", "invoice attached", "overdue payment"
- **Credential harvesting:** "verify your identity", "confirm your password", "click here to login"
- **Authority impersonation:** "IRS", "Social Security Administration", "Microsoft Security Team"

### Implementation Tips

- Don't block — quarantine. Let a human review flagged messages
- Use regex where supported to catch variations ("w1re transfer", "pa$$word")
- Review quarantine weekly and tune false positives
- Layer with whitelisting: trusted senders with flagged keywords still get reviewed

### Sample Rule (Google Workspace)

In Gmail routing rules, create a content compliance rule:

- **Match:** Body contains "verify your account" OR "click here to confirm"
- **Action:** Quarantine message and notify admin

---

## Combining Whitelist + Keywords

### The Layered Approach

Neither strategy works perfectly alone. Together, they cover each other's weaknesses.

```
IF sender NOT IN whitelist:
    → Quarantine (unknown sender)
ELSE IF body CONTAINS suspicious_keywords:
    → Quarantine (compromised account risk)
ELSE:
    → Deliver to inbox
```

**Why this works:**

- Unknown senders can't reach users at all — they have to go through a human gatekeeper
- Compromised trusted accounts get flagged — keywords catch the payload
- Legitimate email flows normally — no friction for day-to-day business
- Phishing success rate drops dramatically — attackers need to compromise a known sender AND avoid all flagged phrases

---

## Quick-Start Checklist

### Week 1: Build Your Whitelist

- ☐ Export 30 days of received email
- ☐ Extract unique sender domains
- ☐ Review and approve legitimate business contacts
- ☐ Add internal domains (@yourcompany.com)
- ☐ Add key vendors and services

### Week 2: Set Up Quarantine

- ☐ Create a shared "Quarantine Review" mailbox
- ☐ Configure routing rule: non-whitelisted → quarantine
- ☐ Assign someone to check quarantine 2x daily
- ☐ Document the "approve sender" process

### Week 3: Add Keyword Filters

- ☐ Create content compliance rules for high-risk phrases
- ☐ Start with 5-10 obvious phishing keywords
- ☐ Route keyword matches to quarantine (not delete!)
- ☐ Monitor false positive rate

### Ongoing Maintenance

- ☐ **Weekly:** Review quarantine, add new legitimate senders
- ☐ **Monthly:** Tune keyword list based on false positives
- ☐ **Quarterly:** Audit whitelist for stale/unused entries
- ☐ **Annually:** Full review of email security posture

---

## Cost

$0. Both Google Workspace and Microsoft 365 include these capabilities in their base plans. You're just not using them.
