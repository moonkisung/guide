# Docker Guide

---
tags:
- linux/docker
- linux/dockerfile
---


[[Ubuntu Guide, Kor]]




# 0. 도커 설치

공식 사이트를 통해 명령어를 복붙하면 된다.

```Shell
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```Shell
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```Shell
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```Shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```


```shell
sudo usermod -aG docker [사용자] //도커 권한 추가
```





## Quick Guide

### 도커 생성

```shell
#1회용 (서버 종료 시 컨테이너 삭제 됨)
docker run -it --rm --gpus all -v /hdd/ym/project/huiino/data:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter

#계속해서 유지되는 컨테이너
docker run -it --gpus all -v /hdd/ym/project/huiino/data:/tf/notebooks --name ym_ct1 -p 8889:8889 -e JUPYTER_PORT=8889 tensorflow/tensorflow:2.5.0-gpu-jupyter
#포트 변경 시 -p 8889:8889 -e JUYPTER_PORT=8889 부분 변경
#서버 허용 포트는 8888-8891까지임
#연결하고 싶은 데이터는 -v /서버 데이터 경로:/tf/notebooks 로 설정
#--name에서 본인이 원하는 container 이름으로 변경


#견본
docker run -it --gpus all --name hjinny -v /hdd/hjinny:/tf/notebooks -p 8888:8888 -e JUPYTER_PORT=8888 tensorflow/tensorflow:2.5.0-gpu-jupyter

docker run -it --gpus all --name kisung -v /hdd/kisung:/tf/notebooks -p 8890:8890 -e JUPYTER_PORT=8890 tensorflow/tensorflow:2.5.0-gpu-jupyter

docker run -it --gpus all --name thomas -v /hdd/thomas:/tf/notebooks -p 8891:8891 -e JUPYTER_PORT=8891 tensorflow/tensorflow:2.5.0-gpu-jupyter


docker run -it --gpus '"device=0,1,2,3"' -p 8895:8888 --name eeg_jinny --shm-size=1G -v /mnt/nas/homes/hjinny:/tf enopus/tensorflow:2.9.1-gpu-anaconda-jupyter 

```



### 컨테이너 확인 및 실행하기

```shell
#컨테이너 확인하기
docker ps -a

#도커 시작하기
docker start ym_ct1

#도커 중지하기
docker stop ym_ct1 
혹은 Ctrl + C로 서버 정지
```


## REF

[도커 공식 설치 문서](https://docs.docker.com/engine/install/ubuntu/)

[[도커 엔진(Docker Engine)\] User guide (tutorial) - 컨테이너에서 hello world (tistory.com)](https://darksoulstory.tistory.com/454)

[초보를 위한 도커 안내서 - 도커란 무엇인가? (subicura.com)](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)







# 1. 도커의 이미지 소개

## 1-1. 오피셜 이미지

해당 프로그램이나 제작사에서 직접 제공하는 이미지.

컨테이너는 linux os를 기반으로 설치되어 배포되는데 리눅스도 버전이 굉장히 많기 때문에 어떤 리눅스를 베이스로 했느냐가 중요하다.

가장 대표적으로 쓰이는 리눅스 버전은 alphine(알파인)으로 용량이 매우 가벼워 대부분 알파인으로 빌드된 이미지가 많다.

중요한 것은 알파인이 기본적인 리눅스이긴 하지만 whl 파일을 지원하지 않기 때문에 파이썬을 돌릴 때는 tar.gz을 gcc로 번거롭게 빌드하여 상당한 오버헤드가 발생하므로 파이썬용으로는 부적합하다. 

파이썬으로는 무난하게 우분투 같은 것이 무난하다.



## 1-2. 개인 이미지

개인이 만든 이미지로 github와 같이 docker hub에 본인이 만든 개인 이미지를 올릴 수 있다.

이미지는 이미 만들어진 이미지에 이것저것 다운받은 뒤 다시 이미지로 만들어버리면 된다.



## 1-3. 이미지 다운

도커는 도커 hub에 이미지가 올라오는데 여기서 이미지를 받을 수 있다.

`docker pull` 명령어를 통해 다운을 받을 수 있는데 우선 login부터 해야 한다.

```shell
docker login

아이디와 비밀번호 입력
```

```shell
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

docker login 을 통해 아이디와 비밀번호를 입력하면 로그인이 된다.



```shell
docker pull [이미지id]
docker pull hello-world
```

```shell
b8dfde127a29: Pull complete
Digest: sha256:5122f6204b6a3596e048758cabba3c46b1c937a46b5be6225b835d091b90e46c
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

도커 hub에선 친절하게 명령어 복사 기능을 지원하고 있으므로 본인이 원하는 이미지를 복사해서 pull 로 다운받을 수 있다.



## 1-4. 이미지 태그

도커의 이미지는 어떤 리눅스 버전을 베이스로 하느냐 혹은 버전을 하는지에 따라 이미지가 바뀐다. 

이 모든 것을 다른 페이지로 관리하면 난잡하기 때문에 docker hub에는 보통 오피셜 이미지가 올라오고,  Tags에서 다른 버전을 다운 받을 수 있다.

예를 들어 tensorflow 오피셜 이미지의 태그를 일부 살펴보면 다음과 같다.

```
2.5.0-gpu-jupyter
docker pull tensorflow/tensorflow:2.5.0-gpu-jupyter

