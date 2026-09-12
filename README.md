# Professional Email on Your Own Domain — Free, Forever, Inside Gmail

Setup guide for a **Namecheap** domain. Send *and* receive from `info@`, `hello@`, `sales@` without ever leaving Gmail. No monthly fee, no credit card.

📄 **PDF version:** [Free-Domain-Email-in-Gmail-Setup-Guide.pdf](Free-Domain-Email-in-Gmail-Setup-Guide.pdf) — click it, then click **Download** (or just read this page, it is the same guide).

Replace `yourdomain.com` and `yourname@gmail.com` throughout with your real values.

---

## 1. Which services we use, and why

Two free services, each doing the half it does best. Neither asks for a card.

| Job | Service | Why this one |
|---|---|---|
| **Receiving** (incoming mail) | ImprovMX (free plan) | Catches mail sent to your domain and forwards it straight into your existing Gmail inbox. Free plan: 1 domain, 25 addresses, 500 forwarded mails/day. It uses SRS, so forwarded mail still passes sender checks at Gmail. |
| **Sending** (outgoing mail) | Brevo SMTP (free plan) | Gmail needs a real SMTP server to send as your domain. Brevo's free plan gives 300 mails/day through `smtp-relay.brevo.com` and — the important part — it signs with DKIM using *your* domain, so SPF/DKIM/DMARC all line up. |

### Why not the other routes

- **Zoho Mail free plan** — real mailboxes, but webmail/app access only. IMAP, POP and SMTP are paid-only, so it cannot be driven from inside Gmail. Ruled out by your "never leave Gmail" requirement.
- **Gmail's "Send through Gmail" option** (no SMTP server) — works, costs nothing, but mail leaves through Google's servers with a `gmail.com` return path. Recipients see "via gmail.com" and your domain's SPF/DKIM do not align, so DMARC reports a fail. Your brief asked for proper authentication, so we use real SMTP instead.
- **Yandex 360** — free domain mail is no longer offered to new sign-ups outside Russia and needs phone verification. Not dependable.

> **Honest note on "forever":** both free tiers are long-standing and advertised as free, but no company can be contractually bound to that. Section 9 lists the drop-in replacements (Cloudflare Email Routing, Zoho) and what you would change if either provider ever alters its terms. You would never lose the addresses themselves — they live on your domain, not on their platform.

---

## 2. Before you start

- Decide your addresses now, 2–5 of them — e.g. `info@`, `hello@`, `sales@`.
- Have your Namecheap login ready (*Domain List → Manage → Advanced DNS*).
- Have the Gmail account open that everything will land in.
- Total time: about 30 minutes of clicking, plus DNS propagation (usually 10–60 minutes).

> ⚠️ **One-time warning:** adding the MX records below moves *all* mail for your domain to ImprovMX. If the domain currently receives mail anywhere else (cPanel mailbox, old host, Namecheap Private Email), that mailbox stops receiving the moment the new MX records go live. Tell me if that applies and we will plan the cut-over.

---

## 3. Step 1 — Create the forwarding addresses (ImprovMX)

1. Go to **improvmx.com**. On the home page, type your domain and your Gmail address into the two boxes and create the free account (or *Sign up* → free plan).
2. Verify your Gmail address from the confirmation mail ImprovMX sends you.
3. In the dashboard, open your domain. Add one alias per address you want:
   - `info` → `yourname@gmail.com`
   - `hello` → `yourname@gmail.com`
   - `sales` → `yourname@gmail.com`
4. Recommended: leave the catch-all alias `*` switched **off**. A catch-all accepts mail to every possible name on your domain and attracts spam.
5. ImprovMX will show your domain as "pending" / red until the DNS records in Step 2 exist. That is expected.

---

## 4. Step 2 — DNS records in Namecheap

Namecheap: *Domain List → Manage* (next to your domain) *→ Advanced DNS*.

### 4a. Switch Mail Settings to Custom MX first

> ⚠️ On the **Advanced DNS** page, scroll to **Mail Settings** and set it to **Custom MX**. If it is left on "Email Forwarding" or "Private Email", Namecheap overrides whatever MX records you type and nothing will work. Delete any MX rows that were already there.

### 4b. Add these records

In Namecheap the domain is appended automatically — type `@` for the root and `mail._domainkey` (not the full hostname) for the DKIM row.

