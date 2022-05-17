2.2 도커 컨테이너 다루기

2.2.1 컨테이너 생성

```
# 도커 엔진의 버전을 확인
PS C:\Users\Playdata> docker -v
Docker version 20.10.12, build e91ed57
```

```dockerfile
# 컨테이너 생성 명령어
PS C:\Users\Playdata> docker run -i -t ubuntu:14.04
Unable to find image 'ubuntu:14.04' locally
14.04: Pulling from library/ubuntu
2e6e20c8e2e6: Pull complete
0551a797c01d: Pull complete
512123a864da: Pull complete
Digest: sha256:60840958b25b5947b11d7a274274dc48ab32a2f5d18527f5dae2962b64269a3a
Status: Downloaded newer image for ubuntu:14.04
```

ubuntu:14.04는 컨테이너를 생성하기위한 이미지의 이름이며

**-i -t옵션은 컨테이너와 상호(interactive) 입출력을 가능하게 한다.**

이 두가지 옵션이 없을 경우 셸을 정상적으로 사용할 수 없습니다.

이유 :  -i는 상호 입출력, -t는 활성화해서 배시(bash)셸을 사용하도록 컨테이너에서 설정

엔진이 존재하지 않으므로 도커 중앙 이미지 저장소인 **도커 허브**에서 자동으로 이미지를 내려받는다.

```
root@0d9a3f1e061d:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@0d9a3f1e061d:/# exit
exit
# exit or ctrl+d
```

컨테이너 내부에서 빠져나오면서 동시에 정지시키는 방법

- exit 말고 ctrl+D도 이용가능

컨테이너를  정지시키지 않고 빠져나오는 방법

Ctrl+P, Q -> 애플리케이션 개발 목적으로 만들어짐



컨테이너의 기본 사용자는 root고 @ 뒤에는 호스 트 이름이며 무작위 해쉬값이 생성되며 최대 16진수 헤시값이 생성됩니다.





```dockerfile
# docker pull 이미지
PS C:\Users\Playdata> docker pull centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:c73f515d06b0fa07bb18d8202035e739a494ce760aa73129f60f4bf2bd22b407
Status: Downloaded newer image for centos:7
docker.io/library/centos:7
# docker images - 이전에 내려받은 이미지의 목록을 보여줌
PS C:\Users\Playdata> docker images
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
docker/getting-started   latest    bd9a9f733898   2 weeks ago     28.8MB
centos                   7         eeb6ee3f44bd   5 months ago    204MB
ubuntu                   14.04     13b66b487594   11 months ago   197MB
# -name 옵션에는 컨테이너의 이름을 설정하여 pull과같은 명령어로 사용가능
PS C:\Users\Playdata> docker create -i -t --name mycentos centos:7
963a6e1e795a30e8eaeba12ed8c0ab6b55e9e4e2b20df03695f79a4ebd88bace
# inspect로 도커 아이디와 다른 특이 사항 확인 가능. or docker inspect 이미지이름 | grep Id
PS C:\Users\Playdata> docker inspect mycentos
[
    {
        "Id": "963a6e1e795a30e8eaeba12ed8c0ab6b55e9e4e2b20df03695f79a4ebd88bace",
```

- 도커 이미지를 내려받을땐 docker pull centos:7 처럼 사용한다.

  ```dockerfile
  # 컨테이너 시작
  PS C:\Users\Playdata> docker start mycentos
  mycentos
  # 컨테이너 접속
  PS C:\Users\Playdata> docker attach mycentos
  [root@963a6e1e795a /]#
  ```

  

  2.2.2 컨테이너 목록 확인

  

  ```dockerfile
  # docker ps 명령어는 정지되지 않은 컨테이너만 출력한다. 정지된 컨테이너를 포함한 모든 컨테이너를 출력하려면 -a 작성
  PS C:\Users\Playdata> docker ps(-a)
  CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                NAMES
  963a6e1e795a   centos:7                 "/bin/bash"              15 minutes ago   Up 8 minutes                         mycentos
  5e52d541c526   docker/getting-started   "/docker-entrypoint.…"   39 minutes ago   Up 39 minutes   0.0.0.0:80->80/tcp   wonderful_mclean
  
  # COMMAND : 커맨드는 컨테이너가 시작될 때 실행될 명령어입니다.
  PS C:\> docker run -i -t ubuntu:14.04 echo hello world!
  hello world!
  
  #
  ```

  - CONTAINER ID : 컨테이너에게 자동으로 할당되는 고유한 ID
  - IMAGE : 컨테이너를 생성할 때 사용된 이미지의 이름
  - COMMAND : 커맨드는 컨테이너가 시작될 때 실행될 명령어입니다. 커맨드는 대부분의 이미지에 미리 내장돼어있기 때문에 별도로 설정할 필요가 없습니다. 컨테이너가 시작될 때 /bin/bash 명령어가 실행됐으므로 상호 입출력이 가능한 셀 환경을 사용할 수 있다.
  - CREATED : 컨테이너가 생성되고 난 뒤 흐른 시간
  - STATUS  : 컨테이너의 상태를 나타내며, 컨테이너가 실행 중임을 나타내는 'UP', 종료된 상태인 'Exited', 일시 중지된 상태인 'Pause' 등이 있다.
  - PORTS : 컨테이너가 개방한 포트와 호스트에 연결한 포트를 나열합니다.
  - NAMES : 컨테이너의 고유한 이름입니다. 

```dockerfile
# 컨테이너 이름 확인하기
PS C:\> docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED             STATUS                      PORTS              NAMES
3f9c6dacb04   ubuntu:14.04             "echo hello world!"      28 minutes ago      Exited (0) 28 minutes ago             gallant_mendel
..
# 컨테이너 이름 바꾸기
PS C:\> docker rename gallant_mendel my_container

# 컨테이너 이름 바꼇는지 확인
PS C:\> docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                      PORTS                NAMES
3ef9c6dacb04   ubuntu:14.04             "echo hello world!"      29 minutes ago   Exited (0) 29 minutes ago                        my_container
..

```



2.2.3 컨테이너 삭제

```dockerfile
# 컨테이너 제거
PS C:\> docker rm my_container
my_container
PS C:\> docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS                   PORTS                NAMES
963a6e1e795a   centos:7                 "/bin/bash"              2 hours ago   Up About an hour                              mycentos
0d9a3f1e061d   ubuntu:14.04             "/bin/bash"              2 hours ago   Exited (0) 2 hours ago                        happy_jackson
5e52d541c526   docker/getting-started   "/docker-entrypoint.…"   2 hours ago   Up 2 hours               0.0.0.0:80->80/tcp   wonderful_mclean

# docker rm mycentos
Error response from daemon: You cannot remove a running container 963a6e1e795a30e8eaeba12ed8c0ab6b55e9e4e2b20df03695f79a4ebd88bace. Stop the container before attempting removal or force remove
- 실행 중이므로 삭제할 수 없음
# docker stop mycentos
# docker rm mycentos  
mycentos 제거 성공
# 두 가지 경우를 합쳐서 쓰는 용어
docker rm -f mycentos
# prune은 모든 컨테이너를 삭제할 수 있다.
PS C:\> docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
0d9a3f1e061d5ad93bb27a089b477689f3a383296954adf6ddc64317564d5828

Total reclaimed space: 8B
# docker ps 명령어의 -a옵션과 -q옵션을 조합해 컨테이너를 삭제할 수도 있습니다. -a는 컨테이너 상태와 관계 없이 모든 컨테이너를, -q는 컨테이너의 ID만 출력하는 역할
PS C:\> docker ps -a -q
5e52d541c526
# 컨테이너의 실행 상태와 관계없이 모든 컨테이너를 정지하고 삭제.
PS C:\> docker stop $(docker ps -a -q)
5e52d541c526
PS C:\> docker rm $(docker ps -a -q)
5e52d541c526
```

- 컨테이너를 삭제하고 확인하는 명령어를 입력해서 확인했습니다.



2.2.4 컨테이너를 외부에 노출

