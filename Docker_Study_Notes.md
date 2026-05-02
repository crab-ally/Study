# Docker

## 1. Docker 개념

### 1.1 Docker란?

실행 환경 자체를 통째로 포장하여 누구나 동일한 상태로 실행할 수 있게 만드는 컨테이너 플랫폼

### 1.2 핵심 구성 요소

| 구성 요소 | 설명 |
|---|---|
| **이미지(Image)** | 컨테이너를 만들기 위한 설계도 |
| **컨테이너(Container)** | 이미지를 실행한 결과물 |
| **볼륨(Volume)** | 컨테이너 외부에 데이터를 저장하는 공간 |
| **네트워크(Network)** | 컨테이너 간 통신 통로 |

### 1.3 Docker vs Docker Compose

| | Docker | Docker Compose |
|---|---|---|
| **역할** | 단일 컨테이너 실행 | 여러 컨테이너를 한 번에 정의하고 실행 |
| **권장 단위** | 1 컨테이너 = 1 프로세스 | 다중 서비스 환경 |

---

## 2. 단일 컨테이너 핵심 명령 실습

### 기본 명령어

| 명령어 | 설명 |
|---|---|
| `docker --version` | Docker 버전 확인 |
| `docker pull <이미지명>` | 이미지 다운로드 |
| `docker images` | 로컬 이미지 목록 확인 |
| `docker ps` | 실행 중인 컨테이너 목록 |
| `docker ps -a` | 전체 컨테이너 목록 (종료된 컨테이너 포함) |

### 컨테이너 생성 및 실행

| 명령어 | 설명 |
|---|---|
| `docker run [옵션] <이미지명>` | 컨테이너 생성 및 실행 |
| `docker run --name <이름> [옵션] <이미지명>` | 이름을 지정하여 실행 |
| `docker run -it --name <이름> <이미지명> bash` | 실행 후 즉시 접속 |
| `docker start <컨테이너명>` | 중지된 컨테이너 시작 |
| `docker exec -it <컨테이너명> bash` | 실행 중인 컨테이너에 접속 |

> **`docker run` 주요 옵션**
> - `-i` : 표준 입력(stdin) 유지
> - `-t` : 터미널(tty) 형태로 연결
> - `-it` : 두 옵션을 함께 사용 (대화형 셸 접속 시 필수)

### 컨테이너 종료 및 삭제

| 명령어 | 설명 |
|---|---|
| `docker stop <컨테이너명>` | 컨테이너 중지 |
| `docker rm <컨테이너명>` | 중지된 컨테이너 삭제 |
| `docker rm -f <컨테이너명>` | 컨테이너 강제 삭제 — 실행 여부 상관 없음|

> **컨테이너 내부에서 종료하려면** `exit` 입력  
> **컨테이너명은 중복 불가** — 동일한 이름으로 새 컨테이너를 생성하려면 기존 컨테이너를 먼저 삭제해야 함

---

## 3. 실무 운영 핵심 (로그/백그라운드/포트/볼륨/inspect)

### 3.1 백그라운드 실행 (`-d`)

```bash
docker run -d --name web nginx
```

| 옵션 | 설명 |
|---|---|
| `-d` | 백그라운드(detached) 모드로 실행 — 터미널을 점유하지 않음 |

### 3.2 포트 포워딩 (`-p`)

```bash
docker run -d -p 8080:80 --name web nginx
```

| 옵션 | 형식 | 설명 |
|---|---|---|
| `-p` | `호스트포트:컨테이너포트` | 호스트의 포트를 컨테이너 포트에 연결하여 외부 접속 허용 |

> 위 예시 기준으로 브라우저에서 `http://localhost:8080` 으로 접속하면 컨테이너 내부의 80번 포트(nginx)에 연결됨

### 3.3 로그 확인 (`docker logs`)

| 명령어 | 설명 |
|---|---|
| `docker logs <컨테이너명>` | 누적 로그 전체 출력 |
| `docker logs -f <컨테이너명>` | 실시간 로그 스트리밍 (종료 Ctrl + C) |
| `docker logs --tail 100 <컨테이너명>` | 최근 100줄만 출력 |
| `docker logs --since 10m <컨테이너명>` | 최근 10분 내 로그만 출력 |

### 3.4 컨테이너 상세 정보 (`docker inspect`)

