---
date: 2017-08-29T00:00:00+00:00
title: AWS S3 Cache
author: drone-plugins
tags: [ cache, amazon, aws, s3 ]
logo: amazon_s3.svg
repo: drone-plugins/drone-s3-cache
image: plugins/s3-cache
---

The S3 cache plugin can be used to preserve files and directories between builds. The below pipeline configuration demonstrates simple usage:

```yaml
kind: pipeline
name: default

steps:
- name: restore
  image: plugins/s3-cache
  settings:
    pull: true
    endpoint: http://minio.company.com
    access_key: myaccesskey
    secret_key: supersecretKey
    restore: true

- name: build
  image: node
  commands:
    - npm install

- name: rebuild
  image: plugins/s3-cache
  settings:
    pull: true
    endpoint: http://minio.company.com
    access_key: myaccesskey
    secret_key: supersecretKey
    rebuild: true
    mount:
      - node_modules
    when:
      event: push

- name: flush
  image: plugins/s3-cache
  settings:
    pull: true
    endpoint: http://minio.company.com
    access_key: myaccesskey
    secret_key: supersecretKey
    flush: true
    flush_age: 14
```

Example configuration using credentials from secrets:

```yaml
kind: pipeline
name: default

steps:
- name: restore
  image: plugins/s3-cache
  settings:
    pull: true
    endpoint: http://minio.company.com
    access_key:
      from_secret: aws_access_key_id
    secret_key:
      from_secret: aws_secret_access_key
    restore: true
```

Example configuration using file credentials:

```yaml
kind: pipeline
name: default

steps:
- name: restore
  image: plugins/s3-cache
  settings:
    pull: true
    root: my-bucket
    file_credentials: /root/.aws/credentials
    region: us-east-1
    restore: true
```

# Parameter Reference

endpoint
: custom endpoint URL (optional, to use a S3 compatible non-Amazon service)

access_key
: amazon access key (optional)

secret_key
: amazon secret key (optional)

file_credentials
: absolute path to amazon credentials file (optional)

profile
: amazon profile used to resolve credentials from the credentials file, default to `default` (optional)

restore
: mode to restore the build environment from cache

rebuild
: mode to rebuild the cache from the build environment and specified `mount`s

flush
: mode to flush the cache of old cache items (please be sure to set this so we don't waste storage)

flush_age
: flush cache files older then # days, defaults to 30 (optional)

mount
: list of files/directories to cache

debug
: enabling more logging for debugging, defaults to `false` (optional)

filename
: filename for the cache (optional)

root
: bucket name with root path prefix for all cache default paths (`path`, `fallback_path`, and `flush_path`), defaults to `/`. (optional)

path
: path to store the cache file, defaults to `[root]/<owner>/<repo>/<branch>/` (optional)

fallback_path
: fallback path for the cache file, defaults to `[root]/<owner>/<repo>/<branch>/` (optional)

flush_path
: path to search for flushable cache files, defaults to `[root]/<owner>/<repo>/` (optional)

region
: bucket region (us-east-1, eu-west-1, etc), if using a s3 endpoint, it will grab the region from that endpoint (optional)

workdir
: path where to restore the cache files to (optional)


## Bucket Name Resolution
The following are methods for resolving the bucket name. Note that the plugin will error if you try to use more than one of these methods together.

- Setting the `root` parameter to the bucket name
- Setting the `endpoint` parameter the bucket name will be grabbed from the endpoint
