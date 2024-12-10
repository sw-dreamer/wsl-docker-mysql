# MySQL 설치 방법 (WLS + Docker)

이 가이드는 ***Windows Subsystem for Linux (WLS)와 Docker**가 *이미 설치된 상태*에서 MySQL 8.0.17을 Docker 컨테이너에 설치하는 방법을 설명합니다.

이 가이드를 따르기 전에 WLS와 Docker가 시스템에 설치되어 있는지 확인해주세요.

  > 만약에 설치가 안되어 있으면 [WLS 설치 가이드](https://github.com/sw-dreamer/wsl-install.git)와 [Docker 설치 가이드](https://github.com/sw-dreamer/wsl-docker.git)를 참조하여 설치를 진행해 주시기 바랍니다.

## 1. Docker에서 MySQL 이미지 확인
  먼저, Docker에서 사용할 MySQL 이미지를 검색합니다.
  ```bash
  docker search mysql
  ```
  이 명령어는 Docker Hub에서 MySQL 관련 이미지를 검색하고 목록을 출력합니다.

## 2. MySQL 8.0.17 이미지 다운로드
  MySQL 8.0.17 버전을 Docker에 다운로드하려면 아래 명령어를 실행하세요.
  ```bash
  docker pull mysql:8.0.17
  ```
  > **주의**
  > DBeaver 연결 시 "Public Key Retrieval is not allowed"라는 에러 메세지가 나오는데 mysql_native_password로 해결하기 위해서 8.0.17 버전을 사용
  > mysql_native_password는 MySQL 8.0 에서 사용되는 인증 플러그인으로, MySQL 9.x에서는 지원되지 않거나 다르게 설정해야 할 수 있습니다.
  > MySQL 9.x에서 인증 플러그인 설정이나 기본 설정이 변경된 경우가 많습니다.
  > 오류 발생 시 MySQL 문서를 참조 하시기 바랍니다.

## 3. 다운로드된 Docker 이미지 확인
  이미지가 제대로 다운로드 되었는지 확인하려면 아래 명령어로 확인할 수 있습니다.
  ```bash
  docker images
  ```

## 4. Docker 컨테이너 생성 및 실행
  MySQL 컨테이너를 생성하고 실행하려면 아래 명령어를 입력하세요.
  
  이 명령어는 MySQL을 컨테이너로 실행하면서 몇 가지 필수 환경설정도 함께 지정합니다.
  
  --name 뒤에는 컨테이너이름, MYSQL_ROOT_PASSWORD 뒤에는 비밀번호를 설정

  ```bash
  docker run --name mysql-container -p 3306:3306 -e MYSQL_ROOT_PASSWORD=rootpassword -e TZ=Asia/Seoul --restart=always -d mysql:8.0.17
  ```
  ### 설명
  - --name: 컨테이너에 이름을 지정합니다 (여기서는 mysql-container).
  - -p: 호스트와 컨테이너의 포트를 연결합니다. 여기서는 3306 포트를 연결합니다.
  - -e: 환경변수를 설정합니다. 예를 들어, MYSQL_ROOT_PASSWORD는 루트 비밀번호를 설정하고, TZ는 시간대를 Asia/Seoul로 설정합니다.
  - --restart=always: Docker가 시스템 부팅 시 자동으로 컨테이너를 시작하도록 설정합니다.
  - -d: 백그라운드에서 컨테이너를 실행합니다.

## 5. 실행 중인 Docker 컨테이너 확인
  현재 실행 중인 컨테이너를 확인하려면 아래 명령어를 입력하세요.
  
  ```bash
  docker ps
  ```

## 6. 모든 컨테이너 확인 (실행 여부 상관없음)
  모든 컨테이너를 확인하려면 아래 명령어를 사용하세요.
  
  ```bash
  docker ps -a
  ```

## 7. Docker 컨테이너의 쉘에 접속

  실행 중인 Docker 컨테이너에 쉘로 접속하려면 아래 명령어를 사용하세요.
  
  ```bash
  docker exec -i -t mysql-container /bin/bash
  ```

  여기서 mysql-container는 실행 중인 컨테이너의 이름입니다.
  
  -i와 -t 옵션은 인터랙티브 모드로 쉘을 실행하도록 설정합니다.

## 8. MySQL 접속
  접속 후 MySQL에 로그인하려면 아래 명령어를 실행하세요.
  
  ```
  mysql -u root -p
  ```

  그리고 설정한 비밀번호(rootpassword)를 입력하세요.

## 9. MySQL 설치 확인
  MySQL이 정상적으로 설치되었는지 확인하려면 MySQL에 접속 후 쿼리를 실행해 보세요.
  
  ```sql
  show databases;
  ```

## 10. 포트 개방

  외부에서 MySQL에 접속할 수 있도록 포트를 개방해야 합니다.
  
  아래 명령어로 포트를 개방할 수 있습니다.
  ```bash
  sudo ufw allow 3306
  ```

## 11. DBeaver 접속 오류 해결
  DBeaver를 사용해 MySQL에 접속하려고 할 때 "Public Key Retrieval is not allowed"라는 오류가 발생할 수 있습니다.
  
  이 오류는 MySQL 8.0에서 기본 인증 방식인 caching_sha2_password와 구버전 클라이언트 간의 호환성 문제로 발생합니다.

  이 문제를 해결하려면, MySQL의 인증 방식을 mysql_native_password로 변경해야 합니다.

  ### 11.1 사용자 인증 방식 변경
  MySQL에 접속 후, 아래 명령어를 실행하여 인증 방식을 변경합니다.
  
  ```sql
  ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'rootpassword';
  ```
  
  설명:
  - 'root'는 MySQL 사용자 이름입니다.
  - '%'는 모든 호스트에서 접속을 허용하는 설정입니다. 특정 IP로 제한하려면 IP를 지정할 수 있습니다.
  - 'rootpassword'는 사용자 비밀번호입니다.

## 12. 변경 사항 적용
  ```sql
  FLUSH PRIVILEGES;
  ```

## 13. DBeaver에서 MySQL 접속 확인
  이제 DBeaver를 통해 MySQL에 정상적으로 접속할 수 있습니다.

## 주의사항
  - MySQL 8.0 이상에서는 기본 인증 방식이 caching_sha2_password로 설정되어 있습니다.
  - 구버전 클라이언트와 호환되도록 mysql_native_password로 변경할 수 있습니다.
  - mysql_native_password는 MySQL 8.0에서 사용되며,
    MySQL 9.x에서는 인증 플러그인 설정이 달라질 수 있습니다. MySQL 9.x에서 인증 방식 관련 오류가 발생할 수 있습니다.
