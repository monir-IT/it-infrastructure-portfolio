# Microsoft 365, Email Domain and Account Recovery

Set up the company's Microsoft 365 tenant and custom email domain, handled two real incidents (a mail outage and a compromised admin account on the previous tenant), then audited and hardened the current tenant using what they taught me.

| | |
| --- | --- |
| **Role** | Sole IT administrator: tenant setup, domain and DNS, user management, incident handling |
| **Platform** | Microsoft 365 Business Standard, domain registered with GoDaddy |
| **Status** | In service; audit complete, hardening in progress (see [Hardening applied](#hardening-applied)) |

## Setup

1. Subscribed to Microsoft 365 Business Standard.
2. Registered the company domain with GoDaddy and verified it in Microsoft 365.
3. Pointed the domain's DNS records at Microsoft 365 so company email works on the custom domain.
4. Created user accounts and mailboxes for staff.

## Incident 1: no email from outside the company

| | |
| --- | --- |
| **Symptom** | Staff could not receive email from outside the company |
| **First check** | The MX record, which tells other mail servers where to deliver email for the domain |
| **Root cause** | The domain registration had expired, so the domain's DNS records stopped working and other servers had nowhere to deliver |
| **Fix** | Renewed the domain; mail flow returned |
| **Lesson** | A domain is critical infrastructure. Auto-renew, and a reminder that does not depend on one person's inbox |

## Incident 2: compromised global admin account (previous tenant)

The most serious incident I have handled. It happened on the company's **previous** Microsoft 365 tenant, which is no longer in use.

| Stage | What happened |
| --- | --- |
| **Compromise** | An attacker gained access to the global administrator account and deleted all of the email |
| **Escalation** | I opened a case with Microsoft support. It repeatedly stalled, so I kept following up and escalating for about a month |
| **Recovery** | Microsoft returned control of the global admin account, and working with their support we recovered the deleted email |

**What it taught me:** MFA on every account matters most; daily work should never happen in a global admin account; you need a way back in that doesn't depend on the account you lost; and recovery through a vendor can take weeks.

## Security audit and hardening of the current tenant

In October 2026 I audited the current tenant (29 users) and applied those lessons. Every step was evidenced with before and after screenshots, kept privately.

### Audit

| Check | How | Finding | Action |
| --- | --- | --- | --- |
| Admin roles | Role assignments | One global admin, also used as a daily account | Fixed (below) |
| Guest accounts | Entra users, creation date, sign-in logs | One guest: a known contact, never signed in | Kept, scheduled for review |
| Third-party apps | Entra enterprise applications, permissions | One app with basic sign-in permissions and anyone in the tenant allowed to use it, no longer needed | Deleted |
| Mail forwarding | PowerShell across all mailboxes (below) | No forwarding, no forwarding inbox rules | Clean |
| Legacy sign-in methods | Sign-in logs filtered to legacy clients | None in the last 7 days | Safe to keep blocked |
| MFA enforcement | Security Defaults, per-user MFA | Security Defaults on; per-user MFA off everywhere (no conflict) | Registration follow-up |

```powershell
Connect-ExchangeOnline

# Mailboxes forwarding mail anywhere
Get-Mailbox -ResultSize Unlimited |
  Where-Object { $_.ForwardingSmtpAddress -or $_.ForwardingAddress } |
  Select-Object DisplayName, ForwardingSmtpAddress, ForwardingAddress

# Inbox rules that forward or redirect mail
Get-Mailbox -ResultSize Unlimited | ForEach-Object {
  Get-InboxRule -Mailbox $_.UserPrincipalName |
    Where-Object { $_.ForwardTo -or $_.RedirectTo -or $_.ForwardAsAttachmentTo } |
    Select-Object MailboxOwnerId, Name, ForwardTo, RedirectTo
}
```

Attackers commonly leave forwarding rules behind to keep reading mail after they lose access, so this check matters after any incident.

### Hardening applied

| Control | Detail | Status |
| --- | --- | --- |
| Emergency (break-glass) admin accounts | Two cloud-only accounts on the `onmicrosoft.com` domain, so they work even if the company domain expires. No licence, MFA on a dedicated phone, passwords sealed offline in a safe | Done |
| Admin role separation | Separate admin account used only in a dedicated browser profile; global admin removed from the daily email account | Done |
| User consent to apps | Set to "Do not allow user consent", blocking consent-phishing attacks | Done |
| Admin consent workflow | Users can request an app; IT reviews and approves | Done |
| Unused accounts | 4 unused brand mailboxes converted to shared mailboxes (no password, access through named users' own accounts); 4 licences freed, about $725 a year | Done |
| MFA for every account | Security Defaults; follow-up on accounts not yet registered, including a plan for a shared workstation account | In progress |
| SPF, DKIM and DMARC | Stop others sending email as the company | Planned |
| Domain protection | Auto-renew, registrar lock, MFA on the registrar account | Planned |
| Self-service password reset | Fewer helpdesk resets, with MFA-verified identity | Planned |

## Skills demonstrated

`Microsoft 365 administration` · `Entra ID` · `DNS (MX, SPF, DKIM, DMARC)` · `Domain management` · `Incident response` · `Account recovery` · `Identity security` · `PowerShell (Exchange Online)` · `Least privilege`
