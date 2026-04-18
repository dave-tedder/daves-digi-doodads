# Railway Custom Domain Setup

Custom domains on Railway require three things, not just a CNAME. Missing any one of them results in "Application not found" or "Waiting for DNS update" indefinitely.

## Required DNS Records

1. **CNAME** - Points the subdomain to the Railway-provided target (e.g., `h0ygqcwl.up.railway.app`)
2. **TXT** - Verification record. Name is `_railway-verify.[subdomain]`, value is provided in Railway's domain settings under "Show DNS records." Without this, Railway won't issue the SSL cert or route traffic.

## Required Railway Configuration

3. **Port** - Must be explicitly set in the domain settings (Edit Port). Railway doesn't auto-detect the port from the running service. Without it, the edge returns "Application not found" even though the service is healthy.

## Process

1. Add the custom domain in Railway (CLI: `railway domain [domain]` or dashboard)
2. Click "Show DNS records" in Railway domain settings to get both the CNAME target and TXT verification value
3. Add both records in your DNS provider (GoDaddy, Cloudflare, etc.)
4. Set the port in Railway domain settings
5. SSL cert provisioning takes a few minutes after DNS propagates

## GoDaddy-Specific

GoDaddy DNS propagation is typically a few minutes. No proxy setting (that's Cloudflare-only). Default TTL is fine.
