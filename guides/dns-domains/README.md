# DNS and Domains: Zero to Ship

> The internet's phone book that turns names you can remember into addresses computers can find.

## Why Should You Care?

You've built an app. It's deployed. It's sitting at some URL like `my-app-abc123.vercel.app`. Now you want it at `yourapp.com`. That requires understanding DNS, the system that translates human-friendly names into the IP addresses that computers actually use to find each other. Without DNS, there is no "visiting a website." There's only typing `142.250.80.46` into your browser and hoping you remembered the right numbers.

DNS is invisible when it works and maddening when it doesn't. You'll change a DNS record and nothing happens for hours. You'll connect a custom domain and get an SSL error. You'll set up email and wonder why messages go to spam. Every one of these problems has a clear explanation once you understand how DNS works. Without that understanding, you're guessing and waiting, sometimes for 48 hours, to find out if your guess was right.

If you build with AI tools, you'll ask them to help you configure DNS, and they'll spit out records that look right. But DNS errors are silent. There's no stack trace. No error log. The site just doesn't load, or loads the wrong thing, or works for you but not your users. You need enough understanding to verify what the AI generates, because the feedback loop for DNS mistakes is measured in hours, not seconds.

## The 30-Second Version

Every website has a name (like `hustlrai.com`) and a server with an IP address (like `76.76.21.21`). DNS is the system that connects the two. When you type a domain into your browser, your computer asks a chain of DNS servers "what's the IP for this name?" until it gets an answer. You control this mapping by setting DNS records at your domain registrar or nameserver provider. An A record points a name to an IP address. A CNAME points a name to another name. You buy domains from registrars, point them at your hosting platform with the right records, and the platform handles the rest, including SSL certificates for HTTPS.

## The Real Explanation (ELI5)

Imagine the entire internet is a city. Every building (server) has a street address (IP address), like 142.250.80.46. But nobody remembers street addresses. So people use names instead: "Google's office," "Stripe's headquarters," "Your app's building."

The city has a phone book service. When you say "I need to get to google.com," the phone book service looks up the name and tells you the street address: "That's at 142.250.80.46. Head there."

But it's not one phone book. It's a chain of them:

1. **Your own memory.** You went to google.com five minutes ago. You still remember the address. No need to look it up again. (This is the browser cache.)

2. **The building's front desk.** Your apartment building keeps a shared list of recently looked-up addresses. Someone in your building visited google.com earlier, so the address is already there. (This is the OS/router cache.)

3. **The local phone book office.** If nobody in your building knows, you call the local office. They handle most requests because they've looked up thousands of addresses recently. (This is the DNS resolver, usually run by your ISP or a service like Cloudflare's 1.1.1.1.)

4. **The city directory.** If the local office doesn't know, they ask the city directory: "Who's in charge of all .com addresses?" The city directory says "Go ask the .com office." (This is the root server pointing to the TLD server.)

5. **The .com office.** The .com office says "For google.com, the building manager is at this address." (This is the TLD server pointing to the authoritative nameserver.)

6. **The building manager.** The building manager has the definitive answer: "google.com is at 142.250.80.46." (This is the authoritative nameserver.)

Now replace "city" with "internet," "building" with "server," "street address" with "IP address," "phone book" with "DNS," and "building manager" with "nameserver." That's DNS.

The analogy is accurate at one important point: this lookup happens in order, and it stops the moment any step has the answer. If your browser remembers the address, it never asks anyone else. This is why DNS changes aren't instant. Old answers are cached at every step in the chain, and each step holds onto its answer for a set amount of time (TTL, time to live) before it's willing to ask again.

## How It Actually Works

### The DNS Lookup Chain

When you type `hustlrai.com` into your browser:

```
Browser cache (visited recently?)
    │ miss
    ▼
OS cache (anyone on this machine visited recently?)
    │ miss
    ▼
Router cache (anyone on this network visited recently?)
    │ miss
    ▼
DNS Resolver (ISP or 1.1.1.1 / 8.8.8.8)
    │ miss
    ▼
Root Server (one of 13 worldwide)
    "I don't know hustlrai.com, but .com is handled by these TLD servers"
    │
    ▼
TLD Server (.com registry)
    "I don't know hustlrai.com, but its nameservers are ns1.cloudflare.com"
    │
    ▼
Authoritative Nameserver (ns1.cloudflare.com)
    "hustlrai.com points to 76.76.21.21"
    │
    ▼
Answer flows back up the chain.
Every step caches the answer for next time.
Browser connects to 76.76.21.21.
```

**Under the hood:** This entire chain happens in 20-100 milliseconds for an uncached lookup. Cached lookups (the common case) take under 1 millisecond. Your browser makes dozens of DNS lookups per page (the main domain, CDN domains, analytics domains, font domains), so caching is critical for performance.

