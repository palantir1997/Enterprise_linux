# 🐧 Rocky Linux 서버 구축 실습 (4.2)

> Rocky Linux 9 기반 HTTP/HTTPS, FTP/SFTP, DNS 서버 구축 실습 정리

---

## 📋 목차

1. [Rocky Linux 설치 및 네트워크 설정](#1단계--rocky-linux-설치-및-네트워크-설정)
2. [HTTP / HTTPS 서버 구축](#2단계--http--https-서버-구축)
3. [FTP / SFTP 서버 구축](#3단계--ftp--sftp-서버-구축)
4. [DNS 서버 구축](#4단계--dns-서버-구축)
5. [트러블슈팅](#-트러블슈팅)
6. [체크리스트](#-최종-체크리스트)

---

## 1단계 — Rocky Linux 설치 및 네트워크 설정

### Rocky Linux 9 ISO 다운로드
- https://rockylinux.org/download

### 네트워크 인터페이스 확인

```bash
ip addr show
```

### NetworkManager 설정 (TUI 방식 - 초보자 추천)

```bash
nmtui
```

### CLI 방식

```bash
nmcli con show
nmcli con mod ens33 ipv4.method auto
nmcli con up ens33
```

### 인터넷 연결 확인

```bash
ping -c 4 8.8.8.8
curl -I https://google.com
```

---

## 2단계 — HTTP / HTTPS 서버 구축

### Apache 설치 및 시작

```bash
dnf install -y httpd mod_ssl

systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

### 방화벽 설정

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

### 자기소개 HTML 페이지 생성

```bash
vi /var/www/html/index.html
```

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>자기소개</title>
  <style>
    body { font-family: sans-serif; max-width: 800px; margin: 50px auto; padding: 20px; }
    h1 { color: #333; border-bottom: 2px solid #4CAF50; padding-bottom: 10px; }
    .info { background: #f9f9f9; border-left: 4px solid #4CAF50; padding: 15px; margin: 20px 0; }
  </style>
</head>
<body>
  <h1>안녕하세요! 홍길동입니다</h1>
  <div class="info">
    <p><strong>이름:</strong> 홍길동</p>
    <p><strong>학과:</strong> 컴퓨터공학과</p>
    <p><strong>서버 OS:</strong> Rocky Linux 9</p>
    <p><strong>관심 분야:</strong> 리눅스 서버 관리, 네트워크</p>
  </div>
  <p>Rocky Linux 웹서버 실습 페이지입니다.</p>
</body>
</html>
```

### HTTPS 인증서 생성

```bash
# SSL 디렉토리 생성
mkdir -p /etc/httpd/conf/ssl

# 자체 서명 인증서 생성 (365일)
openssl req -newkey rsa:2048 -nodes \
  -keyout /etc/httpd/conf/ssl/test.com.key \
  -x509 -days 365 \
  -out /etc/httpd/conf/ssl/test.com.crt

# 인증서 내용 텍스트로 확인
openssl x509 -noout -text -in /etc/httpd/conf/ssl/test.com.crt
```

### SSL 설정 파일 수정

```bash
vi /etc/httpd/conf.d/ssl.conf
```

아래 두 줄 경로 수정:

```apache
SSLCertificateFile    /etc/httpd/conf/ssl/test.com.crt
SSLCertificateKeyFile /etc/httpd/conf/ssl/test.com.key
```

### VirtualHost 설정 (도메인 접속용)

```bash
vi /etc/httpd/conf.d/test.com.conf
```

```apache
<VirtualHost *:80>
    ServerName test.com
    ServerAlias www.test.com
    DocumentRoot /var/www/html
</VirtualHost>

<VirtualHost *:443>
    ServerName test.com
    ServerAlias www.test.com
    DocumentRoot /var/www/html
    SSLEngine on
    SSLCertificateFile    /etc/httpd/conf/ssl/test.com.crt
    SSLCertificateKeyFile /etc/httpd/conf/ssl/test.com.key
</VirtualHost>
```

### Apache 재시작

```bash
# 문법 확인 먼저
httpd -t

# 재시작
systemctl restart httpd
```

### 접속 확인

```bash
curl http://localhost
curl -k https://localhost
```

---

## 3단계 — FTP / SFTP 서버 구축

### vsftpd 설치

```bash
dnf install -y vsftpd
systemctl enable vsftpd
```

### FTPS용 인증서 생성

```bash
openssl req -newkey rsa:2048 -nodes \
  -keyout /etc/vsftpd/vsftpd.pem \
  -x509 -days 365 \
  -out /etc/vsftpd/vsftpd.pem

chmod 600 /etc/vsftpd/vsftpd.pem
```

### vsftpd 설정

```bash
vi /etc/vsftpd/vsftpd.conf
```

아래 항목 추가 또는 수정:

```ini
ssl_enable=YES
rsa_cert_file=/etc/vsftpd/vsftpd.pem
force_local_logins_ssl=YES
force_local_data_ssl=YES
ssl_tlsv1=YES
allow_anon_ssl=NO
pasv_enable=YES
pasv_min_port=50000
pasv_max_port=55000
```

### 방화벽 설정

```bash
firewall-cmd --permanent --add-service=ftp
firewall-cmd --permanent --add-port=50000-55000/tcp
firewall-cmd --reload
```

### FTP 사용자 생성 (사용자명 = 본인 이름)

```bash
# 사용자 생성
useradd 본인이름
passwd 본인이름

# FTP 업로드 디렉토리 생성
mkdir -p /home/본인이름/ftp/uploads
chown 본인이름:본인이름 /home/본인이름/ftp/uploads
chmod 755 /home/본인이름/ftp

# SELinux 허용
setsebool -P ftp_home_dir 1
```

### vsftpd 시작

```bash
systemctl start vsftpd
systemctl status vsftpd
```

### FileZilla로 업로드

```
호스트: 서버IP
사용자명: 본인이름
비밀번호: 설정한 비밀번호
포트: 21

원격 경로: /home/본인이름/ftp/uploads
→ 웹페이지 캡처 이미지 드래그앤드롭 업로드
```

### 업로드 확인

```bash
ls -la /home/본인이름/ftp/uploads/
```

---

## 4단계 — DNS 서버 구축

### BIND 설치

```bash
dnf install -y bind bind-utils
```

### named.conf 설정

```bash
vi /etc/named.conf
```

수정할 항목:

```
listen-on port 53 { 서버IP; };
allow-query     { any; };
recursion no;
```

맨 아래에 zone 추가:

```
zone "test.com" IN {
    type master;
    file "test.com.zone";
    allow-update { none; };
};
```

### zone 파일 생성

```bash
vi /var/named/test.com.zone
```

```
$TTL 86400
@       IN      SOA     ns.test.com.    root.test.com.  (
                        20260402
                        24H
                        15M
                        48H
                        86400 );

        IN      NS      ns.test.com.
        IN      A       서버IP
www     IN      A       서버IP
ns      IN      A       서버IP
```

### 설정 검증

```bash
named-checkconf /etc/named.conf
named-checkzone test.com /var/named/test.com.zone
```

### 방화벽 및 서비스 시작

```bash
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload

systemctl start named
systemctl enable named
```

### DNS 동작 확인

```bash
# 서버에서 직접 조회
nslookup test.com 서버IP
nslookup www.test.com 서버IP
```

---


## 최종 체크리스트

| 항목 | 확인 명령어 |
|------|------------|
| 인터넷 연결 | `ping 8.8.8.8` |
| HTTP 서버 동작 | `curl http://서버IP` |
| HTTPS 서버 동작 | `curl -k https://서버IP` |
| FTP 서버 동작 | `systemctl status vsftpd` |
| 파일 업로드 확인 | `ls /home/본인이름/ftp/uploads/` |
| DNS 서버 동작 | `systemctl status named` |
| 도메인 조회 | `nslookup www.test.com 서버IP` |

---

## 참고 사항

- IP 주소 설정 시 `.0`(네트워크), `.255`(브로드캐스트), `.1~.2`(게이트웨이)는 피할 것
- 수동 IP 설정은 `.10 ~ .200` 사이 권장
- SSH는 기본 포트 22번, 암호화된 원격 접속 프로토콜
- hosts 파일은 DNS보다 우선순위가 높음 (로컬 실습 환경에서 유용)
