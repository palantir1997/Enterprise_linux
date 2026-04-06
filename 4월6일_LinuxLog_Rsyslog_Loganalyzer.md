# Linux 로그 관리 및 중앙 수집 시스템 구축

> 교육 수업 정리 | 2026-04-06

---

## 1. 주요 로그 파일 및 경로

리눅스의 시스템 로그는 대부분 `/var/log` 디렉토리에 저장됩니다.

```bash
cd /var/log
```

| 로그 파일 / 명령어 | 설명 |
|---|---|
| `xferlog` | FTP 서버를 통한 파일 업로드/다운로드 기록 |
| `secure` | 로그인 성공/실패, sudo 사용 등 인증·보안 기록 (해킹 흔적 확인 1순위) |
| `messages` | 시스템 전반 메시지 로그 (오류 발생 시 우선 확인) |
| `.bash_history` | 사용자 홈 디렉토리에 위치, 터미널 명령어 입력 기록 |

---

## 2. 주요 로그 확인 명령어

```bash
# 최근 접속 사용자 목록, 시간, 재부팅 기록
last

# 계정별 마지막 로그인 시간
lastlog

# 현재 로그인된 사용자 및 시스템 상태 (who와 동일)
w

# 특정 문자열이 포함된 줄만 필터링
grep "검색어" /var/log/messages

# 실시간 시스템 로그 확인
journalctl -xe
```

---

## 3. Logrotate (로그 순환 관리)

오래된 로그를 자동으로 압축·삭제하여 디스크 공간을 관리합니다.

```bash
# logrotate 서비스 상태 확인
systemctl status logrotate
systemctl status logrotate.timer

# logrotate 타이머 설정 내용 확인
systemctl cat logrotate.timer

# 애플리케이션별 logrotate 설정 목록 확인
ls /etc/logrotate.d

# Apache(httpd) logrotate 설정 확인
cat /etc/logrotate.d/httpd
```

---

## 4. 호스트명 변경

```bash
# 호스트명을 'client'로 변경
hostnamectl set-hostname client

# 변경 사항 즉시 적용
exec bash
```

---

## 5. Rsyslog 중앙 로그 서버 구축

### 5-1. 설치

```bash
dnf install -y rsyslog rsyslog-doc
```

### 5-2. UDP 수신 설정

`/etc/rsyslog.conf` 파일을 열어 아래 두 줄의 주석을 해제합니다.

```bash
vi /etc/rsyslog.conf
```

```
# UDP 수신 모듈 활성화
$ModLoad imudp          # imudp = Input Module for UDP

# 514번 포트로 수신 대기 (Syslog 표준 포트)
$InputUDPServerRun 514
```

### 5-3. 방화벽 설정 및 서비스 시작

```bash
# 514/tcp 포트 영구 허용
firewall-cmd --permanent --add-port=514/tcp
firewall-cmd --reload

# rsyslog 서비스 시작
systemctl start rsyslog
```

### 5-4. 포트 수신 확인

```bash
# 추가 도구 설치
dnf install -y lsof net-tools

# 514 포트를 사용 중인 프로세스 확인
lsof -i tcp:514

# 네트워크 소켓 상태 확인
netstat -natpl | grep 514
```

---

## 6. Rsyslog + MariaDB 연동 (DB 로그 저장)

```bash
# DB 스키마 확인
cat /usr/share/doc/rsyslog/mysql-createDB.sql

# DB에 Syslog 스키마 적용
mysql -u root -p < /usr/share/doc/rsyslog/mysql-createDB.sql
```

```sql
-- rsyslog 전용 DB 계정 생성 및 권한 부여
GRANT ALL ON Syslog.* TO 'rsyslog'@'localhost' IDENTIFIED BY '패스워드';
```

> **TLS (Transport Layer Security)**: 인터넷 통신을 암호화하여 안전하게 만드는 프로토콜

---

## 7. LogAnalyzer 웹 대시보드 구축

DB에 쌓인 로그를 웹 화면으로 시각화하는 도구입니다.

```bash
# LogAnalyzer 압축 해제
tar -xzvf /tmp/loganalyzer-4.1.13.tar.gz -C /tmp

# 웹 루트에 파일 복사
cp -r /tmp/loganalyzer-4.1.13/src/* /var/www/html/loganalyzer/

# 설정 스크립트 복사 및 실행
cp /tmp/loganalyzer-4.1.13/contrib/configure.sh /var/www/html/loganalyzer/
cat configure.sh
bash configure.sh

# SELinux 임시 비활성화 (테스트 환경)
setenforce 0
```

설치 완료 후 브라우저에서 설치 마법사 접속:
```
http://[웹서버IP]/loganalyzer/install.php?step=3
```

---

## 8. 실습 과제: 웹-DB 분리 구성 및 로그 수집

### 아키텍처

```
클라이언트
    │
    │ HTTP 요청
    ↓
웹 서버 (Web Server)
    │
    │ 3306 포트 (MySQL/MariaDB)
    ↓
DB 서버 (Database Server)
```

- 웹 서버와 DB 서버를 **별도 머신**으로 분리하여 연동 구성
- 클라이언트에서 에러를 발생시켜 로그 수집 확인

### 부하 테스트 명령어 (클라이언트 실행)

```bash
# 웹 서버에 100번 요청을 보내 에러 로그 발생
for i in {1..100}; do
    curl -I -s http://[웹서버IP]/attack-test-$i | grep "HTTP/"
    sleep 0.1
done
```

---

## 📌 핵심 요약

| 항목 | 내용 |
|---|---|
| 로그 저장 위치 | `/var/log/` |
| 보안 로그 | `/var/log/secure` |
| 중앙 로그 수집 | rsyslog + UDP 514 포트 |
| 로그 시각화 | LogAnalyzer 웹 대시보드 |
| DB 연동 | MariaDB `Syslog` 스키마 |
| 로그 순환 | logrotate / logrotate.timer |
