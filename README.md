# 말씀묵상 · 7분법 QT

매일 성경 한 장을 7분법(묵상·찬양·감사·회개·간구·중보·특별대상)으로 묵상하는 iOS 스타일 웹앱.

## 기능

- 📖 매일 자동 진행 (설치일 = Day 1, 마가→마태→누가 순)
- 🙏 7단계 묵상 기록 (1단계만 나눔에 공개)
- 🤖 AI 묵상 요약 (Anthropic Claude API)
- 🔥 연속 묵상 스트릭 · 6주 캘린더
- 💬 커뮤니티 나눔 + 좋아요 + 댓글
- 🔐 아이디/비번 로그인 + 이용약관/개인정보 동의

## 로컬에서 실행

```bash
# 아무 정적 서버면 됩니다
python3 -m http.server 8000
# 또는
npx serve .
```

브라우저로 `http://localhost:8000` 열기.

## Firebase 설정 (선택)

Firebase 미설정 상태로도 모든 기능이 동작합니다(데이터는 이 기기에만 저장). 여러 사용자가 서로 나눔을 공유하려면 Firebase를 연결하세요.

1. https://console.firebase.google.com 에서 새 프로젝트 생성
2. 프로젝트 개요 → 앱 추가 → **웹(</>)** 선택 → 앱 등록
3. 제공되는 `firebaseConfig` 객체를 복사
4. 프로젝트 루트의 `firebase-config.js`를 열어 다음과 같이 수정:

```js
window.FIREBASE_CONFIG = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123...:web:abc"
};
```

5. Firebase Console 좌측 메뉴 → **Firestore Database** → **데이터베이스 만들기** → **테스트 모드**로 시작(개발용)
6. 운영 환경에서는 Firestore 규칙을 다음과 같이 설정:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['uid','name','text','ref','dayIndex']);
      allow update: if request.resource.data.diff(resource.data).affectedKeys().hasOnly(['likes','commentCount']);
      match /comments/{commentId} {
        allow read: if true;
        allow create: if true;
      }
    }
  }
}
```

7. 새로고침하면 설정 화면의 **백엔드 상태**에 `Firebase 연결됨`이 표시됩니다.

## AI 요약 설정

설정 탭 → **AI 묵상 요약** → Anthropic API 키(`sk-ant-...`) 입력.

- 키 발급: https://console.anthropic.com → API Keys
- 키는 이 기기 localStorage에만 저장되며, 요청은 이용자의 Claude 계정으로 직접 호출됩니다.
- Claude Max 구독이 있어도 **API 사용은 별도 과금**입니다(Anthropic 정책). API 크레딧을 충전해 사용하세요.

## GitHub Pages 배포

```bash
git init
git add .
git commit -m "초기 커밋"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

GitHub 저장소 → **Settings** → **Pages** → **Source: Deploy from a branch** → **Branch: main / root** → Save.

1~2분 후 `https://YOUR_USERNAME.github.io/YOUR_REPO/` 에서 접속 가능.

⚠️ **주의**: `firebase-config.js`에 실제 키를 넣어 커밋하면 GitHub에 공개됩니다. Firebase 키는 클라이언트에 노출해도 되지만 Firestore 보안 규칙을 반드시 설정하세요.

## 성경 본문 추가

`data/bible.js`의 `window.BIBLE.chapters` 객체에 장을 추가합니다:

```js
window.BIBLE.chapters['mark-2'] = {
  title: '중풍병자를 고치심',
  verses: [
    { n: 1, t: '...' },
    { n: 2, t: '...' },
    // ...
  ],
};
```

**저작권 안내**: 기본은 **개역한글(공공도메인)** 입니다. 개역개정(대한성서공회)은 저작권 보호 대상이므로 공개 배포용으로 사용하지 마세요.

## 구조

```
index.html              # 앱 진입점
styles.css              # 디자인 시스템
firebase-config.js      # Firebase 설정 (기본 비활성)
firebase-config.js.template
data/
  bible.js              # 성경 본문 + 읽기 계획
  steps.js              # 7분법 단계 정의
js/
  primitives.jsx        # 공용 컴포넌트 (아이콘, 탭바 등)
  backend.js            # Firebase + 로컬 저장 추상화
  store.js              # 묵상 기록 (localStorage)
  ai.js                 # Claude API 호출
  legal.js              # 이용약관/개인정보
  app.jsx               # 메인 앱 + 화면들
frames/
  ios-frame.jsx         # (참조용)
```

## 라이선스

개인 사용 무료. 재배포 시 본문 저작권을 확인하세요.
