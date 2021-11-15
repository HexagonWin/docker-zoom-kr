This project is fully inspired of [sameersbn](https://github.com/sameersbn) [Skype](https://github.com/sameersbn/docker-skype)'s containerization.

# K-ZOOM
```
 _  __    ________   ___  __  __ 
| |/ /   |__  / _ \ / _ \|  \/  |
| ' /_____ / / | | | | | | |\/| |
| . \_____/ /| |_| | |_| | |  | |
|_|\_\   /____\___/ \___/|_|  |_|
   [ KOREAN ZOOM FOR DOCKER ]    
```

![K-ZOOM on Debian GNU/Linux with CDE on systemd](https://github.com/HexagonWin/docker-zoom-kr/blob/master/screenshot.png)

# 서론
GNU/리눅스 환경에서 Zoom을 Docker 내에서 실행하기 위한 Dockerfile입니다.

이 이미지는 Zoom에서 오디오 기능을 활성화 하기 위하여 [X11](http://www.x.org) 및 [Pulseaudio](http://www.freedesktop.org/wiki/Software/PulseAudio/) unix domain socket을 사용합니다.

## 기여

만약 이 이미지가 도움이 되었다면, 도와주세요.
- 업스트림 기능의 패치라면 mdouchement 님의 업스트림 레포지토리에 기여해주세요.
- 제가 메인테이닝 하는 해당 다운스트림 레포지토리의 패치 위에 변경사항은 다운스트림 레포지토리에 기여해주세요.
- 다운스트림 및 업스트림 레포지토리의 Issues 탭에서 문제를 가지고 있는 다른 사람들을 도와주세요.

# 시작하기

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
  hexagonwin/zoom-kr:latest install
```

이 명령어는  `zoom` 으로 실행 가능한 래퍼 스크립트를 생성, 설치합니다.

> **주의**
>
> 만약 호스트에 Zoom이 이미 설치 되어있다면, Docker의 Zoom 대신 호스트의 Zoom 바이너리가 실행될것입니다. 컨테이너 내의 Zoom 실행을 강제하려면 `zoom-kr-wrapper` 스크립트를 사용하세요.

## Web Browser / SSO

Add the following option in `~/.config/zoomus.conf`
```
embeddedBrowserForSSOLogin=false
```

Zoom will spawn Iceweasel (Firefox) included in this image and open SSO provider web page.

## 어떻게 동작하나요?

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


# 관리

## Upgrading

> <설치> 부분에서 다뤘던 명령어를 사용해주세요.

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

## 쉘 접근

For debugging and maintenance purposes you may want access the containers shell. If you are using Docker version `1.3.0` or higher you can access a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it zoomus bash
```

## 문제 해결

* 다음 로그들은 K-Zoom을 실행할 때, 자동으로 실행됩니다.

- Zoom basic logs:
  - `docker logs zoomus` (핑크색 터미널)
  - `ls -l ${ZOOM_HOME:=$HOME}/.zoom/logs` where you can find application logs. (민트색 터미널)
- Screen sharing:
  - Try `xhost +SI:localuser:"$USER"` [#20](https://github.com/mdouchement/docker-zoom-kr/issues/20)
- Transparent login form:
  - A possible workaround is to right click on the Zoom icon from the taskbar and select "About". The about popup could fix the login form render.
  - If `Unrecognized OpenGL version` error is in the logs, it could be an issue with Nvidia drivers [#1](https://github.com/mdouchement/docker-zoom-kr/issues/1)
    - @mzcu made a fix [mzcu@ee177a5](https://github.com/mzcu/docker-zoom-kr/commit/ee177a5e8915a05a51080301996a8ed4b89552ee) that should work
    - I recommend you to try his version: https://github.com/mzcu/docker-zoom-kr
