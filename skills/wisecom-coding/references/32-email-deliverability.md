# 32. Email Deliverability

Apply this chapter when code builds, signs, sends, queues, unsubscribes, suppresses, or diagnoses email, or when DNS for sending domains changes. Research basis: current Gmail/Google Workspace, Microsoft consumer mail and Microsoft 365, Yahoo/AOL, iCloud, GMX, Zoho, Proton, Yandex, and NetEase documentation, protocol RFCs, published research, and patents, checked 28 October 2026. Provider thresholds change; verify current provider documentation before treating a number below as a contract.

Receivers combine authenticated identity, sender reputation, recipient expectations, message content, and local organizational policy. Correct DNS cannot make unwanted mail wanted, and good wording cannot repair broken authentication or a damaged reputation. No provider publishes a reproducible global scoring formula.

## 32.1 Keep outcomes separate

"Spam filtering" covers several distinct outcomes: temporary SMTP deferral, permanent rejection, Spam/Junk placement, quarantine, and later reclassification. Promotions and Updates are inbox categories, not spam verdicts.

- MUST NOT treat SMTP acceptance (`250`) as inbox placement.
- MUST model and report SMTP acceptance, inbox placement, inbox category, complaints, and business outcomes as separate measurements.
- MUST distinguish temporary failures (`4xx`, back off and retry) from permanent failures (`5xx`, classify and suppress where the address is genuinely invalid).

## 32.2 Cross-provider baseline

Implement the strongest common requirement, even where one provider's minimum is weaker:

- [ ] SPF published for the envelope (MAIL FROM) domain, covering every production sending path.
- [ ] DKIM signing on every production sending path, aligned with the visible From domain.
- [ ] DMARC published for the From domain, with aggregate reporting (`rua`).
- [ ] TLS on outbound SMTP.
- [ ] Forward-confirmed reverse DNS for every sending IP, and a valid HELO/EHLO hostname.
- [ ] RFC 8058 one-click unsubscribe plus a visible body link for marketing mail.
- [ ] Recorded consent for every promotional recipient.

Provider differences that change implementation decisions:

| Receiver | Requirement that matters to code |
| --- | --- |
| Personal Gmail | All senders: SPF or DKIM, valid forward/reverse DNS, TLS. Bulk senders (~5,000 messages/24h to personal Gmail, aggregated across the primary domain and subdomains; classification persists once reached): SPF and DKIM, DMARC, alignment, one-click unsubscribe. Keep complaints below 0.1%; never reach 0.3%. Workspace inbound is excluded from these specific rules. |
| Outlook.com/Hotmail/Live/MSN | 5,000+ messages from the same visible From domain: SPF **and** DKIM must pass, DMARC must pass. Failure can return `550 5.7.515`. |
| Microsoft 365 | Tenant policies, bulk thresholds, impersonation protection, and recipient history decide handling. Consumer thresholds do not describe enterprise filtering. |
| Yahoo/AOL | Bulk: SPF and DKIM, passing DMARC, complaints below 0.3%, unsubscribe processed within two days. No published bulk-volume threshold. Yahoo Japan is a separate system. |
| iCloud | Bulk: explicit subscription, SPF/DKIM, published DMARC, reverse DNS, stable identity, unsubscribe, list hygiene. No bulk allowlist, no complaint feedback loop. |
| GMX | Valid aligned DKIM is mandatory; SPF alone is insufficient. Static IP, meaningful PTR, valid HELO/EHLO. |

## 32.3 Keep sender identities straight

| Identity | Purpose |
| --- | --- |
| Visible `From:` | Author shown to the recipient. DMARC evaluates this domain. |
| SMTP `MAIL FROM` | Envelope sender for SPF and bounces; usually appears as `Return-Path`. |
| DKIM `d=` | Domain taking cryptographic responsibility. |
| DKIM `s=` | Selector locating the DNS public key. |
| `Reply-To:` | Where replies go. May differ from From; not authenticated by DMARC. |

Common failure: SPF and DKIM pass for the email platform's own domain, but neither aligns with the customer's visible From domain, so DMARC fails.

Coherent example (reserved names, documentation IP, placeholder key):

```
Visible From:    updates@news.example.com
SMTP MAIL FROM:  bounce+id@bounce.news.example.com
DKIM domain:     news.example.com
DKIM selector:   m202610
```

```
bounce.news.example.com.           TXT "v=spf1 ip4:192.0.2.25 -all"
m202610._domainkey.news.example.com. TXT ("v=DKIM1; k=rsa; p=PUBLIC_KEY")
_dmarc.news.example.com.           TXT ("v=DMARC1; p=none; rua=mailto:dmarc@example.com")
```

DKIM `d=` matches the From domain exactly; the bounce subdomain gives relaxed SPF alignment.