```dockerfile
# 기본적으로 도커는 컨테이너에 172.17.0.x의 IP를 순차적으로 할당합니다. 컨테이너를 새롭게 생성한 후 ifconfig 명령어로 컨테이너의 네트워크 인터페이스를 확인
PS C:\> docker run -i -t --name network_test ubuntu:14.04
root@8b9b255f193d:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:736 (736.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

- 이전의 run 예제와 다른점은 -p 옵션을 추가하여 컨테이너의 포트를 호스트의 포트와 바인딩해 연결할 수 있게 설정합니다.[호스트의 포트]:[컨테이너의 포트]

2.2.4 컨테이너를 외부에 도출

- 컨테이너 웹 서버 설치 중 오류

```dockerfile
root@87d270ead910:/# apt-get update
root@87d270ead910:/# apt-get install apache2 -y
root@87d270ead910:/# service apache2 start
 * Starting web server apache2                                                                                                                                                                                   AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.3. Set the 'ServerName' directive globally to suppress this message
 *
 # 보시는 바와 같이 서버이름을 설정해주지 않아서 오류가 발생합니다.
```



- 아래와 같이 서버이름을 설정해주시면 해결!

```dockerfile
grep ServerName /etc/apache2/apache2.conf
echo "ServerName localhost" >> /etc/apache2/apache2.conf
grep ServerName /etc/apache2/apache2.conf
service apache2 restart
```

참고사이트

https://zetawiki.com/wiki/%EC%9A%B0%EB%B6%84%ED%88%AC_apache2:_Could_not_reliably_determine_the_server%27s_fully_qualified_domain_name



### 											브라우저로 아파치 웹 서버에 접속

호스트 IP의 80번 포트로 접근 -> 80번 포트는 컨테이너의 80번 포트로 포워딩 -> 웹 서버 접근

------------------------

2.2.5 컨테이너 애플리케이션 구축

- 애플리케이션을 하나 혹은 분리된 컨테이너로 구성
- 데이터베이스와 워드프레스 웹 서버 컨테이너를 연동해 워드프레스 기반 블로그 서비스를 만들기

```dockerfile
# http://127.0.0.1:52389/ -> 
PS C:\Users\Playdata># docker run -d --name wordpressdb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7
9ce9c6cddcc591a12085c3f902bf116f7d36e3a5d02379fc8eaf56ada925ddd4
## mysql 이미지를 사용해 데이터베이스 컨테이너

PS C:\Users\Playdata># docker run -d -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password --name wordpress --link wordpressdb:mysql -p 80 wordpress
c120fa257d843ceb250972dd54d8c837a5f2602c23a8b0265f70d8e482467ea0
## 미리 준비된 워드프레스 이미지를 이용해 워드프레스 웹 서버 컨테이너를 생성
```

- 호스트와 바인딩된 포트만 확인하려면 docker port 명령을 사용. wordpress라는 이름의 컨테이너가 사용 중인 호스트의 포트가 출력된 결과

```dockerfile
PS C:\Users\Playdata># docker port wordpress
80/tcp -> 0.0.0.0:52389
```

- port : 127.0.0.1:52389



###  									--- 브라우저로 워드프레스 컨테이너에 접근 --- 



- 컨테이너 내부에서 프로그램을 실행하지 않은 채로 -d 옵션을 설정 ( 포그라운드로써 동작하는 프로그램이 없으므로 컨테이너 시작 불가)

```dockerfile
# 컨테이너가 생성되고 종료된 상태
PS C:\Users\Playdata> docker run -d --name detach_test ubuntu:14.04
7c77ac6ece4e31b1f14d9863905d9c6ecbd469bf0d0681291393406e381a5379
PS C:\Users\Playdata> docker ps -a
CONTAINER ID   IMAGE          COMMAND             CREATED             STATUS          PORTS           NAMES
7c77ac6ece4e   ubuntu:14.04   "/bin/bash"        14 seconds ago      Exited (0) 12 seconds ago     detach_test
```

- 하나의 터미널을 차지하는 mysqld프로그램이 포그라운드로 실행된 로그를 볼 수 있음. 

```
docker run -i -t --name mysql_attach_test -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7
```



```dockerfile
# exec 명령어를 사용하면 컨테이너 내부에서 명령어를 실행한 뒤 그 결과값을 반환받을 수 있음
# -i -t 옵션을 추가해 /bin/bash를 상호 입출력이 가능한 형태로 exec 명령어를 사용
PS C:\Users\Playdata> docker exec -i -t wordpressdb /bin/bash
root@9ce9c6cddcc5:/# echo $MYSQL_ROOT_PASSWORD
password

# 설정된 환경변수가 실제로  MYSQL에 사용됐는지 확인 
# mysql -u root -p (password)
root@9ce9c6cddcc5:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
# 실행됨

```

- --link : A컨테이너에서 B컨테이너로 접근하는 방법 중 가장 간단한 것은 NAT로 할당받은 내부 IP를 쓰는 것이다.

```dockerfile
# wordpress wordpressdb를 둘 다 주이
PS C:\Users\Playdata> docker stop wordpress wordpressdb
wordpress
wordpressdb

# 이유 : 컨테이너를 연결해주는 link는 실행 순서의 의존성도 정의해줌
PS C:\Users\Playdata> docker start wordpress
Error response from daemon: Cannot link to a non running container: /wordpressdb AS /wordpress/mysql
Error: failed to start containers: wordpress
```

- 이처럼 link 옵션은 컨테이너 간에 이름으로 서로를 찾을 수 있게 도와준다.

---------------------------

2.2.6 도커 볼륨

- 도커 이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며 컨테이너의 변경 사항만 별도로 저장. 각 컨테이너의 정보를 보존

![img](https://blog.kakaocdn.net/dn/ck3mTE/btqDj3e8PHY/D4k9YyEeu4XKW8kvlgKXS0/img.png)

- 이미 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 원래 이미지에서 변경된 파일시스템 등을 저장.

- 이미지에 mysql을 실행하는데 필요한 애플리케이션 파일이 들어있다면 데이터베이스를 운용하면서 쌓이는 데이터가 저장

  

  ### 단점

  - mysql 컨테이너를 삭제하면 컨테이너 계층에 저장돼있던 데이터베이스의 정보도 삭제된다
    - 방지방법
      - 볼륨
        - 호스트와 볼륨을 공유, 볼륨 컨테이너 활용, 도커가 관리하는 볼륨을 생성

----------------------------

2.2.6.1 호스트 볼륨 공유

```dockerfile
# mysql 데이터베이스 컨테이너 생성
# -v 옵션 : [호스트의 공유 디렉토리]:[컨테이너의 공유 디렉토리] 형태
# /var/lib/mysql 디렉터리는 MYSQL이 데이터베이스의 데이터를 저장하는 기본 디렉토리
PS C:\Users\Playdata> #docker run -d --name wordpressdb_hostvolume -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress -v /home/wordpress_db:/var/lib/mysql mysql:5.7
2d2bd512e0082c534e78089c5ebdc3c56f88c225c1a22954c5c1cee1def1adc0


# 워드프레스 웹 서버 컨테이너 생성
# -p 옵션으로 컨테이너의 80포트를 외부로 노출
PS C:\Users\Playdata> #docker run -d -e WORDPRESS_DB_PASSWORD=password --name wordpress_hostvolume --link wordpressdb_hostvolume:mysql -p 80 wordpress
c70ba9c91d706e40939d44499b50c46cd313f3b6260445a115da951c2c691a8c
```

```dockerfile
# 호스트에 이미 디렉터리와 파일이 존재하고 컨테이너에도 존재할 때 두 디렉터리를 공유하는 경우는?
# alicek106/volume_test 이미지활용
PS C:\> docker run -i -t --name volume_dummy alicek106/volume_test
Unable to find image 'alicek106/volume_test:latest' locally
latest: Pulling from alicek106/volume_test
56eb14001ceb: Pull complete
7ff49c327d83: Pull complete
6e532f87f96d: Pull complete
3ce63537e70c: Pull complete
587f7dba3172: Pull complete
Digest: sha256:e0287b5cfd550b270e4243344093994b7b1df07112b4661c1bf324d9ac9c04aa
Status: Downloaded newer image for alicek106/volume_test:latest

# /home/testdir_2/test 파일 존재확인
root@66ba8cf2b24a:/# ls /home/testdir_2/
test

