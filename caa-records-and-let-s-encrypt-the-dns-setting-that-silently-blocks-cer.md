# CAA records and Let's Encrypt: the DNS setting that silently blocks certificates

You set up a reverse proxy, DNS points at the server, port 80 is open, and the certificate still fails. One cause that rarely appears in tutorials is a CAA record. It is a DNS record that tells certificate authorities which of them may issue certificates for a domain.

## What CAA does

A CAA record looks like this:

```
example.com.  CAA 0 issue "digicert.com"
```

That record says only DigiCert may issue certificates for `example.com` and its subdomains. Let's Encrypt checks CAA before issuing, sees it is not listed, and refuses. The error usually mentions `CAA record for example.com prevents issuance`.

If a domain has no CAA records at all, any authority may issue. That is the default for most domains.

## Why it appears out of nowhere

- A previous team or agency bought a certificate from another authority and added CAA for it
- A security scanner recommended adding CAA, and someone listed only the current provider
- A DNS provider or CDN added records automatically
- A parent domain has CAA, which applies to subdomains too

## Check it

```
dig +short CAA example.com
dig +short CAA app.example.com
```

Check the exact name and every parent up to the registered domain. Let's Encrypt walks up the tree and uses the first CAA set it finds.

## Fix it

Add Let's Encrypt to the allowed list, keeping existing entries if they are still needed:

```
example.com.  CAA 0 issue "letsencrypt.org"
example.com.  CAA 0 issue "digicert.com"
```

For wildcard certificates, `issuewild` records apply. If `issuewild` exists and does not list Let's Encrypt, wildcard issuance fails even when `issue` allows it.

Optionally add an `iodef` record with an email address to receive reports about refused requests.

## Then retry carefully

DNS changes need time to propagate to the resolvers Let's Encrypt uses. Wait for the record TTL to pass, then trigger a single retry. Repeated failed attempts count toward rate limits, so test against the staging endpoint if you are unsure.

## Prevent it

Document CAA records alongside your other DNS records, and include Let's Encrypt whenever your stack relies on automatic HTTPS. If you use more than one authority, list all of them deliberately.

CAA is only one cause. The full troubleshooting order, from DNS to firewall to rate limits, is in this guide to a [Let's Encrypt certificate not issued](https://peon.sh/blogs/lets-encrypt-certificate-not-issued).

Disclosure: I work on [Peon](https://peon.sh/).
