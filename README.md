# atom-crossroads (통합과학 갈림길)

고등학교 1학년 통합과학 개념을 점검하는 방향키 게임입니다.
원자가 되어 우주를 날아가며, 질문의 정답이 적힌 문으로 들어가면 점수를 얻고 틀린 문이나 가운데 기둥에 부딪히면 생명이 줄어듭니다(생명 3개).

**게임 바로가기:** https://atom-crossroads.vercel.app (로그인 없이 바로 시작)

## 조작

| 키 | 동작 |
|---|---|
| ↑ / ↓ | 누르는 동안 위아래로 이동 |
| → | 누르는 동안 가속 (정답 30점) |
| ← | 누르는 동안 감속 (정답 10점) |
| 아무것도 안 누름 | 보통 속도 (정답 20점) |
| F | 전체화면 |

휴대폰에서는 화면 아래 버튼을 누르거나 화면을 누른 채 위아래로 끌면 됩니다.

## 기능

- **방 입장:** 선생님이 알려 준 방 번호를 입력하거나 QR 코드로 들어갑니다. 방마다 순위표가 따로 있습니다.
- **실시간 경쟁:** 같은 방에서 지금 달리고 있는 친구들의 점수가 게임 화면에 보입니다.
- **교사 모드:** 페이지 아래쪽에서 비밀번호로 엽니다.
  - 입장 허락: 학생이 정한 캐릭터 이름을 허락하거나 거절 (허락된 이름만 순위에 공개)
  - 방 관리: 단원을 골라 방 만들기, 여러 단원을 합친 방 만들기, 방별 QR 코드
  - 단원·문제 관리: 단원 추가와 삭제, 문제 추가·수정·삭제
  - 많이 틀린 개념 / 학생별 틀린 개념: 방별로 오답 분석
  - 학생 화면: 고른 방에 있는 학생들의 게임 화면을 실시간으로 보기

## 개인정보

- 순위표에는 캐릭터 이름과 점수만 공개됩니다.
- 학번, 본명, 틀린 문제 기록은 페이지를 만든 선생님(편집 권한)만 볼 수 있습니다.
- 그 밖의 정보는 저장하지 않습니다.

## 실행에 대해

- `index.html` 한 파일로 동작하며, 기록은 Firebase(Firestore, Realtime Database)에 저장됩니다.
- 학생과 교사 모두 로그인 화면이 없습니다. 기기마다 Firebase 익명 로그인으로 번호를 받습니다.
- **교사 기기:** 처음 교사 비밀번호를 넣은 기기가 교사 기기로 등록됩니다. 다른 기기는 교사 기기에서 만든 '기기 추가 코드'로 추가합니다. 학번·본명·오답 기록은 교사 기기만 읽을 수 있도록 Firebase 보안 규칙으로 막혀 있습니다.
- 같은 컴퓨터를 여러 학생이 쓸 때 다른 학번을 넣으면 새 번호를 받아 기록이 섞이지 않습니다.
- 교사 기기를 모두 잃어버리면 Firebase 콘솔의 Firestore에서 `config/teacher` 문서를 지운 뒤 다시 교사 비밀번호를 넣으면 됩니다.

## 다른 학교에서 쓰기

다른 학교 선생님도 자유롭게 복사해서 쓸 수 있습니다. 학교마다 **자기 Firebase**에 기록이 저장되므로 학생 개인정보가 학교끼리 섞이지 않습니다. 처음 한 번 30분 정도 걸립니다.

### 1. 파일 가져오기
- 이 저장소 오른쪽 위 **Fork**를 눌러 내 GitHub로 복사합니다. (GitHub를 안 쓰면 `index.html`만 내려받아도 됩니다.)

### 2. Firebase 프로젝트 만들기 (무료)
1. https://console.firebase.google.com 에서 **프로젝트 추가** → 이름을 정하고 만들기 (Google 애널리틱스는 꺼도 됩니다)
2. 프로젝트 첫 화면에서 **웹 앱 추가(</>)** → 앱 이름을 넣고 **앱 등록** → 나오는 `firebaseConfig` 값을 메모장에 복사
3. **빌드 → Authentication → 시작하기 → 로그인 방법 → 익명 → 사용 설정**
4. **빌드 → Firestore Database → 데이터베이스 만들기** → 위치 `asia-northeast3 (서울)` → 프로덕션 모드
5. **빌드 → Realtime Database → 데이터베이스 만들기** → 위치 `asia-southeast1 (싱가포르)` → 잠금 모드
   - 화면 맨 위에 나오는 `https://...firebasedatabase.app` 주소를 복사해 둡니다.

### 3. 보안 규칙 붙여 넣기
학생의 학번·본명·오답 기록을 교사 기기만 볼 수 있게 막는 부분입니다. **꼭 넣어 주세요.**