# -v 옵션의 값인  /home/wordpress_db:/home/testdir_2 디렉터리를 확인하면 원래 존재했던 test파일이 없어지고
# 호스트에서 공유된 파일이 존재
# 정리 : -v 옵션을 통한 호스트 볼륨 공유는 호스트의 디렉터리를 컨테이너의 디렉터리에 마운트
PS C:\> docker run -i -t --name volume_overide -v /home/wordpress_db:/home/testdir_2 alicek106/volume_test
root@d13b9d4aa93e:/# ls /home/testdir_2/
auto.cnf    ca.pem           client-key.pem  ib_logfile0  ibdata1  performance_schema  public_key.pem   server-key.pem  wordpress
ca-key.pem  client-cert.pem  ib_buffer_pool  ib_logfile1  mysql    private_key.pem     server-cert.pem  sys

# 태그를 지정하지 않았는데 이미지가 pull된 것은 이미지의 태그를 지정하지 않으면
# 도커 엔진이 latest 태그로 지정된 이미지를 pull하기 때문

```



2.2.6.2 볼륨 컨테이너

```dockerfile
# -v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것
# --volumes-from 옵션을 설정하면 -v 또는 --volume 옵션을 적용한 컨테이너의 볼륨 디렉터리를 공유
# 그러나 직접 볼륨 공유하는 것이 아닌 -v 옵션을 적용한 컨테이너 공유
# 앞에서 생성한 volume_overide 컨테이너에서 볼륨을 공유받는 경우
PS C:\> docker run -i -t --name volumes_from_container --volumes-from volume_overide ubuntu:14.04
root@bad4d8daee94:/# ls /home/testdir_2/
auto.cnf    ca.pem           client-key.pem  ib_logfile0  ibdata1  performance_schema  public_key.pem   server-key.pem  wordpress
ca-key.pem  client-cert.pem  ib_buffer_pool  ib_logfile1  mysql    private_key.pem     server-cert.pem  sys
```

- --volumes-form 옵션을 적용한 컨테이너와 볼륨 컨테이너 사이의 관계

![Docker Volume (컨테이너 볼륨 공유)-컨테이너 볼륨을 다른 컨테이너와 공유하기](https://t1.daumcdn.net/cfile/tistory/9911A7415C9921F905)

### 																									볼륨 컨테이너의 구조

- 이러한 구조를 활용하면 호스트에서 볼륨만 공유하고 별도의 역할을 담당하지 않는 일명 '볼륨 컨테이너' 로서 활용가능
- 볼륨을 사용하려는 컨테이너에 -v 옵션 대신 --volume-from 옵션을 사용
- 볼륨 컨테이너에 연결해 데이터를 간접적으로 공유받음



2.2.6.3 도커 볼륨

- docker volume 명령어 사용

- 이전 방식과 달리 도커 자체에서 제공하는 볼륨 기능 활용하여 데이터 보존

  ```dockerfile
  # create 명령어로 볼륨 생성
  PS C:\> docker volume create --name myvolume
  myvolume
  
  # 볼륨 확인
  # 여기서는 기본적으로 제공되는 드라이버 local사용
  # **( 볼륨을 생성할 때 플러그인 드라이버를 설정해 여러 종류 스토리지 백엔드 가능)
  # 이 볼륨은 로컬 호스트에 저장되며 도커 엔진에 의해 생성되고 삭제
  PS C:\> docker volume ls
  DRIVER    VOLUME NAME
  local     myvolume
  
  
  # [볼륨의 이름]:[컨테이너의 공유 디렉토리]
  # /root/ 디렉터리에 마운트하므로 /root 디렉터리에 파일을 쓰면 해당 파일이 볼륨에 저장
  PS C:\> docker run -i -t --name myvolume_1 -v myvolume:/root/ ubuntu:14.04
  root@a590ba5d873b:/# echo hello, volume! >> /root/volume
  
  # /root 디렉터리에 volume 파일 생성.
  # 다른 컨테이너도 myvolume 볼륨을 쓰면 볼륨을 활용한 디렉터리에 volume 파일 존재.
  # 파일 생성확인
  PS C:\> docker run -i -t --name myvolume_2 -v myvolume:/root/ ubuntu:14.04
  root@ab8199ba21f6:/# cat /root/volume
  hello, volume!
  ```

  

![1부 _ 도커 _ 02 도커 엔진 - 2.2 도커 컨테이너 다루기 : 네이버 블로그](data:image/png;base64,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)

### 																										도커 볼륨 사용 구조

- 볼륨은 디렉토리 하나에 상응하는 단위로서 도커 엔진이 관리.

```dockerfile
# docker inspect를 사용하면 볼륨이 실제로 어디 저장되었는지 확인가능
# 컨테이너, 이미지, 볼륨 등 도커의 모든 구성 단위의 정보 확인할때 사용
# --type 옵션 : image, volume 등을 입력해야 좋게 명시됨

PS C:\> docker inspect --type volume myvolume
[
    {
        "CreatedAt": "2022-03-03T15:01:05Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": {},
        "Scope": "local"
    }
]
# 도커의 모든 명령어는 docker 접두어 다음에 명시하고 특정 구성 단위를 제어하는 명령어 사용가능
# docker container inspect - 컨테이너 정보, docker volume inspect - 볼륨의 정보
```



```dockerfile
# create를 별도로 쓰지않아도 -v 옵션을 입력해서 수행
# -v 옵션 : 볼륨을 자동 생성
PS C:\> docker run -i -t --name volume_auto -v /root ubuntu:14.04

PS C:\> docker volume ls
DRIVER    VOLUME NAME

local     f1e5bdae44b2ec3a089161c06d54c1d53ede8b73b1c8dd1775af696bf7eb0405
local     myvolume

# -v 옵션 대신 --mount 옵션을 사용할 수 있다. 두 옵션의 기능은 같지만, 볼륨의 정보를 나타내는 방법이 다르기 때문에 둘 중 사용하기 편한 옵션을 사용하면 된다.
```





2.2.7 도커 네트워크

​	2.2.7.1 도커 네트워크 구조

​		

```dockerfile
# 이전에 컨테이너 내부에서 ifconfig을 입력해서 네트워크 인터페이스에 eth()과 lo네트워크 인테페이스 확인
# 도커는 컨테이너에 내부 IP를 순차적으로 할당. <컨테이너 재시작시 IP변경될 수 있음>
# 내부 망에서만 쓸 수 있는 IP이므로 외부와 연결 필요
# 외부와 연결되는 과정은 컨테이너를 시작할 때마다 호스트에 veth...라는 네트워크 인테페이스 생성 - veth(virtual eth)
# 각 컨테이너에 외부와의 네트워크를 제공하기 위해 컨테이너마다 가상 네트워크 인터페이스를 호스트에 생성
# veth 인터페이스는 사용자가 직접 생성할 필요가 없음.
# 컨테이너가 생성될 때 도커 엔진이 자동적으로 생성
root@fb69f29cc62d:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          #inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
```

- 여기서 설명한 네트워크 구성은 리눅스를 기준. 윈도우라면 ipconfig를 입력

```dockerfile
# 출력 결과에서 eth()은 공인 IP 또는 내부 IP가 할당되어 실제로 외부와 통신할 수 있음
# veth로 시작하는 인터페이스는 컨테이너를 시작할 때 생성
# veth 인터페이스뿐 아니라 docker()이라는 브리지도 존재함
# docker() 브리지는 각 veth 인테페이스와 바인딩돼 호스트의 eth() 인터페이스와 이어주는 역할
PS C:\Users\Playdata> ipconfig

Windows IP 구성
이더넷 어댑터 이더넷:

   미디어 상태 . . . . . . . . : 미디어 연결 끊김
   연결별 DNS 접미사. . . . : skbroadband

이더넷 어댑터 vEthernet (Default Switch):

   연결별 DNS 접미사. . . . :
   링크-로컬 IPv6 주소 . . . . : fe80::90d9:1319:fe04:230c%46
   ...
   무선 LAN 어댑터 Wi-Fi:

   연결별 DNS 접미사. . . . :
   링크-로컬 IPv6 주소 . . . . : fe80::6954:225c:b760:acb2%7
   IPv4 주소 . . . . . . . . . : 172.30.1.53
   서브넷 마스크 . . . . . . . : 255.255.255.0
   기본 게이트웨이 . . . . . . : 172.30.1.254
  
