# Bucket Lifecycle Configuration Quickstart Guide [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/minio.svg?maxAge=604800)](https://hub.docker.com/r/minio/minio/)

Enable object lifecycle configuration on buckets to setup automatic deletion of objects after a specified number of days or a specified date.

## 1. Prerequisites
- Install MinIO - [MinIO Quickstart Guide](https://docs.min.io/docs/minio-quickstart-guide).
- Install `mc` - [mc Quickstart Guide](https://docs.minio.io/docs/minio-client-quickstart-guide.html)

## 2. Enable bucket lifecycle configuration

- Create a bucket lifecycle configuration which expires the objects under the prefix `old/` on `2020-01-01T00:00:00.000Z` date and the objects under `temp/` after 7 days.
- Enable bucket lifecycle configuration using `mc`:

```sh
$ mc ilm import play/testbucket <<EOF
{
    "Rules": [
        {
            "Expiration": {
                "Date": "2020-01-01T00:00:00.000Z"
            },
            "ID": "OldPictures",
            "Filter": {
                "Prefix": "old/"
            },
            "Status": "Enabled"
        },
        {
            "Expiration": {
                "Days": 7
            },
            "ID": "TempUploads",
            "Filter": {
                "Prefix": "temp/"
            },
            "Status": "Enabled"
        }
    ]
}
EOF
```

```
Lifecycle configuration imported successfully to `play/testbucket`.
```

- List the current settings
```
$ mc ilm ls play/testbucket
     ID     |  Prefix  |  Enabled   | Expiry |  Date/Days   |  Transition  |    Date/Days     |  Storage-Class   |       Tags
------------|----------|------------|--------|--------------|--------------|------------------|------------------|------------------
OldPictures |   old/   |    ✓       |  ✓     |  1 Jan 2020  |     ✗        |                  |                  |
------------|----------|------------|--------|--------------|--------------|------------------|------------------|------------------
TempUploads |  temp/   |    ✓       |  ✓     |   7 day(s)   |     ✗        |                  |                  |
------------|----------|------------|--------|--------------|--------------|------------------|------------------|------------------
```

## 3. Activate ILM versioning features

This will only work with a versioned bucket, take a look at [Bucket Versioning Guide](https://docs.min.io/docs/minio-bucket-versioning-guide.html) for more understanding.

### 3.1 Automatic removal of non current objects versions

A non-current object version is a version which is not the latest for a given object. It is possible to set up an automatic removal of non-current versions when a version becomes older than a given number of days.

e.g., To scan objects stored under `user-uploads/` prefix and remove versions older than one year.
```
{
    "Rules": [
        {
            "ID": "Removing all old versions",
            "Filter": {
                "Prefix": "users-uploads/"
            },
            "NoncurrentVersionExpiration": {
                "NoncurrentDays": 365
            },
            "Status": "Enabled"
        }
    ]
}
```

### 3.2 Automatic removal of delete markers with no other versions

When an object has only one version as a delete marker, the latter can be automatically removed after a certain number of days using the following configuration:

```
{
    "Rules": [
        {
            "ID": "Removing all old versions",
            "Expiration": {
                "Days": 1,
                "ExpiredObjectDeleteMarker": true
            },
            "Status": "Enabled"
        }
    ]
}
```

## Explore Further
- [MinIO | Golang Client API Reference](https://docs.min.io/docs/golang-client-api-reference.html#SetBucketLifecycle)
- [Object Lifecycle Management](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html)
