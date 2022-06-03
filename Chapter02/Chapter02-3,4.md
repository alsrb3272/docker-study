**2.3** **도커 이미지**

도커는 기본적으로 도커 허브라는 중앙 이미지 저장소에서 이미지를 내려받는다.

이미지 저장소를 다른 사람들에게 공개하지 않기 위해 비공개 저장소를 사용하려면 비공개 저장소의 수에 따라 요금을 지불해야 한다. 이를 해결하기 위해 도커 이미지 저장소를 직접 구축해 비공개로 사용할 수 있다.

```dockerfile
# docker search 명령어는 도커 허브에서 이미지를 검색
$ docker search ubuntu
```

![image-20220513144100658](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220513144100658.png)



- 2.3.1 **도커 이미지 생성**

  컨테이너에 애플리케이션을 위한 특정 개발 환경을 직접 구축한 뒤 사용자만의 이미지를 직접 생성해야 하는 경우가 많다. 

  ```dockerfile
  # 이미지로 만들 컨테이너 생성 후 내부에 first라는 이름의 파일을 하나 생성해 기존의 이미지로부터 변경사항 만듬
  $ docker run -i -t --name commit_test ubuntu:14.04
  root@7d3259931d5e:/# echo test_first! >> first
  
  # docker commit [OPTIONS] CONTAINER [REPOSITORY:TAG]
  # 이미지의 이름을 commit_test, 태그를 first. -a 옵션은 author을 뜻하고, -m 옵션은 커밋 메세지를 뜻하며, 이미지에 포함도리 부가 설명이다.
  PS C:\Windows\system32> docker commit -a "alicek106" -m "my first commit" commit_test commit_test:first
  sha256:709cb5b214a7d1bc4cc9205bc65d008de6f09bce4426822ed340512cec5dad5a
  ```

   최상위 디렉터리에 first라는 파일이 있는 도커 이미지가 생성되었다.

  ```dockerfile
  # commit_test;first 이미지로 새로운 이미지를 생성.
  $ docker run -i -t --name commit_test2 commit_test:first
  root@146c95f4fbb5:/# echo test_second! >> second
  
  # commit_test:first 이미지로 컨테이너를 생성한 뒤 second라는 파일을 추가해 commit_test:second라는 이미지를 생성.
  $ docker commit -a "aliicek106" -m "my second commit" commit_test2 commit_test:second
  ```

![image-20220513152427905](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220513152427905.png)

