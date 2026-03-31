# Domains & SSL

Vercel handles custom domains and SSL certificates automatically. Add a domain, update DNS, and you're live with HTTPS.

## Adding a Custom Domain

### Via Dashboard
1. Go to **Project Settings → Domains**
2. Enter your domain (e.g., `yourdomain.com`)
3. Vercel shows the required DNS records
4. Update your DNS provider
5. Vercel auto-provisions an SSL certificate

### Via CLI

```bash
# Add a domain
vercel domains add yourdomain.com

# Inspect domain (shows required DNS config)
vercel domains inspect yourdomain.com

# List all domains
vercel domains ls

# Remove a domain
vercel domains rm yourdomain.com
```

## DNS Configuration

### Apex Domain (yourdomain.com)

Set an **A record** pointing to Vercel's IP:

| Type | Name | Value           |
|------|------|-----------------|
| A    | @    | `76.76.21.21`   |

### Subdomain (www.yourdomain.com or app.yourdomain.com)

Set a **CNAME record** pointing to Vercel:

| Type  | Name | Value                  |
|-------|------|------------------------|
| CNAME | www  | `cname.vercel-dns.com` |
| CNAME | app  | `cname.vercel-dns.com` |

### Both apex + www (recommended)

Set up both records, then configure one to redirect to the other in your Vercel project settings.

## SSL Certificates

- **Automatic:** Vercel provisions and renews SSL certificates for all domains at no cost
- **No configuration needed** — just add the domain and DNS records
- Certificates are issued once DNS propagation completes (minutes to hours)
- Supports wildcard certificates for subdomains

## Domain Aliases

Assign a specific deployment to a domain:

```bash
# Alias a preview deployment to a custom domain
vercel alias <deployment-url> staging.yourdomain.com

# List all aliases
vercel alias ls

# Remove an alias
vercel alias rm staging.yourdomain.com
```

## Wildcard Domains

For multi-tenant apps where each tenant gets a subdomain (e.g., `tenant1.yourdomain.com`):

1. Add `*.yourdomain.com` as a domain in project settings
2. Set a wildcard CNAME: `* → cname.vercel-dns.com`
3. Handle routing in your Next.js middleware/proxy

## Vercel DNS (Nameserver Delegation)

For the simplest setup, delegate your entire domain to Vercel's nameservers:

1. In Vercel Dashboard → Domains, add your domain
2. Choose **Vercel DNS** as the provider
3. Update your registrar's nameservers to the ones Vercel provides
4. Vercel handles all DNS records from that point

## Troubleshooting

| Issue                          | Solution                                                |
|--------------------------------|---------------------------------------------------------|
| Domain shows "Invalid Config"  | DNS records haven't propagated. Wait 15 min–48 hours.   |
| SSL certificate not issued     | DNS must resolve correctly first. Check with `nslookup`. |
| Wrong deployment on domain     | Check which project the domain is assigned to.           |
| Redirect loop                  | Check for conflicting redirects in `next.config` and DNS.|

Verify DNS propagation:
```bash
# Check A record
nslookup yourdomain.com

# Check CNAME
nslookup www.yourdomain.com

# Or use dig
dig yourdomain.com +short
```