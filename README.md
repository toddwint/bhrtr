# toddwint/bhrtr

## Info

`bhrtr` docker image for simple lab testing applications.

Docker Hub: <https://hub.docker.com/r/toddwint/bhrtr>

GitHub: <https://github.com/toddwint/bhrtr>


## Overview

- Download the docker image and github files.
- Configure the settings in `run/config.txt`.
- Start a new container by running `run/create_container.sh`. 
  - The folder `upload` will be created as specified in the `create_container.sh` script.
- An example CSV file `bkhl.csv` is created in the `upload` volume on the first run.
- Fill in the file `upload/bkhl.csv`.
- Modify it as you need, and place it back in the same folder with the same name.
  - Additional columns can be added after the last column.
- Trigger the container to update by restarting it with `./restart.sh` or `./stop.sh` and `./start.sh`. 
  - You can also run `./delete_container` followed by `./create_container` as the `upload` folder will not be removed automatically.
- Open the file webadmin.html to view messages in a web browser.


## Features

- Ubuntu base image
- Plus:
  - iputils-ping
  - iputils-arping
  - tmux
  - python3-minimal
  - iproute2
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


## Sample `config.txt` file

```
# To get a list of timezones view the files in `/usr/share/zoneinfo`
TZ=UTC

# The interface on which to set the IP. Run `ip -br a` to see a list
INTERFACE=eth0

# The IP address that will be set on the docker container
IPADDR=169.254.255.254

# The IP address that will be set on the host to manage the docker container
MGMTIP=169.254.255.253

# The IP subnet in the form NETWORK/PREFIX
SUBNET=169.254.255.240/28

# The IP of the gateway. 
# Don't leave blank. Enter a valid ip from the subnet range
GATEWAY=169.254.255.241

# The ports for web management access of the docker container.
# ttyd tail, ttyd tmux, frontail, and tmux respectively
HTTPPORT1=8080
HTTPPORT2=8081
HTTPPORT3=8082
HTTPPORT4=8083

# The hostname of the instance of the docker container
HOSTNAME=bhrtr01
```


## Sample docker run script

```
#!/usr/bin/env bash
REPO=toddwint
APPNAME=bhrtr
HUID=$(id -u)
HGID=$(id -g)
SCRIPTDIR="$(dirname "$(realpath "$0")")"
source "$SCRIPTDIR"/config.txt

# Make the macvlan needed to listen on ports
# Set the IP on the host and add a route to the container
docker network create -d macvlan --subnet="$SUBNET" --gateway="$GATEWAY" \
  --aux-address="mgmt_ip=$MGMTIP" -o parent="$INTERFACE" \
  "$HOSTNAME"
sudo ip link add "$HOSTNAME" link "$INTERFACE" type macvlan mode bridge
sudo ip addr add "$MGMTIP"/32 dev "$HOSTNAME"
sudo ip link set "$HOSTNAME" up
sudo ip route add "$IPADDR"/32 dev "$HOSTNAME"

# Create the docker container
docker run -dit \
    --name "$HOSTNAME" \
    --network "$HOSTNAME" \
    --ip $IPADDR \
    -h "$HOSTNAME" \
    ` # Volume can be changed to another folder. For Example: ` \
    ` # -v /home/"$USER"/Desktop/upload:/opt/"$APPNAME"/upload \ ` \
    -v "$SCRIPTDIR"/upload:/opt/"$APPNAME"/upload \
    -p "$IPADDR":"$HTTPPORT1":"$HTTPPORT1" \
    -p "$IPADDR":"$HTTPPORT2":"$HTTPPORT2" \
    -p "$IPADDR":"$HTTPPORT3":"$HTTPPORT3" \
    -p "$IPADDR":"$HTTPPORT4":"$HTTPPORT4" \
    -e TZ="$TZ" \
    -e MGMTIP="$MGMTIP" \
    -e GATEWAY="$GATEWAY" \
    -e HUID="$HUID" \
    -e HGID="$HGID" \
    -e HTTPPORT1="$HTTPPORT1" \
    -e HTTPPORT2="$HTTPPORT2" \
    -e HTTPPORT3="$HTTPPORT3" \
    -e HTTPPORT4="$HTTPPORT4" \
    -e HOSTNAME="$HOSTNAME" \
    -e APPNAME="$APPNAME" \
    --cap-add=NET_ADMIN \
    ${REPO}/${APPNAME}
```


## Login page

Open the `webadmin.html` file.

- Or just type in your browser: 
  - `http://<ip_address>:<port1>` or
  - `http://<ip_address>:<port2>` or
  - `http://<ip_address>:<port3>`
  - `http://<ip_address>:<port4>`