2.5.0-gpu
docker pull tensorflow/tensorflow:2.5.0-gpu

2.5.0
docker pull tensorflow/tensorflow:2.5.0
```

이미지 태그는 각 오피셜 이미지마다 다르기 때문에 본인이 쓰고자 하는 오피셜 이미지의 설명을 참고하자.



## 1-5. 이미지 저장 장소 변경

도커는 기본적으로 host 전체를 사용하므로 모든 사용자가 공유한다.

docker의 기본 root dir는 `/var/lib/docker` 이다.

이는 `/etc/docker/daemon.json` 을 수정하여 변경할 수 있다.

```
구버전
{
    "graph": "/path"
}


신버전
{
    "data-root": "/path"
}
```

이후 도커를 재시작한 뒤 `docker info`를 통해 바뀐 경로를 확인할 수 있다.

```
sudo service docker stop
sudo systemctl daemon-reload
sudo service docker start

docker info
```



# 2. 도커의 명령어

## 2-1. 기본 명령어

```shell
docker pull [image]
docker images # 이미지 확인
docker run [image] # 컨테이너 실행, 실행 시 docker pull도 같이 실행된다

docker ps # 작동중인 컨테이너 확인
docker ps -a # 모든 컨테이너 확인
docker ps -s # 컨테이너의 disk usage 확인
docker ps -as # 모든 컨테이너의 disk usage 확인

dokcer rm [id|name] # 컨테이너 삭제
docker rmi [image] # 이미지 삭제

docker start [container] # run과의 차이점은 run은 start + pull
docker stop [container]

docker attach [container] # 컨테이너 접속

docker exec [container] command # 해당 컨테이너에서 해당 명령어를 실행함
docker exec -it [container] /bin/bash # 컨테이너 내부로 진입, -it는 --interactive를 뜻함

```



## 2-2. run 명령어

`-i`: -i는 interactive로 표준 입력(stdin)을 활성화하여 컨테이너와 attach가 되지 않더라도 입력을 유지한다.

`-t`: tty는 컴퓨터와 상호작용을 위한 장비에서 왔는데 pseudo-TTY(TTY 모드)를 사용하는 것을 의미한다. Bash를 사용하려면 켜야하는 명령어로 켜지 않으면 명령을 입력할 수 있지만 셸이 표시되지 않는다.

`-p`: 포트설정. [호스트포트:컨테이너 포트],  `-p 8900:8888` 이라면 호스트의 8900포트가 컨테이너의 8888와 연결되어 있다는 뜻.

`--name`: 컨테이너에 이름을 붙일 때 사용

`-v`: 디렉토리를 마운트 할 때 쓰는 명령어로 데이터를 이미지에 넣고 싶을 때 사용한다. [host path:container path]

`--gpus all`: Host의 Nvidia 드라이버가 컨테이너에도 적용되게 함. `'"device=0,1,2,3"'` 과 같이 개별적인 gpu 할당도 가능

`-w`: WORKDIR를 덮어쓰기 위해 사용. 작업 디렉토리의 이름을 변경한다.

`--rm`: 컨테이너가 종료될 때 컨테이너에 관련된 리소스(파일 시스템, 볼륨)를 제거한다.

`--shm-size`: 공유 메모리를 설정한다. 자세한 내용은 공유 메모리 검색

`--restart`: 재시작 명령어. `no`, `on-failure[:max-retries]`, `always`, `unless-stopped` 4가지가 존재한다. `--restart==always`


## 2-3. exec 명령어

`-d`: --detach=false; 명령을 백그라운드로 실행. 서버에서 많이 사용.

`-it`: run과 동일



## 컨테이너 삭제

```shell
docker ps # 컨테이너 확인
docker ps -a # 모든 컨테이너 확인

docker rm [컨테이너id] 
docker rm [컨테이너id], [컨테이너id]
docker rm -f [컨테이너id] # 컨테이너 강제 삭제
docker ps -a
```



## 이미지 삭제

```shell
docker images # 현재 이미지 확인
docker rmi [이미지id]

docker rmi -f [이미지id] # 컨테이너까지 강제 삭제
```


## REF
[[Docker\] 도커 이미지와 컨테이너 삭제 방법 (brunch.co.kr)](https://brunch.co.kr/@hopeless/10)

[docker prune / 도커 사용 안하는 놈 제거하고 용량 확보하기 :: GM Yankee (tistory.com)](https://gmyankee.tistory.com/277)

[docker 컨테이너에서 GPU 사용 (tistory.com)](https://ykarma1996.tistory.com/92)

[Runtime options with Memory, CPUs, and GPUs | Docker Documentation](https://docs.docker.com/config/containers/resource_constraints/#gpu)

[Docker run 옵션 정리 (tistory.com)](https://nuggy875.tistory.com/49)

[[Docker Basic\] 16. Docker 데이터 저장 개념 / 기본 명령어 - volume :: Watch & Learn (tistory.com)](https://watch-n-learn.tistory.com/34)

[Docker 컨테이너에 데이터 저장 (볼륨/바인드 마운트) | Engineering Blog by Dale Seo](https://www.daleseo.com/docker-volumes-bind-mounts/)

[[Docker] run 옵션 (tistory.com)](https://itstarter.tistory.com/642)

[Docker run reference | Docker Documentation](https://docs.docker.com/engine/reference/run/#restart-policies---restart)



# 3. 도커 환경변수 바꾸기

```shell
docker inspect [컨테이너이름]