**Why it's designed this way:** No single server could handle every DNS lookup on the internet. There are billions of lookups per minute. The hierarchical design distributes the load: root servers only handle "who's in charge of .com?" questions, TLD servers only handle "who's in charge of hustlrai.com?" questions, and authoritative servers only handle "what's the IP for hustlrai.com?" questions. Each level does less work.

### DNS Record Types

DNS records are the entries in the phone book. Each record maps a name to some value. Here are the ones you'll actually use:

**A Record (Address)**

Maps a domain name directly to an IPv4 address.

```
hustlrai.com    A    76.76.21.21
```

"When someone asks for hustlrai.com, send them to this IP address."

This is the most fundamental record. Every domain that serves a website needs at least one A record (or its IPv6 equivalent).

**AAAA Record (IPv6 Address)**

Same as A, but for IPv6 addresses. IPv6 addresses are longer because the internet is running out of IPv4 addresses.

```
hustlrai.com    AAAA    2606:4700:3030::6815:1234
```

Most hosting platforms handle IPv6 automatically. You'll add AAAA records when your platform tells you to.

**CNAME Record (Canonical Name)**

Maps a domain name to another domain name. It's an alias.

```
www.hustlrai.com    CNAME    hustlrai.com
docs.hustlrai.com   CNAME    my-docs-site.vercel.app
```

"When someone asks for www.hustlrai.com, look up hustlrai.com instead and use that answer."

CNAMEs create a chain: the DNS resolver follows the CNAME to the target, then resolves that target normally. This is useful because if the target IP changes, you don't need to update the CNAME. It follows automatically.

**Critical rule:** You cannot put a CNAME on the root domain (hustlrai.com without any subdomain). This is a DNS specification rule, not a platform limitation. CNAMEs only work on subdomains (www, api, docs). For the root domain, use an A record. Some providers (Cloudflare, Route 53) offer a workaround called ALIAS or ANAME that behaves like a CNAME at the root, but it's not a standard DNS record type.

**MX Record (Mail Exchange)**

Tells email servers where to deliver mail for your domain.

```
hustlrai.com    MX    10    mail.google.com
hustlrai.com    MX    20    mail2.google.com
```

"Email sent to anyone@hustlrai.com should be delivered to mail.google.com. If that's down, try mail2.google.com."

The number (10, 20) is the priority. Lower numbers are tried first. If you use Google Workspace, Zoho, or any email provider, they'll give you MX records to add.

**TXT Record (Text)**

Stores arbitrary text. Used for verification and email security.

```
hustlrai.com    TXT    "v=spf1 include:_spf.google.com ~all"
hustlrai.com    TXT    "google-site-verification=abc123..."
```

Common uses:
- **Domain verification:** Proving you own the domain. Google, Vercel, Stripe, and dozens of other services ask you to add a TXT record with a specific value.
- **SPF:** Tells email servers which servers are allowed to send email from your domain. Without it, your emails go to spam.
- **DKIM:** A cryptographic signature that proves an email wasn't tampered with in transit.
- **DMARC:** A policy that tells receiving servers what to do when SPF or DKIM checks fail.

**NS Record (Nameserver)**

Specifies which servers are authoritative for your domain. These are set at your registrar.

```
hustlrai.com    NS    ns1.cloudflare.com
hustlrai.com    NS    ns2.cloudflare.com
```

"For all questions about hustlrai.com, ask these servers."

You change NS records when you want a different provider to manage your DNS (for example, moving DNS management from Namecheap to Cloudflare). This is one of the most impactful DNS changes, because once you change nameservers, all your other records need to exist at the new provider.

### Buying a Domain

A domain registrar is a company authorized to sell domain names. You're not buying the name permanently. You're renting it, typically for 1 year at a time, with auto-renewal.

**Popular registrars:**

| Registrar | Pricing | Notes |
|-----------|---------|-------|
| Cloudflare Registrar | At-cost (no markup) | Cheapest option. Also offers DNS and CDN. No upsells. |
| Namecheap | Low, with occasional sales | Good UI. Free WhoisGuard (privacy). |
| Porkbun | Low | Simple, no-nonsense. Growing in popularity. |
| Google Domains | Was competitive | Sold to Squarespace in 2023. Still works, but no longer Google-run. |
| GoDaddy | Cheap first year, expensive renewals | Aggressive upselling. Dark patterns. Avoid. |

**What you're actually paying for:**
- The right to use the name for the registration period (usually 1 year).
- A record in the global .com (or .io, .dev, etc.) registry that says "this person controls this name."
- The ability to set nameservers, which determine where your DNS records live.

**Tips:**
- Check the renewal price, not just the first-year price. Some registrars charge $2 the first year and $20/year after.
- Enable WHOIS privacy. This hides your personal contact info from public WHOIS lookups. Most registrars include this free now.
- Enable auto-renewal and two-factor authentication. Losing a domain because you forgot to renew is a real and painful scenario.
- `.com` is the default and most trusted TLD. `.io`, `.dev`, and `.app` are common for tech products. Avoid obscure TLDs for anything user-facing.

