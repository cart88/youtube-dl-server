# 我的心得
配置文件里面添加，可以增加配置
proxy: http://admin:654321@10.100.24.221:10811
升级youtube-dl的方法

/usr/bin/youtube-dl:/usr/local/bin/youtube-dl
![Docker Stars Shield](https://img.shields.io/docker/stars/nbr23/youtube-dl-server.svg?style=flat-square)
![Docker Pulls Shield](https://img.shields.io/docker/pulls/nbr23/youtube-dl-server.svg?style=flat-square)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/nbr23/youtube-dl-server/master/LICENSE)

# youtube-dl-server

Simple Web and REST interface for downloading youtube videos onto a server.
[`starlette`](https://www.starlette.io/) +
[`youtube-dl`](https://github.com/rg3/youtube-dl)

Forked from [manbearwiz/youtube-dl-server](https://github.com/manbearwiz/youtube-dl-server).

![screenshot][1]


![screenshot][2]

## Running

For easier deployment, a docker image is available on
[dockerhub](https://hub.docker.com/r/nbr23/youtube-dl-server):

- `nbr23/youtube-dl-server:latest` to use `youtube-dl`
- `nbr23/youtube-dl-server:yt-dlp` to use `yt-dlp`

### Ansible

The ansible role
[ansible-role-youtubedl-server](https://github.com/nbr23/ansible-role-youtubedl-server)
deploys `youtube-dl-server` using `docker` and `nginx`.

### Docker CLI

This example uses the docker run command to create the container to run the
app. Note the `-v` argument to specify the volume and its binding on the host.
This directory will be used to output the resulting videos.

```shell
docker run -d --name youtube-dl -p 8080:8080 -v $HOME/youtube-dl:/youtube-dl nbr23/youtube-dl-server:latest
```

OR for yt-dlp:

```shell
docker run -d --name youtube-dl -p 8080:8080 -v $HOME/youtube-dl:/youtube-dl nbr23/youtube-dl-server:yt-dlp
```

### Docker Compose

This is an example service definition that could be put in `docker-compose.yml`.

```yml
  youtube-dl:
    image: "nbr23/youtube-dl-server:latest"
    volumes:
      - $HOME/youtube-dl:/youtube-dl
      - ./config.yml:/app_config/config.yml:ro # Overwrite the container's config file with your own configuration
    restart: always
```
### HomeAssistant Addon

In the addons store add the repository (https://github.com/unofficial-skills/addons) or click the buton [![adds addon repo.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Funofficial-skills%2Faddons).

Once the addon repository has been added, you should be able to install (YouTube Dl Server) from the Unofficial-skills section.

## Configuration

Configuration is done through the config.yml file at the root of the project.

An alternate configuration path or file path can be forced by setting the environment
variable `YDL_CONFIG_PATH`:

```shell
export YDL_CONFIG_PATH=/var/local/youtube-dl-server/config.yml
```

In the above case, if `/var/local/youtube-dl-server/config.yml` does not exist, it will be created with the default options.

```shell
export YDL_CONFIG_PATH=/var/local/youtube-dl-server/
```

In the above case, if `/var/local/youtube-dl-server/config.yml` does not exist, it will be created with the default options as well.

The configuration file must contain at least the following variables:

```yaml
ydl_server:
  port: 8080
  host: 0.0.0.0
  metadata_db_path: '/youtube-dl/.ydl-metadata.db'

ydl_options:
  output: '/youtube-dl/%(title)s [%(id)s].%(ext)s'
  cache-dir: '/youtube-dl/.cache'
```

Additional youtube-dl parameters can be set in the `ydl_options` sections. To
do this, simply add regular youtube-dl parameters, removing the leading `--`.

For example, to write subtitles in spanish, the youtube-dl command would be:

`youtube-dl --write-sub --sub-lang es URL`

Which would translate to the following `ydl_options` in `config.yml`:

```yaml
ydl_options:
  output: '/tmp/youtube-dl/%(title)s [%(id)s].%(ext)s'
  cache-dir: '/tmp/youtube-dl/.cache'
  write-sub: True
  sub-lang: es
```

## Python

If you have python ^3.3.0 installed in your PATH you can simply run like this,
providing optional environment variable overrides inline.

Install the python dependencies from `requirements.txt`:

```shell
pip install -r requirements.txt
```

OR, if using `yt-dlp`:

```shell
pip install -r requirements-yt-dlp.txt
```

You can run
[bootstrap.sh](https://github.com/nbr23/youtube-dl-server/blob/master/bootstrap.sh)
to download the required front-end libraries (jquery, bootstrap).

```shell
python3 -u ./youtube-dl-server.py
```

To force a specific `youtube-dl` version/fork  (eg `youtube-dlc` or `yt-dlp`), use the
variable `YOUTUBE_DL`:

```shell
YOUTUBE_DL=yt-dlp python3 -u ./youtube-dl-server.py
```

## Usage

### Start a download remotely

Downloads can be triggered by supplying the `{{url}}` of the requested video
through the Web UI or through the REST interface via curl, etc.

### HTML

Just navigate to `http://{{host}}:8080/` and enter the requested `{{url}}`.

### Curl

```shell
curl -X POST  -H 'Content-Type: application/json' --data-raw '{"url":"{{ URL }}","format":"video/best"}' http://{{host}}:8080/api/downloads
```

### Fetch

```javascript
fetch(`http://${host}:8080/api/downloads`, {
  method: "POST",
  body: new URLSearchParams({
    url: url,
    format: "bestvideo"
  }),
});
```

### Bookmarklet

Add the following bookmarklet to your bookmark bar so you can conviently send
the current page url to your youtube-dl-server instance.

#### HTTPS

If your youtube-dl-server is served through https (behind a reverse proxy
handling https for example), you can use the following bookmarklet:

```javascript
javascript:fetch("https://${host}/api/downloads",{body:JSON.stringify({url:window.location.href,format:"bestvideo"}),method:"POST",headers:{'Content-Type':'application/json'}});
```

#### Plain text

If you are hosting it without HTTPS, the previous bookmarklet will likely be
blocked by your browser (as it will generate mixed content when used on HTTPS
sites).

Instead, you can use the following bookmarklet:

```javascript
javascript:(function(){document.body.innerHTML += '<form name="ydl_form" method="POST" action="http://${host}/api/downloads"><input name="url" type="url" value="'+window.location.href+'"/></form>';document.ydl_form.submit()})();
```

## Notes

### Support extra formats

`ffmpeg` is required for format conversion and audio extraction in some
scenarios.

## Implementation

The server uses [`starlette`](https://www.starlette.io/) for the web framework
and [`youtube-dl`](https://github.com/rg3/youtube-dl) to handle the
downloading. The integration with youtube-dl does NOT use their [python
api](https://github.com/rg3/youtube-dl#embedding-youtube-dl) anymore, but calls
the youtube-dl binary directly as the API doesn't provide easy ways to handle
common arguments.

This docker image is based on
[`python:alpine`](https://registry.hub.docker.com/_/python/) and consequently
[`alpine`](https://hub.docker.com/_/alpine/).

[1]:youtube-dl-server.png
[2]:youtube-dl-server-logs.png
