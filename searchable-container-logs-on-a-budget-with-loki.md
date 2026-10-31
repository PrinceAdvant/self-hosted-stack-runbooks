# Searchable container logs on a budget with Loki

`docker logs` is fine for the last few minutes of one container. For questions like "when did these errors start across all services," you need logs in one searchable place. Grafana Loki is light enough to run next to your apps on a small server.

## Why Loki fits small setups

Loki indexes only labels, such as container name and service, not the full text. That keeps storage and memory low. Full-text searches still work by scanning the chunks for a time range, which is fast enough for typical small-team volumes.

## The pieces

- **Loki** stores logs
- **An agent** reads container logs and ships them; Grafana Alloy is the current recommendation, and Promtail is now deprecated
- **Grafana** queries and displays them

A minimal Compose file runs all three. Give Loki a volume for its data and a memory limit around 512 MB to start.

## Labels: keep them few

Good labels: `service`, `environment`, `host`. Bad labels: request IDs, user IDs, anything with many values. High-cardinality labels create many tiny streams and slow Loki down. Put those values in the log line and filter with queries instead.

## Retention

Set retention in Loki's config so disk use stays predictable:

```
limits_config:
  retention_period: 336h
compactor:
  retention_enabled: true
```

Fourteen days is a reasonable start. Estimate disk as daily log volume times days times about 0.2, since Loki compresses well.

## Keep Docker's own limits too

Loki does not replace local log rotation. Keep the `json-file` driver with `max-size` and `max-file`, or the `local` driver, so a chatty container cannot fill the disk if the agent falls behind.

## Useful queries

```
{service="api"} |= "error"
sum by (service) (count_over_time({environment="prod"} |= "timeout" [5m]))
```

The second one turns logs into a quick chart of timeouts per service.

## Log in JSON

Structured JSON logs make filtering far easier with `| json | status >= 500`. Most frameworks have a JSON formatter; enable it in production.

## Protect it

Grafana and Loki should not be public. Put them behind authentication or a VPN, since logs often contain emails and IPs.

Before adding any of this, make sure the basics are right. These [Docker logging best practices](https://peon.sh/blogs/docker-logs-best-practices) cover drivers, rotation and what to log.

Disclosure: I work on [Peon](https://peon.sh/).
