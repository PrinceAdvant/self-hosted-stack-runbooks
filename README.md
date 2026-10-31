# Self-hosted stack runbooks

Runbooks for the services that sit around a self-hosted app: certificates, background workers, logs, object storage, automation, auth backends and data migrations.

- [CAA records and Let's Encrypt: the DNS setting that silently blocks certificates](caa-records-and-let-s-encrypt-the-dns-setting-that-silently-blocks-cer.md): How CAA DNS records work, how they block Let's Encrypt issuance, and how to fix them.
- [Running Celery workers and beat for Django in Docker](running-celery-workers-and-beat-for-django-in-docker.md): A Compose layout for Django with Celery workers and beat, plus settings that prevent lost or duplicate tasks.
- [Searchable container logs on a budget with Loki](searchable-container-logs-on-a-budget-with-loki.md): A small Grafana Loki setup that keeps container logs searchable for weeks on a single VPS.
- [MinIO lifecycle rules and bucket policies for self-hosted storage](minio-lifecycle-rules-and-bucket-policies-for-self-hosted-storage.md): Using lifecycle rules to expire old objects and bucket policies to limit access in a self-hosted MinIO setup.
- [n8n queue mode: scaling executions with Redis and workers](n8n-queue-mode-scaling-executions-with-redis-and-workers.md): When to switch n8n from the default regular mode to queue mode, and the settings that matter.
- [Self-hosted Supabase: SMTP, JWT secrets and keys you must change](self-hosted-supabase-smtp-jwt-secrets-and-keys-you-must-change.md): The configuration values a self-hosted Supabase stack ships with that must be replaced before production.
- [Moving Fly.io volumes and Postgres to a VPS](moving-fly-io-volumes-and-postgres-to-a-vps.md): How to get data out of Fly.io volumes and Fly Postgres and onto a VPS with a short cutover.

Corrections and improvements are welcome through issues or pull requests.
