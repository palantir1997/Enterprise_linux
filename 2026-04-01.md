# 2026-04-01 보안 수업 정리

---

## 1. 시스템 업데이트 및 기본 명령어

```bash
yum update -y        # 패키지 목록 업데이트
yum upgrade -y       # 패키지 업그레이드
```

| 명령어 | 설명 |
|--------|------|
| `cd /` | 루트(홈)로 이동 |
| `pwd` | 현재 디렉토리 경로 출력 |
| `ls -a` | 숨긴 파일 포함 전체 목록 출력 |
| `ls -h` | 도움말(help) 출력 |

---

## 2. 사용자 관리

```bash
useradd test    # test 사용자 추가
passwd test     # test 사용자 비밀번호 설정
```

---

## 3. 경로 이동

```bash
cd tmp      # 상대 경로 이동
cd /tmp     # 절대 경로 이동
mkdir test  # 디렉토리 생성
```

---

## 4. 파일 권한

```
drwxrwxrwx  소유자 / 그룹 / 기타 사용자 순서로 권한 표시
```

```bash
chmod 755 test.txt   # 소유자: rwx, 그룹: r-x, 기타: r-x
```

- `!` : 강제 실행 옵션
- `.conf` : 환경 설정 파일 확장자

---

## 5. 파일 내용 보기

| 명령어 | 설명 |
|--------|------|
| `more` | 긴 파일을 절반씩 출력 |
| `head` | 파일 앞부분 출력 (`-n` 옵션으로 줄 수 지정) |
| `tail` | 파일 끝부분 출력 |
| `less` | 한 페이지씩 출력 |

---

## 6. 서비스 관리 (systemctl)

```bash
systemctl status httpd        # 서비스 상태 확인
systemctl start httpd         # 서비스 시작
```

---

## 7. 방화벽 설정

```bash
firewall-cmd                            # 방화벽 상태 확인 (running이면 정상)
firewall-cmd --permanent --add-port=80/tcp   # 80번 포트(TCP) 영구 허용
firewall-cmd --reload                   # 설정 반영
firewall-cmd --list-all                 # 설정 확인 (ports 항목에 80/tcp 확인)
```

---

## 8. 패키지 설치

```bash
dnf install [패키지명]   # 패키지 설치
httpd -V                 # 아파치(Apache) 버전 확인
php -v                   # PHP 버전 확인
```

---

## 9. PHP 테스트 페이지 생성

```bash
vi /var/www/html/phptest.php
```

```php
<?php phpinfo(); ?>
```

- 브라우저에서 `http://[서버IP]/phptest.php` 접속 시 서버의 PHP 환경 정보 확인 가능

---

## 10. Apache 설정 파일

```bash
vi /etc/httpd/conf/httpd.conf   # Apache 환경 설정 파일
```

---

## 11. 웹 페이지 생성 및 배포

```bash
vi test/test.html               # HTML 파일 생성
# 브라우저: http://[서버IP]/test/test.html

cp test/test.html index.html    # index.html로 복사 (IP만으로 접근 가능)
# 브라우저: http://[서버IP]/
```

---

## 12. MariaDB 설치 및 설정

```bash
systemctl start mariadb         # MariaDB 서비스 시작
mysql_secure_installation       # 보안 설치 마법사 실행
mysql -u root -p                # root 계정으로 DB 접속
```

> 접속 구조: SSH -> 서버 -> DB

---

## 13. MariaDB 기본 쿼리

### 데이터베이스 관리

```sql
SHOW DATABASES;          -- 데이터베이스 목록 조회
USE mysql;               -- 데이터베이스 선택
SHOW TABLES;             -- 테이블 목록 조회
SELECT user, password FROM user;   -- 사용자 목록 조회

CREATE DATABASE test;    -- 데이터베이스 생성
DROP DATABASE test;      -- 데이터베이스 삭제
```

### 테이블 생성 및 수정

```sql
-- 테이블 생성
CREATE TABLE student (
    Name  VARCHAR(10),
    Date  INT(4),
    Phone INT(10)
);

DESC student;   -- 필드(컬럼) 정보 조회

-- 컬럼 추가
ALTER TABLE student ADD Age INT(3);
ALTER TABLE student ADD Address VARCHAR(10) AFTER Date;
ALTER TABLE student ADD MBTI CHAR(4) FIRST;
ALTER TABLE student ADD (Age1 INT(3), Age2 INT(3));

-- 컬럼 순서 변경
ALTER TABLE student MODIFY Name VARCHAR(10) FIRST;

-- 컬럼 삭제
ALTER TABLE student DROP Age1;
```

### 데이터 삽입 및 수정

```sql
-- 전체 필드 삽입
INSERT INTO student VALUES ('Park', 2026, 'Seoul', 12345678, 32);

-- 특정 필드만 삽입
INSERT INTO student (Name, Address) VALUES ('Lee', 'Busan');

-- 데이터 수정
UPDATE student SET Date = 2026;

-- 데이터 필드 삭제
ALTER TABLE 테이블명 DROP COLUMN 필드명;
```

---

*작성일: 2026-04-01*
