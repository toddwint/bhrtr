---
title: README
author: Todd Wintermute
date: 2023-12-21
---

# toddwint/bhrtr


## Info

`bhrtr` docker image for simple lab testing applications.

Docker Hub: <https://hub.docker.com/r/toddwint/bhrtr>

GitHub: <https://github.com/toddwint/bhrtr>


## Overview

Docker image for a quick single physical interface router which will create /32 routes IPs specified.

Pull the docker image from Docker Hub or, optionally, build the docker image from the source files in the `build` directory.

Create and run the container using `docker run` commands, `docker compose` commands, or by downloading and using the files here on github in the directories `run` or `compose`.

**NOTE: A volume named `upload` is created the first time the container is started. Modify the file in that directory. Add a name for the peer device, the IP of the peer device, the gateway of the peer device (bhrtr's IP), and the network subnet mask. Then restart the container.**

Manage the container using a web browser. Navigate to the IP address of the container and one of the `HTTPPORT`s.

**NOTE: Network interface must be UP i.e. a cable plugged in.**

Example `docker run` and `docker compose` commands as well as sample commands to create the macvlan are below.


## Features

- Ubuntu base image
- Plus:
  - fzf
  - iproute2
  - iputils-arping
  - iputils-ping
  - python3-minimal
  - tmux
  - tzdata
  - [ttyd](https://github.com/tsl0922/ttyd)
    - View the terminal in your browser
  - [frontail](https://github.com/mthenw/frontail)
    - View logs in your browser
    - Mark/Highlight logs
    - Pause logs
    - Filter logs
  - [tailon](https://github.com/gvalkov/tailon)
    - View multiple logs and files in your browser
    - User selectable `tail`, `grep`, `sed`, and `awk` commands
    - Filter logs and files
    - Download logs to your computer


## Sample commands to create the `macvlan`

Create the docker macvlan interface.

```bash
docker network create -d macvlan --subnet=169.254.255.240/28 --gateway=169.254.255.241 \
    --aux-address="mgmt_ip=169.254.255.253" -o parent="eth0" \
    --attachable "bhrtr01"
```

Create a management macvlan interface.

```bash
sudo ip link add "bhrtr01" link "eth0" type macvlan mode bridge
sudo ip link set "bhrtr01" up
```

Assign an IP on the management macvlan interface plus add routes to the docker container.

```bash
sudo ip addr add "169.254.255.253/32" dev "bhrtr01"
sudo ip route add "169.254.255.240/28" dev "bhrtr01"
```

## Sample `docker run` command

```bash
docker run -dit \
    --name "bhrtr01" \
    --network "bhrtr01" \
    --ip "169.254.255.254" \
    -h "bhrtr01" \
    -v "${PWD}/upload:/opt/bhrtr/upload" \
    -p "169.254.255.254:8080:8080" \
    -p "169.254.255.254:8081:8081" \
    -p "169.254.255.254:8082:8082" \
    -p "169.254.255.254:8083:8083" \
    -e TZ="UTC" \
    -e MGMTIP="169.254.255.253" \
    -e GATEWAY="169.254.255.241" \
    -e HUID="1000" \
    -e HGID="1000" \
    -e HTTPPORT1="8080" \
    -e HTTPPORT2="8081" \
    -e HTTPPORT3="8082" \
    -e HTTPPORT4="8083" \
    -e HOSTNAME="bhrtr01" \
    -e APPNAME="bhrtr" \
    --cap-add=NET_ADMIN \
    "toddwint/bhrtr"
```


## Sample `docker compose` (`compose.yaml`) file

```yaml
name: bhrtr01

services:
  bhrtr:
    image: toddwint/bhrtr
    hostname: bhrtr01
    ports:
        - "169.254.255.254:8080:8080"
        - "169.254.255.254:8081:8081"
        - "169.254.255.254:8082:8082"
        - "169.254.255.254:8083:8083"
    networks:
        default:
            ipv4_address: 169.254.255.254
    environment:
        - MGMTIP=169.254.255.253
        - GATEWAY=169.254.255.241
        - HUID=1000
        - HGID=1000
        - HOSTNAME=bhrtr01
        - TZ=UTC
        - HTTPPORT1=8080
        - HTTPPORT2=8081
        - HTTPPORT3=8082
        - HTTPPORT4=8083
        - APPNAME=bhrtr
    privileged: true
    cap_add:
      - NET_ADMIN
    volumes:
      - "${PWD}/upload:/opt/bhrtr/upload"
    tty: true

networks:
    default:
        name: "bhrtr01"
        external: true
```
