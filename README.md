# utmh - UTM Harbor Management Tool
`utmh` (UTM Harbor) is a simple project to provide ready to use images (e.g. Linux and BSD) for `Apple Silicon` (`ARM64` based) CPUs with [UTM](https://mac.getutm.app). [UTM](https://mac.getutm.app) is a QEMU based full featured system emulator and virtual machine host for iOS and macOS which allows to run Linux, BSD, Windows and several other operating systems on your iPhone, iPad and Mac. While [UTM](https://mac.getutm.app) is a mac'ish app, `utmh` provides a `server` and `client` tool to fetch ready to use images that can be directly used without further installation or adjustments to credentials, SSH setups or similar. `utmh` requires a utmharbor (a repository) that serves the final images. `umth` is fully written in Python.
  
## Table of Content
- [utmh - UTM Harbor Management Tool](#utmh---utm-harbor-management-tool)
  - [Table of Content](#table-of-content)
  - [Motivation](#motivation)
  - [Features](#features)
  - [Images](#images)
    - [Image Metadata Structure](#image-metadata-structure)
    - [Image Structure](#image-structure)
    - [Image Credentials](#image-credentials)
  - [Client Tool](#client-tool)
    - [Client Requirements](#client-requirements)
    - [Client Configuration](#client-configuration)
    - [Client Usage](#client-usage)
      - [List Images](#list-images)
      - [Download Images](#download-images)
      - [Upload Images](#upload-images)
      - [Remove (local) Images](#remove-local-images)
      - [Start Images](#start-images)
      - [Stop Images](#stop-images)
  - [Server Tool (utm-harbor)](#server-tool-utm-harbor)
    - [Server Requirements](#server-requirements)
    - [Server Repository Structure](#server-repository-structure)
      - [credentials](#credentials)
      - [images](#images-1)
      - [INDEX.yaml](#indexyaml)
    - [Server Configuration](#server-configuration)
    - [Server Usage](#server-usage)
  - [Public Repositories](#public-repositories)
  - [Bugs & Issues](#bugs--issues)
  - [Contact](#contact)

## Motivation
After switching to an ARM64 based `Apple Silicon` CPU my current workflow broke. While `Vagrant` with `VirtualBox` were my daily workflow, this completely broke on the new architecture. However, I searched to alternatives beside of `Docker` since this solution was not cappable of all features I need for developement. Unfortunately, I did not find something similar, but [UTM](https://mac.getutm.app) provides a great tool for virtualizing and emulation of guest systems. It just missed a proper solution like `vagrant init debian/testing`, but there is already a `Gallery` with some usable image. However, these ones may not match the requirements for DevOps. This is where `utmh` becomes handy and fills the gap. All images are fully compatible with `Apple Silicon` CPUs and `ARM64` based. However, you may also ship and use legacy `AMD64` images (which is not the primary goal).

## Features
`utmh` is an overall name for this project which consists of a repository (`utm-harbor`), server tool and client tool.

 * Repository: Serves static files (images, meta files)
 * Server: Generates and updates meta files
 * Client: Downloads and manages image files

## Images
A usable `utmh` image consists of the image itself (`$image.tgz`) and the corresponding meta file (`$image.yaml`) that provides additional information. Therefore, it needs at least these two files:
 * `bsd_freebsd_13.1_minimal_arm64.tgz` - Image Artifact
 * `bsd_freebsd_13.1_minimal_arm64.yaml` - Image Metadata

### Image Metadata Structure

```
bsd_freebsd_13.1_minimal_arm64.tgz:
  sha256: 81dcd909a6b153a131d84bafd90cb2d382ab64926ed14e60f6666ea4c88b7da7
  os_tpye: BSD
  os: FreeBSD
  version: 13.1
  type: minimal
  arch: arm64
  root-pw: Builder1337
```

### Image Structure
Images are shipped in a compressed tar ball which includes the `$imagename.utm`. Mostly the given content is included in the `$imagename.utm`:

```
|-- Images
|   |-- data.qcow2
|   |-- efi_vars.fd
|-- config.plist
|-- view.plist
```

### Image Credentials

## Client Tool
### Client Requirements
 * Python3
 * UTM (https://mac.getutm.app)

Installation of additional packages for Python:
```
pip3 install pyyaml
```

### Client Configuration
The client configuration is stright forward and just needs a simple `YAML` config within the `.utmh` subfolder in your home directory.

`.utmh/config.yaml`:
```
# Required
server: https://utm-harbor.gyptazy.ch
port: 443
ssl_verify: true
image_path: downloads/
log: test.log

# Optional for image uploads
user: gyptazy
password: 1SomE_cRaZy_PaSsWoRd1
```

|Option|Value|Description|
|--|--|--|
|server|https://utm-harbor.gyptazy.ch|HTTP(s) URI to remote repositoy|
|port|443|TCP Port to use (if repository server is running on uncommon port)
|ssl_verify|true|Disable/enable SSL verify|
|image_path|downloads/|Path to download/save images|
|log|test.log|Path to log file|
|user|$username|Username to use for uploading images|
|password|$password|Password to use for uploading images|

By default, the `https://utm-harbor.gyptazy.ch` mirror will be used and can be adjusted to any other. `ssl-verify` allows you to skip further SSL validation on self signed certificates.

### Client Usage
This chapter briefly describes all options of the `utmh` client tool.

#### List Images
All present images on the configured repository can be listed by `utmh list`.

**Example**:
```
$> utmh list
----------------------------------------------------
BSD   | FreeBSD | 13.1     | Minimal | ARM64
BSD   | NetBSD  | 9.2      | Minimal | ARM64
Linux | Debian  | Bullseye | Minimal | ARM64
[...]
```

#### Download Images
Images can be downloaded by using the `utmh download $imagename` command. By default, the registry from the config file will be used.

**Example**:
```
$> utmh download bsd_freebsd_13.1_minimal_arm64
Downloading bsd_freebsd_13.1_minimal_arm64...
Received OK: bsd_freebsd_13.1_minimal_arm64
SHA256 OK: 81dcd909a6b153a131d84bafd90cb2d382ab64926ed14e60f6666ea4c88b7da7
Unpackaged: bsd_freebsd_13.1_minimal_arm64.tgz
Ready to use.
```
#### Upload Images
Images can be uploaded by using the `utmh upload $imagename` command. By default, the registry from the config file will be used. Uploading an image requires to provide the image itself (`$image.tgz`) and the corresponding meta file (`$image.yaml`) that provides additional information. Suffixes like `.tgz` and `.yaml` will automatically be processed.

**Example**:
```
$> utmh upload bsd_freebsd_13.1_minimal_arm64
Uploading bsd_freebsd_13.1_minimal_arm64...
Received OK: bsd_freebsd_13.1_minimal_arm64
SHA256 OK: 81dcd909a6b153a131d84bafd90cb2d382ab64926ed14e60f6666ea4c88b7da7
Ready to use.
```

#### Remove (local) Images
Images can be removed from local storage by using the `utmh remove $imagename` command.

**Example**:
```
$> utmh remove bsd_freebsd_13.1_minimal_arm64
Removed OK: bsd_freebsd_13.1_minimal_arm64
```

*Note: Images will be removed directly without further prompt.*

#### Start Images
Currently there is no possibility in `UTM` (upstream) to run images in headless mode, nor any `cli` tools for managing. As soon as there is any possibility this will be included.

#### Stop Images
Currently there is no possibility in `UTM` (upstream) to run images in headless mode, nor any `cli` tools for managing. As soon as there is any possibility this will be included.

## Server Tool (utm-harbor)
### Server Requirements
The `utm-harbor` can serve the image and metadata itself or use an already present webserver like `Apache`, `nginx` or something similar. Since there are only static files no further configuration will be needed. The repository index will be updated by server tool and only needs the permissions to write within the given `htdocs`. Next to it, `Python3` will be needed.

### Server Repository Structure
Within the `htdocs` the following structure is given:

```
.
├── credentials
│   ├── rootpw
│   ├── utm_harbor_generic
│   └── utm_harbor_generic.pub
├── images
│   ├── bsd_freebsd_13.1_minimal_arm64.tgz
│   ├── bsd_freebsd_13.1_minimal_arm64.yaml
│   ├── bsd_netbsd_9.2_minimal_arm64.tgz
│   ├── bsd_netbsd_9.2_minimal_arm64.yaml
│   ├── linux_archlinux_rolling_minimal_arm64.tgz
│   ├── linux_archlinux_rolling_minimal_arm64.yaml
│   ├── linux_centos_stream_9_minimal_arm64.tgz
│   ├── linux_centos_stream_9_minimal_arm64.yaml
│   ├── linux_debian_bullseye_minimal_arm64.tgz
│   ├── linux_debian_bullseye_minimal_arm64.yaml
│   ├── linux_debian_testing_minimal_arm64.tgz
│   ├── linux_debian_testing_minimal_arm64.yaml
│   ├── linux_gardenlinux_kvm-dev_arm64.tgz
│   ├── linux_gardenlinux_kvm-dev_arm64.yaml
│   ├── linux_ubuntu_22.04_minimal_arm64.tgz
│   ├── linux_ubuntu_22.04_minimal_arm64.yaml
│   └── [...]
├── INDEX.yaml
└── README.txt
```

#### credentials
The `credentials` folder may contain the following static file names:
 * rootpw - This provides the default `root` password for images
 * utm_harbor_generic - The private SSH key
 * utm_harbor_generic.pub - The public SSH key

#### images
The `images` folder provides the images itself, as well as their metadata. All metadata will be slurped by the server tool and written out to the `INDEX.yaml`.

#### INDEX.yaml
This file contains all information of the single images and their metadata and will be used as the source of truth. This file is consumed by the clients and indicates the present images and their configuration.

### Server Configuration
The server configuration is stright forward and just needs a simple `YAML` config.

`server-config.yaml`:
```
srv_engine: nginx
srv_root: /var/www/utm-harbor.gyptazy.ch/htdcos/
log: /var/log/utmh-server.log
```

### Server Usage
The `utmh-server generate -c $server-config.yaml` commands creates the new repo index (`INDEX.yaml`) and should be performed at least after every new image upload. When running a setup with multiple users that may upload images, you may want to run this periodically by a `cronjob` or `systemd-timer`.

**Cronjob Example**:
```
*/5 * * * * $user /opt/utmh-server generate -c /opt/server-config.yaml
```

## Public Repositories
Attached a short overview of public repositories that can be used as an image regestry:

|Server|Port|SSL|Location|Operator|
|--|--|--|--|--|
|utm-harbor.gyptazy.ch|443|True|Frankfurt, Germany|@gyptazy|

## Bugs & Issues
Bugs and Issues can be directly reported for any kind of issue on the [Projects Issue Tracker](https://github.com/gyptazy/utmh/issues/new). Feel free to request new features or to provide further feedback. 

## Contact
Currently, this project is a one man show. If you want to get in touch with me, you may contact me via my contact formular on https://gyptazy.ch/#contacts-e. You can also contact me via encrypted mails by using GPG.
 * Email: contact@gyptazy.ch
 * GPG: 4096R/69BF7050