- 2.3.2 **이미지 구조 이해**

  - 이미지를 좀 더 효율적으로 다루기 위해 컨테이너가 어떻게 이미지로 만들어지며, 이미지의 구조는 어떻게 되는 지 알 필요가 있다. 

    ```dockerfile
    # inspect 명령어는 컨테이너뿐만 아니라 네트워크, 볼륨, 이미지 등 모든 도커 단위의 정보를 얻을 때 사용할 수 있음.
    # 단, 이름이 중복될 경우 컨테이너에 대해 먼저 수행되므로 --type을 명시
    $ docker inspect ubuntu:14.04
    $ docker inspect commit_test:first
    $ docker inspect commit_test:second
    ```

  - ubuntu:14.04, commit_test:first, commit_test:second 이미지의 Layers항목

    | ![image-20220513152941958](file://C:/Users/mink/AppData/Roaming/Typora/typora-user-images/image-20220513152941958.png?lastModify=1652429892) |
    | :----------------------------------------------------------: |
    |                         ubuntu:14.04                         |

    | ![image-20220513153018375](file://C:/Users/mink/AppData/Roaming/Typora/typora-user-images/image-20220513153018375.png?lastModify=1652429892) |
    | :----------------------------------------------------------: |
    |                      commit_test:first                       |

    | ![image-20220513153031655](file://C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220513153031655.png?lastModify=1652429892) |
    | :----------------------------------------------------------: |
    |                      commit_test:second                      |

  - 위에 해당하는 Layers들을 그림으로 쉽게 예시로 보여줌

    ![example](https://user-images.githubusercontent.com/47745785/119522732-2c9bb980-bdb7-11eb-9c24-dd9ecf7d4161.png)

  - docker images에서 위 3개의 이미지 크기가 각각 188MB라고 출력돼도 188MB 크기의 이미지가 3개 존재하는 것은 아니다. 

  - 이미지를 커밋할 떄 컨테이너에서 변경된 사항만 새로우누 레이어로 저장하고, 그 레이어를 포함해 새로운 이미지를 생성하기 때문임.

    ![example](https://user-images.githubusercontent.com/47745785/119523166-800e0780-bdb7-11eb-84e6-b7fd6430f225.png)

  - ```
    # 이러한 이미지의 레이어 구조는 docker history 명령을 통해 좀 더 쉽게 확인할 수 있다. 
    PS C:\Windows\system32> docker history commit_test:second
    
    # 생성한 이미지 삭제하는 방법
    $ docker stop commit_test2
    $ docker rm commit_test2
    $ docker rmi commit_test:first
    
    # 컨테이너가 사용 중인 이미지를 docker rm -f로 강제로 삭제하면 이미지의 이름이 <none>으로 변경되며, 이러한 이미지들을 댕글링(dangling) 이미지라고 함.
    # 댕글링 이미지는 docker images -f dangling=true 명령어를 사용해 별도 확인
    $ docker images -f dangling=true
    
    # 사용 중이지 않은 댕글링 이미지는 docker image prune 명령어로 한꺼번에 제거
    $ docker image prune
    
    # commit_test:first 이미지를 기반으로 하는 하위 이미지인 commit_test:second가 존재하므로 해당 이미지의 레이어 파일이 삭제되지는 않음
    # 이미지를 바로 삭제
    $ docker rmi commit_test:second
    ```

    <img src="C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220513170521188.png" alt="image-20220513170521188" style="zoom:150%;" />

    ![image-20220513170909156](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220513170909156.png)



- 2.3.3 **이미지 추출**

  - 도커 이미지를 별도로 저장하거나 옮기는 등 필요에 따라 이미지를 단일 바이너리 파일로 저장해야 할 때가 있다. 

    ```dockerfile
    # docker save 명령어를 사용하면 컨테이너의 커맨드, 이미지 이름과 태그 등 이미지의 모든 메타데이터를 포함해 하나의 파일로 추출
    $ docker save -o ubuntu_14_04.tar ubuntu:14.04
    # docker load 명령어는 도커에 다시 로드하여 완전히 동일한 이미지가 도커 엔진에 생성
    $ docker load -i ubuntu_14_04.tar
    ```

  

- 2.3.4 **이미지 배포**

  - 2.3.4.1 **도커 허브 저장소**

    도커 허브를 가입하고 저장소를 생성해서 이미지를 올리는 과정

    1. 도커 허브 사이트에서 Repository로 들어간 뒤 Create Repository를 실행(도커 허브 : https://hub.docker.com/)

    2. 이미지 저장소에 대한 정보 입력 
       ex) my-image-name,  This image repository is for testing.

    3. 저장소에 이미지 올리기

       ```dockerfile
       # 기본 이미지 만들기
       $ docker run -it --name commit_container1 ubuntu:14.04
       
       # 이미지 커밋하여 저장
       $ docker commit commit_container1 my-image-name:0.0
       
       # docker tag[기존의 이미지 이름][새롭게 생성될 이미지 이름]
       $ docker tag my-image-name:0.0 alsrb3272/my-image-name:0.0
       
       # 도커 허브 서버에 로그인
       # 도커 엔진에서 로그인한 정보는 /[계정명]/.docker/config.json 파일에 저장된다.
       # 로그인 정보를 삭제하려면 docker logout 실행
       $ docker login
       Username: alsrb3272
       Password: ~~
       Login Succeeded
       
       # push를 이용하여 docker 이미지를 저장소로 올리기
       $ docker push alsrb3272/my-image-name:0.0
       
       # login없이 이미지를 pull로 내려받기
       $ docker pull alsrb3272/my-image-name:0.0
       
       #### 팀 단위 이미지 배포는 유료이기때문에 생략 ####
       ```

       ![image-20220514154607063](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220514154607063.png)

  - **2.3.4.2 도커 시설 레지스트리**

    도커 사설 레지스트리(Docker Private Registry)를 사용하면 개인 서버에 이미지를 저장할 수 있는 저장소를 만들 수 있다. 이 레지스트리는 컨테이너로서 구현되므로 이에 해당하는 도커 이미지가 존재한다. 

    ```dockerfile
    # 사설 레지스트리 컨테이너 생성
    # --restart 옵션은 컨테이너가 종료됐을 때 재시작에 대한 정책을 설정
    # always는 컨테이너가 정지될 때마다 다시 시작하도록 설정
    $ docker run -d --name myregistry -p 5000:5000 --restart=always registry:2.6
    
    # curl은 HTTP 요청을 보내는 도구 중 하나이다
    $ C:\Windows\system32>curl localhost:5000/v2/
    {}
    ```

    ```dockerfile
    # 사설 레지스트리에 이미지 Push하기
    # 레지스트리 컨테이너에 이미지를 올리려면 이미지의 접두어를 레지스트리 컨테이너가 존재하는 호스트의 IP와 레지스트리 컨테이너의 5000번 포트와 연결된 호스트의 포트로 설정
    $ docker tag my-image-name:0.0 ${DOCKER_HOST_IP}:5000/my-image-name:0.0
    
    # 호스트 Ip와 레지스트리 컨테이너 연결
    $ docker tag my-image-name:0.0 192.168.35.43:5000/my-image-name:0.0
    
    # 레지스트리 컨테이너에 이미지 올리기
    $ docker push 192.168.35.43:5000/my-image-name:0.0
    
    # push로 이미지 올리기
    $ docker pull 192.168.35.43:5000/my-image-name:0.0
    ! The push refers to repository [192.168.35.43:5000/my-image-name]
    Get "https://192.168.35.43:5000/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) !
    ### push가 되지않는 이유
    # 도커 데몬은 HTTPS를 사용하지 않은 레지스트리 컨테이너에 접근하지 못하도록 설정하기 때문.172.17.0.0/16
    # HTTPS를 사용하려면 인증서를 적용해 별도로 설정.
    ```

    **Nginx 서버로 접근 권한 생성**

    Nginx 서버 컨테이너를 생성하고 레지스트리 컨테이너와 연동하여 인증 절차를 만드는 방법.

    로그인 인증 기능은 보안을 적용하지 않은 레지스트리 컨테이너에서는 사용할 수 없으므로 여기서는 스스로 인증한(Self-Signed) 인증서와 키를 발급함으로써 TLS를 적용하는 방법

    ```dockerfile
    # 디렉토리 생성
    $ mkdir certs
    
    # Self-signed ROOT 인증서(ca)파일 생성
    $ openssl genrsa -out ./certs/ca.key 2048
    
    # 이 Root 인증서로 레지스트리 컨테이너에 사용될 인증서
    $ openssl req -x509 -new -key ./certs/ca.key -days 10000 -out ./certs/ca.crt
    
    ### 인증 서명 요청 파일인 CSR파일을 생성하고 ROOT 인증서로 새로운 인증서를 발급. 
    $ openssl genrsa -out ./certs/domain.key 2048
    
    # 레지스트리 컨테이너가 존재하는 도커 호스트 서버의 IP나 도메인 이름을 입력 
    $ openssl req -new -key ./certs/domain.key -subj /CN=192.168.35.43 -out ./certs/domain.csr
    
    # IP로 Nginx 서버 컨테이너에 접근
    $ echo subjectAltName = IP:192.168.35.43 > extfile.cnf
    $ openssl x509 -req -in ./certs/domain.csr -CA ./certs/ca.crt -CAkey ./certs/ca.key -CAcreateserial -out ./certs/domain.crt -days 10000 -extfile extfile.cnf
    
    # window htpasswd 설치
    $ sudo apt-get install apache2-utils -y
    
    # 레지스트리에 로그인할 때 사용할 계정과 비밀번호를 저장하는 파일을 생성
    mink@DESKTOP-HS1HH67:~$ htpasswd -c htpasswd alsrb3272
    New password:
    Re-type new password:
    Adding password for user alsrb3272
    mink@DESKTOP-HS1HH67:~$ mv htpasswd certs/
    ```

    ```dockerfile
    # vi certs/nginx.conf 설정 파일 생성
    # Nginx 서버에서 SSL 인증에 필요한 각종 파일의 위치와 레지스트리 컨테이너의 프락시(Proxy) 설정
    ###하단 내용 넣기###
    upstream docker-registry {
      server registry:5000;
    }
    server {
      listen 443;
      server_name 192.168.35.43;
      ssl on;
      ssl_certificate /etc/nginx/conf.d/domain.crt;
      ssl_certificate_key /etc/nginx/conf.d/domain.key;
      client_max_body_size 0;
      chunked_transfer_encoding on;
    
      location /v2/ {
        if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
          return 404;
        }
        auth_basic "registry.localhost";
        auth_basic_user_file /etc/nginx/conf.d/htpasswd;
        add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;
    
        proxy_pass http://docker-registry;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 900;
      }
    }
    ##########################
    # 기존에 생성한 레지스트리 컨테이너 삭제하고 다시 생성
    $ docker stop myregistry; docker rm myregistry
    # Nginx 서버 컨테이너 생성
    # 위에서 생성한 nginx.conf, domain.crt, domain.key 파일이 존재하는 certs 디렉터리를 -v 옵션으로 컨테이너에 공유
    # --link 옵션은 내부 IP를 알 필요 없이 항상 컨테이너에 별명으로 접근하도록 설정
    $ docker run -d --name myregistry --restart=always registry:2.6
    $ docker run -d --name nginx_fronted -p 443:443 --link myregistry:registry -v $(pwd)/certs/:/etc/nginx/conf.d nginx:1.9
    
    # 컨테이너 생성 확인
    $ docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}"
    
    $ sudo cp certs/ca.crt /usr/local/share/ca-certificates/
    $ sudo update-ca-certificates
    $ service docker restart
    $ docker start nginx_frontend
    
    # 이 부분 해결이 안 됨
    $ docker login 192.168.35.43
    ...
    ...
    ### Error response from daemon: Get "https://192.168.35.43/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) ###
    
    $ docker tag uaena:0.0 192.168.35.43/uaena:0.0
    $ docker push 192.168.35.43/uaena:0.0
    ```

    ![image-20220517160032383](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220517160032383.png)

    <img src="C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220517162903678.png" alt="image-20220517162903678" style="zoom:200%;" />

2.4 **Dockerfile**

- **2.4.1 이미지를 생성하는 방법**

  1. **아무것도 존재하지 않는 이미지로 컨테이너 생성**
  2. **애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것을 확인**
  3. **컨테이너를 이미지로 커밋(commit)**

  위의 방법은 애플리케이션이 동작하는 환경을 구성하기 위해 일일이 수작업으로 패키지를 설치해야한다.
  
  그래서 도커는 아래와 같이 일련의 과정을 손쉽게 기록하고 수행할 수 있는 빌드 명령어를 제공한다. 완성된 이미지를 생성하기 위해 컨테이너에 설치해야 하는 패키지, 추가해야 하는 소스코드, 실행해야 하는 명령어와 셸 스크립트 등을 하나의 파일에 기록해 두면 도커는 이 파일을 읽어 컨테이너에서 작업을 수행한 뒤 이미지로 만들어낸다.
  
  
  
  ![example](https://user-images.githubusercontent.com/47745785/120413619-99a0e780-c393-11eb-91b2-6361d6f90561.png)
  
  이러한 작업을 기록한 파일의 이름을 **Dockerfile**이라 하며, 빌드 명령어는 **Dockerfile**을 사용하면 직접 컨테이너를 생성하고 이미지로 커밋해야 하는 번거로움을 덜 수 있을뿐더러 깃과 같은 개발 도구를 통해 애플리케이션의 빌드 및 배포를 자동화할 수 있다.
  
  **도커 허브(Docker Hub)** 등을 통해 배포할 때 이미지 자체를 배포하는 대신 이미지를 생성하는 방법을 기록해 놓은 **Dockerfile**을 배포할 수 있다. 
  
  
  
  **Dockerfile을 작성해야하는 이유**
  
  : 이미지를 생성하는 방법을 기록하는 것뿐만 아니라 이미지의 빌드, 배포측면에서도 매우 유리하며 애플리케이션에 필요한 패키지 설치 등을 명확히 할 수 있고 이미지 생성을 자동화할 수 있으며, 배포할 수 있기 때문이다.



- **2.4.2 Dockerfile 작성**

  Dockerfile을 사용하기 위한 간단한 시나리오로 웹 서버 이미지를 생성하는 예시

  ```dockerfile
  # 이번 절에 사용할 디렉터리 생성 및 안에 HTML 파일을 만들기
  $ mkdir dockerfile && cd dockerfile
  $ echo test >> test.html
  $ ls # html 파일 생성 확인
  
  # 새롭게 생성한 디렉터리 내부에 아래의 내용으로 Dockerfile 이름의 파일을 저장
  # Dockerfile은 한 줄이 하나의 명령어가 되고, 명령어를 명시한 뒤 옵션을 추가
  $ vi Dockerfile
  FROM ubuntu:14.04 # 사용할 베이스 이미지 설정
  LABEL "purpose"="practice" # 생성될 이미지의 라벨 설정
  RUN apt-get update 
  RUN apt-get install apache2 -y # 아파치 웹 서버 설치
  ADD test.html /var/www/html # html 파일을 디렉터리 추가
  WORKDIR /var/www/html # 
  RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
  EXPOSE 80
  CMD apachectl -DFOREGROUND
  
  # FROM : 생성할 이미지의 베이스 이미지. 반드시 한 번 이상 입력
  # LABEL : 이미지의 정보를 키:값 형태로 저장
  # RUN : 컨테이너 내부 명령어. 이미지를 빌드할 때 별도의 입력을 받아야 하는 RUN이 있다면 build 명령어는 이를 오류로 간주하고 빌드를 종료한다. 따라서 -y 옵션을 사용
  # ADD : 파일을 이미지에 추가한다. Dockerfile이 위치한 디렉터리에서 파일을 가져온다.
  # WORKDIR : 명령어를 실행할 디렉터리
  # EXPOSE : Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정
  # CMD : 컨테이너가 시작될 때마다 실행할 명령어를 설정. Dockerfile에서 한 번만 사용가능. CMD는 run 명령어의 이미지 이름 뒤에 입력하는 커맨드와 같은 역할을 하지만 docker run 명령어에서 커맨드 명령줄인자를 입력하면 Dockerfile에서 사용한 CMD 명령어는 run의 커맨드로 덮어 쓰인다. 저 위에서는 ubuntu:14.04 기본 내장 커맨드 /bin/bash가 Dockerfile CMD에 의해 덮어 쓰여진다.
  ```



- **2.4.3 Dockerfile 빌드**

  - **2.4.3.1 이미지 생성**

    dockerfile 디렉터리에서 명령어로 Dockerfile을 빌드하고, 실행

    ```dockerfile
    # -t 옵션은 생성될 이미지의 이름을 설정
    # -t 옵션을 사용하지 않으면 16진수 형태의 이름으로 이미지가 저장되므로 가급적이면 -t 옵션을 사용함.
    $ docker build -t mybuild:0.0 ./
    
    # -P 옵션은 이미지에 설정된 EXPOSE의 모든 포트를 호스트에 연결하도록 설정
    # EXPOSE로 노출된 포트를 호스트에서 사용 가능한 포트에 차례로 연결하므로 이 컨테이너가 호스트의 어떤 포트와 연결됐는지 확인할 필요
    $ docker run -d -P --name myserver mybuild:0.0
    b42473cd1d1997fab904ec281139db7f57f0c49505602b9bd279487ef8117f32
    
    # docker ps 또는 docker port 명령어로 컨테이너와 연결된 호스트의 포트를 확인
    # Dockerfile에서 ADD로 test, html 파일을, RUN으로 test2.html 파일을 웹 서버 디렉터리인 /var/www/html에 추가했으므로 [호스트IP]:[포트]/test.html 혹은 test2.html로 접근
    $ docker port myserver
    80/tcp -> 0.0.0.0:49153
    
    # Dockerfile에 이미지의 라벨을 purpose=practice로 설정했으므로 docker images 명령어의 필터에 이 라벨을 적용
    # --filter 옵션을 통해 해당 라벨을 가지는 이미지만 출력
    $ docker images --filter "label=purpose=practice"
    
    ## 'ip주소:연결된포트'로 접속하면 아파치 웹서버가 실행된것을 확인 ##
    ## test.html, test2.html로 확인가능 ##
    ```

    ![image-20220527155956896](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220527155956896.png)

    

  - 2.4.3.2 빌드 과정 살펴보기

    - 빌드 컨텍스트

      ![example](https://user-images.githubusercontent.com/47745785/120318183-86056a80-c31a-11eb-90d4-769d7e5f2e6d.png)

      - **<빌드 컨텍스트는 이미지를 생성하는 데 필요한 각종 파일, 소스코드, 메타데이터 등을 담고 있는 디렉터리>**

      - 빌드 컨텍스트는 Dockerfile에서 빌드될 이미지에 파일을 추가할 때 사용

      - Dockerfile에서 이미지에 파일을 추가하는 방법은 앞에서 설명할 ADD 외에도 COPY가 있음

      - 이미지 빌드를 시작하면 도커는 가장 먼저 **빌드 컨텍스트**를 읽어 들인 후  위 예제처럼 빌드 경로를 ./로 지정함으로써 test.html 파일을 빌들 컨텍스트에 추가했으며, ADD 명령어를 통해 빌드 컨텍스트에서 test.html 파일을 이미지에 추가

      - .dockerignore라는 파일을 사용해서 빌드 시 이 파일에 명시된 이름의 파일을 컨텍스트에서 제외

      ```dockerfile
      # vi .dockerignore
      test2.html
      # /home/[directory]이라면 *.html은 /home/[directory]/*.html에 해당하는 모든 파일
      *.html
      
      # */*.html -> /home/[directory]/*/*.html에 해당하는 파일을 제외하는 의미
      */*.html
      
      # test.htm을 접두어로 두고 "?" 자리에 임의의 1자리 문자가 들어가는 파일을 제외
      test.htm?
      
      # 특수한 파일을 설정하고 싶다면 "!"를 사용 < 특정 파일을 제외하지 않음을 뜻함>
      !test*.html
      ```

      

    - Dockerfile을 이용한 컨테이너 생성과 커밋

      - ADD, RUN 등의 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성되며 이를 이미지로 커밋
      - Dockerfile에서 명령어 한 줄이 실행될 때마다 이전 Step에서 생성된 이미지에 의해 새로운 컨테이너 생성
      - 이미지의 빌드가 완료되면  Dockerfile의 명령어 줄 수 만큼의 레이어가 존재하게 되며, 중간에 컨테이너도 같은 수만큼 생성되고 삭제

    - 캐시를 이용한 이미지 필드

      ![example](https://user-images.githubusercontent.com/47745785/120319765-56eff880-c31c-11eb-867f-57cde0641d43.png)

      ```dockerfile
      # 빌드 캐시를 사용하기 위한 테스트 Dockerfile
      $ vi Dockerfile2
      FROM ubuntu:14.04
      LABEL "purpose"="practice"
      RUN apt-get update
      
      # build -f or --file 옵션으로 Dockerfile의 이름을 지정
      # 새로운 이미지 빌드
      $ docker build -f Dockerfile2 -t mycache:0.0 ./
      
      # 로그를 보면 Using cache라는 출력 내용과 함께 별도의 빌드 과정이 진행되진 않고 바로 이미지 생성
      # 이전에 빌드했던 dockerfile에 같은 내용이 있다면 build 명령어는 이를 새로 빌드하지 않고 같은 명령어 줄까지 이전에 사용한 이미지 레이어를 활용해 이미지를 생성
      # 이는 같은 명령얼르 여러 번 실행해야 하는 여러 개의 이미지를 빌드 도주우 Dockerfile의 문법과 기타 오류가 발생했을 때 불필요하게 다시 명령어를 실행하지 않음
      
      # Dockerfile에 RUN git clone...을 사용하고 이미지를 빌드했다면 RUN에 대한 이미지 레이어를 계속 캐ㅐ시로 사용하기 때문에 실제 깃 저장소에서 리비전 관리가 일어나도 매번 빌드를 할 때마다 고정된 소스코드를 사용하게 됨
      --> 이 경우 캐시를 사용하지 않으려면 build 명령어에 --no-cache 옵션 추가
      $ docker build --no-cache -t mybuild:0.0 ./
      
      # 특정 Dockerfile을 확장해서 사용한다면 기존의 Dockerfile로 빌드한 이미지를 빌드 캐시로 사용
      # 예시로 도커 허브의 nginx 공식 저장소에서 nginx:latest 이미지를 빌드하는 Dockerfile에 일부 내용을 추가해 사용해본다면 nginx:latest 이미지를 캐시로 사용
      $ docker build --cache-from nginx -t my_extend-nginx:0.0 .
      ```

  - 2.4.3.3 멀티 스테이지를 이용한 Dockerfile 빌드하기

    ```dockerfile
    # Dockerfile을 통해 컴파일될 main.go 파일의 내용
    // main.go
    $ vi main.go
    package main
    import "fmt"
    func main(){
    	fmt.Println("hello world")
    }
    
    # Dockerfile_go
    $ vi Dockerfile_go
    FROM golang
    ADD main.go /root
    WORKDIR /root
    RUN go build -o /root/mainApp /root/main.go
    
    FROM alpine:latest
    WORKDIR /root
    COPY --from=0 /root/mainApp .
    CMD ["./mainAPP"]
    
    $ docker build -f Dockerfile_go -t go_helloworld ./
    # 위의 도커 파일에서 위쪽 문단만 작성하고 이미지를 만들면 golang 패키지 및 라이브러리 때문에 이미지의 크기가 큼
    # 이미지의 크기를 줄이기 위해 멀티 스테이지 빌드 방법을 사용
    # 멀티 스테이지 빌드는 하나의 Dockerfile 안에 여러 개의 FROM 이미지를 정의함으로써 빌드 완료 시 최종적으로 생성될 이미지의 크기를 줄이는 역할을 함
    
    # 두 번째 FROM 아래에서 사용된 COPY 명령어는 첫 번째 FROM에서 사용된 이미지의 최종 상태에 존재하는 /root/mainApp 파일을 두 번째 이미지인 alpine:latest에 복사
    # --from=0은 첫 번째 FROM에서 빌드된 이미지의 최종 상태를 의미
    # 첫 번째 FROM 이미지에서 빌드한 /root/mainApp 파일을 두 번째의 FROM에 명시된 이미지인 alpine:latest 이미지에 복사
    ```

    ![image-20220527182219587](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220527182219587.png)

- **2.4.4 기타 Dockerfile 명령어**

  - 2.4.4.1 ENV, VOLUME, ARG, USER

    - ENV : 컨테이너에서 사용할 환경변수, ${ENV_NAME}, $ENV_NAME 형태로 사용.

      ```dockerfile
      # ENV를 사용하는 Dockerfile
      # vi Dockerfile_env
      FROM ubuntu:14.04
      ENV test /home
      WORKDIR $test
      RUN touch $test/mytouchfile
      
      # 이미지를 빌드하고 컨테이너를 생성해 환경변수를 확인
      # /home 값이 적용된것을 확인
      # run -e 옵션을 사용해 같은 이름의 환경변수를 사용하면 기존의 값은 덮어짐
      $ docker build -f Dockerfile_env -t myenv:0.0 ./
      $ docker run -it --name env_test myenv:0.0 /bin/bash
      $ root@1fsj948e18:/home# echo $test
      
      ```

    - VOLUME : 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리를 설정한다.

      ```dockerfile
      # VOLUME를 사용하는 Dockerfile
      # vi Dockerfile_volume
      FROM ubuntu:14.04
      RUN mkdir /home/volume
      RUN echo test >> /home/volume/testfile
      VOLUME /home/volume
      
      # 이미지를 빌드한 뒤 컨테이너를 생성하고 볼륨의 목록을 확인해보면 볼륨이 생성된 것을 확인
      $ docker build -f Dockerfile_volume -t myvolume:0.0 ./
      $ docker run -it -d --name volume_test myvolume:0.0
      $ docker volume ls
      ```

      ![image-20220530131643319](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220530131643319.png)

    - ARG : build 명령어를 실행할 떄 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정. 다음 Dockerfile은 build 명령어에서 my_arg와 my_arg_2라는 이름의 변수를 추가로 입력받을 것이라고 ARG를 통해 명시한다.

      ```dockerfile
      # ARG를 사용하는 Dockerfile
      # vi Dockerfile_arg
      FROM ubuntu:14.04
      ARG my_arg
      ARG my_arg_2=value2
      RUN touch ${my_arg}/mytouch
      
      # build 명령어를 실행할 때 --build-arg 옵션을 사용해 Dockerfile의 ARG에 값을 입력할 수 있다. 입력하는 방식은 '<키>=<값>'과 같이 쌍을 이루어야 함
      $ docker build -f Dockerfile_arg --build-arg my_arg=/home -t myarg:0.0 ./
      $ docker run -it --name arg_test myarg:0.0
      ```

      ![image-20220530134142311](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220530134142311.png)

    - USER : USER로 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다. 일반적으로 RUN으로 사용자의 그룹과 계정을 생성한 뒤 사용한다. 루트 권한이 필요하지 않다면 USER를 사용하는 것을 권장

      ```dockerfile
      RUN groupadd -r author && useradd -r -g author ihp001
      USER ihp001
      ...
      ...
      # 기본적으로 컨테이너 내부에서는 root 사용자를 사용하도록 설정
      # 컨테이너가 호스트의 root 권한을 가질 수 있다는 것을 의미하기때문에 보안 측면에서도 바람직하지 않음
      # 보안 측면을 강화하기 위해선 컨테이너 애플리케이션을 최종적으로 배포할 때는 컨테이너 내부에서 새로운 사용자를 새롭게 생성해 사용하는 것을 권장
      ```

  - 2.4.4.2 Onbuild. Stopsignal, Healthcheck, shell

    - ONBUILD : 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가한다.

      ```dockerfile
      # ONBUILD가 사용된 Dockerfile
      # vi Dockerfile_onbuild
      # ONBUILD에 RUN echo "onbuild"~을 입력해서 Dockerfile을 빌드할 때 실행되지 않으며, 별도의 정보로 이미지에 저장될 뿐 입니다.
      FROM ubuntu:14.04
      RUN echo "this is onbuild test!"
      ONBUILD RUN echo "onbuild!" >> /onbuild_file
      
      # 하단에 캡처한 내용으론 onbuild_file이 안 보인다.
      ```

      ![image-20220530144249593](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220530144249593.png)

      ```dockerfile
      # 이번에는 위에서 생성한 이미지를 기반으로 하는 새로운 이미지를 Dockerfile로 생성
      # ONBUILD가 적용된 이미지를 기반으로 하는 Dockerfile
      # vi Dockerfile_onbuild_use
      FROM onbuild_test:0.0
      RUN echo "this is child image!"
      
      # 아래에 쓰인 build 명령어에서는 Dockerfile_onbuild_use를 지정하기 위해 -f 옵션을 사용
      
      $ docker build -f ./Dockerfile_onbuild_use ./ -t onbuild_test:0.1
      $ docker run -it --rm onbuild_test:0.1 ls /
      
      # 하단에 캡처한 내용으론 onbuild_file이 있다.
      # 이처럼 ONBUILD는 ONBUILD, FROM, MAINTAINER를 제외한 RUN, ADD 등, 이미지가 빌드될 때 수행돼야 하는 각종 Dockerfile의 명령어를 나중에 빌드될 이미지를 위해 미리 저장해 놓을 수 있다.
      # 단, 이미지의 속성을 설정하는 다른 Dockerfile 명령어와는 달리 ONBUILD는 부모 이미지의 자식 이미지에만 적용
      ```

      ![image-20220530154410803](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220530154410803.png)

    - STOPSIGNAL : 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정. 아무것도 설정하지 않으면 기본적으로 SIGTERM으로 설정됨.

      ```dockerfile
      # STOPSIGNAL를 사용
      FROM ubuntu:14.04
      STOPSIGNAL SIGKILL	
      ```

    - HEALTHCHECK : 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정한다. 컨테이너 내부에서 동작 중인 애플리케이션의 프로세스가 종료되지는 않았으나 애플리케이션이 동작하고 있지 않은 상태를 방지하기 위해 사용될 수 있다.

      ```dockerfile
      # 밑의 Dockerfile은 1분마다 curl -f ~를 실행해 nginx 애플리케이션의 상태를 체크하며, 3초 이상이 소요되면 이를 한 번의 실패로 간주
      # HEALTHCHECK --interval은 컨테이너의 상태를 체크하는 주기
      # 마지막에 지정한 CMD curl ...부분이 상태를 체크하는 명령어
      # --interval에 지정된 주기마다 이를 실행
      # vi Dockerfile_healthcheck
      FROM nginx
      RUN apt-get update -y && apt-get install curl -y
      HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
      
      # 이미지를 빌드한 후 해당 이미지로 컨테이너를 생성하면 docker ps의 출력 중 해당 컨테이너의 STATUS에 정보가 추가된 것을 확인
      $ docker build ./ -t nginx:healthcheck
      $ docker run -dP nginx:healthcheck
      ```

      ![image-20220530163025518](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220530163025518.png)

    - SHELL : Dockerfile에서 기본적으로 사용하는 셸은 리눅스에서 "/bin/sh -c"이다. 이 셸을 따로 지정할 수 있다. 윈도우에서 "cmd /S /C"입니다.

      ```
      FROM node
      RUN echo hello, node!
      SHELL ["/usr/local/bin/node"]
      RUN -V
      ```

  - 2.4.4.3 ADD, COPY

    - ADD, COPY 차이

      COPY는 로컬의 파일만 이미지에 추가할 수 있지만 ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다는 점에서 다르다.

      ADD에서 tar파일은 그대로 추가되지 않고 자동 해제해서 추가한다.

      ```dockerfile
      # COPY는 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할
      COPY test.html /home/
      COPY ["test.html", "/home/"]
      
      # ADD에 추가할 파일을 깃과 같은 외부 URL로 지정
      ADD https://~~
      
      # tar 파일도 추가가능
      ADD test.tar /home
      
      # ADD는 URL이나 tar 파일을 추가할 경우 이미지에 정확히 어떤 파일이 추가될지 알 수 없기 때문에 권장하진 않는다.
      ```

  - 2.4.4.4 ENTRYPOINT, CMD

    - ENTRYPOINT, CMD 차이

      CMD는 컨테이너가 시작될 때 실행할 명령어를 설정한다. 이는 docker run 명령어에서 맨 뒤에 입력했던 커맨드와 같은 역할을 한다.

      ENTRYPOINT도 CMD와 동일하게 컨테이너가 시작될 때 수행할 명령을 지정하는데 커맨드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다.

      ```dockerfile
      # 먼저 ubuntu:14.04 이미지로 컨테이너를 생성하고, 컨테이너가 시작될 때 실행할 명령어를 /bin/bash로 설정
      # 지금은 entrypoint를 입력하지 않아서 설정되지 않았으며, 컨테이너의 커맨드는 /bin/bash
      # # entrypoint: 없음, cmd: /bin/bash
      $ docker run -it --name no_entrypoint ubuntu:14.04 /bin/bash
      
      # # entrypoint: echo, cmd: /bin/bash
      $ docker run -it --entrypoint="echo" --name yes_entrypoint ubuntu:14.04 /bin/bash
      # /bin/bash라는 내용이 터미널에 출력됩니다.
      # 즉, 컨테이너에 entrypoint가 설정되면 run 명령어의 맨 마지막에 입력된 cmd를 인자로 삼아 명령어를 추력합니다.
      ```

      ```dockerfile
      # 일반적으론 스크립트 파일을 entrypoint의 인자로 사용해 컨테이너가 시작될 때마다 해당 스크리브 파일을 실행하도록 설정한다.
      # 단 실행할 스크립트 파일은 컨테이너 내부에 존재해야한다.
      # 이런식으로 Dockerfile에서 COPY, ADD를 쓰면된다.
      $ docker run -it --name entrypoint_sh --entrypoint="/test.sh" ubuntu:14.04 /bin/bash
      -- 해당 파트는 알아본 바에 따르면 그래픽 카드 상에 오류로 인해 해결하려고했으나 권한문제가 있어서 해결되진 못 했다. --
      
      ------------- 오류내용 ---------------
      # docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "/test.sh": stat /test.sh: no such file or directory: unknown.
      # ERRO[0000] error waiting for container: context canceled
      ```
  
      - 이미지를 빌드할 때 이미지가 작동하기 위해서는 다음과 같은 단계를 거칩니다.
        1. 어떤 설정 및 실행이 필요한지에 대해 스크립트로 정리
        2. ADD 또는 COPY로 스크립트를 이미지로 복사
        3. ENTRYPOINT를 이 스크립트로 설정
        4. 이미지를 빌드해 사용
        5. 스크립트에서 필요한 인자는 docker run 명령어에서 cmd로 entrypoint의 스크립트에 전달
  
      ```dockerfile
      # vi Dockerfile_entrypoint
      FROM ubuntu:14.04
      RUN apt-get update
      RUN apt-get install apache2 -y
      ADD entrypoint.sh /entrypoint.sh
      RUN chmod +x /entrypoint.sh
      ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
      $ docker build -f Dockerfile_entrypoint -t entrypoint_image:0.0 ./
      $ docker run -d --name entrypoint_apache_server entrypoint_image:0.0 first second
      $ docker logs entrypoint_apache_server
      first second
      ...
      ...
      ```
  
    - JSON 배열 형태와 일반 형식의 차이점
  
      CMD 또는 ENTRYPOINT에 설정하려는 명령어를 /bin/sh로 사용할 수 없다면 JSON 배열의 형태로 명령어를 설정해야한다.
  
      JSON 배열 형태가 아닌 CMD와 ENTRYPOINT를 사용하면 실제로 이미지를 생성할 때 cmd와 entrypoint에 /bin/sh -c 가 앞에 추가되기 때문이다.
  
      ```dockerfile
      # echo test가 아닌 /bin/sh -c echo test로 설정
      # /entrypoint.sh가 아닌 /bin/sh -c /entrypoint.sh로 설정
      # JSON 배열 형태로 입력하지 않으면 CMD와 ENTRYPOINT에 명시된 명령어의 앞에 /bin/sh -c가 추가된다
      CMD echo test
      $ /bin/sh -c echo test
      
      CMD ["echo", "test"]
      $ echo test
      
      ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
      $ /bin/bash /entrypoint.sh
      ```
  
  - 2.4.5 Dockerfile로 빌드할 때 주의할 점
  
    - Dockerfile을 사용하는 데도 좋은 습관(practice)이라는 것
  
      -  하나의 명령어를 \ (역슬래시)로 나눠서 가독성을 높일 수 있도록 작성
  
      - .dockerignore 파일을 작성해 불필요한 파일을 빌드 컨텍스트에 포함하지 않는 것
  
      - 빌드 캐시를 이용해 기존에 사용했던 이미지 레이어를 재사용하는 방법
  
        ```dockerfile
        # Dockerfile을 사용할 때 백슬래시로 apt-get을 사용하는 좋은 습관
        # vi Dockerfile
        ........
        RUN apt-get install package-1 \
        package-2 \
        package-3
        
        # 여기서 설명하는 부분은 도커의 이미지 구조와 Dockerfile의 관계이다.
        ```
  
  
    - Dockerfile을 사용할 때 이미지를 비효율적으로 빌드하는 예
  
      - fallocate라는 명령어는 100MB 크기의 파일을 가상으로 만들어 컨테이너에 할당
  
      - 이를 이미지 레이어로 빌드 및 파일을 rm 명령어로 삭제
  
      - 빌드가 완료되어 최종 생성된 이미지에는 100MB 크기의 파일인 /dummy가 존재하지 않음
  
        ```dockerfile
        # 잘못된 Dockerfile 사용
        # vi Dockerfile_fallocate_false
        FROM ubuntu:14.04
        RUN mkdir /test
        RUN fallocate -l 100m /test/dummy
        # fallocate는 100MB 크기의 파일을 가상으로 만듬
        RUN rm /test/dummy
        ```
  
    - 실제로 컨테이너에서 사용하지 못하는 파일이 이미지 레이어로 존재하기 때문에 저장 공간은 차지하지만 실제로는 의미가 없는 저장 공간일 수도 있는 것이다
  
    - 이를 방지하기위해 Dockerfile을 작성할 때 &&로 각 RUN 명령을 하나로 묶는 것이다
  
    - 즉, 하나의 RUN으로 여러 개의 명령어를 실행하도록 작성
  
      ```dockerfile
      # Dockerfile 명령어의 바람직한 사용법
      # 여러 개의 RUN 명령어가 하나로 묶일 수 있다면 이미지 레이어의 개수 또한 하나로 줄어들기 때문이다.
      # vi Dockerfile_fallocate_true
      FROM ubuntu:14.04
      RUN mkdir /test && \
      fallocate -l 100m /test/dummy && \
      rm /test/dummy
      ```
  
    - 도커의 레이어 이미지구조는 위의 경웨 단점으로 작용할 수도 있지만 잘만 활용하면 장점으로 활용
    - 예를 들어,  A, B 애플리케이션이 500MB 크기의 같은 라이브러리를 사용해야 한다면 각 애플리케이션의 이미지에 라이브러리를 각기 설치하기보다는 500MB 크기의 라이브러리 이미지를 미리 만들어 놓은 다음 이 이미지를 이용해 A, B 애플리케이션의 이미지를 생성함으로써 저장 공간을 절약
    - 즉, A와 B이 애플리케이션의 이미지는 이 라이브러리로 인해 각각 500MB를 차지해 1GB의 저장 공간을 사용하는 것이 아닌 500MB의 이미지 레이어를 공유
  
    ![image-20220603170457502](C:\Users\mink\AppData\Roaming\Typora\typora-user-images\image-20220603170457502.png)
  
  



7page 씩

