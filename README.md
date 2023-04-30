# ztncui-aio
## ZeroTier network controller user interface in a Docker container

This is to build a Docker image that contains **[ZeroTier One](https://www.zerotier.com/download.shtml)** and **[ztncui](https://key-networks.com/ztncui)** to set up a **standalone ZeroTier network controller** with a web user interface in a container.

Follow us on [![alt @key_networks on Twitter](https://i.imgur.com/wWzX9uB.png)](https://twitter.com/key_networks)

Licensed Under GNU GPLv3

## Credit

Thanks to @kmahyyg for https://github.com/kmahyyg/ztncui-aio from which this build process is forked.

## Further information

Refer to https://github.com/key-networks/ztncui-containerized for the original documentation.

## Build yourself

We support arm64, amd64. armv7 might work, but is not tested. Others are unsupported.

```bash
$ git clone https://github.com/key-networks/ztncui-aio
$ docker build . -t keynetworks/ztncui:latest
```

Change `NODEJS_MAJOR` variable in Dockerfile to use different nodejs version.

Never use `node_lts.x` as your installation script of nodejs whose version might changed without further notice due to time shift.

## Usage

### Golang auto-mkworld

This feature allows you to generate a planet file without using C code and compiler.

Also, due to limitation of IPC of Zerotier-One UI and multiple issues, we do NOT support customized port, you can ONLY use port 9993/udp here.

| MANDATORY | Name | Explanation | Default Value |
|:--------:|:--------:|:--------:|:--------:|
| no | AUTOGEN_PLANET | If set to 1, will use this node identity to generate a `planet` file and put to `httpfs` folder to serve it outside. If set to 2, will use config in `/etc/zt-mkworld/mkworld.config.json`. If set to 0, will do nothing. | 0 |

The reference config file can be found on `ztnodeid/assets/mkworld.conf.json`. `plRecommend` is used to mark to generate planetID and planetBirth timestamp.

You could also define yourself, and check the output to get C header of customized planet. After that, you will find the custom planet file under http file server root and also ca certificate.

### Docker image

```bash
$ git clone https://github.com/key-networks/ztncui-aio # to get a copy of denv file, otherwise make your own
$ docker pull keynetworks/ztncui
$ docker run -d -p3443:3443 -p3180:3180 \
    -v /mydata/ztncui:/opt/key-networks/ztncui/etc \
    -v /mydata/zt1:/var/lib/zerotier-one \
    -v /mydata/zt-mkworld-conf:/etc/zt-mkworld \
    --env-file ./denv <CHANGE HERE ACCORDING TO NEXT PART> \
    --name ztncui \
    keynetworks/ztncui
```

If their one is not updated, try `docker pull ghcr.io/kmahyyg/ztncui-aio:latest` ! (YES, We Love GitHub!)

## Supported Configuration via persistent storage

For ZTNCUI: https://github.com/key-networks/ztncui

| MANDATORY | Name | Explanation | Default Value |
|:--------:|:--------:|:--------:|:--------:|
| YES | NODE_ENV | https://pugjs.org/api/express.html | production |
| no | HTTPS_HOST | Only Listen on HTTPS_HOST:HTTPS_PORT | NO DEFAULT |
| no | HTTPS_PORT | HTTPS_PORT | 3443 |
| no | HTTP_PORT | HTTP_PORT | 3000 |
| no | HTTP_ALL_INTERFACES | Listen on all interfaces, useful for reverse proxy, HTTP only | NO DEFAULT |

Note: If you do NOT set `HTTP_ALL_INTERFACES`, the 3000 port will only get listened inside container.

This image additional specified details:

| MANDATORY | Name | Explanation | Default Value |
|:--------:|:--------:|:--------:|:--------:|
| no | MYDOMAIN | generate TLS certs on the fly (if not exists) | ztncui.docker.test |
| no | ZTNCUI_PASSWD | generate admin password on the fly (if not exists) | password |
| YES | MYADDR | your ip address, public ip address preferred | NO DEFAULT |

Also, this image exposed an http server at port 3180, you could save file in `/mydata/ztncui/httpfs/` to serve it. 
(You could use this to build your own root server and distribute planet file, even though, that won't hurt you, I still suggest to set a protection for this http server in front)

**WARNING: IF YOU DO NOT SET PASSWORD, YOU HAVE TO USE `docker exec -it <CONTAINER NAME> bash`, and then `cat /var/log/docker-ztncui.log` to get your random password. This is gatekeeper.**

## Chinese users only

This script use https:///ip.sb for public IP detection purpose, which is blocked in some area of China Mainland. Under this circumstance, the program will try to detect public IP using `ifconfig` tool and might lead to unwanted result, to prevent this, make sure you set `MYADDR` environment variable when docker container is up.

The upstream repo (https://github.com/kmahyyg/ztncui-aio) only accept Issues and PRs in English. Other languages will be closed directly without any further notice. If you come from some non-English countries, use Google Translate, and state that at the beginning of the text body.

