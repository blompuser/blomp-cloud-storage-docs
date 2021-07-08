# openstack-cli - use with Blomp.com

<!-- TOC -->

- [openstack-cli - use with Blomp.com](#openstack-cli---use-with-blompcom)
    - [environment variables](#environment-variables)
        - [Environment variable examples](#environment-variable-examples)
            - [set environment variables for linux](#set-environment-variables-for-linux)
            - [blomprc examples](#blomprc-examples)
                - [blomprc - Preset your credentials](#blomprc---preset-your-credentials)
                - [blomprc - Ask for credentials](#blomprc---ask-for-credentials)
                - [blomprc - Ask only for password](#blomprc---ask-only-for-password)
    - [Upload large files](#upload-large-files)
        - [Known issues uploading large files to Blomp.com](#known-issues-uploading-large-files-to-blompcom)
        - [Upload 6gb.img example openstack-cli](#upload-6gbimg-example-openstack-cli)
        - [Upload 6gb.img example rclone](#upload-6gbimg-example-rclone)
    - [swift capabilities - Blomp.com](#swift-capabilities---blompcom)
        - [swift help](#swift-help)

<!-- /TOC -->

## environment variables

### Environment variable examples

Environment variables can be set in different ways.

#### set environment variables for linux

create a file .blomprc with your variables (see examples in example configurations).

If you want your variables to be loaded on system start, add `.blomprc` to your `.profile`

```bash
# blomp, set openstack variables
if [ -n "$BASH_VERSION" ]; then
    # include .blomprc if it exists
    if [ -f "$HOME/.blomprc" ]; then
	. "$HOME/.blomprc"
    fi
fi
```

#### .blomprc examples

You can create several different, if you use always the same account and never need to change, then you can preset your credentials. If you work on a pc where other people have access to, then you can set asking user for credentials, see examples below.

##### .blomprc - Preset your credentials

```bash
unset OS_SERVICE_TOKEN
export OS_USERNAME='your@email.com'
export OS_PASSWORD='YourPassword'
export OS_AUTH_URL=https://authenticate.blomp.com/v2.0/
export OS_TENANT_NAME=storage
export OS_TENANT_ID=8b989f118e624ca6957e102775583f6f
export OS_IDENTITY_API_VERSION=2
export OS_REGION_NAME='RegionOne'
```

##### .blomprc - Ask for credentials

```bash
unset OS_SERVICE_TOKEN
echo "Please enter your Blomp Password: "
read -sr OS_USERNAME_INPUT
export OS_USERNAME=$OS_USERNAME_INPUT
echo "Please enter your Blomp Password: "
read -sr OS_PASSWORD_INPUT
export OS_PASSWORD=$OS_PASSWORD_INPUT
export OS_AUTH_URL=https://authenticate.blomp.com/v2.0/
export OS_TENANT_NAME=storage
export OS_TENANT_ID=8b989f118e624ca6957e102775583f6f
export OS_IDENTITY_API_VERSION=2
export OS_REGION_NAME='RegionOne'
```

##### .blomprc - Ask only for password

```bash
unset OS_SERVICE_TOKEN
export OS_USERNAME='your@email.com'
echo "Please enter your Blomp Password: "
read -sr OS_PASSWORD_INPUT
export OS_PASSWORD=$OS_PASSWORD_INPUT
export OS_AUTH_URL=https://authenticate.blomp.com/v2.0/
export OS_TENANT_NAME=storage
export OS_TENANT_ID=8b989f118e624ca6957e102775583f6f
export OS_IDENTITY_API_VERSION=2
export OS_REGION_NAME='RegionOne'
```

## Upload large files

Swift capabilities can be listed by running `swift capabilities`. At the time of writing current document, max file size is `5368709122` bytes. ***All files above max file size need to be segmented or chunked***.

For segmented uploads, max segment amount is 1000 and setting segment size to `5368709122` bytes results in max file size 5000GB/4,88TB (`1000*5368709122=5368709122000` which is `5000.000001863` GB)

Currently there are 3 different methods how one could upload large files

method | max size | can be linked
:- | -: | :-:
browser | **100 GB** | **yes***¹
[openstack-cli](#upload-250gbimg-example-openstack-cli) | **5000 GB** | **yes***¹
rclone | **unlimited***³ | **no***²

*¹ - *can be done over blomp dashboard*

*² - *links for rclone chunked files are not possible*

*³ - *it is limited by set max size of composite file and max chunks amount and seize*

### Known issues uploading large files to Blomp.com

- Error 403 Forbidden
  ```log
  INFO:swiftclient:RESP STATUS: 403 Forbidden
  INFO:swiftclient:RESP HEADERS: {'Content-Length': '73', 'Content-Type': 'text/html; charset=UTF-8', 'X-Trans-Id': 'tx4135fea0dc864390a48a3-0060e7069a', 'X-Openstack-Request-Id': 'tx4135fea0dc864390a48a3-0060e7069a', 'Date': 'Thu, 08 Jul 2021 14:07:23 GMT'}
  INFO:swiftclient:RESP BODY: b'<html><h1>Forbidden</h1><p>Access was denied to this resource.</p></html>'
  ERROR:swiftclient.service:Container PUT failed: http://swiftproxy-01.acs.ai.net:8080/v1/AUTH_8b989f118e624ca6957e102775583f6f/your%40email.com 403 Forbidden  [first 60 chars of response] b'<html><h1>Forbidden</h1><p>Access was denied to this resourc'
  ```
- it seems there is no support available at all
  - all information here required a user to find it all out instead of providing working configurations
- unannounced offline/maintenance
  - uploading large and segmented files can fail due to blomp service going suddenly offline
- using [openstack-cli](https://docs.openstack.org/python-designateclient/latest/user/shell-v2.html)
  - If using big chunk/segment size, then many uploads fail, with `5368709122` bytes segment/chunk size, every second upload fails
  - Bigger segment/chunk size results in less '401 Unauthorized' errors. Even with 1G size quite often it fails. Using size of 128M works well.
  - Smaller segment/chunk size reduces max possible file size for upload as on blomp only 1000 segments are allowed per object. 
  - If using large segment/chunk size's when uploading large files, it often results in error 401, despite using `--retries` it sometimes fails to upload one segment which results in most cases in a need to reupload whole image. 
- using browser, upload over dashboard.blomp.com
  - Max. upload file size is equeal to `102400 MB` (`100 GB`) 
  - segment upload fails, similar to mentioned openstack-cli issue, process stops and upload fails
  - Especially if dealing with more than one file, it is slower than cli
- BlompLive ([Windows](https://www.blomp.com/wp-content/Downloads/BlompInstall-0.9.13.exe)/[Mac](https://www.blomp.com/wp-content/Downloads/BlompInstall-0.9.13.app.zip)/[Linux](https://www.blomp.com/wp-content/Downloads/BlompInstall-0.9.13))
  - ***upload of files equal or larger than 5GB is currently not possible***
    - *the reason for this is no segments virtual container for most if not all blomp users, there is not a single user who is not part of blomp team which could show logs of successful segmented upload.*
- BlompGo (*blomp's mounting drive as X: for windows, uses rclone*)
  - *BlompGo is available* ***only for Windows***
  
    OS | 64-Bit | 32-Bit
    :- | -: | -:
    Windows 7 | [x64](https://www.blomp.com/wp-content/Downloads/blompgo-win7-x86.exe) | [x86](https://www.blomp.com/wp-content/Downloads/blompgo-win7-x86.exe)
    Windows 10 | [x64](https://www.blomp.com/wp-content/Downloads/blompgo-win10-x64.exe) | not available
  - ***upload of files equal or larger than 5GB is currently not possible***
    - *the reason for this is no segments virtual container for most if not all blomp users, there is not a single user who is not part of blomp team which could show logs of successful segmented upload.*

### Upload 6gb.img example (openstack-cli)

```bash
#!/bin/bash
unset OS_SERVICE_TOKEN
export OS_USERNAME='your@email.com'
export OS_PASSWORD='YourPassword'
export OS_AUTH_URL=https://authenticate.blomp.com/v2.0/
export OS_TENANT_NAME=storage
export OS_TENANT_ID=8b989f118e624ca6957e102775583f6f
export OS_IDENTITY_API_VERSION=2
export OS_REGION_NAME='RegionOne'

# set file which you want to upload
# if you use paths, then on blomp it will be saved in same file path
# Example: /home/user/6gb.img would save it in desired container on blomp and create [container]/home/user folder where 6gb.img would be put. 
uploadObject="6gb.img"

# set upload folder/container where to upload uploadObject
uploadContainer="testimages"

# set retries variable
# default is 3, use higher values to be sure that segments upload, otherwise you need to restart your upload
retries=10

# set custom segments container
# currently there is bug on blomp backend and there is no separate virtual segments folder resulting in forbidden error.
# Workaround is to use custom segments containter to which a user has write access
segments_container="${OS_USERNAME}/.segments"

# set segment size
# segment size is set to max possible size shown by swift capabilities
segment_size=5368709122

# Optional, set upload options
optional="--debug --verbose --changed --skip-identical --ignore-checksum"
segment_options="--object-threads 8 --segment-threads 8 --leave-segments --use-slo"

# upload to trezor
swift upload --retries ${retries} \
             --segment-size ${segment_size} \
             --segment-container ${segments_container} \
             ${segment_options} \
             ${optional} \
             "${OS_USERNAME}/${uploadContainer}" \
             "${uploadObject}"
```


### Upload 6gb.img example (rclone)

You can add [chunker](https://rclone.org/chunker/) overlay in rclone. This is useful for those who want to mount their drives and do not care about inability to create download links for rclone overlay files.

[Chunker](https://rclone.org/chunker/) can be used together or independently from segment, but it is not recommended to use both, if you use rclone chunker, then set in your rclone's remote config for blomp `chunk_size = 1P` to disable segmented uploads and add rclone's [chunker](https://rclone.org/chunker/) overlay, here is [chunker](https://rclone.org/chunker/) overlay example for `rclone.conf`:

Blomp [chunker](https://rclone.org/chunker/) example:

```ini
[blomp]
type = swift
env_auth = false
user = your@email.com
key = YourPassword
auth = https://authenticate.blomp.com/v2.0/
tenant = storage
tenant_id = 8b989f118e624ca6957e102775583f6f
region = RegionOne
auth_version = 2
endpoint_type = public
leave_parts_on_error = true
chunk_size = 1P
no_chunk = false
storage_policy = Policy-0

[blomp-chunker]
type = chunker
remote = blomp:your@email.com
chunk_size = 5242879K
name_format = *.#####
start_from = 1
hash_type = none
meta_format = simplejson
fail_hard = false
transactions = norename
```
openstack-cli
You can copy/sync files and folders to `blomp-chunker:` where all files will be chunked and uploaded only if file size is bigger than `5242879K`:

```bash
rclone copy ~/6gb.img blomp-chunker:testimages/ --progress
```

## swift capabilities - Blomp.com

```conf
Core: swift
 Options:
  account_autocreate: True
  account_listing_limit: 10000
  allow_account_management: True
  container_listing_limit: 10000
  extra_header_count: 0
  max_account_name_length: 256
  max_container_name_length: 256
  max_file_size: 5368709122
  max_header_size: 8192
  max_meta_count: 90
  max_meta_name_length: 128
  max_meta_overall_size: 4096
  max_meta_value_length: 256
  max_object_name_length: 1024
  policies: [{'default': True, 'name': 'Policy-0', 'aliases': 'Policy-0'}]
  strict_cors_mode: True
  version: 2.17.1
Additional middleware: account_quotas
Additional middleware: bulk_delete
 Options:
  max_deletes_per_request: 10000
  max_failed_deletes: 1000
Additional middleware: bulk_upload
 Options:
  max_containers_per_extraction: 10000
  max_failed_extractions: 1000
Additional middleware: container_quotas
Additional middleware: container_sync
 Options:
  realms: {}
Additional middleware: ratelimit
 Options:
  account_ratelimit: 0.0
  container_listing_ratelimits: []
  container_ratelimits: []
  max_sleep_time_seconds: 60.0
Additional middleware: slo
 Options:
  max_manifest_segments: 1000
  max_manifest_size: 8388608
  min_segment_size: 1
  yield_frequency: 10
Additional middleware: swift3
 Options:
  allow_multipart_uploads: True
  max_bucket_listing: 1000
  max_multi_delete_objects: 1000
  max_parts_listing: 1000
  max_upload_part_num: 1000
  version: 1.12.1.dev9
```

### swift help

```bash
usage: swift [--version] [--help] [--os-help] [--snet] [--verbose]
             [--debug] [--info] [--quiet] [--auth <auth_url>]
             [--auth-version <auth_version> |
                 --os-identity-api-version <auth_version> ]
             [--user <username>]
             [--key <api_key>] [--retries <num_retries>]
             [--os-username <auth-user-name>]
             [--os-password <auth-password>]
             [--os-user-id <auth-user-id>]
             [--os-user-domain-id <auth-user-domain-id>]
             [--os-user-domain-name <auth-user-domain-name>]
             [--os-tenant-id <auth-tenant-id>]
             [--os-tenant-name <auth-tenant-name>]
             [--os-project-id <auth-project-id>]
             [--os-project-name <auth-project-name>]
             [--os-project-domain-id <auth-project-domain-id>]
             [--os-project-domain-name <auth-project-domain-name>]
             [--os-auth-url <auth-url>]
             [--os-auth-token <auth-token>]
             [--os-storage-url <storage-url>]
             [--os-region-name <region-name>]
             [--os-service-type <service-type>]
             [--os-endpoint-type <endpoint-type>]
             [--os-cacert <ca-certificate>]
             [--insecure]
             [--os-cert <client-certificate-file>]
             [--os-key <client-certificate-key-file>]
             [--no-ssl-compression]
             [--force-auth-retry]
             <subcommand> [--help] [<subcommand options>]

Command-line interface to the OpenStack Swift API.

Positional arguments:
  <subcommand>
    delete               Delete a container or objects within a container.
    download             Download objects from containers.
    list                 Lists the containers for the account or the objects
                         for a container.
    post                 Updates meta information for the account, container,
                         or object; creates containers if not present.
    copy                 Copies object, optionally adds meta
    stat                 Displays information for the account, container,
                         or object.
    upload               Uploads files or directories to the given container.
    capabilities         List cluster capabilities.
    tempurl              Create a temporary URL.
    auth                 Display auth related environment variables.
    bash_completion      Outputs option and flag cli data ready for
                         bash_completion.

Examples:
  swift download --help

  swift -A https://api.example.com/v1.0 \
      -U user -K api_key stat -v

  swift --os-auth-url https://api.example.com/v2.0 \
      --os-tenant-name tenant \
      --os-username user --os-password password list

  swift --os-auth-url https://api.example.com/v3 --auth-version 3\
      --os-project-name project1 --os-project-domain-name domain1 \
      --os-username user --os-user-domain-name domain1 \
      --os-password password list

  swift --os-auth-url https://api.example.com/v3 --auth-version 3\
      --os-project-id 0123456789abcdef0123456789abcdef \
      --os-user-id abcdef0123456789abcdef0123456789 \
      --os-password password list

  swift --os-auth-token 6ee5eb33efad4e45ab46806eac010566 \
      --os-storage-url https://10.1.5.2:8080/v1/AUTH_ced809b6a4baea7aeab61a \
      list

  swift list --lh

```