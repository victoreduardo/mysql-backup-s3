# mysql-backup-s3

[![Docker Image Version (latest by date)](https://img.shields.io/docker/v/victoreduardoss/mysql-backup-s3?sort=semver)](https://hub.docker.com/r/victoreduardoss/mysql-backup-s3) ![Docker Image Size (tag)](https://img.shields.io/docker/image-size/victoreduardoss/mysql-backup-s3/latest) ![Docker Pulls](https://img.shields.io/docker/pulls/victoreduardoss/mysql-backup-s3)

Periodicaly backup MySQL to S3.

## Usage

```sh
$ docker run -e SCHEDULE="0 0 * * *" -e S3_ACCESS_KEY_ID=key -e S3_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket -e S3_PREFIX=backup -e MYSQL_USER=user -e MYSQL_PASSWORD=password -e MYSQL_HOST=localhost -e MYSQL_DATABASE=my-db victoreduardoss/mysql-backup-s3
```

docker-compose:

```yaml
services:
  mysql_backup:
    image: victoreduardoss/mysql-backup-s3
    environment:
    SCHEDULE: 30 13 * * * # every day at 13:30
    S3_PREFIX: mysql
    MYSQLDUMP_DATABASE: my-db
    MYSQL_HOST: localhost
    MYSQL_USER: user
    MYSQL_PASSWORD: password
    S3_ACCESS_KEY_ID: key
    S3_SECRET_ACCESS_KEY: secret
    S3_BUCKET: my-bucket
```

## Manual backup

With the container already running (same environment variables as usual), trigger one backup immediately:

```sh
docker exec <container_name_or_id> sh /backup.sh
```

With Docker Compose (replace `mysql_backup` with your service name):

```sh
docker compose exec mysql_backup sh /backup.sh
```

The script uses the container’s existing environment; you do not need to pass `-e` again unless you override them for that one run (e.g. `docker exec -e MYSQLDUMP_DATABASE=other_db …`).

## Environment variables

- `SCHEDULE` crontab-like syntax to schedule your backups
- `SUCCESS_WEBHOOK` url to notify on success
- `MYSQLDUMP_OPTIONS` mysqldump options (default: --quote-names --quick --add-drop-table --add-locks --allow-keywords --disable-keys --extended-insert --single-transaction --create-options --comments --net_buffer_length=16384)
- `MYSQLDUMP_DATABASE` list of databases you want to backup (default: --all-databases)
- `MYSQL_HOST` the mysql host _required_
- `MYSQL_PORT` the mysql port (default: 3306)
- `MYSQL_USER` the mysql user _required_
- `MYSQL_PASSWORD` the mysql password _required_
- `S3_ACCESS_KEY_ID` your AWS access key _required_
- `S3_SECRET_ACCESS_KEY` your AWS secret key _required_
- `S3_BUCKET` your AWS S3 bucket path _required_
- `S3_PREFIX` path prefix in your bucket (default: 'backup')
- `S3_FILENAME` a consistent filename to overwrite with your backup. If not set will use a timestamp.
- `S3_REGION` the AWS S3 bucket region (default: us-west-1)
- `S3_ENDPOINT` the AWS Endpoint URL, for S3 Compliant APIs such as [minio](https://minio.io) (default: none).
- `DELETE_OLDER_THAN` delete old backups, see explanation and warning below.

- `S3_S3V4` set to `yes` to enable AWS Signature Version 4, required for [minio](https://minio.io) servers (default: no)
- `MULTI_DATABASES` Allow to have one file per database if set `yes` default: no)

## Delete Old Backups

You can additionally set the DELETE_OLDER_THAN environment variable like -e DELETE_OLDER_THAN="30 days ago" to delete old backups.

WARNING: this will delete all files in the S3_PREFIX path, not just those created by this script.

---

This project was originally forked from [schickling/dockerfiles](https://github.com/schickling/dockerfiles/tree/master/mysql-backup-s3).
