# 📋 보안 수업 정리 - 4.3 NFS / Samba / SELinux

---

## 1. /etc/shadow

> 리눅스에서 사용하지 않는 도메인 선택 시 확인하는 파일

- 사용자들의 **패스워드가 암호화**되어 저장됨
- 직접 열어서 확인 가능 (root 권한 필요)

---

## 2. NFS (Network File System)

### 환경 구성

| 역할 | 호스트 |
|------|--------|
| NFS 서버 | Rocky-server (nfs-s) |
| NFS 클라이언트 | DNS 서버 (nfs-client) |

---

### 🖥️ 서버 설정

```bash
# 필요 패키지 설치
dnf install -y nfs-utils

# 서비스 시작
systemctl start nfs-server
systemctl start rpcbind

# 호스트명 확인 및 변경
hostnamectl status
hostnamectl set-hostname Server

# 공유 디렉토리 생성 및 권한 설정
mkdir /nfsserver         # 기본 권한: 755
chmod 756 /nfsserver

# exports 설정 (공유 대상 IP는 마스킹 처리)
vi /etc/exports
```

**`/etc/exports` 내용:**
```
/nfsserver  172.16.xx.xxx(rw,sync,no_root_squash)
```

```bash
# 설정 적용
exportfs -a

# 방화벽 포트 오픈
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --reload
```

---

### 💻 클라이언트 설정

```bash
# 서비스 시작
systemctl start rpcbind

# 마운트 포인트 생성
mkdir /nfsclient

# NFS 마운트 (서버 IP는 마스킹 처리)
mount -t nfs 172.16.xx.xxx:/nfsserver /nfsclient/

# 마운트 확인
df -h

# 마운트 해제
umount /nfsclient
```

> ✅ **핵심 흐름**: 서버에서 디렉토리 공유(export) → 클라이언트에서 마운트

---

## 3. Samba (SMB/CIFS)

### NFS vs Samba 비교

| 구분 | NFS | Samba |
|------|-----|-------|
| 주요 개발사 | Sun Microsystems | Microsoft (오픈소스 구현) |
| 주 사용 환경 | Linux / Unix 간 공유 | **Windows ↔ Linux 간 공유** |
| 마운트 방식 | 리눅스 파일시스템처럼 마운트 | 윈도우 네트워크 드라이브 방식 |
| 설정 난이도 | 비교적 간단하고 가벼움 | 설정 많지만 유연함 |
| 성능 | 작은 파일 많을 때 빠름 | 일반 파일 전송에 안정적 |
| 보안 방식 | IP/호스트 기반 인증 (비교적 취약) | **사용자 계정/비밀번호 기반** |
| 호환성 | Windows에서 별도 설정 필요 | 거의 모든 OS에서 기본 지원 |

---

### 🖥️ Samba 서버 설정

```bash
# 패키지 설치
dnf install -y samba samba-common samba-client

# 공유 디렉토리 생성
mkdir /sambatest
chmod 777 /sambatest

# Samba 사용자 비밀번호 설정
smbpasswd -a sambauser
```

**`/etc/samba/smb.conf` 설정 추가:**
```ini
[Sambashare]
    path = /sambatest
    browsable = yes
    writable = yes
    guest ok = yes
    create mask = 0777
    directory mask = 0777
```

```bash
# 설정 확인
testparm

# 서비스 시작
systemctl start smb nmb

# Samba 연결 상태 확인
smbstatus

# 방화벽 설정
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

---

### 💻 Windows에서 접속

```
Windows + R → \\서버IP 입력
```

> ⚠️ 전원 껐다 켠 후에는 `systemctl start nmb smb` 실행 필요

---

## 4. SELinux

```bash
# SELinux 상태 확인
sestatus
```

> Samba 사용 시 SELinux 설정이 접속에 영향을 줄 수 있으므로 상태 확인 필수

---

## 📂 디렉토리 확인

```bash
# Samba 공유 폴더에서 생성된 파일 확인
cd /sambatest
ls
```

---

## 🔑 핵심 명령어 요약

| 명령어 | 설명 |
|--------|------|
| `exportfs -a` | NFS exports 설정 적용 |
| `df -h` | 마운트된 파일시스템 확인 |
| `testparm` | Samba 설정 유효성 검사 |
| `smbstatus` | Samba 연결 현황 확인 |
| `sestatus` | SELinux 상태 확인 |
| `hostnamectl status` | 호스트명 확인 |

---

> 📌 **참고**: 문서 내 IP 주소는 보안상 일부 마스킹 처리되었습니다.