### Nameservers: What They Are and When to Change Them

When you buy a domain from Namecheap, Namecheap's nameservers are set by default. This means Namecheap's servers answer all DNS queries for your domain, and you manage your DNS records through Namecheap's dashboard.

You change nameservers when you want a different provider to manage your DNS. The most common reason: moving DNS to Cloudflare for their free CDN, DDoS protection, and faster global DNS resolution.

**How to change nameservers:**

```
1. Sign up at the new DNS provider (e.g., Cloudflare)
2. Add your domain. The provider gives you new nameservers:
   ns1.cloudflare.com
   ns2.cloudflare.com
3. Go to your registrar (where you bought the domain)
4. Find the nameserver settings
5. Replace the old nameservers with the new ones
6. Wait (up to 48 hours, usually 1-4 hours)
7. Manage all DNS records at the new provider from now on
```

**Important:** When you change nameservers, the old provider's DNS records stop being used. You need to recreate all your records at the new provider before (or immediately after) the switch. Otherwise, your site, email, and everything else goes down until the records exist at the new provider.

### Connecting a Domain to Your Hosting Platform

This is the part you'll do most often. You have a deployed app and want it on your custom domain.

**Vercel:**

```
1. Vercel Dashboard > Project > Settings > Domains
2. Add your domain: hustlrai.com
3. Vercel tells you to add these records:

   hustlrai.com       A       76.76.21.21
   www.hustlrai.com   CNAME   cname.vercel-dns.com

4. Go to your DNS provider, add the records
5. Wait for verification (usually 1-5 minutes)
6. Vercel auto-provisions an SSL certificate
7. Your site is live at hustlrai.com
```

**Netlify:**

```
1. Netlify Dashboard > Site > Domain management > Add custom domain
2. Add your domain: hustlrai.com
3. Netlify tells you to add:

   hustlrai.com       A       75.2.60.5
   www.hustlrai.com   CNAME   your-site.netlify.app

4. Add records at your DNS provider
5. Netlify auto-provisions SSL
```

**Railway:**

```
1. Railway Dashboard > Service > Settings > Domains
2. Add custom domain: api.hustlrai.com
3. Railway gives you a CNAME target:

   api.hustlrai.com   CNAME   your-service.up.railway.app

4. Add the CNAME at your DNS provider
5. Railway provisions SSL after verification
```

**What each record does in this context:**
- The **A record** on the root domain (`hustlrai.com`) points directly to the platform's IP address. Traffic goes straight there.
- The **CNAME** on `www` points to the platform's DNS name. This lets the platform change its underlying IP without you needing to update anything.
- The platform uses the domain to route traffic to your specific project (Vercel hosts millions of sites on the same IPs, so it uses the domain name in the request to figure out which project to serve).

### SSL/TLS and HTTPS

When you visit `https://hustlrai.com`, two things are happening that don't happen with plain `http://`:

1. **Encryption.** Everything between your browser and the server is encrypted. No one on the network (your ISP, the coffee shop WiFi, a malicious router) can read the data.
2. **Identity verification.** The server proves it's actually hustlrai.com, not an impersonator.

Both of these are provided by an SSL/TLS certificate, a cryptographic file on the server that the browser verifies.

**How it works:**

```
Browser: "I want to connect to hustlrai.com securely"
         │
         ▼
Server:  "Here's my SSL certificate. It was issued by Let's Encrypt
          and it's valid for hustlrai.com"
         │
         ▼
Browser: "I trust Let's Encrypt. The certificate matches the domain.
          The certificate hasn't expired. Connection is secure."
         │
         ▼
         🔒 Padlock appears. HTTPS is active.
```

**Let's Encrypt** is a free, automated certificate authority. Before Let's Encrypt (launched 2015), SSL certificates cost $50-300/year and required manual renewal. Now they're free and auto-renew. This is why HTTPS went from "only for banks" to "every website."

**On modern platforms (Vercel, Netlify, Railway, Fly.io):** SSL is fully automatic. When you add a custom domain, the platform provisions a Let's Encrypt certificate for you. It auto-renews before expiration. You don't touch any certificate files. You don't configure anything. It just works.

**When SSL doesn't "just work":** The most common cause is incorrect DNS records. The platform needs to verify you own the domain before issuing a certificate. If the DNS records aren't pointing to the platform yet (because of propagation delay), the certificate can't be issued. Wait for DNS propagation, then the certificate will provision automatically.

### Subdomains

A subdomain is a prefix added to your domain: `api.hustlrai.com`, `docs.hustlrai.com`, `staging.hustlrai.com`. Each subdomain can point to a completely different server, platform, or service.

```
hustlrai.com          → Vercel (main website)
api.hustlrai.com      → Railway (backend API)
docs.hustlrai.com     → GitBook or Mintlify (documentation)
staging.hustlrai.com  → Vercel preview (staging environment)
```

**How to set them up:**

Each subdomain gets its own DNS record:

