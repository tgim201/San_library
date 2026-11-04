# 🕯️ 심야의 고요한 실내 서재 (The Silent Library)

아늑한 3가지 실내 서재 테마와 커스텀 MP3 ASMR 사운드(벽난로 장작불, 봄을 기다리는 밤, 비 오는 도시뷰 + 한옥 풍경소리), 그리고 **'다 읽었어요(완독 🏆)' / '이어서 읽기(진행 중 📖)'** 독서 추적 시스템을 갖춘 온라인 실내 도서관 웹 애플리케이션입니다.

---

## 📁 프로젝트 파일 구성

- `index.html` : 빌드 과정 없이 브라우저에서 즉시 실행 가능한 단일 완성형 애플리케이션
- `firebase-config.js` : Firebase 연동 설정 파일 (비워둘 경우 자동으로 `localStorage` 기반 Offline Mock Mode로 작동)
- `firestore.rules` : Firebase Cloud Firestore 보안 규칙 (`reading_logs` 컬렉션 읽기/쓰기 검증)
- `firestore.indexes.json` : Firestore 복합 인덱스 정의 파일 (`status` 및 `createdAtMs` 정렬용)
- `firebase.json` : Firebase CLI 배포 설정 파일
- `seed-data.json` : 공유 독서 기록 서가의 기본 도서 4종 시드 데이터
  1. `세계를 건너 너에게 갈게(이꽃님)`
  2. `고백해도 되는 타이밍(황영미)`
  3. `톰소여의모험(마크 트웨인)`
  4. `해리포터와 비밀의방(조앤.K.롤링)`
- `.nojekyll` : GitHub Pages 배포 시 한글/특수문자 파일명(MP3 등)이 누락 없이 서빙되도록 보장하는 설정 파일
- **배경음 MP3 파일 (3종)**
  - Map 1 (앤틱 벽난로 서재): `🔥 장작불 타는 소리 ASMR 🔥 2시간 벽난로 소리 Cozy Fireplace Sounds (compressed by happyscribe (mp3cut.net).mp3`
  - Map 2 (비 내리는 창가 도서관): `[ASMR] 봄을 기다리는 밤 💨🌳🦉 바람소리, 밤부엉이와 코요테 소리를 담은 한 밤 중 숲 속의 페이크윈도우(가짜창문) (1) (1).mp3`
  - Map 3 (심야 한옥 다실 서재): `[공부ASMR] 비 오는 도시뷰 앞에서 1시간 집중 타이머 빗소리로 집중력 높이기 l 1-Hour Focus Timer with Rain Sounds (1).mp3` (+ 은은한 풍경소리 레이어링 유지)

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
| 필드명 | 타입 | 설명 | 예시 |
| :--- | :--- | :--- | :--- |
| `id` | `string` | 기록 고유 ID | `"log_1793779200000"` |
| `bookTitle` | `string` | 읽은 책 제목 (저자 포함) | `"세계를 건너 너에게 갈게(이꽃님)"` |
| `status` | `string` | `"completed"` (완독 🏆) 또는 `"reading"` (읽는 중 📖) | `"completed"` |
| `totalSeconds` | `number` | 총 누적 독서 시간 (초) | `10860` |
| `quote` | `string` | 오늘의 필사 한 문장 | `"시간을 건너서라도 너에게 닿을 수 있다면..."` |
| `memo` | `string` | 한 줄 감상평 / 메모 | `"벽난로 앞에서 따뜻하게 완독했어요."` |
| `progressNote` | `string` | 읽은 페이지 분량 또는 `"완독 완료"` | `"~132p"` |
| `nickname` | `string` | 작성자 익명 닉네임 | `"벽난로 앞의 독서가"` |
| `mapId` | `number` | 독서한 서재 맵 번호 (`0`, `1`, `2`) | `0` |
| `mapName` | `string` | 독서한 서재 맵 이름 | `"앤틱 벽난로 서재"` |
| `uid` | `string` | Firebase Anonymous Auth UID | `"anon_uid_..."` |
| `createdAt` | `string` | 화면 표시용 날짜 문자열 | `"2026. 11. 04"` |
| `createdAtMs` | `number` | 최신순 정렬용 Unix Timestamp (ms) | `1793779200000` |
| `timestamp` | `timestamp` | Firestore 서버 타임스탬프 | `serverTimestamp()` |

### Step 3. `firebase-config.js` 입력
`firebase-config.js` 파일에 본인의 Firebase 설정값을 입력하고 저장합니다.

---

## 🌐 GitHub Pages 배포 가이드

1. GitHub에서 새 저장소(Repository)를 생성합니다.
2. 프로젝트 폴더의 모든 파일(`index.html`, `firebase-config.js`, `.nojekyll`, `firestore.rules`, MP3 파일 등)을 커밋 & 푸시합니다.
   > **💡 대용량 MP3 파일 업로드 팁:** GitHub 기본 파일당 용량 제한(100MB)으로 인해 `🔥 장작불 타는 소리...mp3`(약 176MB) 파일을 일반 `git push`로 올릴 때 제한이 걸릴 경우, 아래 명령어로 **Git LFS(Large File Storage)**를 활성화하거나 오디오 인코딩 비트레이트를 낮춰(100MB 이하) 푸시하시면 정상 배포됩니다:
   > ```bash
   > git lfs install
   > git lfs track "*.mp3"
   > git add .gitattributes
   > ```
3. GitHub 저장소의 **Settings > Pages > Build and deployment**에서 `main` 브랜치 `/ (root)` 폴더를 선택하고 **Save**를 누릅니다.
4. Firebase Console > **Authentication > 설정 > 승인된 도메인**에 배포된 `<아이디>.github.io` 도메인을 등록합니다.
