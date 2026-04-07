# Linux & Windows Server 보안 실습 정리

## 1. 리눅스 사용자 계정 구조

리눅스는 사용자 정보와 비밀번호 정보를 분리하여 보안을 강화한다.

| 파일 | 역할 | 접근 권한 |
|------|------|-----------|
| `/etc/passwd` | 사용자 기본 정보 (UID, GID, 홈디렉토리, 쉘 등) | 모든 사용자 읽기 가능 |
| `/etc/shadow` | 암호화된 비밀번호 및 계정 만료 정책 | root만 접근 가능 |

---

## 2. 파일 권한 및 소유자 변경

```bash
chown root ttt.txt       # 파일 소유자를 root로 변경
chgrp root ttt.txt       # 파일 그룹을 root로 변경
chmod 755 ttt.txt        # 파일 권한 변경 (rwxr-xr-x)
```

---

## 3. 리눅스 주요 디렉토리 구조

| 디렉토리 | 설명 |
|----------|------|
| `/bin` | 기본 명령어 (ls, cp, mv 등) |
| `/sbin` | 관리자 명령어 |
| `/etc` | 시스템 설정 파일 |
| `/var` | 로그 및 가변 데이터 |
| `/tmp` | 임시 파일 저장 |
| `/usr` | 응용 프로그램 |
| `/home` | 일반 사용자 홈 디렉토리 |
| `/root` | root 사용자 홈 |
| `/boot` | 부팅 관련 파일 |
| `/dev` | 장치 파일 |
| `/proc` | 프로세스 정보 (가상 파일 시스템) |
| `/lib` | 라이브러리 파일 |

---

## 4. 로그 및 사용자 활동 확인

```bash
cat ~/.bash_history      # 사용자가 입력한 명령어 기록 확인
who                      # 현재 로그인된 사용자 목록
tty                      # 현재 사용 중인 터미널 확인
id                       # 현재 사용자의 UID, GID 확인
```

---

## 5. 파일 검색 및 관리 명령어

```bash
find / -name sshd_config           # 특정 파일 이름으로 전체 검색
find / -perm 4000 -user root       # SUID 권한이 설정된 root 소유 파일 검색
ls -ltr                            # 최근 수정된 파일 순으로 정렬
ls -lhS                            # 파일 크기 기준 내림차순 정렬
ls --hide=*.txt                    # 특정 확장자 숨김
rmdir dirname                      # 빈 디렉토리 삭제
rm -rf dirname                     # 디렉토리 강제 삭제 (주의)
```

---

## 6. 명령어 검색 및 로그 분석

```bash
apropos clear                      # 키워드로 관련 명령어 검색
grep 'root' /etc/passwd            # 특정 문자열 검색
whereis ls                         # 명령어 위치 찾기
tac filename                       # 파일 내용을 역순으로 출력
```

> `find`와 `grep`을 함께 사용하면 시스템 관리 및 보안 분석에서 강력한 도구가 된다.
> 예: `find /var/log -name "*.log" | xargs grep "Failed password"`

---

## 7. Windows Server - IIS (웹 서버)

IIS(Internet Information Services)는 Windows 환경의 기본 웹 서버다.

관리 경로: `IIS Manager > Sites > Default Web Site`

주요 작업 흐름:
- 사이트 추가/삭제 및 포트 설정
- 응용 프로그램 풀 관리
- 바인딩 설정 (IP, 포트, 호스트명)
- 로그 위치: `C:\inetpub\logs\LogFiles`

---

## 8. Windows Server - FTP 서버

FTP(File Transfer Protocol)는 파일 전송을 위한 서비스로, IIS에 역할을 추가하여 구성한다.

### 설치 및 설정 흐름

```
서버 관리자 > 역할 및 기능 추가 > Web Server(IIS) > FTP 서버 체크 후 설치
```

### 주요 설정 항목

| 항목 | 설명 |
|------|------|
| FTP 사이트 추가 | IIS Manager > 사이트 우클릭 > FTP 사이트 추가 |
| 바인딩 | IP 주소 및 포트 지정 (기본: 21) |
| SSL 설정 | 없음 / 허용 / 필수 중 선택 |
| 인증 방식 | 익명 / 기본 인증 선택 |
| 권한 설정 | 읽기 / 쓰기 권한 부여 |

### 방화벽 설정

FTP 사용 시 방화벽에서 포트를 열어야 한다.

```
Windows Defender 방화벽 > 고급 설정 > 인바운드 규칙 > FTP 서버 허용
```

### 접속 확인

```bash
ftp [서버 IP]            # 터미널에서 FTP 접속
```

---

## 9. Windows Server - DNS 서버

DNS(Domain Name System)는 도메인 이름을 IP 주소로 변환하는 서비스다.

### 설치 흐름

```
서버 관리자 > 역할 및 기능 추가 > DNS 서버 선택 후 설치
```

### 주요 설정 항목

| 항목 | 설명 |
|------|------|
| 전방 조회 영역 | 도메인명 -> IP 변환 (일반적인 DNS 조회) |
| 역방향 조회 영역 | IP -> 도메인명 변환 |
| A 레코드 | 호스트 이름과 IP 주소 매핑 |
| CNAME 레코드 | 도메인 별칭 설정 |
| MX 레코드 | 메일 서버 지정 |

### 관리 도구

```
DNS Manager > 전방 조회 영역 > 영역 추가 > 기본 영역 선택
> 영역 이름 입력 > 레코드 추가
```

### 동작 확인

```cmd
nslookup [도메인명]      # DNS 조회 결과 확인
ipconfig /flushdns       # DNS 캐시 초기화
```

---

## 10. 내일 학습 예정

- 리눅스 파일 권한 심화 (chmod / SUID / SGID / Sticky bit)
- 로그 분석 실습 (`/var/log/auth.log`, `journalctl`)
- 웹 서버 및 DB 서버 구조 이해

---

## 참고 자료

- Linux Manual (`man` 명령어)
- RedHat / Rocky Linux 공식 문서
- Microsoft IIS 공식 문서
