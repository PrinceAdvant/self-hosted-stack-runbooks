# Moving Fly.io volumes and Postgres to a VPS

Fly.io makes it easy to attach volumes and run Postgres close to your app. Moving away means getting that data out cleanly. Compute is easy to recreate; volumes and databases need a plan.

## Inventory first

```
fly volumes list -a myapp
fly apps list
```

Note every volume, its size and the app that mounts it. Check whether your Postgres is the older unmanaged Fly Postgres app or the newer managed offering, since the export path differs.

## Postgres: dump over a proxy

For unmanaged Fly Postgres, open a local tunnel and dump:

```
fly proxy 15432:5432 -a myapp-db
pg_dump -h localhost -p 15432 -U postgres -Fc -d myapp > myapp.dump
```

For managed Postgres, use the connection string from the dashboard or CLI with `pg_dump`. Time the dump and a restore on the new server; that is your downtime estimate.

## Volumes: copy files out

Volumes attach to a single machine, so copy from inside it. One approach is to SSH in and stream a tar archive:

```
fly ssh console -a myapp -C "tar czf - /data" > data.tar.gz
```

For larger volumes, run `rsync` or an object storage client from inside the machine to push data to S3-compatible storage, then pull it on the VPS. Check that the archive is complete by comparing file counts and sizes.

## Prepare the VPS

- Same Postgres major version or newer, with extensions installed
- Volume paths matching what the app expects, for example `/data`
- Secrets copied from `fly secrets list` names; values must be re-entered, since Fly does not display them

## Cutover

1. Scale the Fly app down or enable a maintenance page
2. Take a final dump and volume copy
3. Restore on the VPS and start the app
4. Test with `curl --resolve` against the new IP
5. Switch DNS, with TTLs lowered in advance

## Regions

If you ran in several Fly regions, decide whether you really need that. Most small apps are well served by one location near their users, plus a CDN for static assets.

## Clean up

Keep the Fly resources for a few days for rollback, then destroy volumes and apps to stop billing. Volumes are billed even when machines are stopped.

For comparing where to move, including self-hosted options, see this roundup of [Fly.io alternatives](https://peon.sh/blogs/fly-io-alternative).

Disclosure: I help build [Peon](https://peon.sh/).
