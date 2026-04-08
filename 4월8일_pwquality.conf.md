## 보안 수업 정리 (2026-04-08)

### 1. 비밀번호 정책 설정 (pwquality)

Linux에서 비밀번호 복잡도 정책은 `/etc/security/pwquality.conf` 파일에서 설정한다.
해당 설정은 사용자 계정의 비밀번호 생성 시 복잡도와 보안 수준을 강제하는 역할을 한다.

#### 주요 옵션 설명

| 속성(Property) | 설명(Description)                        | 권장 설정값(예시)    |
| ------------ | -------------------------------------- | ------------- |
| minlen       | 비밀번호 최소 길이                             | 8 이상          |
| minclass     | 사용해야 하는 문자 종류 수 (대문자, 소문자, 숫자, 특수문자 중) | 3 또는 4        |
| dcredit      | 숫자(Digit) 포함 개수 가중치                    | -1 (최소 1개 필수) |
| ucredit      | 대문자(Uppercase) 포함 개수 가중치               | -1 (최소 1개 필수) |
| lcredit      | 소문자(Lowercase) 포함 개수 가중치               | -1 (최소 1개 필수) |
| ocredit      | 특수문자(Other) 포함 개수 가중치                  | -1 (최소 1개 필수) |
| difok        | 이전 비밀번호와 최소 몇 글자 이상 달라야 하는지 설정         | 3 ~ 5         |
| retry        | 비밀번호 입력 실패 시 재시도 횟수                    | 3             |
| maxrepeat    | 동일 문자의 연속 사용 가능 횟수                     | 2 또는 3        |

#### credit 옵션 동작 방식

credit 계열 옵션(dcredit, ucredit, lcredit, ocredit)은 **양수와 음수에 따라 의미가 다르게 동작한다.**

**음수 설정 (-N)**
최소 N개 이상 해당 문자를 반드시 포함해야 한다.
강제성이 있는 정책이다.

예시

```
dcredit = -1  → 숫자 최소 1개 필수
ucredit = -2  → 대문자 최소 2개 필수
```

**양수 설정 (N)**
해당 문자가 포함될 경우 비밀번호 길이 계산 시 보너스 점수를 주는 방식이다.
강제성은 낮고 권장 수준의 정책이다.

예시

```
dcredit = 1 → 숫자가 있으면 길이 계산 시 +1 보너스
```

---

### 2. PAM 설정 확인

Linux 인증 정책은 PAM(Pluggable Authentication Module)을 통해 관리된다.

관련 설정 디렉토리

```
cd /etc/pam.d/
```

해당 디렉토리에는 로그인, ssh, sudo 등의 인증 정책 파일이 존재하며
pwquality 설정이 이 PAM 정책을 통해 실제로 적용된다.

---

### 3. authselect

`authselect`는 Linux에서 인증 정책(PAM 구성을 포함)을 관리하는 도구이다.

주요 기능

* PAM 인증 정책 프로파일 관리
* 시스템 인증 방식 변경
* 보안 정책 적용 자동화

예시 명령어

```
authselect current
authselect list
```

---

### 4. 로그 분석 환경 구성

이번 실습에서는 로그 분석을 위해 **Loganalyzer + NFS 환경**을 구성하였다.

#### 서버 구조

**221 Server**

역할

* Loganalyzer 설치
* 로그를 시각적으로 분석하고 보여주는 서버
* NFS Client로 동작하여 로그 데이터를 가져옴

기능

* 로그 모니터링
* 웹 기반 로그 분석 화면 제공

**222 Client**

역할

* NFS Server
* 로그 데이터를 제공하는 서버

기능

* 시스템 로그 저장
* NFS를 통해 Loganalyzer 서버로 로그 공유

구조 개념

```
[222 Server]
NFS Server
(로그 저장)

       ↓ NFS 공유

[221 Server]
NFS Client
Loganalyzer
(로그 분석 및 시각화)
```

---

### 5. 과제 내용

이번 실습 과제는 다음과 같다.

1. 웹페이지 화면 캡쳐
2. Loganalyzer 로그 분석 화면 캡쳐
3. FileZilla에서 FTP 접속 화면 캡쳐

FTP 접속을 통해 파일 업로드 및 서버 연결 확인을 수행한다.

---

### 6. 오늘 학습 핵심 내용

1. Linux 비밀번호 보안 정책 설정 (pwquality)
2. PAM 인증 구조 이해
3. authselect를 이용한 인증 정책 관리
4. Loganalyzer 기반 로그 분석 환경 구성
5. NFS를 활용한 로그 서버 구조 이해
6. FTP(FileZilla)를 이용한 서버 접속 실습

---