```
api.hustlrai.com      CNAME    api-service.up.railway.app
docs.hustlrai.com     CNAME    your-team.gitbook.io
staging.hustlrai.com  CNAME    staging-project.vercel.app
```

Then add the subdomain as a custom domain on the target platform. The platform needs to know it should respond to requests for that subdomain.

**Subdomains are independent.** Changing a record for `api.hustlrai.com` has zero effect on `hustlrai.com` or `docs.hustlrai.com`. They're separate entries in the DNS phone book. This is why subdomains are the standard way to separate services: your API can go down without affecting your marketing site.

**Wildcard subdomains** (`*.hustlrai.com`) route all unmatched subdomains to one destination. Useful for multi-tenant apps where each customer gets `customer-name.hustlrai.com`. Set up with:

```
*.hustlrai.com    CNAME    your-app.vercel.app
```

### Propagation and TTL

DNS changes aren't instant. When you update a record, the old value is cached at DNS resolvers around the world. Each cached value has a TTL (Time To Live) that says "keep this answer for X seconds before asking again."

```
You change A record: hustlrai.com → new IP

What happens:
  Resolver in Mumbai:  still has old IP cached (TTL: 3500s remaining)
  Resolver in NYC:     cache expired, gets new IP immediately
  Resolver in London:  still has old IP cached (TTL: 1200s remaining)

Result: some users see the old site, some see the new site.
This resolves as each cache expires and re-fetches.
```

**Common TTL values:**

| TTL | Duration | When to use |
|-----|----------|-------------|
| 300 | 5 minutes | Before and during DNS changes. Minimizes propagation time. |
| 3600 | 1 hour | Default for most records. Good balance. |
| 86400 | 24 hours | Records that never change (MX, NS). |

**The TTL strategy for DNS changes:**

```
1. Lower TTL to 300 seconds (5 minutes)
2. Wait for the OLD TTL to expire
   (if it was 3600, wait at least 1 hour)
3. Make your DNS change
4. Changes propagate within 5-10 minutes globally
5. After everything is confirmed working, raise TTL back to 3600
```

Most people skip steps 1-2 and wonder why their changes take "forever." If your TTL was 86400 (24 hours) and you change a record, some resolvers will serve the old value for up to 24 hours.

**How to check if your changes are live:**

```bash
# Check what a specific DNS server returns
dig hustlrai.com @1.1.1.1

# Check what your machine sees
dig hustlrai.com

# Quick check for specific record types
dig hustlrai.com A
dig hustlrai.com CNAME
dig hustlrai.com MX
dig hustlrai.com TXT

# Alternative: nslookup
nslookup hustlrai.com
nslookup hustlrai.com 8.8.8.8

# Check propagation worldwide
# Visit: https://www.whatsmydns.net
# Enter your domain, select record type, see results from resolvers globally
```

`dig` output explained:

```
;; ANSWER SECTION:
hustlrai.com.    300    IN    A    76.76.21.21
│                │      │     │    │
│                │      │     │    └── The value (IP address)
│                │      │     └── Record type
│                │      └── Internet class (always IN)
│                └── TTL in seconds (300 = 5 minutes)
└── The domain queried
```

## The Mental Model

**Think of DNS as a chain of phone books, where each book caches answers and won't check again until its timer runs out.** This means whenever you make a DNS change, the question isn't "did I do it right?" but "has everyone's cache expired yet?" You can verify your change is correct immediately using `dig @1.1.1.1`, but the rest of the world might still see the old value until their caches expire.

**Think of DNS records as forwarding instructions at a post office.** An A record is "deliver mail for this name to this address." A CNAME is "this name is an alias for that other name, follow the alias." An MX record is "email for this domain goes to that mail server." This means whenever you set up a new service (email, docs, API), you're adding a new forwarding instruction, not changing the existing ones.

