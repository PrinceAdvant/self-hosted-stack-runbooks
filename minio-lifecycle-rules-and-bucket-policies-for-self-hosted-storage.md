# MinIO lifecycle rules and bucket policies for self-hosted storage

A self-hosted MinIO server tends to grow forever: old backups, abandoned uploads, temporary exports. Lifecycle rules and bucket policies keep it tidy and safe. Both use the same S3 concepts you would use on AWS, managed with the `mc` client.

## Set up the client

```
mc alias set local https://s3.example.com ACCESS_KEY SECRET_KEY
```

Use a dedicated admin user for this, not the root credentials baked into the container.

## Expire old objects

Delete objects in a `tmp/` prefix after seven days:

```
mc ilm rule add local/exports --prefix "tmp/" --expire-days 7
```

For a backup bucket, keep 30 days:

```
mc ilm rule add local/backups --expire-days 30
```

List rules with `mc ilm rule ls local/backups`. Lifecycle processing runs in the background, so objects disappear some time after they become eligible, not at an exact moment.

## Versioning and noncurrent versions

If versioning is on, deleting an object only adds a delete marker and the old versions remain. Add a rule for noncurrent versions or storage keeps growing:

```
mc ilm rule add local/backups --noncurrent-expire-days 14
```

## Clean up failed uploads

Interrupted multipart uploads leave parts behind. Recent MinIO releases clean stale uploads automatically after a default period; check your version's documentation and the `api` configuration if you see unexplained growth.

## One user per application

Create a user per app and attach a policy that allows only its bucket:

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject","s3:PutObject","s3:DeleteObject","s3:ListBucket"],
    "Resource": ["arn:aws:s3:::app-uploads","arn:aws:s3:::app-uploads/*"]
  }]
}
```

Then `mc admin policy create local app-uploads policy.json` and attach it to the user. A leaked key now exposes one bucket, not all of them.

## Public files, carefully

If an app serves public assets, allow anonymous read on a specific prefix only:

```
mc anonymous set download local/app-uploads/public
```

Never set public access on a whole bucket that also holds private files.

## Watch capacity

Check `mc admin info local` for disk usage and set an alert before disks pass 80%. Erasure-coded setups need free space to heal.

Installation, TLS and sizing are covered in this guide to [self-hosting MinIO](https://peon.sh/blogs/self-host-minio-object-storage).

Disclosure: I help build [Peon](https://peon.sh/).
