This project is fully inspired of [sameersbn](https://github.com/sameersbn) [Skype](https://github.com/sameersbn/docker-skype)'s containerization.

# hexagonwin/zoom-kr-KR

# Introduction
GNU/리눅스 환경에서 Zoom을 Docker 내에서 실행하기 위한 Dockerfile입니다.

이 이미지는 Zoom에서 오디오 기능을 활성화 하기 위하여 [X11](http://www.x.org) 및 [Pulseaudio](http://www.freedesktop.org/wiki/Software/PulseAudio/) unix domain socket을 사용합니다.

## Contributing

만약 이 이미지가 도움이 되었다면, 도와주세요.
- 업스트림 기능의 패치라면 mdouchement 님의 업스트림 레포지토리에 기여해주세요.
- 제가 메인테이닝 하는 해당 다운스트림 레포지토리의 패치 위에 변경사항은 다운스트림 레포지토리에 기여해주세요.
- 다운스트림 및 업스트림 레포지토리의 Issues 탭에서 문제를 가지고 있는 다른 사람들을 도와주세요.

# Getting started

## Installation

~~Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/mdouchement/zoom-kr) and is the recommended method of installation.~~
* 해당 다운스트림 포크는 자동 빌드 이미지가 존재하지 않습니다. *

직접 빌드를 하는 명령어는 다음과 같습니다.

```bash
docker build -t hexagonwin/zoom-kr github.com/hexagonwin/docker-zoom-kr
```

로컬 이미지가 생성된 이후, 다음 명령어를 수퍼유저로 실행하여 래퍼 스크립트를 설치해주세요.

```bash
docker run -it --rm \
  --volume /usr/local/bin:/target \
  mdouchement/zoom-kr:latest install
```

이 명령어는  `zoom` 으로 실행 가능한 래퍼 스크립트를 생성, 설치합니다.

> **Note**
>
> If Zoom is installed on the the host then the host binary is launched instead of starting a Docker container. To force the launch of Zoom in a container use the `zoom-kr-wrapper` script. For example, `zoom-kr-wrapper zoom` will launch Zoom inside a Docker container regardless of whether it is installed on the host or not.

## Web Browser / SSO

Add the following option in `~/.config/zoomus.conf`
```
embeddedBrowserForSSOLogin=false
```

Zoom will spawn Iceweasel (Firefox) included in this image and open SSO provider web page.

## How it works

The wrapper scripts volume mount the X11 and pulseaudio sockets in the launcher container. The X11 socket allows for the user interface display on the host, while the pulseaudio socket allows for the audio output to be rendered on the host.

When the image is launched the following directories are mounted as volumes

- `${HOME}/.zoom`
- `${HOME}/.config`
- `XDG_DOWNLOAD_DIR` or if it is missing `${HOME}/Downloads`
- `XDG_DOCUMENTS_DIR` or if it is missing `${HOME}/Documents`

This makes sure that your profile details are stored on the host and files received via Zoom are available on your host in the appropriate download directory.

**Don't want to expose host's folders to Zoom?**

Add `ZOOM_HOME` environment variable to namespace all Zoom folders:

```sh
export ZOOM_HOME=${HOME}/zoomus
```


# Maintenance

## Upgrading

To upgrade to newer releases:

  1. Download the updated Docker image:

  ```bash
  docker pull mdouchement/zoom-kr:latest
  ```

  2. Run `install` to make sure the host scripts are updated.

  ```bash
  docker run -it --rm \
    --volume /usr/local/bin:/target \
    mdouchement/zoom-kr:latest install
  ```

## Uninstallation

```bash
docker run -it --rm \
  --volume /usr/local/bin:/target \
  mdouchement/zoom-kr:latest uninstall
```

## Shell Access

For debugging and maintenance purposes you may want access the containers shell. If you are using Docker version `1.3.0` or higher you can access a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it zoomus bash
```

## Troubleshooting

- Zoom basic logs:
  - `docker logs zoomus`
  - `ls -l ${ZOOM_HOME:=$HOME}/.zoom/logs` where you can find application logs.
- Screen sharing:
  - Try `xhost +SI:localuser:"$USER"` [#20](https://github.com/mdouchement/docker-zoom-kr/issues/20)
- Transparent login form:
  - A possible workaround is to right click on the Zoom icon from the taskbar and select "About". The about popup could fix the login form render.
  - If `Unrecognized OpenGL version` error is in the logs, it could be an issue with Nvidia drivers [#1](https://github.com/mdouchement/docker-zoom-kr/issues/1)
    - @mzcu made a fix [mzcu@ee177a5](https://github.com/mzcu/docker-zoom-kr/commit/ee177a5e8915a05a51080301996a8ed4b89552ee) that should work
    - I recommend you to try his version: https://github.com/mzcu/docker-zoom-kr