**Think of nameservers as choosing which phone book company manages your listing.** Changing nameservers doesn't change your address. It changes who you trust to answer questions about your address. This means whenever you switch nameservers, make sure your new phone book company has all the same listings as the old one, or things disappear.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| DNS | The system that translates domain names to IP addresses. | Every time any device connects to any website. |
| Domain name | A human-readable address like hustlrai.com. | Buying domains, setting up hosting. |
| IP address | A numeric address that identifies a server on the internet (e.g., 76.76.21.21). | DNS records, server configuration. |
| Registrar | A company you buy domain names from (Cloudflare, Namecheap, Porkbun). | Domain purchase, renewal, nameserver changes. |
| Nameserver | A server that answers DNS queries for your domain. | NS records, switching DNS providers. |
| DNS record | An entry that maps a name to a value (IP, another name, text, etc.). | Every time you connect a domain to a service. |
| A record | Maps a name to an IPv4 address. | Pointing your domain at a server. |
| AAAA record | Maps a name to an IPv6 address. | Same as A record, for IPv6. |
| CNAME | Maps a name to another name (alias). Only for subdomains. | Pointing subdomains at hosting platforms. |
| MX record | Specifies which server handles email for the domain. | Setting up email (Google Workspace, Zoho). |
| TXT record | Stores text data, used for verification and email auth. | Domain verification, SPF, DKIM, DMARC. |
| NS record | Specifies which nameservers are authoritative for the domain. | Changing DNS providers. |
| TTL | Time To Live. How long a DNS answer should be cached before re-checking. | DNS changes, propagation. |
| Propagation | The time it takes for a DNS change to be visible worldwide. | After making any DNS change. |
| Subdomain | A prefix on your domain (api.hustlrai.com, docs.hustlrai.com). | Separating services, staging environments. |
| SSL/TLS | Encryption protocol that makes HTTPS work. | Custom domains, security, certificates. |
| Certificate | A cryptographic file that proves a server is who it claims to be. | HTTPS, Let's Encrypt, SSL errors. |
| Let's Encrypt | A free certificate authority that auto-issues and auto-renews SSL certificates. | Every modern hosting platform uses this. |
| WHOIS | The public directory of domain ownership information. | Domain lookup, privacy. |
| Root domain | The bare domain without any subdomain (hustlrai.com, not www.hustlrai.com). | Also called "apex domain" or "naked domain." |
| Wildcard | A DNS record for *.yourdomain.com that matches all subdomains. | Multi-tenant apps, catch-all routing. |

## Common Patterns

### Pattern 1: Custom Domain on Vercel

**When to use it:** You've deployed a Next.js app on Vercel and want it on your own domain.

**How it works:** Add the domain in Vercel's dashboard, create DNS records at your provider, and Vercel handles SSL automatically.

**Steps:**

```
1. Vercel Dashboard > Project > Settings > Domains > Add "hustlrai.com"

2. Add these DNS records at your provider:

   Type    Name    Value
   A       @       76.76.21.21
   CNAME   www     cname.vercel-dns.com

3. Wait 1-5 minutes for verification

4. Vercel provisions SSL certificate automatically

5. Configure redirect: www.hustlrai.com → hustlrai.com
   (Vercel offers this in the domain settings)
```

The `@` symbol means the root domain (hustlrai.com itself, no subdomain). Some providers use `@`, others leave the name field blank. Check your provider's docs.

### Pattern 2: Email Setup with MX Records

**When to use it:** You want `you@hustlrai.com` email using Google Workspace, Zoho, or another provider.

**How it works:** MX records tell the world's email servers where to deliver mail addressed to your domain.

**Google Workspace example:**

```
Type    Name    Priority    Value
MX      @       1           ASPMX.L.GOOGLE.COM
MX      @       5           ALT1.ASPMX.L.GOOGLE.COM
MX      @       5           ALT2.ASPMX.L.GOOGLE.COM
MX      @       10          ALT3.ASPMX.L.GOOGLE.COM
MX      @       10          ALT4.ASPMX.L.GOOGLE.COM
```

Also add SPF to prevent your emails from going to spam:

```
Type    Name    Value
TXT     @       "v=spf1 include:_spf.google.com ~all"
```

And DKIM (Google Workspace provides the value):

```
Type    Name                Value
TXT     google._domainkey   "v=DKIM1; k=rsa; p=MIIBIjAN..."
```

Email DNS is fiddly. Follow your provider's exact instructions. Missing one record means emails go to spam or don't arrive at all.

### Pattern 3: Subdomain for API or Docs

**When to use it:** Your API runs on a different platform than your frontend, or you use a hosted docs service.

**How it works:** Create a CNAME record for the subdomain pointing to the service, then register the subdomain as a custom domain on that service.

```
# API on Railway
Type    Name    Value
CNAME   api     your-service.up.railway.app

# Docs on GitBook
Type    Name    Value
CNAME   docs    your-team.gitbook.io

# Staging on Vercel (different project)
Type    Name    Value
CNAME   staging staging-project.vercel.app
```

Then in each platform's dashboard, add the custom domain (`api.hustlrai.com`, `docs.hustlrai.com`). The platform needs this so it knows to accept requests for that subdomain and provision an SSL certificate.

### Pattern 4: Domain Redirect (www to non-www)

**When to use it:** You want `www.hustlrai.com` to redirect to `hustlrai.com` (or vice versa). You should pick one as canonical and redirect the other.

**How it works on Vercel/Netlify:** Both platforms handle this natively.

```
Vercel:
  1. Add both hustlrai.com and www.hustlrai.com as domains
  2. Vercel shows a "redirect" option for one of them
  3. Select www.hustlrai.com → redirects to hustlrai.com

Netlify:
  1. Primary domain: hustlrai.com
  2. Domain alias: www.hustlrai.com (auto-redirects to primary)
```

**If your platform doesn't handle it:** Use a redirect rule at the DNS or server level. Cloudflare offers "Page Rules" that can redirect `www.hustlrai.com/*` to `hustlrai.com/$1`.