```

![97. [Docker + Network] Docker 컨테이너의 Macvlan 사용해보기 : 네이버 블로그](https://mblogthumb-phinf.pstatic.net/MjAxNzA0MTVfNDcg/MDAxNDkyMjQ2MzQ5ODMx.Jl6qkAAjUZn8jVr94ZhbRGKtzoYungSm4Hth4J4Zt2Ag.OqAoPP2rrQSbguDx50_WkWiBvOPyI-YyGeFuM8CrjdMg.PNG.alice_k106/%EC%BA%A1%EC%B2%98.PNG?type=w2)

- 정리하면 컨테이너의 eth() 인터페이스는 호스트의 veth..라는 인터페이스와 연결됐으며 veth 인터페이스는 docker() 브리지와 바인딩돼 외부와 통신



2.2.7.2 도커 네트워크 기능

- 컨테이너를 생성하면 기본적으로 docker() 브리지를 통해 외부와 통신할 수 있는 환경 구축
- 사용자의 선택에따라 여러 네트워크 드라이버를 쓸 수 있음.
- 대표적인 네트워크 드라이버 ['브리지(Bridge)', '호스트(Host)', '논(None)', '컨테이너(Container)', '오버레이(Overlay)' ]
- 서드파티(Third-Party) 플러그인 솔루션으로는 weave, flannel, openvswitch 등이 있음

```dockerfile
# 도커에서 기본적으로 쓸 수 있는 네트워크
# 도커의 네트워크를 다루는 명령어 = docker network로 시작
# 이미 브리지, 호스트, 논 네트워크가 있음
PS C:\Users\Playdata> docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
58168758ef4c   bridge    bridge    local
d1bdbbbd8c75   host      host      local
9835d3434648   none      null      local

# docker network inspect 명령어를 이용하면 네트워크의 자세한 정보를 살펴볼 수 있음
# docker inspect --type network를 사용해도 동일한 출력 가능
PS C:\Users\Playdata> docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "58168758ef4caa81770bbb1529c7bdb2922bd70f2b7a4719940ce7e462fe8951",
        ...


```



- 브리지 네트워크

  

```dockerfile
# 사용자 정의 브리지를 새로 생성해 각 컨테이너에 연결하는 네트워크 구조
# 기본적으로 존재하는 docker()을 사용하는 브리지 네트워크가 아닌 새로운 브리지 타입의 네트워크 생성
# 브리지 타입의 mybridge 네트워크 생성
PS C:\Users\Playdata> docker network create --driver bridge mybridge
c67469460b1e925ae418e4c86b9ccb3ef26015f3bb022b1464fb6ea3951a7718

# docker run 또는 create 명령어에 --net 옵션의 값을 설정
# 컨테이너가 이 네트워크를 사용하도록 설정
# ifconfig을 입력하면 새로운 IP 대역이 할당된 것을 확인
PS C:\Users\Playdata> docker run -i -t --name mynetwork_container --net mybridge ubuntu:14.04
root@dc7ab97316b8:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:12:00:02
          inet addr:172.18.0.2  Bcast:172.18.255.255  Mask:255.255.0.0
          ..
```

```dockerfile
# docker network disconnet, connect를 통해 컨테이너에 유동적으로 붙이고 뗄 수 있음 
PS C:\Users\Playdata> docker network disconnect mybridge mynetwork_container
PS C:\Users\Playdata> docker network connect mybridge mynetwork_container

# 네트워크의 서브넷, 게이트웨이, IP 할당 범위 등을 임의로 설정
# --subnet, --ip-range, --gateway 옵션을 추가
# 단 --subnet과 --ip-range는 같은 대역
PS C:\Users\Playdata> docker network create --driver=bridge --subnet=172.72.0.0/16 --ip-range=172.72.0.0/24 --gateway=172.72.0.1 my_custom_network
4d037714f210f3931bd683119379d1e980bbda969fe074a5bcf5fa87d5ed7475
```



- 호스트 네트워크

```dockerfile
# 네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 사용
# 브리지 드라이버와 달리 호스트 드라이버의 네트워크는 별도 생성x
# 기존의 host라는 이름의 네트워크 사용
# --net 옵션을 입력해 호스트를 설정
# 호스트 머신에서 설정한 호스트 이름도 컨테이너가 물려받음
# 컨테이너의 호스트 이름도 무작위 16진수가 아닌 도커 엔진이 설치
PS C:\Users\Playdata> docker run -i -t --name network_host --net host ubuntu:14.04
root@docker-desktop:/# echo "inner containenr"
inner containenr
root@docker-desktop:/# ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:6a:3f:50:6e
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:6aff:fe3f:506e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:526 (526.0 B)

eth0      Link encap:Ethernet  HWaddr 02:50:00:00:00:01
          inet addr:192.168.65.3  Bcast:192.168.65.255  Mask:255.255.255.0
          inet6 addr: fe80::50:ff:fe00:1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:17 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:1286 (1.2 KB)
          ....

