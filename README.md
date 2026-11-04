# 🕯️ San 고요한 서재 (The Silent Library)

아늑한 3가지 실내 서재 테마(**벽난로 서재**, **비내리는 창가도서관**, **심야 한옥 서재**)와 고음질 ASMR 사운드(**벽난로 장작소리 MP3**, **천둥번개 없는 잔잔한 창가 빗소리**, **고즈넉한 밤바람·귀뚜라미·풍경소리**), 그리고 **'다 읽었어요(완독 🏆)' / '이어서 읽기(진행 중 📖)'** 독서 추적 시스템을 갖춘 온라인 실내 도서관 웹 애플리케이션입니다.

---

## 📁 프로젝트 파일 구성

- `index.html` : 빌드 과정 없이 브라우저에서 즉시 실행 가능한 단일 완성형 애플리케이션
- `firebase-config.js` : Firebase 연동 설정 파일 (비워둘 경우 자동으로 `localStorage` 기반 Offline Mock Mode로 작동)
- `firestore.rules` : Firebase Cloud Firestore 보안 규칙 (`reading_logs` 컬렉션의 익명 닉네임 및 독서 기록 필드만 엄격히 검증)
- `firestore.indexes.json` : Firestore 복합 인덱스 정의 파일 (`nickname` + `createdAtMs`, `status` + `createdAtMs` 정렬용)
- `firebase.json` : Firebase CLI 배포 설정 파일
- `seed-data.json` : 공유 독서 기록 서가의 기본 도서 4종 시드 데이터
  1. `세계를 건너 너에게 갈게(이꽃님)`
  2. `고백해도 되는 타이밍(황영미)`
  3. `톰소여의모험(마크 트웨인)`
  4. `해리포터와 비밀의방(조앤.K.롤링)`
- `.nojekyll` : GitHub Pages 배포 시 한글 파일명(MP3 등)이 누락 없이 서빙되도록 보장하는 설정 파일
- **맵별 배경음 구성 (3종)**
  - Map 1 (**벽난로 서재**): `장작소리.mp3`
  - Map 2 (**비내리는 창가도서관**): 48kHz 무손실 Web Audio 합성 — **천둥번개 없는 잔잔한 창가 빗소리 & 빗방울 토독거림**
  - Map 3 (**심야 한옥 서재**): 48kHz 무손실 Web Audio 합성 — **고즈넉한 밤바람 백색소음 + 은은한 귀뚜라미 풀벌레 소리 + 처마 끝 풍경소리**

---

## 🔥 Firebase 연동 및 데이터베이스 구성 가이드

### Step 1. Firebase 프로젝트 생성 및 익명 인증 활성화
1. [Firebase Console](https://console.firebase.google.com/)에서 새 프로젝트를 생성하고 **웹(`</>`) 앱**을 등록합니다.
2. **빌드 > Authentication > 로그인 방법(Sign-in method)**에서 **익명(Anonymous)** 로그인을 **사용 설정**합니다.

### Step 2. Cloud Firestore 데이터베이스 및 보안 규칙 적용
1. **빌드 > Firestore Database > 데이터베이스 만들기**를 클릭하여 생성합니다.
2. Firestore 콘솔의 **규칙(Rules)** 탭에 `firestore.rules` 파일의 내용을 붙여넣고 **게시(Publish)**를 클릭합니다.
3. 최초 접속 시 `reading_logs` 컬렉션이 비어 있으면 애플리케이션(`ensureFirestoreSeedLogs`)이 자동으로 기본 도서 4권(`seed_1` ~ `seed_4`)을 Firestore에 초기화하며, 이후 모든 방문자의 기록이 실시간(`onSnapshot`)으로 공유됩니다.

#### 📊 Firestore `reading_logs` 문서 데이터 구조 (Schema)
특정 익명 닉네임(`nickname`)과 그에 따른 독서기록 필드만 저장되며, 가벼운 인사(`greetings`) 등은 DB에 저장하지 않습니다.

| 필드명 | 타입 | 설명 | 예시 |
| :--- | :--- | :--- | :--- |
| `id` | `string` | 기록 고유 ID | `"log_1793779200000"` |
| `nickname` | `string` | 입장 시 설정한 익명 닉네임 (독서기록 연결 기준 키) | `"사각거리는 흑연"` |
| `bookTitle` | `string` | 읽은 책 제목 (저자 포함) | `"세계를 건너 너에게 갈게(이꽃님)"` |
| `status` | `string` | 완독 여부: `"completed"` (완독 🏆) 또는 `"reading"` (읽는 중 📖) | `"completed"` |
| `totalSeconds` | `number` | 총 누적 독서 시간 (초) | `10860` |
| `quote` | `string` | 오늘의 필사 한 문장 | `"시간을 건너서라도 너에게 닿을 수 있다면..."` |
| `memo` | `string` | 한 줄 감상평 / 메모 | `"벽난로 앞에서 따뜻하게 완독했어요."` |
| `progressNote` | `string` | 읽은 페이지 분량 또는 `"완독 완료"` | `"~132p"` |
| `mapName` | `string` | 독서한 서재 맵 이름 | `"벽난로 서재"` |
| `createdAt` | `string` | 화면 표시용 날짜 문자열 | `"2026. 11. 04"` |
| `createdAtMs` | `number` | 최신순 정렬용 Unix Timestamp (ms) | `1793779200000` |

### Step 3. `firebase-config.js` 입력
`firebase-config.js` 파일에 본인의 Firebase 설정값을 입력하고 저장합니다.

---

## 🌐 GitHub Pages 배포 가이드

1. GitHub에서 새 저장소(Repository)를 생성합니다.
2. 프로젝트 폴더의 모든 파일(`index.html`, `firebase-config.js`, `.nojekyll`, `firestore.rules`, `장작소리.mp3` 등)을 커밋 & 푸시합니다. (`장작소리.mp3` 파일이 11.6MB로 압축되어 있고 빗소리·밤소리는 브라우저 내장 무손실 Web Audio 합성으로 작동하므로 Git LFS 없이 일반 `git push`로 바로 업로드 가능합니다.)
3. GitHub 저장소의 **Settings > Pages > Build and deployment**에서 `main` 브랜치 `/ (root)` 폴더를 선택하고 **Save**를 누릅니다.
4. Firebase Console > **Authentication > 설정 > 승인된 도메인**에 배포된 `<아이디>.github.io` 도메인을 등록합니다.
