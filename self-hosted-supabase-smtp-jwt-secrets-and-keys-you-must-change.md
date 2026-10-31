# Self-hosted Supabase: SMTP, JWT secrets and keys you must change

The self-hosted Supabase Docker setup starts quickly because it ships with example values. Several of them must be changed before real users sign up. Leaving them in place is the most common security mistake in self-hosted Supabase.

## JWT secret and API keys

The `.env` file includes a `JWT_SECRET` plus `ANON_KEY` and `SERVICE_ROLE_KEY`, which are JWTs signed with that secret. The example values are public. Anyone who knows them can mint a service role token and bypass row level security.

Generate a new random secret of at least 32 characters, then generate new anon and service role keys signed with it. Supabase's self-hosting docs include a generator. Update all three together; keys signed with the old secret stop working.

Keep the service role key on the server side only. It should never appear in frontend code or mobile apps.

## Database password

Change `POSTGRES_PASSWORD` before the first start. Changing it after the database is initialized requires updating the roles inside Postgres too, not just the env file.

## Dashboard login

Studio is protected by `DASHBOARD_USERNAME` and `DASHBOARD_PASSWORD` through the API gateway. Set strong values, and consider restricting the dashboard to your IP or VPN.

## SMTP for auth emails

Without SMTP settings, sign-up confirmations, magic links and password resets do not arrive, or are limited to testing. Configure:

```
SMTP_HOST=smtp.provider.com
SMTP_PORT=587
SMTP_USER=...
SMTP_PASS=...
SMTP_ADMIN_EMAIL=no-reply@example.com
SMTP_SENDER_NAME=Example
```

Use a transactional email provider and set up SPF, DKIM and DMARC for the sending domain. Test with Gmail and Outlook addresses.

## Site URL and redirects

Set `SITE_URL` to your real frontend URL and list allowed redirect URLs in `ADDITIONAL_REDIRECT_URLS`. Wrong values break OAuth and magic links.

## Storage backend

By default, Storage writes files to a local volume. For anything beyond testing, back that volume up or point Storage at S3-compatible object storage.

## Before launch

- No example values left in `.env`
- Ports for Postgres and internal services not published publicly
- Backups of the database and storage tested
- HTTPS in front of the API gateway

Sizing, optional services and updates are covered in this guide to [self-hosting Supabase on a VPS](https://peon.sh/blogs/self-host-supabase-vps).

Disclosure: I work on [Peon](https://peon.sh/).