**Why this matters:** Without a redirect, Google sees `www.hustlrai.com` and `hustlrai.com` as two different sites with duplicate content. This hurts SEO. Pick one, redirect the other.

### Pattern 5: Domain Verification with TXT Records

**When to use it:** A service asks you to "verify domain ownership" by adding a DNS record.

**How it works:** The service gives you a unique string. You add it as a TXT record. The service checks DNS for that record. If it finds the string, you've proven you control the domain.

```
# Google Search Console verification
Type    Name    Value
TXT     @       "google-site-verification=abc123xyz789..."

# Stripe domain verification
Type    Name    Value
TXT     @       "stripe-verification=sv_test_abc123..."

# Postmark (email service) verification
Type    Name                          Value
TXT     20230101._domainkey.hustlrai  "v=DKIM1; k=rsa; p=MIGfMA0G..."
```

Some services ask for the TXT record on the root domain (`@`). Others ask for it on a specific subdomain (like `_domainkey`). Follow the exact instructions. The name and value must match precisely.

**You can have multiple TXT records on the same domain.** This is fine and normal. Google verification, Stripe verification, SPF, and DMARC can all coexist as separate TXT records on `@`.

## Mistakes Everyone Makes

### 1. Changing nameservers and wondering why it takes 48 hours

**What people do wrong:** They change nameservers at their registrar and expect the new DNS records to work immediately.

**Why it seems right:** You made the change. It should be instant, like changing a setting.

**What actually happens:** Nameserver changes propagate through a global system. The old nameservers were cached at resolvers worldwide. The TLD servers (.com registry) need to update their records. This genuinely takes up to 48 hours, though most propagation happens within 4-6 hours. During this time, some users hit the old nameservers (which may not have your records anymore) and others hit the new ones.

**What to do instead:** Before changing nameservers, recreate all your existing DNS records at the new provider. This way, both old and new nameservers return the same answers during propagation. Check current records with `dig hustlrai.com ANY @old-nameserver` and make sure every record exists at the new provider. Expect 2-48 hours for full propagation. Don't panic if things are spotty during this window.

### 2. CNAME on the root domain

**What people do wrong:**

```
hustlrai.com    CNAME    my-app.vercel.app    ← This violates DNS specs
```

**Why it seems right:** CNAMEs work fine for `www.hustlrai.com`. Why not for the root?

**What actually happens:** The DNS specification (RFC 1034) says a CNAME cannot coexist with other records at the same name. The root domain needs other records (MX for email, TXT for verification, NS for nameservers). A CNAME would conflict with all of them. Some DNS providers let you try, then your email breaks because the MX records are ignored.

**What to do instead:** Use an A record for the root domain:

```
hustlrai.com    A    76.76.21.21
```

Or, if your DNS provider supports it, use an ALIAS/ANAME record (Cloudflare calls this "CNAME flattening"):

```
hustlrai.com    ALIAS    my-app.vercel.app
```

This resolves the CNAME at the DNS provider level and returns an A record to the client, sidestepping the specification issue. Cloudflare, Route 53, and DNSimple support this. Standard DNS providers may not.

### 3. Forgetting to set up www redirect

**What people do wrong:** They set up `hustlrai.com` and forget about `www.hustlrai.com`. Users who type `www.hustlrai.com` get an error page.

**Why it seems right:** "Who types www anymore?"

**What actually happens:** Plenty of people type www, or they have bookmarks with www, or they click a link that includes www. If you didn't set up a DNS record and redirect for `www`, those users see a DNS resolution error or an SSL error. You lose traffic and look unprofessional.

**What to do instead:** Always set up both:

```
hustlrai.com        A       76.76.21.21
www.hustlrai.com    CNAME   cname.vercel-dns.com
```

Then configure your hosting platform to redirect one to the other. Pick `hustlrai.com` (non-www) as canonical. Most platforms have a one-click option for this.

### 4. SSL certificate errors after domain change

**What people do wrong:** They change their DNS to point to a new platform and immediately see "Your connection is not private" or "NET::ERR_CERT_AUTHORITY_INVALID."

**Why it seems right:** "I pointed the domain, the site should work."

**What actually happens:** SSL certificates are domain-specific. The new platform needs to provision a certificate for your domain, which requires verifying domain ownership via DNS. If DNS hasn't propagated yet, or the records are wrong, the platform can't verify ownership, can't issue a certificate, and browsers refuse the connection because the old certificate (from the previous platform) is either expired or doesn't match.

**What to do instead:** Expect a brief SSL gap when switching platforms. To minimize it:
1. Lower TTL to 300 seconds before switching.
2. Add the domain to the new platform first (it starts trying to provision the certificate).
3. Update DNS records.
4. Wait 5-15 minutes for DNS propagation and certificate issuance.
5. If the certificate doesn't provision after 30 minutes, check your DNS records with `dig` to make sure they point to the new platform.

### 5. Not lowering TTL before making DNS changes