## 32.4 Authentication and DNS checks

SPF

- [ ] Exactly one SPF TXT record at the actual envelope domain.
- [ ] All authorized sending paths included.
- [ ] Ten or fewer evaluated lookup-causing terms (`include`, `a`, `mx`, `ptr`, `exists`, `redirect`), counted recursively through nested includes. Exceeding the limit is `permerror`. The limit is not "ten providers" or "ten DNS packets".

DKIM

- [ ] The active sending path actually signs. A published key alone proves nothing.
- [ ] Signing is the last step: after MIME serialization, personalization, link/tracking rewrites, and footer insertion. Later substantive changes invalidate the signature.
- [ ] RSA-SHA256 with a 2048-bit key as baseline. SHA-1 is prohibited.
- [ ] `List-Unsubscribe` and `List-Unsubscribe-Post` are covered by the signature when present.

DMARC

- [ ] At least one passing, aligned SPF or DKIM identity. Some providers require both to pass.
- [ ] `p=none` understood as monitoring, not disabled authentication.
- [ ] Enforcement (`quarantine`/`reject`) chosen for spoofing protection, not expected to buy inbox placement.
- [ ] Current standard: RFC 9989 (May 2026), superseding RFCs 7489 and 9091. Verify receiver adoption before relying on new semantics.

DNS and SMTP identity

- [ ] Sending IP → PTR hostname → A/AAAA → same sending IP.
- [ ] Valid SMTP host identity in HELO/EHLO.
- [ ] IPv6 tested independently when enabled; its PTR and authentication must hold too.
- [ ] MX records are not mistaken for outbound authentication; MX concerns receiving mail.

Forwarding

- Forwarding can break SPF; mailing-list modification can break DKIM. ARC preserves prior authentication results but is not a universal trust override.

## 32.5 Message construction

- MUST build messages with a maintained MIME library, not string concatenation.
- MUST use correct encodings and multipart boundaries, a valid `Date`, a unique `Message-ID`, and no duplicated single-instance headers.
- MUST make `text/plain` and `text/html` alternatives carry the same information.
- SHOULD keep HTML simple and tested for email clients: no scripts, forms, iframes, or embedded objects.
- SHOULD keep essential information in readable text; images support it and carry alt text.
- SHOULD send attachments only when expected, clearly named, without double extensions; prefer a link to an authenticated account page.
- SHOULD make link labels match their destinations, using a recognizable domain related to the sender. Avoid URL shorteners.

Content patterns:

| Higher risk | Trustworthy alternative |
| --- | --- |
| `Re:`/`Fwd:` subject with no real thread | Accurate subject: "The project estimate you requested" |
| Sender name impersonating a CEO, bank, or authority | Stable company/team name on an authorized domain |
| "FINAL WARNING: verify your password immediately" | A real event and deadline; never request a password by reply |
| Unexpected bank-account change with a vague invoice | Reference the actual order; use an established payment route |
| Homoglyphs, zero-width characters, unrelated filler text | Ordinary accurate language, including legitimate non-English text |
| Familiar link label pointing to an unrelated domain | Clear action leading to the expected domain |
| Entire message in one promotional image | Readable text with supporting images |
| Plain-text part contradicting the HTML part | Equivalent meaning in both parts |

State why the recipient is receiving the message.

## 32.6 One-click unsubscribe (marketing mail)

