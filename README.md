# dns-blocklist

A small personal DNS blocklist for a home network running pfSense with pfBlockerNG (DNSBL, Python mode).

It is meant to be used **in addition to** these upstream lists, not instead of them:

- [oisd big](https://oisd.nl/)
- [HaGeZi Threat Intelligence Feeds](https://github.com/hagezi/dns-blocklists#tif)
- [HaGeZi native tracker lists](https://github.com/hagezi/dns-blocklists#native): Windows/Office, Apple, Amazon, Samsung, OPPO/Realme
- [hkamran80 smart-tv](https://github.com/hkamran80/blocklists)

Everything here is telemetry, analytics, ad attribution or crash reporting that showed up in our own DNS logs and got past all of the lists above. It is not a general-purpose list and is not trying to be complete.

## Files

| File | What it is |
|---|---|
| `blocklist.txt` | Domain feed, one name per line, `#` starts a comment. pfBlockerNG fetches it daily from the raw URL. |
| `regex.txt` | Patterns for names a plain list cannot express (every subdomain under a name that is itself a subdomain). pfBlockerNG cannot fetch regex from a URL, so this file is the source of truth and is copied by hand into DNSBL → Python Group Policy → Regex. Format: `pattern # Name`. |

Raw feed URL:

```
https://raw.githubusercontent.com/yuliwilliam/dns-blocklist/main/blocklist.txt
```

## How pfBlockerNG treats an entry

pfBlockerNG decides this per name, not per list:

- A registrable domain (`example.com`, `example.co.uk`) blocks the domain **and all of its subdomains**.
- Anything deeper (`metrics.example.com`) blocks **only that exact name**. `a.metrics.example.com` still resolves.

So a name goes into `blocklist.txt` when blocking the whole registrable domain is fine, or when the exact name is all that needs blocking. When every subdomain under a deeper name has to go (random per-app or per-region prefixes, for example), it goes into `regex.txt`.

## What is deliberately not here

Some names look like tracking but are left alone because blocking them breaks something people use:

- `app.adjust.com`: click-through and deep-link redirects. Only `analytics.adjust.com` is blocked.
- `is.snssdk.com` and the rest of `snssdk.com`: ByteDance app APIs. Only the `log*` hosts are blocked.

## Adding an entry

1. Confirm the name is not already blocked by an upstream list (it reached the internet in the DNS reply log).
2. Check what else lives under the same domain before blocking a whole registrable domain.
3. Add it with a short comment saying what it is. Group entries by vendor.
4. After pfBlockerNG's next update (or a manual Force Reload of DNSBL), flush the name from the Unbound cache and check that it now resolves to the DNSBL VIP.