```bash
docker inspect <컨테이너명> # JSON 형태
```

- 전체 JSON 출력에서 핵심 항목만 필터링:

| 명령어 | 확인 내용 |
|---|---|
| `docker inspect web \| grep IPAddress` | 컨테이너 IP 주소 |
| `docker inspect web \| grep Mounts` | 볼륨 마운트 경로 |
| `docker inspect web \| grep -i status` | 컨테이너 상태 |

### 3.5 볼륨 마운트 (`-v`)

컨테이너와 호스트 간에 디렉토리를 공유하여 컨테이너가 삭제되어도 데이터를 유지하는 기능

```bash
docker run -v <호스트경로>:<컨테이너경로> <이미지명>
```

**예시**

```bash
# nginx의 HTML 파일을 호스트에서 직접 수정
docker run -d -p 8080:80 -v ~/mysite:/usr/share/nginx/html --name web nginx

# MySQL 데이터 영속화
docker run -d -v ~/mysql-data:/var/lib/mysql --name db mysql
```

---

## 4. Docker Compose (멀티 컨테이너 운영)

여러 컨테이너를 하나의 `docker-compose.yml` 파일로 정의하고 함께 실행·관리하는 도구

### 4.1 `docker-compose.yml` 기본 구조

```yaml
services:
  ros2:
    image: osrf/ros:humble-desktop
    container_name: ros2_container
    stdin_open: true
    tty: true

    volumes:
      - ./workspace:/workspace

    working_dir: /workspace

    command: bash

  web:
    image: nginx
    ports:
      - "8080:80"
```

| 키 | 의미 |
| ---| --- |
| `services` | 실행할 컨테이너(서비스) 목록 |
| `ros2/web` | 서비스 이름 (임의 지정) |
| `image` | 사용할 Docker 이미지 (`osrf/ros:humble-desktop`) |
| `container_name` | 컨테이너 이름을 직접 지정 |
| `stdin_open` | 표준 입력(STDIN)을 열어둠 (`-i` 옵션과 동일) |
| `tty` | 터미널 환경 활성화 (`-t` 옵션과 동일) |
| `volumes` | 호스트와 컨테이너 디렉토리 연결 (`./workspace → /workspace`) |
| `working_dir` | 컨테이너 시작 시 기본 작업 디렉토리 |
| `command` | 컨테이너 실행 시 수행할 명령어 (`bash`) |
| `ports` | 포트 연결 (호스트:컨테이너) |

> **`.yml` vs `.yaml`** — 완전히 동일한 형식. 초기에는 일부 구형 OS가 확장자를 3글자로 제한하여 `.yml`이 생겨났으며, 현재는 둘 다 혼용.

### 4.2 핵심 명령어

**서비스 실행**

| 명령어 | 설명 |
|---|---|
| `docker compose up` | 모든 서비스 시작 |
| `docker compose up --build` | 이미지를 새로 빌드 후 시작 |
| `docker compose build` | 실행 없이 이미지만 빌드 |

**상태 확인**

| 명령어 | 설명 |
|---|---|
| `docker compose ps` | 서비스(컨테이너) 목록 및 상태 확인 |
| `docker compose logs` | 전체 서비스 로그 출력 |
| `docker compose logs -f` | 실시간 로그 스트리밍 (종료 Ctrl + C) |
| `docker compose logs -f <서비스명>` | 특정 서비스 로그만 실시간 확인 |

**컨테이너 접속**

| 명령어 | 설명 |
|---|---|
| `docker compose exec <서비스명> bash` | 실행 중인 컨테이너에 접속 |

> `docker compose exec`는 보통 이미 인터랙티브라서 `-it` 생략 가능

**서비스 종료**

| 명령어 | 설명 |
|---|---|
| `docker compose stop` | 컨테이너 중지 |
| `docker compose down` | 컨테이너 중지 및 삭제 |
| `docker compose down -v` | 컨테이너 + 연결된 볼륨까지 삭제 |
| `docker compose down --rmi all` | 컨테이너 + 이미지까지 삭제 |

> `down`은 컨테이너와 네트워크를 삭제하지만 볼륨과 이미지는 기본적으로 유지됨.

---

## 5. ROS2 실습을 위한 Docker 실무 미니 프로젝트 (완성 단계)