```
List-Unsubscribe: <https://news.example.com/u/OPAQUE_TOKEN>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

The endpoint is a trust boundary and a data-integrity path:

- [ ] HTTPS `POST` works without login, cookies, confirmation step, or redirect.
- [ ] Token is unguessable and verified; it does not expose the address or a sequential ID.
- [ ] Suppression is committed durably before success is returned.
- [ ] Repeated requests are idempotent and harmless.
- [ ] A plain `GET` (link scanners, prefetchers) does not unsubscribe anyone.
- [ ] A visible unsubscribe link also exists in the body.
- [ ] Suppression is shared across every sending system and re-applied at dispatch time, so queued campaigns honor recent opt-outs.
- [ ] Processing completes well within two days (Yahoo requirement).

## 32.7 Sending behavior and list hygiene

| Increases risk | Decreases risk |
| --- | --- |
| Purchased lists, accidental or unverified signups | Explicit consent, address confirmation, recorded subscription scope |
| Turning a monthly update into daily sales mail | Purpose and frequency the recipient selected |
| Continuing after complaints or unsubscribe | Prompt, durable, shared suppression |
| Repeatedly mailing nonexistent addresses | Bounce classification and suppression of genuinely invalid recipients |
| Sudden unexplained spikes, aggressive retries | Predictable queues, gradual ramp-up, adaptive backoff |
| Abused shared pools, compromised accounts, unsafe linked domains | Monitored infrastructure, secured accounts, controlled sending |

- MUST record the user action or subscription that justifies each message.
- SHOULD separate transactional and promotional streams (subdomains, IPs, or ESP streams). This aids operations and diagnosis but does not guarantee independent reputation.
- SHOULD NOT use ordinary Exchange Online mailboxes as a bulk-mail platform.
- MUST back off on `4xx` deferrals; retries are bounded and idempotent per Message-ID (see chapter 16).
- SHOULD ramp new domains/IPs gradually with real opted-in traffic, adjusting to observed responses.

## 32.8 Monitoring and diagnostics

- Keep full SMTP response text, enhanced status codes, sender identities, Message-ID, template version, and protected `.eml` samples. Treat samples as personal data.
- Read the receiver-generated `Authentication-Results` header on a delivered message. Self-added `spf=pass` or favorable spam-score headers establish nothing.
- Investigate the receiver's stated reason before rewording content.
- Low or zero complaint rates can mean mail already lands in Spam, where users cannot report it. Low-volume data may be withheld and dashboards lag. Missing data is not health.
- Pixel opens are not a reputation signal: Google does not use them; Apple Mail Privacy Protection prefetches content.
- Microsoft 365: SCL no longer determines the cloud spam verdict (August 2026). Inspect `CAT`, `DIR`, `BCL`, `compauth`, the applicable policy, and the detection technology instead of an old SCL table.

Provider tooling:

| Receiver | Evidence source |
| --- | --- |
| Gmail | Postmaster Tools: authentication, reputation, compliance, campaign feedback |
| Yahoo | Complaint Feedback Loop for enrolled DKIM domains; feed it into suppression |
| Microsoft consumer | SNDS and JMRP |
| Microsoft 365 | Delivered headers, message trace, detector, tenant policy |

## 32.9 Never

- NEVER send marketing mail without recorded consent, or to purchased lists.
- NEVER modify a message after DKIM signing.
- NEVER publish multiple SPF records for one domain or exceed ten lookup-causing terms.
- NEVER let the visible From domain go unaligned with every passing authenticated identity.
- NEVER forge, copy, or self-insert `Authentication-Results`, spam-score, or other trust headers.
- NEVER fake reply/forward threads, impersonate people or institutions, or invent urgency.
- NEVER ask for passwords or credentials by email, or announce payment-route changes without an established verification path.
- NEVER obfuscate text (homoglyphs, zero-width characters, hidden filler) to evade filters.
- NEVER disguise link destinations.
- NEVER include scripts, forms, iframes, or embedded objects.
- NEVER keep mailing after an unsubscribe, complaint, or hard bounce, including from already-queued campaigns.
- NEVER unsubscribe on `GET`, or require login/confirmation for the one-click `POST`.
- NEVER retry permanent failures, or retry deferrals aggressively without backoff.
- NEVER report delivery success from SMTP acceptance, a DNS dashboard, or a generic spam checker.
- NEVER log full message bodies, tokens, or recipient lists into unprotected telemetry.

## 32.10 Do not enforce myths

The evidence does not support these blanket rules. Do not encode them as validators, lint rules, or review blockers:

- "Never use free, invoice, or discount."
- "Every email needs a 60:40 text-to-image ratio."
- "Plain-text email automatically lands in the inbox."
- "A dedicated IP is always better."
- "An emoji, attachment, or no-reply address automatically means spam."
- "Passing a generic spam checker predicts Gmail's decision."

Words still matter in context. Microsoft 365 Advanced Spam Filter options (off by default, not in Standard/Strict presets) can act on an undisclosed sensitive-word list, URL forms, and HTML elements. Zoho administrators can define subject/body patterns. NetEase names keyword examples such as 发票 ("invoice") and 展会 ("trade fair"). A local administrator can always be stricter than the provider default; diagnose per receiver.

## 32.11 Release checklist

Require evidence, not configuration screenshots:

- [ ] A message delivered through the real ESP to each target receiver shows SPF, DKIM, and DMARC `pass` with alignment in the receiver's `Authentication-Results`.
- [ ] Forward and reverse DNS and TLS verified on every production sending path, IPv4 and IPv6.
- [ ] Each message type has a recorded eligibility basis; transactional and promotional streams are separated.
- [ ] From name and domain are stable and recognizable; Reply-To and support routes work.
- [ ] Content is accurate; links resolve to expected domains; attachments are expected and safe; plain-text and HTML parts agree.
- [ ] One-click unsubscribe tested end to end: `POST` suppresses, `GET` does not, repeats are harmless, queued sends honor it.
- [ ] Bounce classification and complaint feedback reach the suppression system.
- [ ] SMTP diagnostics, Message-ID, and template version are captured for every send.
- [ ] Launch volume ramps gradually with backoff on deferrals.
