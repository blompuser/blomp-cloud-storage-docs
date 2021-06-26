# blomp-cloud-storage-docs
Blomp Cloud Storage documentatioin

## Table of content

![](https://dashboard.blomp.com/images/logo_light.png)

<!-- TOC -->

- [blomp-cloud-storage-docs](#blomp-cloud-storage-docs)
  - [Table of content](#table-of-content)
  - [Rclone - Installation and documentation](#rclone---installation-and-documentation)
    - [Rclone Features](#rclone-features)
    - [Rclone Installation & documentation](#rclone-installation--documentation)
    - [Rclone Downloads](#rclone-downloads)
  - [Create rclone config for Blomp cloud storage (first time install)](#create-rclone-config-for-blomp-cloud-storage-first-time-install)
    - [Table of demo remotes](#table-of-demo-remotes)
      - [Table description](#table-description)
    - [Start rclone config](#start-rclone-config)
    - [Create remote config for Blomp](#create-remote-config-for-blomp)
      - [Optional - advanced config](#optional---advanced-config)
  - [Local overlays - chunker, compress and crypt](#local-overlays---chunker-compress-and-crypt)
    - [overlays over local overlays](#overlays-over-local-overlays)
  - [Advanced Configuration for Blomp with rclone](#advanced-configuration-for-blomp-with-rclone)
    - [Create alias for remote for blomp-remote](#create-alias-for-remote-for-blomp-remote)
    - [Create chunker overlay for blomp](#create-chunker-overlay-for-blomp)
      - [Optional, advanced chunker settings](#optional-advanced-chunker-settings)
    - [Create crypt overlay for blomp chunker](#create-crypt-overlay-for-blomp-chunker)
      - [Optional advanced settings](#optional-advanced-settings)
    - [Create compress overlay for blomp crypt overlay](#create-compress-overlay-for-blomp-crypt-overlay)
      - [Optional advanced settings](#optional-advanced-settings-1)
    - [Create compress overlay for blomp chunker overlay](#create-compress-overlay-for-blomp-chunker-overlay)
      - [Optional advanced settings](#optional-advanced-settings-2)
    - [Finished, remote overview and quit rclone config](#finished-remote-overview-and-quit-rclone-config)
  - [Demo config (local filesystem) - rclone.conf (recommended)](#demo-config-local-filesystem---rcloneconf-recommended)
  - [Demo config (using remote only) - rclone.conf (not recommended for use)](#demo-config-using-remote-only---rcloneconf-not-recommended-for-use)
  - [Demo Systemd service files](#demo-systemd-service-files)
    - [Why do I recommend to use local filesystem for compress, chunking and crypt](#why-do-i-recommend-to-use-local-filesystem-for-compress-chunking-and-crypt)
    - [local overlays (recommended)](#local-overlays-recommended)
    - [non local overlays (not recommended)](#non-local-overlays-not-recommended)
    - [How to enable service](#how-to-enable-service)
    - [How to start/stop/restart service](#how-to-startstoprestart-service)

<!-- /TOC -->

## Rclone - Installation and documentation

Repository contains outdated rclone version which lacks on several features where one of them is selfupdate function. You can install from repository.

I suggest to either [download latest binaries](https://github.com/rclone/rclone/releases) or compile and install from source. On later points you will upgrade binary directly with `rclone selfupdate` as well for beta `rclone selfupdate --beta` for those who want to test latest version.

For installation guides and more info visit [official rclone description](https://github.com/rclone/rclone) which is always kept up to date.

Please see [the full list of all storage providers and their features](https://rclone.org/overview/)

### Rclone Features

  * MD5/SHA-1 hashes checked at all times for file integrity
  * Timestamps preserved on files
  * Partial syncs supported on a whole file basis
  * [Copy](https://rclone.org/commands/rclone_copy/) mode to just copy new/changed files
  * [Sync](https://rclone.org/commands/rclone_sync/) (one way) mode to make a directory identical
  * [Check](https://rclone.org/commands/rclone_check/) mode to check for file hash equality
  * Can sync to and from network, e.g. two different cloud accounts
  * Optional large file chunking ([Chunker](https://rclone.org/chunker/))
  * Optional transparent compression ([Compress](https://rclone.org/compress/))
  * Optional encryption ([Crypt](https://rclone.org/crypt/))
  * Optional FUSE mount ([rclone mount](https://rclone.org/commands/rclone_mount/))
  * Multi-threaded downloads to local disk
  * Can [serve](https://rclone.org/commands/rclone_serve/) local or remote files over HTTP/WebDav/FTP/SFTP/dlna

### Rclone Installation & documentation

Please see the [rclone website](https://rclone.org/) for:

  * [Installation](https://rclone.org/install/)
  * [Documentation & configuration](https://rclone.org/docs/)
  * [Changelog](https://rclone.org/changelog/)
  * [FAQ](https://rclone.org/faq/)
  * [Storage providers](https://rclone.org/overview/)
  * [Forum](https://forum.rclone.org/)
  * ...and more

### Rclone Downloads

  * https://rclone.org/downloads/

## Create rclone config for Blomp cloud storage (first time install)

As example we will use several rclone modules combined to offer some general use example. For more info about [swift](https://rclone.org/swift/), [chunker](https://rclone.org/chunker/), [crypt](https://rclone.org/crypt/) and [compress](https://rclone.org/compress/) modules as well as description of [mount command](https://rclone.org/commands/rclone_mount/) can be found at official rclone documentation.


### Table of demo remotes

Here is a table of demo configuration remotes and alias.

name | storage type | chunk size*¹ | encrypted | compressed | max. file size
:- | -:  | :-: | :-: | :-: | -:
`blomp-remote` | `swift` | `1P`| no | no |`5242879K`
`blomp-alias` | `alias` | `1P` | no | no | `5242879K`
`blomp-chunker` | `chunker` | `5242879K` | no | no | **unlimited***²
`blomp-trezor` | `crypt` | `5242879K` | **yes** | no | **unlimited***²
`blomp-chunker-archive` | `compress` | `5242879K` | no | **yes** | **unlimited***²
`blomp-trezor-archive` | `compress` | `5242879K` | **yes** | **yes** | **unlimited***²


- _*¹ - setting swift to 1P technically disables chunker and only files equal/smaller than max. file size can uploaded._

- _*² - as long as there is free space and chunk naming allows enough chunks_

#### Table description

- blomp-remote is main blomp storage
- blomp chunker is chunker overlay which chunks data. This can be used for different purposes, one of them would be for abililty to get error Blomp's max file size restriction or for users who care about privacy and security and want their files to be chunked into some specific size to hide actual file size. In current demo, we will use max. allowed file size as chunk_size, all files smaller than 5G will not be chunked, only files bigger than 5G
- blomp-chunker-archive is compress overlay with max gzip compression using chunker. This is especially usefull for archive purpose as well for storage of disk images which do normally contain free space, compressing such images on the fly.
- blomp-trezor is overlay using chunker for ability to upload files bigger than 5G which do require manual chunking or for security purpose to hide real file size.
- blomp-trezor-archive is copmress overlay for encrypted and chunked data. Similar to blomp-chunker-archive, this is very usefull for image backups which contain some sensible data and one would like to have it stored encrypted

### Start rclone config

- to add/edit/encrypt rclone config file, run 

```shell
rclone --config ~/.config/rclone/blomp-demo.conf config`
```

### Create remote config for Blomp

1. Add new Blomp remote

   ```log
   <5>NOTICE: Config file "/home/tor/.config/rclone/blomp-demo.conf" not found - using defaults
   No remotes found - make a new one
   n) New remote
   s) Set configuration password
   q) Quit config
   n/s/q> n
   ```

1. select name for your remote, in our example we use `blomp-remote`

    ```log
    name> blomp-remote
    ```

1. select storage type, choose `swift`

   ```log
   Storage> swift
   ```
   
   ℹ️ - *type either `swift` or the number from the list, at the time of creation of current document it is `28`*

   *See help for swift backend at: https://rclone.org/swift/*

   ```log
   28 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
   ```

1. For current example we will not use swift credentials from environment variables as we might want use different accounts from same provider. Default value is `false`, leave it blank or enter false and press enter.

   ```log
   env_auth> 
   ```

1. Set credentials, your user (`demo@mail.com`) and password (`demopasswd123`)

   ```log
   User name to log in (OS_USERNAME).
   Enter a string value. Press Enter for the default ("").
   user> demo@mail.com
   API key or password (OS_PASSWORD).
   Enter a string value. Press Enter for the default ("").
   key> demopasswd123
   ```

1. Set Authentification URL to `https://authenticate.blomp.com`

   ```log
   auth> https://authenticate.ain.net
   ```
   ℹ️ *- you can use 2 different domains, `https://authenticate.ain.net` is used by blomp rclone app, where `https://authenticate.blomp.com` is preffered URL and blomp dashboard says it should be used*

1. `user_id` and `user_domain` are optional for _v3 auth_, leave it blank (ℹ️ *- Blomp uses v2 auth*)

   ```log
   user_id> 
   domain> 
   ```

1. Set tenant name to `storage`

   ```log
   tenant> storage
   ```

1. Leave tenant tenant ID blank

   ```log
   tenant_id> 
   ```

1. leave blank all following options until you get to `auth_version`, type `2` and press enter

   ```log
   auth_version> 2
   ```

   ℹ️ *- Blank options before you are asked for auth_version*
   ```log
   Tenant ID - optional for v1 auth, this or tenant required otherwise (OS_TENANT_ID)
   Enter a string value. Press Enter for the default ("").
   tenant_id> 
   Tenant domain - optional (v3 auth) (OS_PROJECT_DOMAIN_NAME)
   Enter a string value. Press Enter for the default ("").
   tenant_domain> 
   Region name - optional (OS_REGION_NAME)
   Enter a string value. Press Enter for the default ("").
   region> 
   Storage URL - optional (OS_STORAGE_URL)
   Enter a string value. Press Enter for the default ("").
   storage_url> 
   Auth Token from alternate authentication - optional (OS_AUTH_TOKEN)
   Enter a string value. Press Enter for the default ("").
   auth_token> 
   Application Credential ID (OS_APPLICATION_CREDENTIAL_ID)
   Enter a string value. Press Enter for the default ("").
   application_credential_id> 
   Application Credential Name (OS_APPLICATION_CREDENTIAL_NAME)
   Enter a string value. Press Enter for the default ("").
   application_credential_name> 
   Application Credential Secret (OS_APPLICATION_CREDENTIAL_SECRET)
   Enter a string value. Press Enter for the default ("").
   application_credential_secret> 
   AuthVersion - optional - set to (1,2,3) if your auth URL has no version (ST_AUTH_VERSION)
   Enter a signed integer. Press Enter for the default ("0").
   ```

1. Select `public` Endpoint type

   ```log
   endpoint_type> public
   ```

1. Keep storage policy blank

   ```log
   storage_policy> 
   ```
1. Set advanced options (_optional_)

   ```log
   Edit advanced config? (y/n)
   y) Yes
   n) No (default)
   y/n> 
   ```
   #### Optional - advanced config

1. avoid calling abort upload on a failure

   ```log
   leave_parts_on_error> true
   ```
1. Set above which size files will be chunked into a _segments container

   ```log
   chunk_size> 1P
   ```

   ℹ️ *- swift default and max size is 5G (5242880K) which currently does not work with blomp with error message `Permission denied`. Setting it to 1P disables chunking and resolves Permission Error, max file size is 5242880K.*

1. Don't chunk files during streaming upload.

   ```log
   no_chunk> false
   ```

   ℹ️ *When doing streaming uploads (e.g. using rcat or mount) setting this
flag will cause the swift backend to not upload chunked files.*

   *This will limit the maximum upload size to 5GB. However non chunked
files are easier to deal with and have an MD5SUM.*

   *Rclone will still chunk files bigger than chunk_size when doing normal copy operations. Default value is `false`*

1. Set encoding for backend, leave blank and press enter unless you know what you do.

   ```log
   encoding> 
   ```

1. Finished, check your config once again and confirm with `y` if everything is OK

   ```conf
   Remote config
   --------------------
   [blomp-remote]
   type = swift
   user = demo@mail.com
   key = demopasswd123
   auth = https://authenticate.ain.net
   tenant = storage
   auth_version = 2
   endpoint_type = public
   leave_parts_on_error = true
   chunk_size = 1P
   no_chunk = false
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   ```

   ```log
   y/e/d> y
   ```

   if you are finished, quit config by typing `q` and press enter, for current guide continue and create new remote alias

   ```log
   Current remotes:

   Name    Type
   ====    ====
   blomp-remote         swift

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> 
   ```

## Local overlays - chunker, compress and crypt

With blomp you can use overlays directly, but it is not recommended.

Use instead [local](https://rclone.org/local/) overlays of [chunker](https://rclone.org/chunker/), [compress](https://rclone.org/compress/) and [crypt](https://rclone.org/crypt/).

[Example config for local filesystem, chunker, compress and crypt](.config/rclone/blomp-local-demo.conf):

```conf
# configure local filesystem, in current case we want to use one_file_system, for more info read https://rclone.org/local/#restricting-filesystems-with-one-file-system
[local]
type = local
nounc = true
one_file_system = true
case_sensitive = true
no_preallocate = true
no_set_modtime = true

# alias for Blomp online storage for easier access
[blomp-alias]
type = alias
remote = blomp-remote:demo@mail.com

# local filesystem alias for easier/simpler access.
# remote in current case is the folder
# ensure that your user is the owner of that folder and has write access
[blomp-local]
type = alias
remote = local:/media/yourusername/blomp-local

# local chunker for Blomp, chunking only files above chunk_size
# norename is used to speed up the process
# max number of chunks is 99999, to change it, edit amount of # in name_format
[blomp-local-chunker]
type = chunker
remote = blomp-local:
chunk_size = 5242879K
name_format = *.#####
start_from = 1
hash_type = md5
meta_format = simplejson
fail_hard = false
transactions = norename

# local compression to gzip with highest compression, level 9 is highest compression
[blomp-local-gzip]
type = compress
remote = blomp-local:
level = 9

# local encrypt/decrypt module. 
# Crypt module should be created with rclone config where your password will be save encrypted to rclone.conf
[blomp-local-trezor]
type = crypt
remote = blomp-local:trezor
filename_encryption = standard
directory_name_encryption = true
password = ***ENCRYPTED-PASS***
password2 = ***ENCRYPTED-PASS***
no_data_encryption = false
```

### overlays over local overlays

Here are few examples of how different local overlays can be combined.

```conf
[blomp-local-gzip-chunked]
type = compress
remote = blomp-local-chunker:
level = 9

[blomp-local-chunked-gzip]
type = chunker
remote = blomp-local-gzip:
chunk_size = 5242879K
name_format = *.#####
start_from = 1
hash_type = md5
meta_format = simplejson
fail_hard = false
transactions = norename

[blomp-local-trezor-chunked]
type = crypt
remote = blomp-local-chunker:trezor
filename_encryption = standard
directory_name_encryption = true
password = ***ENCRYPTED-PASS***
password2 = ***ENCRYPTED-PASS***
no_data_encryption = false

[blomp-local-trezor-gzip-chunked]
type = crypt
remote = blomp-local-gzip-chunked:trezor
filename_encryption = standard
directory_name_encryption = true
password = ***ENCRYPTED-PASS***
password2 = ***ENCRYPTED-PASS***
no_data_encryption = false

# do not use
[blomp-local-trezor-chunked-gzip]
type = crypt
remote = blomp-local-chunked-gzip:trezor
filename_encryption = standard
directory_name_encryption = true
password = ***ENCRYPTED-PASS***
password2 = ***ENCRYPTED-PASS***
no_data_encryption = false

```

## Advanced Configuration for Blomp with rclone

Advanced configuration example allows a user combination of mountable remotes for different use case.

In simple words:
- Normal usage without file size limit
  - Archive, compressed drive with gzip, max compression
- Encryption without file size limit
  - Encryption archive (compressed with gzip, max compression)

### Create alias for remote for blomp-remote

*If you quit rclone config, follow [Start rclone config](#start-rclone-config) to restart config.*

We will use alias for easier access in further configuration as well for easier usage in terminal and spare us typing buckett which for blomp is your email address.

```shell
rclone --config ~/.config/rclone/blomp-demo.conf config`
```

1. Create alias for remote - add new remote `alias`

   which we will call `blomp` for simple use in future and as last select remote/path to alias `blomp-remote` 

   ```log
   Current remotes:

   Name    Type
   ====    ====
   blomp-remote         swift

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   e/n/d/r/c/s/q> n
   ```

   ```log
   name> blomp-alias
   ```

1. Create alias for remote - select storage type

   ```log
   Storage> alias
   ```

1. Create alias for remote - select remote to alias

   ```log
   remote> blomp-remote:demo@mail.com
   ```

1. Create alias for remote - Confirm alias settings

   ```log
   Remote config
   --------------------
   [blomp-alias]
   type = alias
   remote = blomp-remote:demo@mail.com
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   ```

   ```log
   y/e/d> y
   ```

### Create chunker overlay for blomp

At the time of writting this guide, `--swift-chunk-size` option is not working with Blomp. Currently files bigger than `5242879K` can be uploaded with:

- [x] web browser on your dashboard
- [x] additional chunker overlay in `rclone.conf`
- [ ] BlompLive 
  _Only BlompSupport could confirm that it works, until the time of creation of current document it never worked for me_)
- [ ] BlompGo

*Due to file size restriction, all other overlays will use chunker as their remote. That way we get around file size limit without any loss of time compared to uploads without chunker overlay.*

*If you quit rclone config, follow [Start rclone config](#start-rclone-config) to restart config.*

1. Create new chunker remote for `blomp-alias`

   ```log
   Name                 Type
   ====                 ====
   blomp-alias          alias
   blomp-remote         swift

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> n
   name> blomp-chunker    
   Storage> chunker
   remote> blomp-alias:
   chunk_size> 5242879K
   hash_type> none
   ```

   #### Optional, advanced chunker settings

1. Set string format of file names*³

   ```log
   name_format> *.#####
   ```

   ___*³__ - * is filename and # stands for max possible amount of chunks, `#####` means max **99999** chunks are available, **with 5G** chunk size, max. file size would be **488T** (488.276367188T)_. Rclone's default is `*.rclone_chunk.###`

1. Set minimum valid chunk number, default is `1`

   ```log
   start_from> 1
   ```

1. Set metadata object format

   We will use norename and it requires `meta_format` **not to be** `none`
   ```log
   meta_format> simplejson
   ```

1. Choose how chunker should handle files with missing or invalid chunks.

   For current guide we will use: `Warn user, skip incomplete file and proceed.`

   ```log
   fail_hard> false
   ```

1. Choose how chunker should handle temporary files during transactions.

   We do not want chunker to rename files as it requires additional copy and delete operation which for Blomp takes very long time and is very slow.

   ```log
   transactions> norename
   ```

1. Confirm chunker settings

   ```log
   Remote config
   --------------------
   [blomp-chunker]
   type = chunker
   remote = blomp-alias:
   chunk_size = 5.000G
   hash_type = none
   name_format = *.#####
   start_from = 1
   meta_format = simplejson
   fail_hard = false
   transactions = norename
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   ```

   ```log
   y/e/d> y
   ```

   ⚠️ - Ensure to change chunk_size from 5.000G to `5242879K`, as otherwise chunker will fail. You can set any lower value like `1G`.

### Create crypt overlay for blomp chunker

1. Create new crypt remote for blomp-chunker

   ```log
   Current remotes:

   Name                 Type
   ====                 ====
   blomp-alias          alias
   blomp-chunker        chunker
   blomp-remote         swift

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> n
   name> blomp-trezor
   Storage> crypt
   remote> blomp-chunker:trezor
   filename_encryption> standard
   directory_name_encryption> true
   ```

1. Set your password for encryption

   ```log
   Password or pass phrase for encryption.
   y) Yes type in my own password
   g) Generate random password
   y/g> y
   ```

   Set password for encryption:

   ```log
   password: cryptpass12345
   ```

   Confirm the password:

   ```log
   password: cryptpass12345
   ```

2. Set password or pass phrase for salt

   ```log
   Password or pass phrase for salt. Optional but recommended.
   Should be different to the previous password.
   y) Yes type in my own password
   g) Generate random password
   n) No leave this optional password blank (default)
   y/g/n> y
   ```

   Set password for salt:
   ```log
   password: cryptpassSalt1
   ```

   Confirm password for salt:

   ```log
   password: cryptpassSalt1
   ```

   #### Optional advanced settings

1. Allow server-side operations (e.g. copy) to work across different crypt configs

   ```log
   erver_side_across_configs> false
   ```
1. Encrypt file data

   ```log
   no_data_encryption> false
   ```

1. Confirm crypt settings

   ```log
   Remote config
   --------------------
   [blomp-trezor]
   type = crypt
   remote = blomp-chunker:trezor
   filename_encryption = standard
   directory_name_encryption = true
   password = *** ENCRYPTED ***
   password2 = *** ENCRYPTED ***
   server_side_across_configs = false
   no_data_encryption = false
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   y/e/d> y
   ```

### Create compress overlay for blomp crypt overlay

1. Create new crypt overlay over blomp-chunker

   ```log
   Current remotes:

   Name                 Type
   ====                 ====
   blomp-alias          alias
   blomp-chunker        chunker
   blomp-remote         swift
   blomp-trezor         crypt

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> n
   name> blomp-trezor-archive
   Storage> compress
   remote> blomp-trezor:archive
   mode> gzip
   ```

   #### Optional advanced settings

1. Set compression level

   ```log
   level> 9
   ```

   GZIP compression level (-2 to 9).
   - Generally -1 (default, equivalent to 5) is recommended.
   - Levels 1 to 9 increase compressiong at the cost of speed.. Going past 6 generally offers very little return.
   - Level -2 uses Huffmann encoding only. Only use if you now what you are doing
   - Level 0 turns off compression.


1. Some remotes don't allow the upload of files with unknown size.

   ```log
   ram_cache_limit> 20M
   ```

1. Confirm compress overlay settings

   ```log
   Remote config
   --------------------
   [blomp-trezor-archive]
   type = compress
   remote = blomp-trezor:archive
   mode = gzip
   level = 9
   ram_cache_limit = 20M
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   ```

   ```log
   y/e/d> y
   ```

### Create compress overlay for blomp chunker overlay

1. Create new crypt overlay over blomp-chunker

   ```log
   Current remotes:

   Name                 Type
   ====                 ====
   blomp-alias          alias
   blomp-chunker        chunker
   blomp-remote         swift
   blomp-trezor         crypt
   blomp-trezor-archive compress

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> n
   name> blomp-chunker-archive
   Storage> compress
   remote> blomp-chunker:archive
   mode> gzip
   ```

   #### Optional advanced settings

2. Set compression level

   ```log
   level> 9
   ```

   GZIP compression level (-2 to 9).
   - Generally -1 (default, equivalent to 5) is recommended.
   - Levels 1 to 9 increase compressiong at the cost of speed.. Going past 6 generally offers very little return.
   - Level -2 uses Huffmann encoding only. Only use if you now what you are doing
   - Level 0 turns off compression.

3. Some remotes don't allow the upload of files with unknown size.

   ```log
   ram_cache_limit> 20M
   ```

4. Confirm compress overlay settings

   ```log
   Remote config
   --------------------
   [blomp-chunker-archive]
   type = compress
   remote = blomp-chunker:archive
   mode = gzip
   level = 9
   ram_cache_limit = 20M
   --------------------
   y) Yes this is OK (default)
   e) Edit this remote
   d) Delete this remote
   ```

   ```log
   y/e/d> y
   ```

   ### Finished, remote overview and quit rclone config

   ```log
   Current remotes:

   Name                 Type
   ====                 ====
   blomp-alias          alias
   blomp-chunker        chunker
   blomp-chunker-archive compress
   blomp-remote         swift
   blomp-trezor         crypt
   blomp-trezor-archive compress

   e) Edit existing remote
   n) New remote
   d) Delete remote
   r) Rename remote
   c) Copy remote
   s) Set configuration password
   q) Quit config
   ```

   ```log
   e/n/d/r/c/s/q> q
   ```

## Demo config (local filesystem) - [rclone.conf](.config/rclone/blomp-local-demo.conf) (recommended)

This configuration is recommended. Chunker should not be required at all, but as swift-chunk-size on blomp does not work for anybody except blomp team, we are forced to use it as workaround.

ℹ️ - *I have to mention that in my local example for chunker I use max allowed file size of 5GB - 1KB. If you want to combine your blomp with several other cloud storages in one drive, then choose for chunker the size of storage which offers smallest size, take as example dropbox, onedrive and other which have its own restrictions on max file size. You can set it to small sizes like 3M or 5M if you want*.

```conf
# configure local filesystem, in current case we want to use one_file_system, for more info read https://rclone.org/local/#restricting-filesystems-with-one-file-system
[local]
type = local
nounc = true
one_file_system = true
case_sensitive = true
no_preallocate = true
no_set_modtime = true

# alias for Blomp online storage for easier access
[blomp-alias]
type = alias
remote = blomp-remote:demo@mail.com

# local filesystem alias for easier/simpler access.
# remote in current case is the folder
# ensure that your user is the owner of that folder and has write access
[blomp-local]
type = alias
remote = local:/media/yourusername/blomp-local

# local chunker for Blomp, chunking only files above chunk_size
# norename is used to speed up the process
# max number of chunks is 99999, to change it, edit amount of # in name_format
[blomp-local-chunker]
type = chunker
remote = blomp-local:
chunk_size = 5242879K
name_format = *.#####
start_from = 1
hash_type = md5
meta_format = simplejson
fail_hard = false
transactions = norename

# local compression to gzip with highest compression, level 9 is highest compression
[blomp-local-gzip]
type = compress
remote = blomp-local:
level = 9

# local encrypt/decrypt module. 
# Crypt module should be created with rclone config where your password will be save encrypted to rclone.conf
[blomp-local-trezor]
type = crypt
remote = blomp-local:trezor
filename_encryption = standard
directory_name_encryption = true
password = ***ENCRYPTED-PASS***
password2 = ***ENCRYPTED-PASS***
no_data_encryption = false
```

## Demo config (using remote only) - [rclone.conf](.config/rclone/blomp-demo.conf) (not recommended for use)

```conf
[blomp-remote]
type = swift
user = demo@mail.com
key = demopasswd123
auth = https://authenticate.ain.net
tenant = storage
auth_version = 2
endpoint_type = public
leave_parts_on_error = true
chunk_size = 1P
no_chunk = false

[blomp-alias]
type = alias
remote = blomp-remote:demo@mail.com

[blomp-chunker]
type = chunker
remote = blomp-alias:
chunk_size = 5242879K
hash_type = none
name_format = *.#####
start_from = 1
meta_format = simplejson
fail_hard = false
transactions = norename

[blomp-trezor]
type = crypt
remote = blomp-chunker:trezor
filename_encryption = standard
directory_name_encryption = true
password = 8QSBAdBXqPpgQbH1-ri2jsK6t52fgjVPhR79qazL
password2 = uWe6EbZuUOZwjCoOxzj6czp-15xCafDOa9NmjAZ0
server_side_across_configs = false
no_data_encryption = false

[blomp-trezor-archive]
type = compress
remote = blomp-trezor:archive
mode = gzip
level = 9
ram_cache_limit = 20M

[blomp-chunker-archive]
type = compress
remote = blomp-chunker:archive
mode = gzip
level = 9
ram_cache_limit = 20M
```

## Demo Systemd service files

Example how some basic service file mounting drive on system launch could look like. Here are [5 service files](etc/systemd/system) which could be enabled, you can test those by simply replacing credentials in rclone.conf.

### Why do I recommend to use local filesystem for compress, chunking and crypt

- it is much faster because operations and processes on remote
  - in my examples I use norename for chunker simply because I tested one huge file and I wanted to spare uploading unrenamed chunks before huge file is uploaded, for smaller files than 20 gb you can and probably should use rename in chunker config.
- requires much less:s
  - time
  - bandwidth

*Using overlays directly on remote was only for testing purpose if it works and it does work, but it should not be used as users will have pretty bad experience compared to local filesystem.*
### local overlays (recommended)

I recommend to use local overlays for whatever should be done in regard of chunking, compression and encryption/decryption.
One of main reasons is that local overlays are much faster, due to very long wait times for copy/move, you would end up wasting a lot bandwidth and time, exceeding in few times the real amount of data which you need to upload.

- In configs I used max. allowed file size for blomp which is currently 5Gbyte-1Kbyte.
  - **Info for union remote**:
    - the smaller the chunk size, the better it is in regard of using max available free space.
    - the bigger the chunk size, the better it is in regard of performance but may leave few accounts with <chunk_size amount of free space.

- [blomp](etc/systemd/system/blomp-local-chunker.service)  (*local remote chunked, shows chunked files*)
- [blomp-local-compress](etc/systemd/system/blomp-local-compress.service)  (*local remote compress, shows compressed files as one*)
- [blomp-trezor](etc/systemd/system/blomp-local-trezor.service)  (*local remote crypt, shows decrypted files and folders*)
### non local overlays (not recommended)

- [blomp](etc/systemd/system/blomp.service)
  - [blomp-chunker](etc/systemd/system/blomp-chunker.service)
    - [blomp-chunker-archive](etc/systemd/system/blomp-chunker-archive.service)
  - [blomp-trezor](etc/systemd/system/blomp-trezor.service)
    - [blomp-trezor-archive](etc/systemd/system/blomp-trezor-archive.service)

### How to enable service

Enable service, example with blomp-trezor-archive:

```shell
sudo systemctl enable blomp-trezor-archive
```

Disable service, example with blomp-trezor-archive:

```shell
sudo systemctl enable blomp-trezor-archive
```

### How to start/stop/restart service

You can use start to start, stop to stop and restart to restart running service.

All examples with blomp-trezor-archive:

```shell
sudo systemctl start blomp-trezor-archive
```

```shell
sudo systemctl stop blomp-trezor-archive
```

```shell
sudo systemctl restart blomp-trezor-archive
```