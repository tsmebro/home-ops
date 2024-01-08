CloudNative Postgres

CloudNative Postgres uses an Operator to watch for new database cluster- and backup- definitions as provided by CRDs.
Operator

The operator is deployed via helm release
Cluster & Backups

S3-compatible storage can be used as a live WAL backup. Since the native PVC/PV is using k8s-local local-path, the backup should use NAS-based s3.

An example of a db deployment can be found in _example-db
Restore

k8s-at-home discussion

    On versioning: If you end up with an older cloudnative pg backup (i.e. i have a pg14 one) but you're using a newer operator that defaults to newer (i.e. > 15.1), you wont be able to restore as it will whine about versions. Add an image tag to the cluster spec to deploy a second cluster with a specifically downgraded ver to restore without having to juggle around changing the operator helm etc

CLI

Install the kubectl plugin


# cloudnative-pg

## S3 Configuration

1. Create `~/.mc/config.json`

    ```json
    {
        "version": "10",
        "aliases": {
            "minio": {
                "url": "https://s3.<domain>",
                "accessKey": "<access-key>",
                "secretKey": "<secret-key>",
                "api": "S3v4",
                "path": "auto"
            }
        }
    }
    ```

2. Create the outline user and password

    ```sh
    mc admin user add minio postgresql <super-secret-password>
    ```

3. Create the outline bucket

    ```sh
    mc mb minio/postgresql
    ```

4. Create `postgresql-user-policy.json`

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:ListBucket",
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Effect": "Allow",
                "Resource": ["arn:aws:s3:::postgresql/*", "arn:aws:s3:::postgresql"],
                "Sid": ""
            }
        ]
    }
    ```

5. Apply the bucket policies

    ```sh
    mc admin policy add minio postgresql-private postgresql-user-policy.json
    ```

6. Associate private policy with the user

    ```sh
    mc admin policy set minio postgresql-private user=postgresql
    ```
