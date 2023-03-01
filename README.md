

[Smokeping](https://oss.oetiker.ch/smokeping/) keeps track of your network latency. 

[![smokeping](https://camo.githubusercontent.com/e0694ef783e3fd1d74e6776b28822ced01c7cc17/687474703a2f2f6f73732e6f6574696b65722e63682f736d6f6b6570696e672f696e632f736d6f6b6570696e672d6c6f676f2e706e67)](https://oss.oetiker.ch/smokeping/)

## Application Setup

* Once running the URL will be `http://<host-ip>/smokeping/smokeping.cgi`. For example a full URL might look like `https://smokeping.yourdomain.com/smokeping/smokeping.cgi`.
* Basics are, edit the `Targets` file to ping the hosts you're interested in to match the format found there.
* Wait 10 minutes.

## Usage

Here are some example snippets to help you get started creating a container.

### docker build 

```
docker build --no-cache -t DOCKER_HUB_USER/smokeping:latest .
```


### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
version: "2.1"
services:
  smokeping:
    image: davarski/smokeping:latest
    container_name: smokeping
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /path/to/smokeping/config:/config
      - /path/to/smokeping/data:/data
    ports:
      - 80:80
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=smokeping \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 80:80 \
  -v /path/to/smokeping/config:/config \
  -v /path/to/smokeping/data:/data \
  --restart unless-stopped \
  davarski/smokeping:latest

```
### Example (using ./docker-compose.yaml and ./config)

```
docker-compose up -d
```
<img src="pictures/smokeping-rrdtool-charts.png?raw=true" width="800">

<img src="pictures/smokeping-rrdtool.png?raw=true" width="800">


## Parameters

Container images are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 80` | Allows HTTP access to the internal webserver. |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Etc/UTC` | specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `-v /config` | Configure the `Targets` file here |
| `-v /data` | Storage location for db and application data (graphs etc) |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

Will set the environment variable `PASSWORD` based on the contents of the `/run/secrets/mysecretpassword` file.

## Umask for running applications

For smokeping image we have the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask).

## User / Group Identifiers

When using volumes (`-v` flags) permissions issues can arise between the host OS and the container. Avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```bash
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

## Support Info

* Shell access whilst the container is running: `docker exec -it smokeping /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f smokeping`
* container version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' smokeping`
* image version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' davarski/smokeping:latest`

