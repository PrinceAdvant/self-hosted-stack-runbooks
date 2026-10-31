# n8n queue mode: scaling executions with Redis and workers

A single n8n container runs the editor, receives webhooks and executes workflows in one process. That is perfect for light use. When workflows get heavy or numerous, one slow execution can make the editor sluggish and webhooks time out. Queue mode splits the work across processes.

## Signs you need it

- The editor becomes slow while workflows run
- Webhook callers report timeouts during busy periods
- CPU on the n8n container sits near its limit
- You want to restart or upgrade without dropping executions

If none of these apply, stay in regular mode. Queue mode adds moving parts.

## The architecture

- **Main process**: editor UI, API and triggers
- **Redis**: the queue of executions
- **Workers**: separate n8n processes started with `n8n worker` that pull jobs and run them
- **Postgres**: required, since SQLite cannot be shared between processes

Optionally, dedicated **webhook processes** receive production webhook calls, keeping the main process free for the UI.

## Key settings

All processes share these environment variables:

```
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=redis
DB_TYPE=postgresdb
N8N_ENCRYPTION_KEY=<same value everywhere>
```

The encryption key must be identical across main and workers, or workers cannot decrypt credentials. This is the most common setup mistake.

## Worker concurrency

Each worker runs several executions in parallel, controlled by `--concurrency`, default 10. For CPU-heavy workflows, lower it and add workers; for workflows that mostly wait on APIs, the default is fine.

## Redis settings

Use a Redis instance with `noeviction` and persistence enabled. An evicted job key is a lost execution.

## Binary data

Workflows handling files keep binary data in memory by default. In queue mode, workers and the main process need to see the same data, so configure an external store, such as S3-compatible storage, if workflows pass files between nodes.

## Upgrades

Upgrade main and workers to the same version together. Mixed versions can fail on changed node definitions.

## Monitoring

Watch the queue length in Redis and execution failures in n8n. A growing queue means you need more workers or slower triggers.

For the base installation, backups and reverse proxy setup, start with this guide to [self-hosting n8n](https://peon.sh/blogs/self-host-n8n-automation).

Disclosure: I work on [Peon](https://peon.sh/).