sudo su 
service docker stop
```

컨테이너 id 알아내기



````shell
/var/lib/docker/containers/[container-id]/config.v2.json
````

도커의 config 파일 변경



```shell
service dokcer start
docker strat [컨테이너이름]
```



## Update 명령어 사용하기

다만 update는 volume 추가 및 변경은 지원하지 않음

volume 추가 및 변경 시 commit을 통해 이미지로 바꾼 뒤 다시 run으로 컨테이너화 시킬 것

```shell
docker update [OPTION] [CONTAINER ID]
docker update --memory=100g ym_test
```


## REF

[Docker 저장소 위치 변경 – Hooni's Playground (hooni-playground.com)](https://hooni-playground.com/751/)

[실행중인 도커 컨테이너에 추가 포트 오픈하기?! (tistory.com)](https://ongamedev.tistory.com/entry/실행중인-도커-컨테이너에-추가-포트-오픈하기)

[실행중인 docker 컨테이너 환경변수 변경 (tistory.com)](https://behonestar.tistory.com/216)

[Docker 컨테이너에 자원 할당 및 관리 (tistory.com)](https://cumulus.tistory.com/35)

[[Docker Basic\] 15. Docker Container 자원 제한 :: Watch & Learn (tistory.com)](https://watch-n-learn.tistory.com/32)



# 4. 도커파일 

`주석(#)`: 파이썬과 동일하게 \#이 주석문.

`FROM`: 베이스가 될 이미지 파일. FROM <이미지>:<태그>로 사용한다.

`WORKDIR`: 작업할 디렉토리. 터미널에서 cd로 이동해서 명령을 실행하는 것과 동일하다.

`RUN`: bash에서 실행할 명령어. 정확하게는 도커파일로부터 도커 이미지를 빌드하는 순간에 실행되는 명령어를 뜻한다. 
통상적으로는 터미널 창에서 명령어 치는 것과 동일하다.   백슬래쉬( \\ ) 를 이용해 다음 줄로 넘어갈 수 있다.

`COPY`: 호스트 컴퓨터에 있는 디렉토리나 파일을 도커의 이미지 파일 시스템으로 복사하기 위해 사용된다. 절대경로와 상대경로를 모두 지원한다.

`ADD`: 일반 파일뿐만이 아니라 압축 파일이나 네트워크 상의 파일도 사용. 통상적으로는 COPY를 더 자주 사용함.

`ENTRYPOINT`: 컨테이너가 생서되고 최초로 실행될 때 수행되는 명령어를 지칭한다. CMD와 다른 점은 CMD는 `run` 명령어 사용 시 변경이 가능하나 ENTRYPOINT는 고정이다.

`CMD`: 컨테이너가 생서되고 최초로 실행될 때 수행되는 명령어를 지칭한다. 변경이 가능하다.

```dockerfile
# FROM에서 앞에 아무것도 붙이지 않으면 docker hub에서 이미지를 따오기 때문에 docker login 필요
# FROM tensorflow/tensorflow:2.9.1-gpu

# --platform=linux/amd64를 붙이면 로컬에 설치되어 있는 도커 이미지를 들고 온다
FROM --platform=linux/amd64 tensorflow/tensorflow:2.9.1-gpu

MAINTAINER ym
RUN cd root \
	curl -o conda.sh https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh

RUN	chmod 777 conda.sh \
	./conda.sh -b \
	conda install juypter -y \
    conda install numpy pandas scikit-learn matplotlib seaborn pyyaml h5py keras -y && \
    conda upgrade dask
	
RUN rm Anaconda3-5.3.1-Linux-x86_64.sh

COPY conf/.jupyter /root/.jupyter
COPY run_jupyter.sh /

VOLUME /tf/notebooks

CMD ["bash"]
```

위 dockerfile에서 `conda.sh -b` 뒤에 `-p`를 붙여서 설치 경로 설정도 가능하다.

ex) `./conda.sh -b -p /home/ym/`

또한 `export PATH="~/anaconda3/bin:$PATH"`는 재시작 시 초기화 되므로 `./bashrc` 파일을 생성해 이를 명시해야 컨테이너를 껐다켜도 `conda`가 제대로 작동한다.

이는 anaconda의 설치 방법 참조



## Anaconda - Jupyter 버전

```dockerfile
FROM --platform=linux/amd64 tensorflow/tensorflow:2.9.1-gpu
MAINTAINER ym

RUN su -
WORKDIR /root

RUN	mkdir /tf && \
  curl -o conda.sh https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh && \
	chmod 777 conda.sh && \
	./conda.sh -b && \
	echo "export PATH=\"~/anaconda3/bin:\$PATH\"" > ~/.bashrc && \
	source ~/.bashrc && \
	rm conda.sh && \
	pip install tensorflow-gpu==2.9.1 && \
	conda install jupyter -y
	
EXPOSE 8888 6006

CMD ["bash", "-c", "source ~/.bashrc && jupyter notebook --ip 0.0.0.0 --allow-root --port=8888 --notebook-dir=/tf"]
```