**What people do wrong:** Their TTL is 86400 (24 hours). They make a DNS change. It takes a full day to propagate globally. Meanwhile, some users see the old site and others see the new one.

**Why it seems right:** High TTL is good for performance (fewer DNS lookups). Why change it?

**What actually happens:** The high TTL means DNS resolvers worldwide cache the old value for up to 24 hours. You can't force them to refresh. You just wait.

**What to do instead:** Follow the TTL strategy: lower TTL to 300 (5 min) at least 24 hours before any planned DNS change. After the change propagates and everything works, raise TTL back to 3600 (1 hour). The 24-hour wait before the change is the part everyone skips. You need to wait for the old, high TTL to expire before the new, low TTL takes effect.

### 6. Copying DNS records wrong (trailing dots, missing quotes)

**What people do wrong:** They get a TXT record from a service like `"v=spf1 include:_spf.google.com ~all"` and paste it without the quotes. Or they add a CNAME value without the trailing dot that some providers expect.

**Why it seems right:** "I copied exactly what the instructions said."

**What actually happens:** Some DNS providers auto-add quotes around TXT values, others don't. Some expect the trailing dot on CNAME targets (`cname.vercel-dns.com.`), others add it automatically. If the value is wrong by even one character, the record is technically invalid or doesn't match what the verifying service expects.

**What to do instead:** After adding a record, verify it with `dig`:

```bash
dig hustlrai.com TXT    # Check TXT records
dig www.hustlrai.com CNAME    # Check CNAME
```

Compare the output to what the service expects. If it doesn't match, adjust. Each DNS provider has slightly different UI conventions for entering records. When in doubt, check the provider's documentation for how they handle quotes and trailing dots.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Lower TTL before any DNS change.** Set it to 300 seconds at least 24 hours in advance. This one step prevents most propagation headaches.

2. **Use A records for root domains, CNAME for subdomains.** Don't fight the DNS specification. A records for `yourdomain.com`, CNAME for `www`, `api`, `docs`, and every other subdomain.

3. **Always set up both root and www.** Configure DNS records for both, then redirect one to the other. Which one is canonical doesn't matter much. Just be consistent.

4. **Verify every DNS change with `dig` immediately after making it.** Don't wait and hope. Run `dig yourdomain.com @1.1.1.1` to check if the authoritative answer is correct, even before global propagation.

5. **Keep DNS records at one provider.** Don't split records between your registrar and another DNS provider. Pick one (Cloudflare is the common choice), move everything there, and manage it in one dashboard.

6. **Document your DNS records somewhere outside the DNS provider.** A simple text file or spreadsheet listing every record, its purpose, and when it was added. When something breaks at 2 AM, you need to know why each record exists without reverse-engineering it.

7. **Enable DNSSEC if your registrar and DNS provider support it.** It adds cryptographic verification to DNS responses, preventing cache poisoning attacks. Cloudflare makes this a one-click toggle.

8. **Never let a domain expire accidentally.** Enable auto-renewal. Enable 2FA on your registrar account. A lost domain can be registered by someone else within hours of expiration. Getting it back is expensive and sometimes impossible.

## The "Just Tell Me What to Do" Quickstart

You'll buy a domain, connect it to a Vercel project, set up www redirect, and verify everything works. Total time: under 10 minutes (plus propagation time).

**Prerequisites:** A deployed project on Vercel (even the default Next.js starter), a credit card for the domain purchase.

**Step 1: Buy a domain**