# 컨테이너의 네트워크를 호스트 모드로 설정
# 컨테이너 내부의 애플리케이션을 별도의 포트 포워딩 없이 이용가능
```

- 논 네트워크

```dockerfile
# none은 말 그대로 아무런 네트워크를 쓰지 않는 것
# 다음과 같이 컨테이너 생성 시 외부와 연결이 단절
PS C:\Users\Playdata> docker run -i -t --name network_none --net none ubuntu:14.04
root@845e4de19963:/# ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
```

- 컨테이너 네트워크

```dockerfile
# --net 옵션으로 container를 입력
# 다른 컨테이너의 네트워크 네임스페이스 환경을 공유
# 공유되는 속성은 내부IP, 네트워크 인터페이스의 맥(MAC) 주소 등
# --net 옵션의 값으로 container:[다른 컨테이너 ID]
PS C:\Users\Playdata> docker run -i -t -d --name network_container_2 --net container:network_container_1 ubuntu:14.04
90aeb47f66f5845c528e2c27c2dcfd56eda4debc00b1d782d5afc470901ed7b3
```

- -i, -t, -d 옵션을 함께 사용하면 컨테이너 내부에서 셸을 실행하지만 내부로 들어가지 않으며 컨테이너도 종료되지 않는다. 위와 같이 테스트용으로 컨테이너를 생성할 때 유용하게 씀

- 위와 같이 다른 컨테이너의 네트워크 환경을 공유하면 내부 IP를 새로 할당받지 않음

- 호스트에 veth로 시작하는 가상 네트워크 인터페이스로 생성되지 않음

- network_container_2 컨테이너의 네트워크와 관련된 사항은 전부 network_container_1과 같게 설정

  

```dockerfile
# 두 컨테이너의 eth() 정보 동일. 
PS C:\Users\Playdata> docker exec network_container_1 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:03
          inet addr:172.17.0.3  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1076 (1.0 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

PS C:\Users\Playdata> docker exec network_container_2 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:03
          inet addr:172.17.0.3  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1076 (1.0 KB)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

![img](https://miro.medium.com/max/1066/1*3MG2AKT_y_4nMiNfY7kS4Q.png)

​																								-- 호스트 머신 -- 



- 브리지 네트워크와 --net-alias

https://ihp001.tistory.com/188?category=859773 참조 사이트

브리지 네트워크와 --net-alias

- 브리지 타입의 네트워크와 run 명령어의 --net-alias 옵션을 함께 쓰면 특정 호스트 이름으로 컨테이너 여러 개 접근 가능

```dockerfile
# --net-alias 옵션의 값은 alicek106으로 설정 - 나는 실수로 06으로 설정함.!
# 다른 컨테이너에서 alicek106이라는 호스트 이름으로 아래 3개의 컨테이너 접근
PS C:\Users\Playdata> docker run -itd --name network_alias_container1 --net mybridge --net-alias alicek06 ubuntu:14.04
95d34bef4bf9056f0b7df4763aa2c39b80fd35a9741d91c5a5f073c1bdf8426c
PS C:\Users\Playdata> docker run -itd --name network_alias_container2 --net mybridge --net-alias alicek06 ubuntu:14.04
6623add3021ee78fbeedc8dbd74e6c9e9ee064b3f709857b1f9c57406980841d
PS C:\Users\Playdata> docker run -itd --name network_alias_container3 --net mybridge --net-alias alicek06 ubuntu:14.04
134b809aeae7422e857e88ea416479ded70fa4564c8e7e9f41e7e3246fa62c83
```

- `docker inspect network_alias_container1 | grep 172` 이런식으로 컨테이너 3개 전부 IP 확인하고, 이 세 개의 컨테이너에 접근할 컨테이너를 생성한 뒤 alicek106이라는 호스트 이름으로 ping 요청을 전송한다.

  

```dockerfile
# 컨테이너 3개의 IP로 각각 ping이 전송된 것을 확인할 수 있다. 
# 매번 달라지는 IP를 결정하는 것은 별도의 알고리즘이 아닌 라운드 로빈 방식이다. 
# 도커 엔진에 내장된 DNS가 alicek06이라는 호스트 이름을 --net-alias 옵션으로 alicek106을 설정한 컨테이너로 변환하기 때문에 가능하다.
# ping 명령어는 이 IP 리스트에서 첫 번째 IP를 사용하므로 매번 다른 IP로 ping을 전송
PS C:\Users\Playdata> docker run -i -t --name network_alias_ping --net mybridge ubuntu:14.04
root@21e656b4247a:/# ping -c 1 alicek106
ping: unknown host alicek106
root@21e656b4247a:/# ping -c 1 alicek06
PING alicek06 (172.19.0.3) 56(84) bytes of data.
64 bytes from network_alias_container2.mybridge (172.19.0.3): icmp_seq=1 ttl=64 time=0.138 ms

--- alicek06 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.138/0.138/0.138/0.000 ms
root@21e656b4247a:/# ping -c 1 alicek06
PING alicek06 (172.19.0.2) 56(84) bytes of data.
64 bytes from network_alias_container1.mybridge (172.19.0.2): icmp_seq=1 ttl=64 time=0.187 ms

--- alicek06 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.187/0.187/0.187/0.000 ms
root@21e656b4247a:/# ping -c 1 alicek06
PING alicek06 (172.19.0.4) 56(84) bytes of data.
64 bytes from network_alias_container3.mybridge (172.19.0.4): icmp_seq=1 ttl=64 time=0.091 ms

--- alicek06 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.091/0.091/0.091/0.000 ms
```

![example](https://user-images.githubusercontent.com/47745785/118841704-7ee66180-b903-11eb-9b7c-1937e3edc2af.png)

- --link 옵션 : 컨테이너의 IP가 변경돼도 별명으로 컨테이너를 찾을 수 있게 DNS에 의해 자동적 관리
- 단 이 경우는 디폴트 브리지 네트워크의 컨테이너 DNS라는 점이 다름
- --net-alias 옵션 또한 --link 옵션과 비슷한 원리



```dockerfile
# 핑을 전송했는지 확인하기위해선 dig라는 도구를 사용
# dig는 DNS로 도메인 이름에 대응하는 IP를 조회할 때 쓰는 도구
# dig는 ubuntu:14.04 이미지에 설치돼 있지 않음.
# 컨테이너 내부에 설치 및 반환되는 IP 확인
root@21e656b4247a:/# apt-get update

root@21e656b4247a:/# apt-get install dnsutils

root@21e656b4247a:/# dig alicek06

; <<>> DiG 9.9.5-3ubuntu0.19-Ubuntu <<>> alicek06
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64691
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;alicek06.                      IN      A

;; ANSWER SECTION:
alicek06.               600     IN      A       172.19.0.3
alicek06.               600     IN      A       172.19.0.4
alicek06.               600     IN      A       172.19.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Mar 07 13:47:44 UTC 2022
;; MSG SIZE  rcvd: 98
```



- MacVLAN 네트워크

  - 호스트의 네트워크 인터페이스 카드를 가상화해 물리 네트워크 환경을 컨테이너에게 동일하게 제공

  - MacVLAN을 사용하면 컨테이너는 물리 네트워크상에서 가상의 맥(MAC) 주소를 가짐

  - 해당 네트워크에 연결된 다른 장치와의 통신이 가능

  - MacVLAN에 연결된 컨테이너는 기본적으로 할당되는 IP대역인 172.72.X.X대신 네트워크 장비의 IP할당받음

    

![example](https://user-images.githubusercontent.com/47745785/118844433-eac9c980-b905-11eb-9366-2518a5f09d20.png)

- 따라서 MacVLAN을 사용하는 컨테이너들과 동일한 IP 대역을 사용하는 서버 및 컨테이너들은 서로 통신이 가능하다.

```dockerfile
# MacVLAN 내용은 더 알아보기!
PS C:\Users\Playdata> docker network create -d macvlan --subnet=192.168.0.0/24 --ip-range=192.168.0.64/28 --gateway=192.168.0.1 -o macvlan_mode=bridge -o parent=eth0 my_macvlan
19aba3697abcd727e5f0fac5ee355af63553fe079486eb5db24ab3773047a4af
PS C:\Users\Playdata> docker run -it --name c1 --hostname c1 --network my_macvlan ubuntu:14.04
root@c1:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
...
root@c1:/# ping 192.168.0.128 -c 1
PING 192.168.0.128 (192.168.0.128) 56(84) bytes of data.
From 192.168.0.64 icmp_seq=1 Destination Host Unreachable

```



2.2.8 컨테이너 로깅

- 2.2.8.1 json-file 로그 사용하기
  - 컨테이너 내부에서 어떤 일이 일어나는지 아는 것은 디버깅뿐만 아니라 운영 측면도 중요
  - 애플리케이션 레벨에서 로그가 기록되도록 개발해 별도의 로깅 서비스 사용 가능
  - 단 도커는 컨테이너의 표준 출력(StdOut)과 에러(StdErr) 로그를 별도의 메타데이터 파일저장

```dockerfile
# 컨테이너를 생성해 간단한 로그 생성. mysql 5.7 버전의 컨테이너
# 애플리케이션이 잘 구동되는지 여부를 알 순 없음
PS C:\Users\Playdata> docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
ffcde61123ab836ea04679300724a7f448a2e86a4d9023bd280b4ea7fb1cd16e

# docker logs 명령어를 써서 컨테이너의 표준 출력을 확인함으로써 애플리케이션의 상태 알 수 있음
# docker logs 명령어는 컨테이너 내부에서 출력을 보여주는 명령어
PS C:\Users\Playdata> docker logs mysql
2022-03-07 14:35:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
2022-03-07 14:35:58+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-03-07 14:35:58+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
....

# 다른 방법으로 컨테이너 생성. 동일한 mysql 컨테이너 생성, -e 옵션 제외
PS C:\Users\Playdata> docker run -d --name no_passwd_mysql mysql:5.7
860001c98fca3b35a0966c804fbf823ffd0f75f4256fe636e9a85498ea81e092

# docker ps 명령어로 목록을 확인하면 컨테이너는 생성됐으나 실행되지 않음. start도 마찬가지
PS C:\Users\Playdata> docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"
CONTAINER ID   STATUS             PORTS                   NAMES
ffcde61123ab   Up 6 minutes       3306/tcp, 33060/tcp     mysql
134b809aeae7   Up About an hour                           network_alias_container3
6623add3021e   Up About an hour                           network_alias_container2
95d34bef4bf9   Up About an hour                           network_alias_container1
193c9949d53a   Up 2 hours         0.0.0.0:59318->80/tcp   wordpress
5d1b2095af6b   Up 2 hours         3306/tcp, 33060/tcp     wordpressdb
74027231bfa7   Up 2 hours                                 charming_cerf

```

- 이럴 때 docker logs 명령어를 사용하면 애플리케이션에 무슨 문제가 있는지 확인가능

- 컨테이너가 정상적으로 실행 및 동작하지 않고 docker attach 명령어도 사용하지 못하는 환경에선 docker logs 명령어를 쓰면 간단하고 빠르게 에러 확인

  

```dockerfile
# 컨테이너에서 발생한 로그들
PS C:\Users\Playdata> docker logs no_passwd_mysql
2022-03-07 14:41:09+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
2022-03-07 14:41:11+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-03-07 14:41:11+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
2022-03-07 14:41:12+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD

# --tail 옵션을 써서 출력할 줄의 수를 설정
PS C:\Users\Playdata> docker logs --tail 2 mysql
2022-03-07T14:37:48.241621Z 0 [Note] mysqld: ready for connections.
Version: '5.7.37'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```



```dockerfile
# since 옵션에 유닉스 시간을 입력해 특정시간 이후의 로그를 확인
PS C:\Users\Playdata> docker logs --since 1474765979 mysql
2022-03-07 14:35:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
2022-03-07 14:35:58+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
..

# -t 옵션으로 타임스탬프를 표시
# -f 옵션을 써서 로그를 스트림으로 확인 <애플리케이션 개발할 때 유용>
PS C:\Users\Playdata> docker logs -f -t mysql
2022-03-07T14:35:57.979848000Z 2022-03-07 14:35:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.37-1debian10 started.
2022-03-07T14:35:58.060013300Z 2022-03-07 14:35:58+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
...

# docker logs 명령어는 run 명령어에서 -i -t 옵션을 설정해 docker attach 명령어를 사용가능
# 컨테이너 내부에서 bash 셀 등을 입출력한 내용 확인가능
PS C:\Users\Playdata> docker run -i -t --name logstest ubuntu:14.04
root@e6281cd0b861:/# echo test!
test!
PS C:\Users\Playdata> docker logs logstest
root@e6281cd0b861:/# echo test!
test!
root@e6281cd0b861:/# exit
exit

```

- 기본적으로 위와 같은 컨테이너 로그는 JSON 형태로 도커 내부에 저장
- 이 파일은 다음 경로에 컨테이너ID로 시작하는 파일명으로 저장된다.

```dockerfile
# 아래의 log 파일의 내용을 cat, vi 편집기 등으로 확인하면 logs 명령으로 정제되지 않은 JSON 데이터 확인
# 
# sudo에서
$ cat /var/lib/docker/containers/컨테이너ID/컨테이너ID-json.log

# --log-opt 옵션으로 컨테이너 json 로그 파일의 최대 크기를 지정
# max-size는 로그 파일의 최대 크기, max-file은 로그 파일의 개수를 의미
PS C:\Users\Playdata> docker run -it --log-opt max-size=10k --log-opt max-file=3 --name log-test ubuntu:14.04
```

- 어떠한 설정도 하지 않았다면 도커는 위와 같이 컨테이너 로그를 JSON 파일로 저장하지만 그 밖에도 각종 로깅 드라이버를 사용하게 설정해 컨테이너 로그를 수집할 수 있다. 사용 가능한 드라이버의 대표적인 예로 syslog, journald, fluentd, awslogs 등이 있다.

2.2.8.2 SYSLOG 로그

- 컨테이너의 로그는 JSON뿐 아니라 syslog로 보내 저장하도록 설정
- syslog는 커널, 보안 등 시스템과 관련된 로그, 애플리케이션의 로그 등 다양한 종류의 로그 수집 및 저장, 그리고 분석
- 다음 명령어를 입력해 syslog에 로그를 저장하는 컨테이너를 생성하고 로그를 확인

```dockerfile
# 리눅스 위에서 돌리신다면 /var/log/syslog or /var/log/messages에서 확인 가능
PS C:\Users\Playdata> docker run -d --name syslog_container --log-driver=syslog ubuntu:14.04 echo syslogtest
043cb065125afd3a705360779de1e633b4e1fa5173c81ecce0d903add6b8e17d
docker: Error response from daemon: failed to initialize logging driver: Unix syslog delivery error.

```

- syslog 로깅 드라이버는 기본적으로 로컬호스트의 syslog에 저장하므로 운영체제 및 배포관에따라 syslog 파일의 위치를 알아야 이를 확인
- 우분투 14.04는 /var/log/syslog, CemtOS와 RHEL은 /var/log/messages 파일에서
- 우분투 16.04와 CoreOS는 journalctl -u docker.service 명령어로 확인

```dockerfile
# 서버 호스트에 rsyslog 서비스가 시작하도록 설정된 컨테이너를 구동하고 rsyslog 컨테이너를 생성
# rsyslog.conf 파일을 열어 syslog 서버를 구동시키는 항목의 주석을 해체한 후 변경사항 저장

PS C:\Users\Playdata> docker run -i -t -h rsyslog --name rsyslog_server -p 514:514 -p 514:514/udp ubuntu:14.04
root@rsyslog:/# vi /etc/rsyslog.conf
..
# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
..
# 저장 후 서비스 재시작
root@rsyslog:/# service rsyslog restart
 * Stopping enhanced syslogd rsyslogd                                                          [ OK ]
 * Starting enhanced syslogd rsyslogd                                                          [ OK ]
```

- --log-opt는 로깅 드라이버에 추가할 옵션을 뜻하며, syslog-address에 rsyslog 컨테이너에 접근할 수 있는 주소를 입력
- tag는 로그 데이터가 기록될 때 함께 저장될 태그이며 로그를 분류하기 위해 사용
- 또한 --log-opt는 syslog--facility를 쓰면 로그가 저장될 파일을 변경
- facility는 로그를 생성하는 주체에 따라 로그를 다르게 저장하는 것
- 여러 애플리케이션에서 수집되는 로그를 분류하는 방법
- facility 옵션을 쓰면 rsyslog 서버 컨테이너에 해당 facility에 해당하는 새로운 로그 파일 생성



2.2.8.3 fluentd 로깅

- fluentd는 각종 로그를 수집하고 저장할 수 있는 기능을 제공하는 오픈소스 도구로서, 도커 엔진의 컨테이너 로그를 fluentd를 통해 저장할 수 있도록 플러그인을 공식적으로 제공한다.
- fluentd은 데이터 포맷으로 JSON을 사용하기 때문에 쉽게 사용할 수 있을뿐만 아니라 수집되는 데이터를 AWS S3, HDFS, MongoDB 등 다양한 저장소에 저장할 수 있는 장점이 있음
- fluentd와 몽고 DB를 연동해 데이터를 저장하는 방법
- 어떤 컨테이너가 fluentd에 접근하고, fluentd는 몽고DB에 데이터를 저장하는 구조이다.

![example](https://user-images.githubusercontent.com/47745785/119222217-2c3cbd80-bb2e-11eb-9c2a-b148f97725ac.png)

- 위 그림대로 도커 서버,fluentd 서버, 몽고DB 서버로 구성되므로 서버는 최소 3대가 필요하지만 사용자의 환경에 맞게 변경해서 사용

  

```dockerfile
# 몽고DB설치 
PS C:\Users\Playdata> docker run --name mongoDB -d -p 27017:27017 mongo
Unable to find image 'mongo:latest' locally

# 우분투로 접속 < 파워셀에선 이용불가 >
mink@DESKTOP-U4KHBSV:~$ vi fluent.conf

# fluentd.conf 파일이 저장된 디렉토리에서 fluentd 컨테이너 생성
# -v옵션을 이용해 fluentd 컨테이너에 공유 및 설정 파일 사용
mink@DESKTOP-U4KHBSV:~$  docker run -d --name fluentd -p 24224:24224 -v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf -e FLUENTD_CONF=fluent.conf alicek106/fluentd:mongo
Unable to find image 'alicek106/fluentd:mongo' locally
mongo: Pulling from alicek106/fluentd

```

- 도커 허브의 fluentd 이미지에는 몽고DB에 연결하는 플러그인이 내장돼 있지 않습니다. alicek106/fluentd:mongo 이미지는 공식 fluentd 이미지에 몽고 DB 플러그인을 설치하는 것

```dockerfile
# --log-driver를 fluentd로 설정하고 --log-opt의 fluentd-address 값에 fluentd 서버 주소 지정
# --log-opt tag를 명시함으로써 로그의 태그를 docker.nginx.webserver로 지정했지만
mink@DESKTOP-U4KHBSV:~$ docker run -p 80:80 -d --log-driver=fluentd --log-opt fluentd-address=192.168.0.101:24224 --log-opt tag=docker.nginx.webserver nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx

# 이후는 오류 발생 후에 공부할 예정.


```

2.2.8.4 아마존 클라우드워치 로그

- AWS(Amazon Web Service)에서는 로그 및 이벤트 등을 수집하고 저장해 시각적으로 보여주는 클라우드워치(CloudWatch)를 제공

- 컨테이너에서 드라이버 옵션을 설정하는 것만으로 클라우드워치 로깅 드라이버를 사용

  - 클라우드워치를 사용하는 것은 아래와 같은 단계

    - 클라우드워치에 해당하는 IAM 권한 생성

    - 로그 그룹 생성

    - 로그 그룹에 로그 스트림(LogStream)생성

    - 클라우드워치의 IAM 권한을 사용할 수 있는 EC2 인스턴스 생성과 로그 전송

      

   1. 클라우드워치에 해당하는 IAM 권한 생성 (※ 모든 생성은 다 서울로 해야함!)

      - AWS 사이트 접속한 다음 콘솔에 로그인하기

      - 관리 콘솔 메뉴 중 '보안, 자격 증명 및 규정 준수' 들어가기

      - 왼쪽 사이드 바의 [역할] 탭을 클릭한 후 [역할 만들기] 버튼을 눌러 새로운 IAM 권한을 생성

      - '역할을 사용할 서비스 선택' 항목에서 [EC2]를 선택

      - [권한 추가] 항목의 권한 정책 필터에서 CloudWatchFull를 입력한 뒤 체크하기

      - 마지막으로 해당 IAM 역할의 이름을 입력해 새로운 권한생성

        



  2. 로그 그룹 생성

     - 홈에서 검색창에 [CloudWatch]를 검색하여 접속

     - [로그] 항목 안에 [로그 그룹]으로 들어가 [로그 그룹 생성]을 하기

     - 로그 그룹을 생성했으면 만들어진 로그를 클릭하고 안에 들어가 우측 하단에 [로그 스트림 생성] 버튼 클릭하기

       

3. EC2 생성하기

   - 검색 창에 EC2 검색하고 들어가기

   - EC2 [인스턴스] 항목 안에 [인스턴스] 들어가서 인스턴스 시작하기

   - 인스턴스에서 사용할 운영체제를 선택 (Ubuntu Server 18.04 LTS)

   - 높은 성능의 도커를 사용할 시 이용요금을 많이 지불해야하기때문에 무료로 제공되는 t2 micro클릭하고 [다음:인스턴스 세부 정보 구성] 들어가기

   - EC2 인스턴스에서 mycloudwatch라는 클라우드워치를 사용하도록 권한을 추가한다.

   - 마지막으로 검토 및 시작하여 검토를 다 하면 시작하기를 한다.

   - 그러면 이렇게 기존 키 페어에대한 생성이 뜬다. 

   - 키 페어 이름을 정하고 키 페어를 다운받고 시작하면 된다.

     

 4. Mobaxtem으로 접속하여 ssh 연결하기

    - 만들어 놓은 컨테이너에 정보를 확인하면 퍼블릭 IPv4 DNS 주소를 복사한다.

    - 그 다음 Mobaxtem에서 [session]을 클릭 후 [ssh]에 접속하여 host와 key값을 설정한다.

    - 그리고 접속하면 [login as:] 가 뜨는데 여기엔 ubuntu라고 적는다

    - 다음 과정은 https://deepmal.tistory.com/21 이 사이트를 참고한다.

    - 과정을 마치면  밑에 코드를 작성하여 container를 생성한다. 

      

      ```dockerfile
      # 컨테이너 생성후 테스트하기
      # 로깅 드라이버로 awslogs(클라우드워치)를 사용할 수 있게 설정한다
      # 로그 그룹과 스트림은 mylog과 mylogstream을 사용
      # 리전으로는 EC2 인스턴스가 아닌 로그 그룹 및 스트림이 생성된 리전을 입력해야 함
      # ap-northeast-2는 아시아 태평양(서울) 리전의 코드입니다.
      $ sudo docker run -i -t --log-driver=awslogs --log-opt awslogs-region=ap-northeast-2 --log-opt awslogs-group=mylog --log-opt awslogs-stream=mylogstream ubuntu:14.04   
      root@7e20cacde40e:/# echo test!
      test!
      ```

      

      

    - 클라우드워치의 mylogstream 로그 스트림 내의 로그를 확인하면 로그가 정상적으로 수집됌을 알 수 있다.

      



2.2.9 컨테이너 자원 할당 제한

- 컨테이너를 생성하는 run, create 명령어에서 컨테이너의 자원 할당량을 조정하도록 옵션을 입력할 수 있다.

- 컨테이너에 자원 할당 옵션을 설정하면 호스트와 다른 컨테이너의 동작을 방해하지 않게해준다.

- 컨테이너에 설정된 자원 제한을 확인하는 가장 쉬운 방법은 docker inspect 명령어를 입력하는 것이다.

  ```dockerfile
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker inspect rsyslog_server
  [
      {
  ```

  

```dockerfile
- run 명령어에서 설정된 컨테이너의 자원 제한을 변경하려면 update 명령어를 사용합니다.
# docker update (변경할 자원 제한) (컨테이너 이름)
# docker update --cpuset-cpus=1 centos ubuntu
```

2.2.9.1 컨테이너 메모리 제한

- docker run 명령어에 --memory를 지정해 컨테이너의 메모리를 제한할 수 있다.

- 입력할 수 있는 단위는 m(megabyte), g(gigabyte)이며 제한할 수 있는 최소 메모리는 4MB입니다.

  ```dockerfile
  # 컨테이너의 메모리 사용량을 1GB로 제한
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --memory="1g" --name memory_1g nginx
  aa93f63b698b095c7360a66f0db37ca1cbc157da2064ce74389aef153b3d6554
  
  # inspect명령어로 메모리의 값을 확인(1G)
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker inspect memory_1g | grep \"Memory\"
              "Memory": 1073741824,
  
  # 컨테이너 메모리를 매우 적게할 시 에러문자가 뜨며 6MB 이상이여야 생성된다고한다.
  # 컨테이너 목록을 조회해도 생성되지 않았음.
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name memory_4m --memory="4m" mysql:5.7
  docker: Error response from daemon: Minimum memory limit allowed is 6MB.
  See 'docker run --help'.
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker ps -a --format "table {{.ID}}\t{{.Status}}\t{{.Names}}"
  CONTAINER ID   STATUS                      NAMES
  aa93f63b698b   Up 6 minutes                memory_1g
  ...
  # 기본적으로 컨테이너의 Swap 메모리는 메모리의 2배로 설정되지만 별도로 지정할 수 있다.
  # Swap메모리는 500MB, 메모리를 200MB로 설정해 컨테이너 생성
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -it --name swap_500m --memory=200m --memory-swap=500m ubuntu:14.04
  root@367e0be39c57:/#
  
  ```



2.2.9.2 컨테이너 CPU 제한

##  --CPU-SHARES

- --cpu-shares 옵션은 컨테이너에 가중치를 설정해 해당 컨테이너가 CPU를 상대적으로 얼마나 사용했는 지 알 수 있다.

- 시스템에 존재하는 CPU를 어느 비중만큼 나눠 쓸 것인지를 명시하는 옵션입니다.

  ```dockerfile
  # --cpu-shares 옵션은 상대적인 값을 가짐.
  # 아무런 설정을 하지 않았을 때 컨테이너가 가지는 값은 1024로, 이는 CPU 할당에서 1의 비중을 뜻함.
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -it --name cpu_share --cpu-shares 1024 ubuntu:14.04
  root@0382217a1af3:/#
  
  # 1개의 CPU를 가지는 호스트에서 간단 테스트
  # cpu_1024 컨테이너는 --cpu-shares 옵션을 이용해 1024의 값을 할당
  # 컨테이너의 명령어는 1개의 프로세스로 CPU에 부하를 주는 명령어(stress --cpu 1)로 설정
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name cpu_1024 --cpu-shares 1024 alicek106/stress stress --cpu 1
  Unable to find image 'alicek106/stress:latest' locally
  latest: Pulling from alicek106/stress
  f7277927d38a: Pull complete
  8d3eac894db4: Pull complete
  edf72af6d627: Pull complete
  3e4f86211d23: Pull complete
  d6e1f41c61e5: Pull complete
  Digest: sha256:35e7f4fb3481e223d0640888b1770d0c78731091344d6c3c5ed5c695b08a28de
  Status: Downloaded newer image for alicek106/stress:latest
  acd514a9088ce52351bda2cb094f04090a4b3aeacc223659e6a6ca8659769d42
  
  # cpu_1024 컨테이너의 CPU 사용률 확인
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ ps aux | grep stress
  mink      1519  0.0  0.0   8164   732 pts/2    S+   22:28   0:00 grep --color=auto stress
  
  ```

  

--cpuset-cpu

- 호스트에 CPU가 여러 개 있을 때 --cpuset-cpus 옵션을 지정해 컨테이너가 특정 CPU만 사용하도록 설정
- CPU 집중적인 작업이 필요하다면 여러 개의 CPU를 사용하도록 설정해 작업을 적절하게 분배하는 것

```dockerfile
# 컨테이너가 3번째 CPU만 사용하도록 설정
mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name cpuset_2 --cpuset-cpus=2 alicek106/stress stress --cpu 1
03623573c3173da6e7f59141a2bde16fd7a6bf6a75edd4646cdc5cd7b4e1f934
```



--cpu-period, --cpu-quota

- 컨테이너의 CFS(Completely Fair Scheduler) 주기는 기본적으로 100ms로 설정되지만 run 명령어의 옵션 중 --cpu-period와 --cpu-quota로 이 주기를 변경할 수 있다.

  ```dockerfile
  # --cpu-period의 값은 기본적으로 100000이며,  100ms이다.
  # --cpu-quota는 --cpu-period에 설정된 시간 중 CPU 스케쥴링에 얼마나 할당할 것인지를 설정
  # 100000(--cpu-period) 중 25000(--cpu-quota)만큼을 할당해 CPU 주기가 1/4로 줄었으므로 일반적인 컨테이너보다 CPU 성능이 1/4 정도로 감소
  # 즉, 컨테이너는 [--cpu-quota 값]/[--cpu-period 값]만큼 CPU시간을 할당
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name quota_1_4 --cpu-period=100000 --cpu-quota=25000 alicek106/stress stress --cpu 1
  03d09de8e34b3841ca5854131d366a9fde6bb48af8486ede84282cceaee89e66
  
  # 성능 비교를 위해 다음 명령어로 컨테이너를 추가로 생성
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name quota_1_1 --cpu-period=100000 --cpu-quota=100000 alicek106/stress --cpu 1
  1ebf8164a4b8395c5675e07e3b64583dfb6384443c7fbaa6eff1e651cf4dbf5f
  
  # stress를 확인하려고 하면 값이 제대로 안 나옴
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ ps aux | grep stress
  ```

  

--cpus

- --cpus 옵션은 --cpu-period, --cpu-quota와 동일한 기능을 하지만 좀 더 직관적으로 CPU의 개수를 직접 지정한다는 점에서 다르다.

  ```dockerfile
  # --cpus 옵션에 0.5를 설정하면 --cpu-period=100000 또는 --cpu-quota=500000과 동일하게 컨테이너의 CPU를 제한
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -d --name cpus_container --cpus=0.5 alicek106/stress stress --cpu 1
  d516c83303f329545cd3500f37c9ae7ba617c5d54eceb35c5c22dddb744bce3b
  
  ```

  - 병렬 처리를 위해 CPU를 많이 소모하는 워크로드를 수행해야 한다면 --cpu-share, --cpus, --cpu-period. --cpu-quota 옵션보다는 --cpuset-cpu 옵션을 사용하는 것이 좋다.
  - --cpuset-cpu 옵션을 사용하면 특정 컨테이너가 특정 CPU에서만 동작하는 CPU 친화성(Alfinity)을 보장할 수 있고, CPU 캐시 미스 또는 컨텍스트 스위칭과 같이 성능을 하락시키는 요인을 최소화할 가능성이 높아지기 때문이다.

2.2.9.3 Block I/O 제한

- 컨테이너를 생성 할 때 아무런 옵션도 설정하지 않으면 컨테이너 내부에서 파일을 읽고 쓰는 대역폭에 제한이 설정되지 않습니다.

- run 명령어에서 --device-write-bps, --device-read-bps, --device-write-iops, --device-read-iops 옵션을 지정해 블록 입출력을 제한

- Direct I/O의 경우에만 블록 입출력이 제한되며, Buffered I/O는 제한되지 않는다.

- --device-write-bps, --device-read-bps는 각기 쓰고 읽는 작업의 초당 제한을 설정한다.

- kb, mb, gb 단위로 제한

  ```dockerfile
  # 다음 명령어로 컨테이너를 생성하면 초당 쓰기 작업의 최대치가 1MB로 제한
  # Block I/O를 제한하는 옵션은 [디바이스 이름]:[값] 형태로 설정
  # AWS의 EC2 인스턴스에서 테스트했기 때문에 /dev/xvda라는 디바이스를 사용
  # 이 디바이스에서 컨테이너의 저장 공간을 할당받고 있어 /dev/xvda:1mb로 설정
  # devicemapper를 스토리지 드라이버를 사용하는 도커 엔진에서 루프 디바이스를 스토리지로 사용한다면 /dev/loop0:1mb와 같은 형식
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -it --device-write-bps /dev/xvda:1mb ubuntu:14.04
  WARNING: Your kernel does not support BPS Block I/O write limit or the cgroup is not mounted. Block I/O BPS write limit discarded.
  
  # 생성된 컨테이너에서 쓰기 작업을 테스트해보면 속도가 초당 1MB로 제한되는 것을 확인
  # 10MB의 파일을 Direct I/O를 통해 쓰기 작업을 수행
  root@a599ccee5f81:/# dd if=/dev/zero of=test.out bs=1M count=10 oflag=direct
  10+0 records in
  10+0 records out
  10485760 bytes (10 MB) copied, 0.0224329 s, 467 MB/s
  ```

  ```dockerfile
  # CPU의 자원 할당에서 --cpu-share에 상대적인 값을 입력했던 것처럼 --device-write-iops,   --device-read-iops에도 상대적인 값 입력
  # --device-write-iops의 값이 2배 차이 나는 컨테이너로 쓰기 작업을 수행하면 수행 시간 또는 2배 가량 차이가 남
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -it --device-write-iops /dev/xvda:5 ubuntu:14.04
  WARNING: Your kernel does not support IOPS Block write limit or the cgroup is not mounted. Block I/O IOPS write limit discarded.
  root@15421ed9cdcd:/# dd if=/dev/zero of=test.out bs=1M count=10 oflag=direct
  10+0 records in
  10+0 records out
  10485760 bytes (10 MB) copied, 0.0447474 s, 234 MB/s
  
  mink@DESKTOP-U4KHBSV:/mnt/c/Users/Playdata$ docker run -it --device-write-iops /dev/xvda:10 ubuntu:14.04
  WARNING: Your kernel does not support IOPS Block write limit or the cgroup is not mounted. Block I/O IOPS write limit discarded.
  root@e538d78a5ff6:/# dd if=/dev/zero of=test.out bs=1M count=10 oflag=direct
  10+0 records in
  10+0 records out
  10485760 bytes (10 MB) copied, 0.029512 s, 355 MB/s
  ```

  

2.2.9.4 스토리지 드라이버와 컨테이너 저장 공간 제한

- 도커 엔진은 컨테이너 내부의 저장 공간을 제한하는 기능을 보편적으로 제공하지는 않지만, 도커의 스토리지 드라이버나 파일 시스템 등이 특정 조건을 만족하는 경우에만 이 기능을 제한적으로 사용 할 수 있다.
- 컨테이너 애플리케이션이 해당 스토리지 드라이버에 적합하지 않다면 이 기능을 사용하지 않는 것이 좋을 수도 있다.
- 컨테이너의 저장 공간을 제한하지 않는다는 선택지로 고려





























































