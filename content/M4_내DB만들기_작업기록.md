# M4 — 내 DB 만들기 (Supabase 테이블 + REST) 작업 기록 (완료)

**작업자:** 신순화 (안성 지역활동 아키비스트)
**날짜:** 2026-06-28
**목표:** 내 데이터베이스(Supabase)에 데이터 넣고, URL(REST)로 꺼내보기
**운전:** 브라우저(Supabase 대시보드) + PowerShell, 챗 Claude가 조수석에서 안내
**현재 상태:** ✅ **완료** — 터미널에서 내 notes 데이터가 JSON으로 돌아옴 (완료 기준 충족)

---

## 큰 그림 (M4 전체 흐름)

| 시간 | 단계 | 내용 | 상태 |
|---|---|---|---|
| 0~10' | 1 | Supabase 가입 + 새 프로젝트 생성 | ✅ 완료 |
| 10~25' | 2 | SQL Editor에서 `notes` 테이블 + 데이터 1줄 | ✅ 완료 |
| 25~40' | 3 | RLS 켜기 + anon SELECT 정책 | ✅ 완료 |
| 40~60' | 4 | 터미널에서 REST GET 확인 (`curl`) | ✅ 완료 |

> **M4 완료 기준:** 터미널에서 내 notes 데이터가 JSON으로 돌아오면 성공. → **달성함.**

---

## 시작 전 결정사항

- **Supabase란?** PostgreSQL 데이터베이스를 클릭 몇 번으로 만들고, 자동으로 REST API까지 붙여주는 서비스. 내 데이터를 인터넷 URL로 넣고 꺼낼 수 있게 해줌.
- **로그인:** GitHub 계정(`a01026886152-prog`)으로 가입. M2에서 만든 그 계정 그대로 사용 → 새 비번 만들 필요 없음.
- **플랜:** Free ($0/month). 카드 등록 없이 사용.
- **키 두 종류 (중요):**
  - `anon` (public) 키 = 일반 사용자 수준. RLS 켜져 있으면 **브라우저/코드에 노출돼도 OK.**
  - `service_role` (secret) 키 = 관리자 전권, **RLS 무시.** 절대 클라이언트·채팅·캡처에 노출 금지.
  - → 이번 작업엔 **anon 키만** 사용. service 키는 끝까지 안 건드림.

---

## 1단계 — Supabase 가입 + 프로젝트 생성 ✅

- `supabase.com` → **Start your project** → **Continue with GitHub**
- OAuth 승인 화면: "이메일 주소(읽기 전용)"만 요청 → 안전, 승인.
- **조직(Organization) 생성:** 이름 자동값 그대로, Type=Personal, Plan=**Free** 확인 후 생성.
  - 메모: Supabase는 "조직 안에 프로젝트" 구조. 조직은 한 번만 만들면 됨.
- **새 프로젝트 생성:**
  - Project name: `ansungact-archive` (영문·숫자·하이픈만, 한글 불가)
  - GitHub 연결(optional): **건너뜀** (SQL·curl로 할 거라 불필요)
  - **Database password:** `Generate a password`로 자동 생성 → **안전한 곳(메모장/비번관리앱)에 복사 저장.** 이 화면 지나면 다시 안 보임. ⚠️ 채팅창엔 절대 붙여넣지 않음.
- 생성 직후 STATUS가 **Unhealthy**로 보였으나, 1~2분 뒤 **Healthy(초록)**로 바뀜 = 정상. DB 켜지는 중이었던 것.
- 프로젝트 URL 확보: `https://uyqgrnkouvyfwtsmemsn.supabase.co`

---

## 2~3단계 — 테이블 + 데이터 + RLS 정책 ✅

**위치:** Supabase 대시보드 → **SQL Editor** (주소: `.../project/<프로젝트ID>/sql/new`)

### (가) 테이블 만들기 시도 → "이미 있음" 에러
- 처음 `CREATE TABLE notes (...)` 실행 시:
  > `ERROR: 42P07: relation "notes" already exists`
- = **테이블이 이미 있었음** (M4를 전에 한 번 시작했던 흔적). 잘못 아님.
- 단, `CREATE TABLE`에서 에러 나면서 **그 아래 줄(INSERT·RLS·정책)이 실행 안 됨** → 따로 처리 필요.

### (나) 데이터·정책만 안전하게 재실행
```sql
-- 테스트 데이터 1줄 (이미 있어도 또 넣어도 무방)
INSERT INTO notes (title, body)
VALUES ('첫 번째 노트', '안성 지역활동 아카이브 시작!');

-- RLS 켜기 (이미 켜져 있어도 에러 안 남)
ALTER TABLE notes ENABLE ROW LEVEL SECURITY;

-- 같은 이름 정책 있으면 지우고 새로
DROP POLICY IF EXISTS "anon_read" ON notes;
CREATE POLICY "anon_read" ON notes
  FOR SELECT
  USING (true);
```
→ `Success. No rows returned` (테이블 설정/정책은 데이터를 안 돌려줘서 "No rows"가 정상).

### (다) 데이터 확인 + 중복 정리
- `SELECT * FROM notes;` → **2 rows** (INSERT를 두 번 돌려서 중복 생김)
- 중복 정리: 같은 (title, body) 중 가장 오래된 1개만 남기고 삭제 → **1 row** 깔끔.