Go to [Cloudflare Registrar](https://dash.cloudflare.com/) (or Namecheap or Porkbun). Search for a cheap domain. `.com` domains are typically $10-12/year. Buy it.

If using Cloudflare: you'll need to add the domain to your Cloudflare account first (free plan), then register/transfer it.

**Step 2: Add the domain to Vercel**

```
Vercel Dashboard > Your Project > Settings > Domains
Add: yourdomain.com
```

Vercel shows the DNS records you need to add.

**Step 3: Add DNS records**

At your DNS provider (Cloudflare, Namecheap, etc.), add:

```
Type    Name    Value               TTL
A       @       76.76.21.21         300
CNAME   www     cname.vercel-dns.com 300
```

(Use the specific values Vercel gives you. They may differ slightly.)

**Step 4: Verify with dig**

```bash
# Check root domain
dig yourdomain.com A

# Expected: 76.76.21.21

# Check www
dig www.yourdomain.com CNAME

# Expected: cname.vercel-dns.com
```

If you don't have `dig` installed, use [whatsmydns.net](https://www.whatsmydns.net) to check from multiple locations.

**Step 5: Configure redirect in Vercel**

Back in Vercel's domain settings, you'll see both `yourdomain.com` and `www.yourdomain.com`. Set `www.yourdomain.com` to redirect to `yourdomain.com` (Vercel offers this as a toggle).

**Step 6: Verify SSL**

Visit `https://yourdomain.com` in your browser. You should see the padlock icon. Vercel provisions the certificate automatically once DNS is verified. If you see an SSL error, wait 5-10 minutes and try again.

**Step 7: Verify redirect**

```bash
curl -I https://www.yourdomain.com
```

You should see a `301` or `308` redirect to `https://yourdomain.com`.

**Step 8: Raise TTL**

Once everything works, go back to your DNS provider and change the TTL on both records from 300 to 3600 (1 hour).

That's it. Your site is on a custom domain with SSL, www redirect, and proper DNS configuration.

## How to Prompt AI Tools About This

### Context to provide

When asking Claude, Cursor, or Copilot about DNS and domains, always specify:
- Your registrar ("Cloudflare," "Namecheap")
- Your DNS provider if different from registrar
- Your hosting platform ("Vercel," "Netlify," "Railway")
- What you're trying to do ("connect custom domain," "set up email," "add subdomain")
- Any error messages you're seeing

### Example prompts that produce good results

**Connecting a domain:**
> "I bought hustlrai.com on Cloudflare. My Next.js app is on Vercel. What DNS records do I need to add to connect the domain? Include both root and www, and the redirect setup."

**Email setup:**
> "I have hustlrai.com on Cloudflare DNS. I want to set up Google Workspace email. Give me the exact MX, SPF, DKIM, and DMARC records I need to add, in the format Cloudflare expects."

**Debugging DNS:**
> "I added a CNAME for api.hustlrai.com pointing to my-service.up.railway.app 2 hours ago. `dig api.hustlrai.com` still returns NXDOMAIN. My registrar is Namecheap, DNS is on Cloudflare. What could be wrong?"

**Subdomain setup:**
> "I need three subdomains: api.hustlrai.com → Railway, docs.hustlrai.com → GitBook, staging.hustlrai.com → separate Vercel project. Give me all the DNS records and the steps to configure each platform."

### What to watch out for in AI-generated DNS advice

- **Suggesting CNAME on root domain.** AI frequently recommends `hustlrai.com CNAME my-app.vercel.app`. This violates DNS specs. Always use an A record (or ALIAS if your provider supports it) for the root domain.
- **Outdated IP addresses.** Platform IPs change. Don't blindly use the IPs AI gives you. Check the platform's current documentation for the correct values.
- **Missing www configuration.** AI often only configures the root domain and forgets www (or vice versa). Always ask for both.
- **Generic TTL values.** AI defaults to 3600 or "auto." For changes, you want 300. Ask specifically about TTL strategy if you're migrating.
- **Ignoring propagation.** AI will say "add the record and it's done." It won't mention the waiting period or the TTL strategy. Add that context yourself.

### Key terms to use in prompts

Using these terms gets better output: "A record," "CNAME," "root domain vs subdomain," "TTL," "nameserver," "propagation," "SSL certificate," "ALIAS record," "MX record," "SPF," "DKIM," "domain verification."

## Ship It: Build This

Buy a domain, connect it to a live project, set up subdomains, and verify everything with command-line tools.

**What you're building:** A complete domain setup for a project with three endpoints: the main app on the root domain, a docs subdomain, and an API subdomain. You'll configure DNS records, SSL, www redirect, and email verification records. You'll use `dig` and `nslookup` to verify every step.

**Why this project specifically:** It exercises every DNS concept that matters: root domain A records, subdomain CNAMEs, www redirect, TXT verification records, TTL management, and troubleshooting with `dig`. After this, you'll never be confused by DNS again.

**Rough architecture:**

- Buy a cheap domain ($2-12/year for a .com or alternative TLD)
- DNS managed at one provider (Cloudflare recommended)
- Root domain (yourdomain.com) pointing to Vercel via A record
- www redirect to root domain
- `docs.yourdomain.com` subdomain pointing to a docs platform (or second Vercel project)
- `api.yourdomain.com` subdomain pointing to a backend service (or another Vercel project)
- TXT record for domain verification (Google Search Console or similar)
- SSL verified on all endpoints
- All records verified with `dig` from the command line
- A text file documenting every DNS record, its purpose, and its TTL

**Estimated time:** 1-2 hours for a vibe coder using AI tools (plus up to 24 hours of propagation waiting if nameservers change).

## Go Deeper (Optional)

- **Learn DNSSEC** when you need to protect your domain from DNS cache poisoning and spoofing attacks, especially for applications handling financial data or sensitive user information.
- **Learn Cloudflare Workers and edge routing** when you need to make routing decisions (A/B tests, geo-redirects, header manipulation) at the DNS/CDN layer before traffic reaches your server.
- **Learn email authentication in depth (SPF, DKIM, DMARC)** when your transactional emails are going to spam or when you're sending email at scale and need to protect your domain's reputation.
- **Learn anycast and global DNS architecture** when you're operating at scale and need to understand why some DNS providers (Cloudflare, Route 53) are faster than others and how global routing works.
- **Learn domain transfer and registrar migration** when you need to move a domain between registrars without downtime, which involves understanding transfer locks, auth codes, and the ICANN transfer process.