주피터 노트북에선 `!bash -c "conda command"` 사용



## Anaconda만 설치

```dockerfile
FROM --platform=linux/amd64 tensorflow/tensorflow:2.9.1-gpu
MAINTAINER ym

RUN	mkdir /tf && \
  curl -o conda.sh https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh && \
	chmod 777 conda.sh && \
	./conda.sh -b && \
	echo "export PATH=\"~/anaconda3/bin:\$PATH\"" > ~/.bashrc && \
	source ~/.bashrc && \
	rm conda.sh && \
	pip install tensorflow-gpu==2.9.1
	
EXPOSE 8888 6006

CMD ["bash"]
```



## PyTorch 설치 버전

```dockerfile
# 파이토치는 기본적으로 conda가 설치되어 있음
# 경로는 /opt/conda

FROM --platform=linux/amd64 pytorch/pytorch:1.12.0-cuda11.3-cudnn8-runtime
MAINTAINER ym
	
RUN	mkdir /tf && conda install jupyter -y
	
EXPOSE 8888 6006

CMD ["bash", "-c", "source ~/.bashrc && jupyter notebook --ip 0.0.0.0 --allow-root --port=8888 --notebook-dir=/tf"]
```





```shell
# --platform 확인 코드
docker inspect --format='{{.Os}}/{{.Architecture}}' [IMAGE_NAME]
```



## 도커 build

```shell
docker build -t [ID]/[Repository]:[Tag] [Dockerfile Path] . -f [Dockerfile Path]
```

Dockerfile은 확장자가 없고 Dockerfile이라는 이름이 default이다.

`-f`를 사용하면 Dockerfile이 아니더라도 build 할 수 있다.



## REF