| Type | Host | Value | Priority | TTL |
|---|---|---|---|---|
| MX | `@` | `mx1.improvmx.com` | 10 | Automatic |
| MX | `@` | `mx2.improvmx.com` | 20 | Automatic |
| TXT | `@` | `v=spf1 include:spf.improvmx.com include:spf.brevo.com ~all` | — | Automatic |
| TXT | `@` | `brevo-code:<the code Brevo shows you in Step 3>` | — | Automatic |
| TXT | `mail._domainkey` | `<the DKIM value Brevo shows you in Step 3>` | — | Automatic |
| TXT | `_dmarc` | `v=DMARC1; p=none; rua=mailto:yourname@gmail.com; fo=1` | — | Automatic |

**Rules that bite people here:**

- You may have **only one** SPF record (one TXT starting `v=spf1`). If one already exists, merge the two `include:` parts into it — do not add a second.
- Multiple *other* TXT records on `@` are perfectly fine (the SPF one and the brevo-code one can coexist).
- Do the MX + SPF + DMARC rows now; come back and add the brevo-code and DKIM rows after Step 3, when Brevo has generated them. Brevo may show the DKIM as a CNAME pair (`brevo1._domainkey` / `brevo2._domainkey`) instead of a TXT — if so, add exactly what it shows.

Back in ImprovMX, click **Check again**. Once it turns green, incoming mail is already working — test it right away by mailing `info@yourdomain.com` from your phone.

---

## 5. Step 3 — Sending credentials (Brevo)

1. Sign up free at **brevo.com** and confirm your account (it asks for a website/company name — any answer is fine).
2. Go to **Senders, Domains & Dedicated IPs → Domains → Add a domain**. Enter `yourdomain.com` and choose the manual / "I'll do it myself" option rather than letting it log into your registrar.
3. Brevo now shows you the **Brevo code** and the **DKIM** record. Copy both into Namecheap exactly as shown (rows 4 and 5 of the table above). Copy/paste — never retype a DKIM key.
4. Wait a few minutes, then click **Authenticate this email domain** in Brevo. You want green ticks on Brevo code and DKIM.
5. Go to **Senders** and add each address (`info@yourdomain.com` etc.) as a sender. With the domain authenticated these are accepted without separate per-address verification.
6. Go to **SMTP & API → SMTP**. Note the three values — server `smtp-relay.brevo.com`, port `587`, login (your Brevo account email) — and click to generate a new **SMTP key**. Copy it now; it is shown once. That key is the password Gmail will use.

> Brevo free allows 300 outgoing mails per day, shared across everything you send. For business correspondence that is a very high ceiling — but it is a ceiling, so do not use these addresses for bulk mailing.

---

## 6. Step 4 — Wire it into Gmail

### 6a. Sending ("Send mail as")

1. Gmail → gear icon → **See all settings** → **Accounts and Import** tab.
2. Under *Send mail as*, click **Add another email address**.
3. **Name:** what recipients should see (your name, or your business name).
   **Email address:** `info@yourdomain.com`.
   Leave **Treat as an alias** ticked. Click *Next Step*.
4. If Gmail offers a choice, pick **Send through yourdomain.com SMTP servers**, not "Send through Gmail". Then fill in:

   ```
   SMTP Server : smtp-relay.brevo.com
   Port        : 587
   Username    : your Brevo login e-mail
   Password    : the Brevo SMTP key from Step 3.6
   Security    : Secured connection using TLS  (recommended)
   ```

5. Click **Add Account**. Gmail mails a verification code to `info@yourdomain.com` — which ImprovMX forwards into this same inbox, usually within seconds. Paste the code in, or click the link in it.
6. Repeat 2–6 for each address. The same Brevo login and SMTP key are reused every time.

### 6b. Replying from the right address

Still in *Accounts and Import*:

- Set **When replying to a message:** to *Reply from the same address the message was sent to*. This is what makes a mail that arrived at `sales@` automatically reply as `sales@` — without it, everything goes out from your personal Gmail address.
- Under *Send mail as*, use *make default* if you want a domain address to be the default for brand-new mails.

### 6c. Keeping the inbox tidy (optional)

Gmail → Settings → **Filters and Blocked Addresses** → *Create a new filter*. In **To:** put `info@yourdomain.com`, then *Create filter* → **Apply the label** → new label "Info". One filter per address, each with its own colour-coded label in the sidebar.

---

## 7. Final checklist — proving it all works

Everything here is free and takes about ten minutes.

