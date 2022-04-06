APT-REPO-SERVER
=========================

apt-repo-server is a debian repository server. It monitors file changing event(inotify), then reproduce index file(Packages.gz) automatically.

This has been modified to collect NGINX packages using a certificate and key, then host them in a container acting as a repo for demo environment purposes.

Packages collected are:
- app-protect (and all immediate dependencies)
- app-protect-attack-signatures
- app-protect-threat-campaigns

Usage
=======================

Build container
```
git clone https://github.com/aknot242/apt-repo-server
cd apt-repo-server
cp <your nginx repo cert> nginx-repo.crt
cp <your nginx repo key> nginx-repo.key

DOCKER_BUILDKIT=1 docker build --no-cache --secret id=nginx-crt,src=nginx-repo.crt --secret id=nginx-key,src=nginx-repo.key -t aknot242/apt-repo-server .
```


Run server

```
docker run --restart unless-stopped -d -p 10000:80 aknot242/apt-repo-server

```

Test to make sure packages are serving

```
curl http://localhost:10000/dists/focal/main/binary-amd64/
```

File structure in the container looks similar to:
```
$ tree /data/
data/
└── dists
    ├── bionic
    │   └── main
    │       ├── binary-amd64
    │       │   └── Packages.gz
    │       └── binary-i386
    │           └── Packages.gz
    └── focal
        └── main
            ├── binary-amd64
            │   ├── Packages.gz
            │   └── qnap-fix-input_0.1_all.deb
            └── binary-i386
                └── Packages.gz
```

Packages.gz looks like
```
$ zcat data/dists/focal/main/binary-amd64/Packages.gz
Package: qnap-fix-input
Version: 0.1
Architecture: all
Maintainer: Doro Wu <dorowu@qnap.com>
Installed-Size: 33
Filename: ./qnap-fix-input_0.1_all.deb
Size: 1410
MD5sum: 8c08f13d61da1b8dc355443044bb2608
SHA1: 6deef134c94da7f03846a6b74c9e4258c514868f
SHA256: 7441f1616810d5893510d31eac2da18d07b8c13225fd2136e6a380aefe33c815
Section: utils
Priority: extra
Description: QNAP fix
 UNKNOWN
```

Update /etc/apt/sources.list
```
$ echo deb http://127.0.0.1:10000 focal main | sudo tee -a /etc/apt/sources.list
```


License
==================

apt-repo is under the Apache 2.0 license. See the LICENSE file for details.