[Dockerfile 여러개 두고 골라서 쓰는 방법 (tistory.com)](https://bscnote.tistory.com/109)

[How to build Anaconda Python Data Science Docker container (hands-on.cloud)](https://hands-on.cloud/how-to-build-python-data-science-docker-container-based-on-anaconda/)

[docker - How can I use a local image as the base image with a dockerfile? - Stack Overflow](https://stackoverflow.com/questions/20481225/how-can-i-use-a-local-image-as-the-base-image-with-a-dockerfile)

[초보를 위한 도커 안내서 - 이미지 만들고 배포하기 (subicura.com)](https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html)

[Dockerfile에서 자주 쓰이는 명령어 | Engineering Blog by Dale Seo](https://www.daleseo.com/dockerfile/)



# 5. 도커 Commit으로 이미지화

```shell
docker commit [Container ID|Container Name] [REPOSITROY[:TAG]]

sudo docker commit ym_ct1 test:0.01
sha256:527e9eddcf269ea0bfd6c38ad854c2eb9f2327dee7cecbe1658fd71244d91951


# Result
ym@kwon-System-Product-Name:/hdd/ym$ sudo docker images
REPOSITORY              TAG                 IMAGE ID       CREATED          SIZE
test                    0.01                527e9eddcf26   6 seconds ago    6.65GB
```



## 이미지의 Repository와 Tag 변경

```shell
docker image tag [previous imageName[:TAG]] [imageName[:TAG]]
```


## REF

[실행 중인 Docker 컨테이너를 파일로 저장하고 다시 불러오기 – ~/xo.dev –](https://xo.dev/export-and-import-docker-container/)

[도커 이미지 만들기 강의](https://www.youtube.com/watch?v=59P19DFB0sc)

[docker image commit 방법 (tistory.com)](https://cofs.tistory.com/405)

[[Docker\] docker commit 명령어 사용하는 방법! (tistory.com)](https://somjang.tistory.com/entry/Docker-docker-commit-명령어-사용하는-방법)

[docker push | Docker Documentation](https://docs.docker.com/engine/reference/commandline/push/)




# Docker sudo 명령어 없이 사용하기
특별히 sudo를 생략하는게 중요한 부분은 아니나 컨테이너 이름의 자동완성 때문이라도 권한을 올리는 것이 좋다.

```shell
sudo groupadd docker

sudo usermod -aG docker {userID}

sudo service docker restart

sudo systemctl reboot


# 에러 발생 시 /var/run/docker.sock의 권한을 666으로 변경할 것

```


# Nvidia driver install 및 Docker 세팅
[[Ubuntu Guide, Kor#Nvidia Drivier 세팅]]

![[Ubuntu Guide, Kor#Nvidia Drivier 세팅]]



# Nvidia Cuda / Tensorflow 

Nvidia는 도커로 어떻게 작동하는가?

[NVIDIA/nvidia-docker: Build and run Docker containers leveraging NVIDIA GPUs (github.com)](https://github.com/NVIDIA/nvidia-docker)

해당 사이트의 이미지에서도 확인할 수 있지만 가장 중요한 것은 host 전체에 *`nvidia driver`*가 설치되어 있어야 한다는 점이다.

그렇지만 cuda를 비롯한 부속 패키지는 설치할 필요가 전혀 없다.

cuda 등은 tensorflow 이미지에 몽땅 포함이 되어 있기 때문에 설치할 필요가 없으며 이것이 도커 설치의 가장 큰 이점이다.

특히 tensorflow의 버전에 따라 cuda 등의 버전도 바뀌어야 하는데 이를 docker를 사용하면 여러 버전을 사용할 수 있다.

```shell
docker run --gpus all nvidia/cuda:11.0-base nvidia-smi
```

우선 tensorflow를 돌리기 전에 해당 cuda 이미지를 통해 gpu가 제대로 잡히는지 확인하는 것이 좋다. 

해당 `nvidia/cuda:version-base`이미지는 GPU 지원의 유무를 확인할 뿐, 계속해서 작동하는 컨테이너가 아니므로 이 이미지로 무언가 할 수 있다는 생각은 하지말자.



```shell
docker run -it --rm tensorflow/tensorflow bash
//Start a CPU-only container

docker run -it --rm --runtime=nvidia tensorflow/tensorflow:latest-gpu python
//Start a GPU container, using the Python interpreter.
	
docker run -it --rm -v $(realpath ~/notebooks):/tf/notebooks -p 8888:8888 tensorflow/tensorflow:latest-jupyter
//Run a Jupyter notebook server with your own notebook directory (assumed here to be ~/notebooks). To use it, navigate to localhost:8888 in your browser.



```

도커를 이용한 tensorflow의 가장 빠른 사용은 주피터 노트북을 이용하는 것이다. 

이 때 주의사항으로는 --gpus all 을 명시해줘야 제대로 gpu가 잡힌다.

다른 로컴 컴퓨터에서 서버 컴퓨터의 주피터 노트북을 사용하고 싶을 땐 해당 서버 컴퓨터의 ip:8888로 접속한다.

서버의 종료 방법은 `Ctrl + C` 이지만 `Ctrl + Q` 혹은 `Ctrl + P`를 눌려서 서버를 종료하지 않고도 다시 host로 넘어오는 것이 가능하다.



```shell
docker run -it --rm --gpus all -v $(realpath ~/notebooks):/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter
docker run -it --rm --gpus all -v /hdd/ym/project/data:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter
docker run -it --rm --gpus all -v /hdd/hjinny:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter
docker run -it --rm --gpus all -v /hdd/kisung:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter
docker run -it --rm --gpus all -v /hdd/thomas:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.5.0-gpu-jupyter
```

실제 실행 코드는 위와 같다

```python
from tensorflow.python.client import device_lib
device_lib.list_local_devices()

#안 할 경우 0부터 사용하기 때문에 여러 명이 사용할 때는 번호를 지정하고 사용하는 것이 좋음
import os
os.environ["CUDA_DEVICE_ORDER"]="PCI_BUS_ID"
os.environ["CUDA_VISIBLE_DEVICES"]="1"
```

GPU 작동 여부는 위의 두 코드를 이용해 확인할 수 있다.



```python
import torch
torch.cuda.is_available()
torch.cuda.get_device_name(0)
torch.cuda.device_count()
```

토치 GPU 작동 여부는 위의 코드를 사용한다.



추가적으로 설치법은 이것저것 갈릴텐데 공식 문서를 보고 따라하자.

[도커 공식문서]([Runtime options with Memory, CPUs, and GPUs | Docker Documentation](https://docs.docker.com/config/containers/resource_constraints/#gpu))

[nvidia 공식 문서]([Installation Guide — NVIDIA Cloud Native Technologies documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker))

도커 공식문서에서는 `nvidia-docker2` 같은 패키지는 전혀 설치하지 않는데 아마 docker측에서 설치하지 않아도 돌아갈 수 있게 변경한 것으로 보인다.

즉 도커 공식문서를 그대로 따라해보고 안 되면 nvidia 공식문서대로 진행하면 된다.



# Docker UFW 미적용 문제


## Docker 보안 세팅
주로 Jupyter Notebook과 같이 컨테이너 독자적으로 port가 개방되는 경우 컨테이너는 독자적인 iptable 체인룰이 적용된다.

그러므로 이를 옵션에서 disable 하거나 컨테이너 별로 별개로 설정해야 해야 호스트에서 설정한 범용적인 체인룰이 적용된다.



## REF
[Docker 보안 하드닝 | 코마의 훈훈한 공간 (code-machina.github.io)](https://code-machina.github.io/2019/09/02/Docker-Security-Hardening.html)

[What is the best practice of docker + ufw under Ubuntu - Stack Overflow](https://stackoverflow.com/questions/30383845/what-is-the-best-practice-of-docker-ufw-under-ubuntu)






# Ubuntu Guide
---
tags:
- linux/ubuntu
---

[[How to Use Docker, Kor]]

# 우분투 설치 시

/boot 파티션은 별도로 나누면 향후 번거롭다. (옛날 방식)

최근에는 /(루트)의 용량이 크기 때문에 굳이 나눌 필요가 없음





# 소유권 설정

기본적인 소유권에 관한 설정은 chmod나 chown을 사용한다.  (ch = change의 약자)

그러나 chmod나 chown으로 설정할 수 있는 소유자 권한은 한정적이므로 더 많은 기능을 사용할 때는 ACL을 사용한다.



```shell
chown -R [owner name]:[group name][filename or directory]

ex)
chown -R ym /hdd/ym
```



```shell
chmod 777 [fileName]
```





# Sudo 권한 주기

```shell
usermod -aG sudo username
```
권한을 주고 난 뒤 로그아웃 → 로그인을 해야 제대로 sudo 권한이 적용됨


# 계정 관리


## 계정 생성

```shell
useradd는 전부 설정해줘야 함
sudo useradd [id]

adduser가 한 번에 셋팅 다 됨 (루트 폴더까지)
sudo adduser [id]

```


## 계정 목록 확인

```shell
cat /etc/passwd
cut -f1 -d: /etc/passwd
```


## 계정 삭제

```Shell
sudo userdel [username]
sudo userdel -rf [username] # 홈 디렉토리를 포함한 모든 정보 삭제

```



# curl 명령어

```shell
# 원본 파일 그대로 저장
curl -O https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh

#사용자 이름으로 파일명 저장
curl -o anaconda3.sh https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh

```

다운로드 시 `wget`와 같이 주로 사용

`wget`이 default로 설치되지 않은 것과는 달리 curl은 설치되어 있는 것으로 확인됨

`apt install curl`이 안 될 때는 `apt update`로 업데이트를 할 것.


# cat 명령어

`cat [fileName]` 해당 파일을 출력한다.

`cat > [fileName]` 파일을 생성한다. 안에 내용을 입력할 수 있는데 저장은 `ctrl+D`이다.



# echo 명령어

기본적으로 무언가를 출력하는 명령어지만 이를 파일로도 만들 수 있기 때문에 매우 자주 쓰인다.

`echo "string"` 기본적인 출력 명령어

`echo "string" > [fileName]` 



# watch 명령어
`nvidia-smi` 와 같이 실시간으로 확인이 필요한 명령어를 사용 시 `watch` 명령어를 사용한다.

```shell
# 1초 간격으로 nvidia-smi를 refresh 하여 나타냄
# 여기서 -d 는 변경 부분을 하이라이트 시켜 화면에 나타낸다.
# -n 은 --interval=<seconds>를 나타냄
watch -d -n 1 nvidia-smi
watch -d -n 5 'cat /proc/uptime'

```

## REF
[watch 명령을 이용한 linux 시스템 모니터링 | SharedIT - IT 지식 공유 네트워크](https://www.sharedit.co.kr/posts/2000)



# 디스크 용량 확인하기

```shell
#inode 남은 공간, 사용 공간, 사용 퍼센트 출력
df -i 

#사람이 읽을 수 있는 단위로 출력
df -h 

#1KB를 1000단위로 용량 표시
df -H 

du
```



# 의존성 문제

```shell
#의존성 문제 발생 시 의존성이 맞지 않는 부분 확인
sudo dpkg --configure -a

sudo apt-get --fix-broken install

#파티션이 꽉차면 autoremove 사용 (구버전이 있을 경우)
sudo apt-get autoremove

autoremove 
```



## /boot 파티션이 꽉 찼을 시

```shell
uname -a
Linux kwon-System-Product-Name 5.4.0-104-generic #118~18.04.1-Ubuntu SMP Thu Mar 3 13:53:15 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

#5.4.0-104-generic 이게 리눅스 버전
#이 버전을 제외한 나머지 리눅스 버전 삭제
apt-get remove linux-image-4.4.0-21-generic

#다만 이것도 /boot 디렉토리가 꽉 찼을 시 먹히지 않음
#이럴 땐 boot 디렉토리에 직접 접근할 것

cd /boot
ls
System.map-5.4.0-100-generic  config-5.4.0-100-generic  efi   initrd.img-5.4.0-100-generic  lost+found      memtest86+.elf            vmlinuz-5.4.0-100-generic
System.map-5.4.0-104-generic  config-5.4.0-104-generic  grub  initrd.img-5.4.0-104-generic  memtest86+.bin  memtest86+_multiboot.bin  vmlinuz-5.4.0-104-generic

#여기서 위 uname -a에서 나온 리눅스 버전과 맞지 않는 나머지 버전들의 4가지를 삭제해서 용량 확보
#<System.map-5.4.0-104-generic config-5.4.0-104-generic initrd.img-5.4.0-104-generi vmlinuz-5.4.0-104-generic>

```



# NFS와 Mount

```shell
#nfs 설치
sudo apt-get install nfs-common

#NAS 마운트 ( -t 없이 해도 상관없음)
mount -t nfs [아이피]:[NAS 경로] [워크스테이션 디렉토리 경로]
mount -t nfs ip_number:/nas_path /directory
mount -t nfs 10.125.220.161:/volume1/project /mnt/nas1
mount -t nfs 10.125.220.161:/volume1/project /mnt/nas/project
mount -t nfs 10.125.220.161:/volume1/homes /mnt/nas/homes

#type error가 뜬다면 -t 없이 작성
mount 10.125.220.161:/volume1/project /mnt/nas1

#NAS의 경로는 시놀로지에 접속해서 확인가능
#/volume1/project 연구실 NAS 경로
```


mount는 재부팅 시 초기화 되므로 fstab에 등록해야 함

```shell
vim /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p2 during installation
UUID=f274ab16-9002-4a87-a47b-405f9edee7cf /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/nvme0n1p1 during installation
UUID=8C00-8117  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
# NAS
10.125.220.161:/volume1/project /mnt/nas1       nfs     defaults	rw      0       0
10.125.220.161:/volume1/homes /mnt/nas/homes/   nfs     defaults        rw      0       0
10.125.220.161:/volume1/project /mnt/nas/project        nfs     defaults        rw      0       0

```





## REF

[재부팅 NAS 자동 마운트 (nfs) | 개발자 상현에 하루하루 (hyeon.pro)](https://hyeon.pro/dev/reboot-nfs-auto-mount/)


# htop (cpu 실시간 확인)

```shell
sudo apt install htop

htop

```



# bash에서 tab 자동완성이 안 될 때

```shell
sudo apt install bash-completion

sudo vim /root/.bashrc

#아래 부분 주석 제거
if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
   . /etc/bash_completion
   
sudo -s source /root/.bashrc
```



# Nvidia Drivier 세팅

## Nvidia Driver 설치하기

```shell
sudo lshw -C display

[sudo] password for kwon:
  *-display
       description: VGA compatible controller
       product: NVIDIA Corporation
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:1a:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:99 memory:b4000000-b4ffffff memory:a0000000-afffffff memory:b0000000-b1ffffff ioport:7000(size=128) memory:b5000000-b507ffff
  *-display
       description: VGA compatible controller
       product: NVIDIA Corporation
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:68:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:100 memory:d7000000-d7ffffff memory:c0000000-cfffffff memory:d0000000-d1ffffff ioport:b000(size=128) memory:c0000-dffff

```

lshw 명령어를 통해 그래픽 카드의 정보를 확인



```
ubuntu-drivers devices

== /sys/devices/pci0000:16/0000:16:00.0/0000:17:00.0/0000:18:10.0/0000:1a:00.0 ==
modalias : pci:v000010DEd00002204sv00001458sd00004043bc03sc00i00
vendor   : NVIDIA Corporation
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-460 - distro non-free
driver   : nvidia-driver-495 - distro non-free
driver   : nvidia-driver-470 - distro non-free recommended
driver   : nvidia-driver-460-server - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

```

recommended 된 드라이버를 설치한다



설치는 순서는 아래와 같다

```shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update

sudo apt-get install nvidia-driver-470

sudo reboot
```



만약 기존에 설치된 파일이 남아있을 경우 아래 명령어를 통해 삭제할 것

```shell
sudo apt --purge autoremove nvidia*
```





## nvidia-docker2 설치

```shell
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

위와 같은 오류가 발생했다면 nvidia-container-toolkit을 설치해야 한다.

![nvidia-docker2](https://blog.kakaocdn.net/dn/baaAJh/btrtCUw7mUO/42JJ0UbmTPKqPWHlz8uNLK/img.png)


```Shell
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list


sudo apt-get update

sudo apt-get install -y nvidia-docker2

sudo systemctl restart docker

sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi



```



## REF

[Docker NVIDIA Container Toolkit(NVIDIA Docker)의 동작원리 (tistory.com)](https://engineer-mole.tistory.com/265)

[우분투 18.04 - NVIDIA 드라이버를 설치하는 방법 (codechacha.com)](https://codechacha.com/ko/install-nvidia-driver-ubuntu/)

[nvidia-docker 와 nvidia container runtime의 차이 (tistory.com)](https://koobh.tistory.com/60)

[Installation Guide — NVIDIA Cloud Native Technologies documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

# Snap 삭제 방법

Snap이란? 

우분투에서 자체 지원하는 패키지 관리자로 apt와 비슷한 기능



```shell
#snap list로 snap 패키지 리스트 확인
snap list

# 패키지 삭제
snap remove --purge snap-store
snap remove --purge gtk-common-themes
snap remove --purge gnome-3-38-2004

snap remove --purge core20 
snap remove --purge bare 

#위의 순서대로 삭제 하면 된다. 
#core20 삭제는 snapd에서 사용 중이라 실패 할 수 도 있는데 상관말고 다음 단계로 진행 하면 삭제 됨.
#snapd 는 다음 단계에서 삭제 가능


#snap core에서 사용 중인 마운트 해제
umount /snap/core20/1405
umount /snap/core20/1434


#snapd 삭제
apt autoremove --purge snapd


#남아 있는 snap 폴더들 삭제 
rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd


#snapd를 설치 안 되게 마크 하기
apt-mark hold snapd 

```





## REF

[Ubuntu 20.04 free from Snaps by following these simple steps | Ubunlog](https://ubunlog.com/en/a-las-bravas-como-liberar-a-ubuntu-20-04-de-la-tirania-de-los-snaps/)

[Daddy Makers: 우분투 업데이트, 용량 늘리기 및 각종 에러 해결 방법 (daddynkidsmakers.blogspot.com)](http://daddynkidsmakers.blogspot.com/2020/07/apt-update.html)

[Ubuntu 18.04에서는 Snap을 기본 패키지 관리자로 사용 — Steemit](https://steemit.com/ubuntu/@calmglow/ubuntu-18-04-snap)



# SSH 세팅

## SSH 설치
```shell
sudo apt update
sudo apt install openssh-server
```

설치 후 `systemctl`을 통해 작동하는지 확인이 가능하다.

```shell
sudo systemctl status ssh

sudo systemctl enable ssh
sudo systemctl start ssh

sudo ufw allow ssh # 이건 ufw allow 22이랑 같으므로 ssh 포트가 다른 포트면 ssh가 아닌 해당 포트 번호로 allow 해야함
```


## SSH 포트 변경

`/etc/ssh/sshd_config`에서 포트 번호 변경 가능

변경 이후 `service ssh restart` 를 통해 재적용 시킨다

`netstat -tnlp`로 port 번호 확인 가능


## SSH Root 직접 로그인 허용 유무

`/etc/ssh/sshd_config` 에서 변경이 가능 

```shell
PermitRootLogin no

# 주석 제거와 no를 해주면 된다.
```


# Hostname 변경하기
## 명령어 모음

| 명령어                                | 역할                    |
| ------------------------------------- | ----------------------- |
| hostnamectl status                    | 현재 호스트명 설정 조회 |
| hostnamectl set-hostname `호스트명`   | 시스템 호스트명 설정    |
| hostnamectl set -icon-name `아이콘명` | 호스트 아이콘명 설정    |
| hostnamectl set -chasssis `섀시명`    | 호스트명 섀시 유형 설정 |


## hosts 수정
`sudo` 권한으로 `vim /etc/hosts` 를 수정한다.

127.0.1.1의 기존에 있던 hostname을 삭제하고 새롭게 정의한 hostname을 넣어준다.


## REF
[linux hostname(리눅스 호스트네임) 설정하는 법 (lesstif.com)](https://www.lesstif.com/lpt/linux-hostname-105644119.html)

[127.0.0.1 과 localhost의 차이 (intrepidgeeks.com)](https://intrepidgeeks.com/tutorial/difference-between-127001-and-localhost)

[ubuntu 20.04 hostname 변경:hostnamectl 사용 (tistory.com)](https://seonghyuk.tistory.com/199)


# 방화벽 설정

우분투를 비롯한 리눅스에서 가장 많이 쓰이는 방화벽은 iptables이다. 

그러나 iptables는 넓은 기능을 가지고 있지만 사용법이 까다로워 러닝커브가 높다는 단점이 있다.

우분투에서는 iptables를 조금 더 쉽게 쓸 수 있도록 ufw 라는 소프트웨어를 제공한다. 

ufw 또한 근본적으로는 iptables를 사용하는 것은 동일하지만 조금 더 사용자 편의적으로 만들었다고 볼 수 있다.

다만 그만큼 세부적인 설정은 불가능하므로 전문적인 보안 설정에는 ufw보단 iptables를 직접 사용하는 것이 권장된다.



## UFW 명령어

### 기본 세팅

가장 기본이 되는 Rule이 존재하는데 incoming, outgoing, routed다.

이는 `ufw status verbose`를 통해 확인이 가능하다.

```shell
sudo ufw status verbose

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
```

default의 설정은 아래와 같이 설정할 수 있다.

```shell
sudo ufw default deny incoming
sudo ufw default allow outcoming
sudo ufw default allow routed
```

여기서 중요한 것은 `ufw deny incoming` 인데 외부에서 내부로 들어오는 기본 값은 기본적으로 반드시 `deny`로 설정한다.

모두 틀어막은 상태에서 특정한 port만 열어야 보안적으로 안전하다.


```shell
ufw allow from 164.125.219.140 # ip에 대해 모두 열음
ufw allow from 164.125.219.140 to any port 8888
ufw allow from 164.125.219.141 to any port 8888
ufw allow from 164.125.219.142 to any port 8888
ufw allow from 164.125.219.143 to any port 8888
ufw allow from 164.125.219.144 to any port 8888
ufw allow from 164.125.219.150 to any port 8888
```

```shell
포트가 여러 개면 반드시 tcp나 udp를 명시해야 함 (주피터는 tcp만 허용해도 사용 가능)
ufw allow from 10.125.219.140 to any port 6006:6020 proto tcp 
ufw allow from 10.125.219.150 to any port 6006:6020 proto tcp 
ufw allow from 164.125.219.140 to any port 6006:6009 proto udp
ufw allow from 164.125.219.144 to any port 6006:6009 proto tcp 
ufw allow from 164.125.219.150 to any port 6006:6009 proto tcp
```



### 넘버링 된 것 보기

```shell
sudo ufw status numbered

sudo ufw delete [number]

sudo ufw insert [number] command
ex) sudo ufw insert 14 allow from 164.125.219.140 to any port 8888

echo "y" | ufw delete [number]

```



### 재적용

```shell
sudo systemctl
혹은
sudo ufw disable
sudo ufw enable

```

## 방화벽 끄기

```shell
sudo ufw disable
sudo systemctl disable ufw
```



## 방화벽 REF

[ufw 정책이 docker 컨테이너에 적용이 안될때. | 마르스 블로그 (lasel.kr)](https://lasel.kr/archives/536)

[Ubuntu 일반 사용자에게 sudo 권한주기. – Blog-boxcorea](https://blog.boxcorea.com/wp/archives/2847)

[TWpower's Tech Blog](https://twpower.github.io/64-use-chown-to-subfiles-and-subfolders)

[우분투 방화벽 설정 (tistory.com)](https://wlsvud84.tistory.com/23)

[useradd와 adduser의 차이 (tistory.com)](https://kit2013.tistory.com/187)

[[ Ubuntu \] UFW를 활용한 방화벽 설정 방법 (tistory.com)](https://dev-joo.tistory.com/60)



# 기본 pip 경로 변경

```shell
which pip


# ~/.bashrc에서 설정 변경
alias python_default='.../bin/python3'
alias python_conda='.../bin/python3'
```