| ✓ | Check | How, and what you should see |
|---|---|---|
| ☐ | MX records live | At `mxtoolbox.com/SuperTool.aspx` run *MX Lookup* on your domain. Expect `mx1.improvmx.com` (10) and `mx2.improvmx.com` (20) and nothing else. Command line: `nslookup -type=mx yourdomain.com` |
| ☐ | SPF valid | MXToolbox *SPF Record Lookup*. Exactly one record; both includes present; no "more than one SPF record" error. |
| ☐ | DKIM published | MXToolbox *DKIM Lookup*, selector `mail`. Or `nslookup -type=txt mail._domainkey.yourdomain.com`. And: Brevo's domain page shows green ticks. |
| ☐ | DMARC published | `nslookup -type=txt _dmarc.yourdomain.com` returns your `v=DMARC1; p=none; ...` record. |
| ☐ | Receiving works | From an unrelated account (phone, a friend, another webmail) mail each address in turn. Each one lands in your Gmail inbox. Check the ImprovMX logs if one does not. |
| ☐ | Sending works | Compose in Gmail, switch the *From* to `info@yourdomain.com`, send to another mailbox you own. It arrives, shows your domain address as the sender, and there is **no** "via gmail.com" or "on behalf of" next to the name. |
| ☐ | Round trip | Reply to that mail from the other mailbox. The reply comes back to your Gmail, and hitting Reply sends it out as the domain address again. |
| ☐ | Authentication passes | Send one mail from `info@` to the address shown at `mail-tester.com` (free, 3/day). Aim for 9–10/10 with SPF, DKIM and DMARC all green and DKIM signed by *your* domain. |
| ☐ | Headers confirm it | In Gmail, open a received test mail → three dots → *Show original*. Look for `SPF: PASS`, `DKIM: 'PASS' with domain yourdomain.com`, `DMARC: 'PASS'`. |
| ☐ | Not in spam | Send a test to a Gmail, an Outlook/Hotmail and a Yahoo address. All three should reach the inbox, not the junk folder. |

> ✅ **The one check that matters most** is the Gmail "Show original" line reading `DKIM: 'PASS' with domain yourdomain.com`. That single line is the difference between this setup and the free shortcuts — it proves your domain really authorised the message, which is what keeps you out of spam folders long-term.

---

## 8. If something misbehaves

| Symptom | Cause and fix |
|---|---|
| ImprovMX stays red / "MX not found" | Namecheap Mail Settings is not on **Custom MX**, or an old MX row is still present. Fix both, then wait 30 minutes — Namecheap's default TTL means old values can linger. |
| Incoming mail never arrives | Check the ImprovMX dashboard logs: they tell you whether the mail reached them (a DNS problem) or was rejected after (a forwarding problem). Also check Gmail Spam. |
| Gmail: "Authentication failed. Please check your username/password" | The SMTP key was mistyped, or you used your Brevo *account password* instead of the generated SMTP key. Generate a fresh key and paste it. |
| Gmail: "TLS Negotiation failed" / connection refused | Wrong port. Use 587 with TLS. If your network blocks 587, Brevo also accepts 2525. |
| Gmail verification code never arrives | Incoming side is not finished. Confirm Step 2 is green in ImprovMX first, then retry — Gmail lets you resend the code. |
| Recipients see "via brevo" or "on behalf of" | DKIM is not authenticated yet in Brevo. Finish Step 3.4 and the label disappears. |
| Brevo blocks sending | The sender address was not added under *Senders*, or the 300/day cap was hit. Both are visible on the Brevo dashboard. |
| Everything looks right but mail lands in spam | A brand-new domain has no sending reputation. Keep volumes low and natural for the first week or two; it settles. Do not send mass mail from these addresses. |

---

## 9. Long-term: keeping it free

- **Once a year**, log into ImprovMX and Brevo. Both can deactivate dormant free accounts; a single login resets that.
- **Renew the domain.** The only bill in this whole setup is your Namecheap renewal. If the domain lapses, the addresses go with it.
- **Keep DMARC at `p=none`** for the first few weeks and read the reports that arrive at your Gmail. Once you see nothing but passes, tighten it to `p=quarantine`.
- **If ImprovMX ever changes its free tier:** swap in *Cloudflare Email Routing* — free, unlimited addresses, no daily cap. It needs your domain's nameservers pointed at Cloudflare (free, registration stays at Namecheap); you then replace the two MX rows with Cloudflare's three. Nothing else in this guide changes.
- **If Brevo ever changes its free tier:** alternatives with free SMTP tiers include Mailjet (200/day) and SMTP2GO (1,000/month). You would only re-do Step 3 and the password field in Step 4a.
- **If you later want true mailboxes** that exist independently of your Gmail, Zoho's free plan gives five of them — webmail and mobile app only, so it is a companion to this setup, not a replacement.

---

*Prepared for Freelancer project 40707214 — Lifetime Free Domain Emails Setup. Send me your domain name and I will return the record table pre-filled, and I will stay on hand through propagation and the checklist.*
