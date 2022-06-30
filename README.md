<p align="center">
<img src="logo.png" alt="Logo" />
</p>

<h1 align="center">Dockerize-Radicale</h1>


<p align="center">
Enhanced Docker image for <a href="http://radicale.org">Radicale</a>, the CalDAV/CardDAV server.
</p>

## Changelog

:page_with_curl: See [CHANGELOG.md](CHANGELOG.md)

## Latest version
## Running

### Option 1: **Basic** instruction

```
docker run -d --name radicale \
    -p 5232:5232 \
    -v ./data:/data \
    s101/docker-radicale
```

### Option 2: **Recommended, Production-grade** instruction (secured, safe...) :rocket:

This is the most secured instruction:

```
docker run -d --name radicale \
    -p 127.0.0.1:5232:5232 \
    --init \
    --read-only \
    --security-opt="no-new-privileges:true" \
    --cap-drop ALL \
    --cap-add CHOWN \
    --cap-add SETUID \
    --cap-add SETGID \
    --cap-add KILL \
    --pids-limit 50 \
    --memory 256M \
    --health-cmd="curl --fail http://localhost:5232 || exit 1" \
    --health-interval=30s \
    --health-retries=3 \
    -v ./data:/data \
    s101/docker-radicale
```

A [Docker compose file](docker-compose.yml) is included.

Note on capabilities:
- `CHOWN` is used to restore the permission of the `data` directory. Remove this if you do not need the `chown` to be run (see [below](#volumes-versus-bind-mounts))
- `SETUID` and `SETGID` are used to run radicale as the less privileged `radicale` user (with su-exec), and are required.
- `KILL` is to allow Radicale to exit, and is required.

## Custom configuration

To customize Radicale configuration, first get the config file:

* (Recommended) use this repository preconfigured [config file](config),
* Or, use [the original Radicale config file](https://raw.githubusercontent.com/Kozea/Radicale/master/config) and:
  1. set `hosts = 0.0.0.0:5232`
  1. set `filesystem_folder = /data/collections`

Then:
1. create a config directory (eg. `mkdir -p /my_custom_config_directory`)
2. copy your config file into the config folder (eg. `cp config /my_custom_config_directory/config`)
3. mount your custom config volume when running the container: `-v /my_custom_config_directory:/config:ro`.
The `:ro` at the end make the volume read-only, and is more secured.

## Volumes versus Bind-Mounts

This section is related to the error message `chown: /data: Permission denied`.

With [Docker volumes](https://docs.docker.com/storage/volumes/), and not [bind-mounts](https://docs.docker.com/storage/bind-mounts/) like shown in the examples above, you may need to disable the container trying to make the `data` directory writable.

This is done with the `TAKE_FILE_OWNERSHIP` environment variable.  
The variable will tell the container to perform or skip the `chown` instruction.  
The default value is `true`: the container will try to make the `data` directory writable to the `radicale` user.  

To disable the `chown`, declare the variable like this:

```
docker run -d --name radicale s101/docker-radicale \
    -e "TAKE_FILE_OWNERSHIP=false"
```

## Running with Docker compose

A [Docker compose file](docker-compose.yml) is included. 
It can also be [extended](https://docs.docker.com/compose/production/#modify-your-compose-file-for-production). 

## Multi-architecture

The correct image type for your architecture will be automatically selected by Docker, whether it is amd64, arm64 or armv7 (Raspberry Pi).