```sql
DELETE FROM notes a
USING notes b
WHERE a.title = b.title
  AND a.body  = b.body
  AND a.created_at > b.created_at;
SELECT * FROM notes;
```

---

## 4단계 — 터미널에서 REST GET 확인 ✅

### (가) Project URL + anon 키 확보
- **Project URL / API URL:** `Settings → API` (또는 Data API 페이지)
  - `https://uyqgrnkouvyfwtsmemsn.supabase.co/rest/v1/`
- **anon 키:** `Settings → API Keys` → **"Legacy anon, service_role API keys"** 탭
  - ⚠️ Supabase가 키 체계를 새로 바꿔서, 예전 방식 anon 키는 **Legacy 탭**에 있음. (새 방식은 "Publishable key"라는 이름.)
  - `anon` `public` 키의 **Copy** 버튼으로 복사. `service_role` 키는 안 건드림.
  - ⚠️ **"Disable JWT-based API keys" 버튼 누르지 말 것** — 누르면 쓰려는 anon 키가 꺼짐.

### (나) curl 시도 → PowerShell 함정
- 깨끗한 PowerShell 창에서 anon 키를 변수에 저장:
  ```
  $KEY = "<anon 키>"
  ```
- `curl ... -H "apikey: $KEY" ...` 실행 → **빨간 에러** (`ParameterBindingException`, `IDictionary로 변환 불가`)
  - **원인:** Windows PowerShell에서 `curl`은 진짜 curl이 아니라 **`Invoke-WebRequest`의 별명**. `-H "..."` 같은 진짜 curl 문법을 못 알아들음.
- `curl.exe`로 다시 시도 → 이번엔 명령 두 개가 줄바꿈 없이 엉켜 붙음 → `curl: (6) Could not resolve host: curl.exe`, `400 Bad Request`.

### (다) 성공한 방법 — Invoke-RestMethod
PowerShell 네이티브 명령으로 한 줄 실행:
```
Invoke-RestMethod -Uri "https://uyqgrnkouvyfwtsmemsn.supabase.co/rest/v1/notes?select=*" -Headers @{ apikey = $KEY; Authorization = "Bearer $KEY" }
```
→ **데이터가 표로 돌아옴:**
```
id         : 37dc8ee9-8548-4b94-8608-4130a4ae311c
title      : 첫 번째 노트
body       : 안성 지역활동 아카이브 시작!
created_at : 2026-06-28T05:31:20.550232+00:00
```
한글 안 깨짐, 빈 배열 `[]` 아님, 정책 제대로 걸림 = **M4 완료.** 🏁

---

## 핵심 개념 메모

- **Windows PowerShell의 `curl`은 가짜다.** 진짜 curl(`-H`, `-d` 등 문법)을 쓰려면:
  - `curl.exe`를 직접 부르거나 (단, 명령 한 줄로 또박또박),
  - **`Invoke-RestMethod -Headers @{ ... }`** 방식 사용 ← 제일 안전하고 추천.
- **헤더 두 개 다 필요:** Supabase REST는 `apikey`와 `Authorization: Bearer` 둘 다 같은 키로 넣어야 함.
- **빈 배열 `[]`만 오면** = RLS 정책 빠진 것. `CREATE POLICY ... FOR SELECT USING (true);` 확인.
- **anon vs service:** anon은 공개 OK(RLS 켜져 있을 때), service는 절대 노출 금지.
- **키·비번은 터미널/메모장 안에만. 채팅창엔 절대 붙여넣지 않기.**

---

## 반복된 혼란 패턴 (계속 유효)

- 마법사/대화형 화면은 **그 창에서 직접 키보드**로 답해야 함. Claude Code 채팅창에 "default 골라줘"라고 쳐도 마법사는 못 알아들음.
- 명령을 **붙여넣을 때 두 개가 엉켜 붙는** 일 잦음 → 붙여넣기 전 Enter 한 번으로 줄 비우고, 붙여넣은 뒤 한 줄인지 눈으로 확인.
- 한/영 키 확인: `npx`가 `ㅜㅔㅌ`처럼 나오면 한글 모드.
- URL은 브라우저 **주소창**에 (검색창 말고).

---

## M4 체크리스트

- [x] Supabase 가입 (GitHub OAuth, `a01026886152-prog`)
- [x] 조직 생성 (Free 플랜)
- [x] 프로젝트 생성 (`ansungact-archive`) + DB 비번 저장
- [x] STATUS Healthy 확인
- [x] `notes` 테이블 (이미 존재) + 데이터 1줄
- [x] RLS 켜기 + `anon_read` SELECT 정책
- [x] 중복 데이터 정리 (1 row)
- [x] Project URL + anon 키 확보 (Legacy 탭)
- [x] 터미널에서 REST GET 성공 (`Invoke-RestMethod`) ← **M4 완료**

---

## 다음에 해볼 수 있는 것 (선택)

- **POST로 데이터 넣기:** 같은 anon 키로 `notes`에 새 글 추가 (INSERT 정책 추가 필요).
- **M3 위키 ↔ M4 DB 연결:** Quartz 위키 페이지에서 Supabase 데이터를 불러와 보여주기.
- **보안 마무리:** 작업 끝난 PowerShell 창은 닫기(`exit`) → `$KEY` 변수 사라짐.

> **오늘의 성과:** M3(Quartz 위키 로컬 미리보기)와 M4(Supabase DB + REST)를 하루에 둘 다 완료.
