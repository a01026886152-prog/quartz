# M2 — Git + GitHub 작업 기록 (완료)

**작업자:** 신순화 (안성 지역활동 아키비스트)
**날짜:** 2026-06-26 (1단계) ~ 2026-06-27 (2·3단계)
**목표:** `~/ansungact-ai` 폴더를 GitHub 레포로 만들기
**운전:** Claude Code (챗 Claude가 조수석에서 안내)
**결과:** ✅ M2 완료 — GitHub에 레포 생성 + push 성공

---

## 전체 흐름 (3단계)

| 단계 | 내용 | 상태 |
|---|---|---|
| 1 | git 초기화 + 첫 커밋 (내 PC 안에 저장) | ✅ 완료 |
| 2 | GitHub에 빈 레포 만들고 remote 연결 | ✅ 완료 |
| 3 | HTTPS + Personal Access Token(PAT)으로 push | ✅ 완료 |

> **완료 기준:** GitHub 웹에서 내 레포 → README.md + CLAUDE.md 보이면 성공. → **달성 ✅**

**최종 레포 주소:** `github.com/a01026886152-prog/ansungact-ai` (Private)

---

## 진행 방식 결정

- 운전자: **Claude Code에 프롬프트 던지기** 방식 (챗 Claude는 조수석 안내)
- README 문구는 신순화님 역할에 맞게 **"신순화 — 안성 지역활동 아키비스트"**로 수정
- 인증: SSH 말고 **HTTPS + PAT** 방식, 윈도우 환경

---

## 1단계 — git 초기화 + 첫 커밋 ✅

### 실행한 명령 (한 줄씩, 각각 허락 → Yes)
```
git init
git add .
git commit -m "첫 커밋: 안성 지역활동 아카이브 시작"
```

### 결과
- 커밋 번호: **`397db7a`**
- 담긴 파일: **7개** (README.md, CLAUDE.md, M1 작업기록, `.obsidian` 설정 파일들)

---

## 2단계 — GitHub 레포 생성 + remote 연결 ✅

### GitHub 계정
- 처음엔 "계정 있다"고 생각했으나, Gmail에서 `github` 검색 → 메일 없음 → **계정 없었음 확인**
- 새로 가입: 이메일 `a01026886152@gmail.com`, Username `a01026886152-prog`
- (가입 시 GitHub Copilot 체크는 불필요 → 끄는 게 좋음)

### 빈 레포 만들기 (GitHub 웹에서)
- `github.com/new` 또는 Dashboard의 "Create repository"
- Repository name: `ansungact-ai`
- Description: 안성 지역활동 아카이브
- **Private** 선택
- ⚠️ **README / .gitignore / license 전부 체크 안 함** (이미 로컬에 있어서 충돌 방지 → 완전히 빈 레포로)

### remote 연결
- GitHub이 준 URL: `https://github.com/a01026886152-prog/ansungact-ai.git`
- 이 URL을 Claude Code에 알려주니 `git remote add origin ...` 실행 → **fetch/push 둘 다 정상 연결**

---

## 3단계 — PAT 만들고 push ✅

### PAT(Personal Access Token) 만들기
- 위치: `github.com/settings/tokens` → **Tokens (classic)** → Generate new token (classic)
- 설정값:
  - **Note**: `ansungact-ai push` (토큰 이름, 아무거나 OK)
  - **Expiration**: 90 days
  - **Select scopes**: ⚠️ **`repo` 만 체크** (하위 항목 자동 선택됨. 다른 건 안 건드림)
- Generate token → 토큰(`ghp_...`)은 **그 화면 벗어나면 다시 못 봄** → 즉시 복사해서 메모장에 보관

### push 실행
- Claude Code가 준 방식: 토큰을 URL에 직접 끼워서 push
  ```
  git push https://[토큰]@github.com/a01026886152-prog/ansungact-ai.git master
  ```
  (`[토큰]`자리에 복사한 값, 토큰과 `@github.com` 사이 띄어쓰기 없음)
- **결과: `Pushed to master` → push 성공!**

### ⚠️ 보안 마무리 (중요)
- push 중 토큰이 화면에 노출됐었음 → **즉시 폐기**
- `github.com/settings/tokens` → 해당 토큰 **Delete**
- 토큰을 폐기해도 **이미 올라간 파일은 그대로 남음** (토큰은 업로드용 일회용 열쇠)

---

## 막혔던 지점과 해결법

### (1) Claude Code 화면의 명령어를 Enter 쳐도 반응 없음
- **원인:** 화면에 흐릿하게 보이는 건 "안내(보여주기)"일 뿐, 실제 입력된 게 아님
- **해결:** **직접 타이핑하거나 복사-붙여넣기**해야 진짜 명령으로 인식됨

### (2) git이 "이 커밋 누가 한 거야?" 물음 (Author identity unknown)
- **원인:** 이 PC에서 git 처음 써서 작성자 정보 없음
- **해결:** 한 번만 등록 (이후 자동)
  ```
  git config --global user.name "신순화"
  git config --global user.email "a01026886152@gmail.com"
  ```

### (3) 주소창 vs GitHub 검색창 혼동
- **증상:** `github.com/settings/tokens`를 GitHub 검색창에 넣어 엉뚱한 검색 결과가 나옴
- **해결:** **브라우저 맨 위 주소창**에 입력해야 "그 페이지로 이동"이 됨

### (4) "계정 있는 줄 알았는데 없었음"
- Gmail에서 `github` 검색 → 메일 0건 → 가입한 적 없음 확정 → 새로 가입

### (5) push 인증의 함정 (가장 많이 막히는 곳)
- **함정:** "Password" 물으면 **진짜 비밀번호가 아니라 PAT 토큰**을 넣어야 함
- 토큰을 URL에 끼워 넣는 방식이면 별도 입력 없이 한 번에 통과
- 토큰 입력 시 화면에 안 보여도 정상 (보안)

### (6) Auto-update failed / LF→CRLF 경고
- **판단:** 둘 다 **무해.** 무시해도 됨

---

## 핵심 개념 메모

- **지금 상태:** PC 폴더 ↔ GitHub 레포가 연결되고, 파일이 인터넷(GitHub)에 올라감
- **PAT = 비밀번호 대신 쓰는 임시 열쇠.** 터미널에선 비번 대신 토큰을 씀. 한 번 보여주고 다시 안 보여줌 → 즉시 복사
- **브랜치 이름:** 이번엔 `master`로 올라감 (Claude Code가 URL 직접 방식으로 진행). `main`이든 `master`든 파일은 동일하게 올라감. 나중에 정리 가능

---

## 남은 마무리 (다음에)

- **M2 작업기록(.md)을 GitHub에 올리기:** 어제 폴더에 넣었지만 아직 커밋·push 안 함.
  → `git add . → git commit -m "M2 기록 추가" → git push` 한 번 더 하면 됨
- **다음 자산:** 5종 자산 2번 — 위키 (Obsidian + Quartz + Cloudflare Pages)

---

## M2 완료 체크리스트

- [x] git 초기화 + 첫 커밋 (`397db7a`, 7개 파일)
- [x] git 작성자 정보 등록
- [x] GitHub 계정 생성 (`a01026886152-prog`)
- [x] GitHub 빈 레포 생성 (`ansungact-ai`, Private)
- [x] remote 연결
- [x] PAT 생성 (repo 권한)
- [x] push 성공 (`Pushed to master`)
- [x] 토큰 폐기 (보안 마무리)
- [x] GitHub 웹에서 파일 확인 (README.md, CLAUDE.md 등)
