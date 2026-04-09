---

# Linux & MariaDB Security / System Administration Notes

## 1. Linux Kernel Parameter 확인

### Swappiness 확인 

```bash
sysctl -a | grep swappiness
```

* Linux의 **메모리 관리 정책** 중 하나인 `swappiness` 값을 확인하는 명령어
* swap 영역 사용 비율을 조절하는 커널 파라미터

### 커널 매개변수 설정 파일

```
/etc/sysctl.conf
```

* 리눅스 **커널 매개변수(Kernel Parameters)** 를 **영구적으로 설정하는 핵심 파일**
* 시스템 재부팅 후에도 설정 유지

예시

```
vm.swappiness=10
```

적용

```bash
sysctl -p
```

---

# 2. 시스템 리소스 확인

### 메모리 사용량 확인

```bash
free
```

* 시스템 **메모리 / swap 사용량 확인**

### 상세 메모리 정보 확인

```bash
cat /proc/meminfo
```

* 리눅스 메모리 상세 정보 확인
* 총 메모리, 사용 가능 메모리, 캐시 등 확인 가능

### CPU 정보 확인

```bash
cat /proc/cpuinfo
```

* CPU 모델
* 코어 개수
* CPU 클럭
* 캐시 정보 확인 가능

---

# 3. MariaDB 사용자 및 권한 관리

## 사용자 생성

```sql
create user 'estuser'@'localhost' identified by '123456';
create user 'winuser'@'172.16.11.219' identified by '123456';
create user 'hoon'@'%' identified by '123456';
```

설명

* `localhost` → 로컬 접속만 허용
* 특정 IP → 해당 IP에서만 접속 가능
* `%` → 모든 IP 접속 허용

---

## 데이터베이스 생성

```sql
create database est default character set utf8;
```

---

## 권한 부여

```sql
grant all privileges on est_db.* to 'estuser'@'localhost';
```

설명

* `est_db.*`
* 해당 DB의 모든 테이블 권한 부여

특정 권한만 부여

```sql
GRANT CREATE, DROP ON est_db.* TO 'winuser'@'172.16.11.219';
```

---

## 권한 확인

```sql
SHOW GRANTS FOR 'estuser'@'localhost';
```

---

## 권한 제거

```sql
revoke all privileges on *.* from 'edushare'@'localhost';
```

---

## 사용자 삭제

```sql
drop user 'winuser'@'172.16.11.219';
```

---

# 4. MariaDB Replication (Master / Slave)

MariaDB Replication은 **Master 서버의 데이터를 Slave 서버로 복제**하는 구조이다.

목적

* 데이터 백업
* 부하 분산
* 고가용성

---

# 4.1 Master 서버 설정

## MariaDB 설치

```bash
dnf update -y
dnf install mariadb-server
```

## 방화벽 설정

```bash
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```

---

## 설정 파일 수정

```bash
vi /etc/my.cnf.d/mariadb-server.cnf
```

추가

```
[mysqld]

server-id = 1
log_bin = /var/log/mariadb/mysql-bin.log
expire_logs_days = 10
max_binlog_size = 100M
```

설명

* `server-id`
  서버의 고유 식별자

* `log_bin`
  Binary Log 활성화 (데이터 변경 기록)

* `expire_logs_days`
  Binary Log 보관 기간

* `max_binlog_size`
  Binary Log 최대 크기

---

## Slave 계정 생성 (Master에서 실행)

```sql
mariadb -u root -p
```

```sql
grant replication slave on *.* to 'slave_db'@'%' identified by 'slave_password';
```

---

## MariaDB 재시작

```bash
systemctl restart mariadb
```

---

## Master 상태 확인

```sql
show master status;
```

Binary Log 상태 확인

```sql
SHOW VARIABLES LIKE '%log_bin%';
```

---

## 데이터 변경 방지 잠금

Replication 설정 전 데이터 변경을 막기 위해 사용

```sql
FLUSH TABLES WITH READ LOCK;
```

이유

* Binary Log의 **좌표(Position)** 값이 변하는 것을 방지

---

# 4.2 Slave 서버 설정

## 설정 파일 수정

```bash
vi /etc/my.cnf.d/mariadb-server.cnf
```

```
server-id = 2
```

---

## server-id 확인

```sql
show variables like 'server_id';
```

---

## MariaDB 재시작

```bash
systemctl restart mariadb
```

---

## Master 서버 연결

DB 접속

```bash
mysql -u root -p
```

Replication 설정

```sql
CHANGE MASTER TO
MASTER_HOST="192.168.100.101",
MASTER_USER="slave_db",
MASTER_PASSWORD="slave_password",
MASTER_PORT=3306,
MASTER_LOG_FILE="mysql-bin.000002",
MASTER_LOG_POS=342;
```

---

## Slave 시작

```sql
start slave;
```

정지

```sql
stop slave;
```

초기화

```sql
reset slave;
```

---

## Replication 상태 확인

```sql
show slave status;
```

자세한 출력

```sql
show slave status\G
```

---

# 5. Replication 테스트

Master에서 데이터 생성

```sql
CREATE DATABASE retest;

USE retest;

CREATE TABLE users (
name VARCHAR(255)
);

INSERT INTO users (name) VALUES ('홍길동');
INSERT INTO users (name) VALUES ('홍길순');
```

확인

* Slave 서버에서 동일한 데이터베이스와 테이블이 생성되면 **Replication 정상 동작**

---

원하면 내가 이거 **GitHub용으로 더 깔끔한 README (목차, 아키텍처 그림, 보안 설명 포함)**으로도 다시 만들어 줄게.
보안 수업 정리라면 **교수님들이 좋아하는 수준으로 업그레이드도 가능하다.**