**Firestore Database → 규칙** 탭: 원래 내용을 지우고 붙여 넣은 뒤 **게시**

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function signedIn() { return request.auth != null; }
    function teacherPath() { return /databases/$(database)/documents/config/teacher; }
    function invitePath() { return /databases/$(database)/documents/config/invite; }
    function isTeacher() { return signedIn() && exists(teacherPath()) && request.auth.uid in get(teacherPath()).data.uids; }
    function own(id) { return signedIn() && request.auth.uid == id; }

    match /config/teacher {
      allow read: if signedIn();
      allow create: if signedIn() && request.resource.data.keys().hasOnly(['uids'])
        && request.resource.data.uids == [request.auth.uid];
      allow update: if isTeacher() || (signedIn()
        && request.resource.data.keys().hasOnly(['uids', 'lastCode'])
        && request.resource.data.uids == resource.data.uids.concat([request.auth.uid])
        && exists(invitePath())
        && request.resource.data.lastCode == get(invitePath()).data.code
        && request.resource.data.lastCode != resource.data.get('lastCode', '')
        && get(invitePath()).data.exp > request.time.toMillis());
      allow delete: if false;
    }
    match /config/invite { allow read, write: if isTeacher(); }

    match /units/{id}     { allow read: if signedIn(); allow write: if isTeacher(); }
    match /rooms/{id}     { allow read: if signedIn(); allow write: if isTeacher(); }
    match /approvals/{id} { allow read: if signedIn(); allow write: if isTeacher(); }
    match /scores/{id}    { allow read: if signedIn(); allow write: if own(id) || isTeacher(); }

    match /names/{id}     { allow read, write: if own(id) || isTeacher(); }
    match /stats/{id}     { allow read, write: if own(id) || isTeacher(); }
    match /requests/{id}  { allow read, write: if own(id) || isTeacher(); }
  }
}
```

**Realtime Database → 규칙** 탭: 원래 내용을 지우고 붙여 넣은 뒤 **게시**

```json
{
  "rules": {
    "lobby": {
      ".read": "auth != null",
      "$uid": { ".write": "auth != null && auth.uid === $uid" }
    },
    "score": {
      "$room": {
        ".read": "auth != null",
        "$uid": { ".write": "auth != null && auth.uid === $uid" }
      }
    },
    "screen": {
      "$room": {
        ".read": "auth != null",
        "$uid": { ".write": "auth != null && auth.uid === $uid" }
      }
    },
    "watch": {
      "$room": { ".read": "auth != null", ".write": "auth != null" }
    }
  }
}
```

### 4. `index.html`에서 두 군데만 바꾸기
`index.html`에서 `학교마다 바꿀 곳`을 찾으면 바로 아래에 있습니다. (GitHub에서는 파일을 열고 연필 아이콘으로 고친 뒤 **Commit changes**)

1. `FIREBASE_CONFIG`: 2단계에서 복사한 `firebaseConfig` 값으로 바꿉니다. `databaseURL` 줄이 없으면 Realtime Database 주소를 넣어 추가합니다.
2. `TEACHER_PASSWORD`: 우리 학교 교사 모드 비밀번호로 바꿉니다.

### 5. 인터넷 주소 만들기 (둘 중 하나)
- **Vercel:** https://vercel.com 에 GitHub로 로그인 → **Add New → Project** → 내 저장소 고르기 → **Deploy**
- **GitHub Pages:** 저장소 **Settings → Pages** → Branch를 `main` / `(root)`로 → **Save**

QR 코드와 방 링크는 올린 주소에 맞춰 자동으로 만들어집니다.

### 6. 처음 한 번 (학생에게 링크를 주기 전에)
1. 내 컴퓨터에서 만든 주소를 열고, 맨 아래 **교사 모드 열기 → 교사 비밀번호**를 넣습니다. 이 기기가 교사 기기로 등록됩니다.
2. **단원·문제 관리** 탭에서 **기본 단원 6개와 방 7개 넣기**를 누릅니다.
3. **방 관리** 탭의 QR 코드나 링크를 학생들에게 나눠 줍니다.

### 알아 두기
- **무료 사용량:** Firebase 무료 요금제(Spark)로 한 학교 수업은 충분합니다. 결제 정보를 넣지 않으면 요금이 나가지 않습니다.
- **개인정보:** 학번과 본명은 각 학교 Firebase에만 저장되고 그 학교 교사 기기만 볼 수 있습니다. 학교의 개인정보 처리 방침에 맞게 안내해 주세요.
- **교사 기기를 모두 잃어버렸을 때:** Firebase 콘솔 → Firestore → `config` → `teacher` 문서를 지우고, 새 기기에서 교사 비밀번호를 다시 넣으면 됩니다.

## 출처

- 기본 문항: 2022 개정 교육과정 통합과학 성취기준을 참고해 자체 제작
- 글꼴: Black Han Sans, Gowun Dodum (Google Fonts, SIL Open Font License)
- QR 생성: [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) (MIT License)
- 데이터 저장: Firebase JavaScript SDK (Apache License 2.0)
