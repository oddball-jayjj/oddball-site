# 오드볼 디저트 홈페이지

GitHub Pages용 정적 사이트. `index.html` 하나와 `images/` 폴더로 동작합니다.

## 올리는 법
1. 이 폴더 전체를 GitHub 저장소에 올립니다.
2. Settings → Pages → Branch: main, folder: / (root) 저장.
3. 1~2분 뒤 `https://<계정>.github.io/<저장소>/` 에서 확인.

## 도메인 연결
- 저장소에 `CNAME` 파일을 만들고 도메인(예: `oddball.kr`)만 한 줄 적습니다.
- 도메인 업체 DNS에 A 레코드 4개(185.199.108.153 / 185.199.109.153 / 185.199.110.153 / 185.199.111.153)와
  `www` CNAME → `<계정>.github.io` 를 추가합니다.
- Settings → Pages → Custom domain 에 도메인 입력, Enforce HTTPS 체크.

## 제품 사진 바꾸기 (제품당 2~5장)
1. `images/` 폴더에 사진을 넣습니다. 예: `financier-1.jpg`, `financier-2.jpg` (가로 4:3, 1200px 정도 권장)
2. `index.html`에서 `const PHOTOS` 부분에 경로를 적습니다.
   ```js
   financier: ["images/financier-1.jpg", "images/financier-2.jpg", "images/financier-3.jpg"],
   ```
   첫 장이 대표 사진입니다. 사진이 3장보다 적으면 맛별 그림이 뒤에 채워집니다.
3. `HERO_PHOTO`에 소개 화면 큰 사진 경로를 적습니다. 비워 두면 제품 그림이 나옵니다.
4. 카카오톡·SNS 공유 썸네일을 쓰려면 `images/og.jpg`(1200×630)를 넣고 `index.html` head에 `<meta property="og:image" content="images/og.jpg">`를 추가합니다.

## 제품·굽는 법 수정
- 제품: `const CATALOG` (맛, 규격)
- 굽는 법: `const BAKE` (우녹스 기준 예열·단계별 온도·시간), `const STORE` (보관)
- 전화번호·카카오톡 채널: `const CONFIG`

## 데이터 (Supabase 프로젝트: oddball-homepage)
| 표 | 내용 |
|---|---|
| client_registrations | 신규 거래처 등록 신청 (사업자등록증 파일은 Storage `biz-certs` 버킷) |
| diagnosis_logs | 문제 해결 진단 기록 |
| customer_questions | 진단 후 해결 여부, 질문 |
| page_events | 페이지·제품 조회 |

무료 플랜은 7일 동안 접속이 없으면 일시정지됩니다. 사이트가 공개되어 방문이 생기면 유지됩니다.

## 거래처 등록 → 카카오톡 알림 (최초 1회 설정)
1. developers.kakao.com → 내 애플리케이션 → 애플리케이션 추가
2. 앱 키 → **REST API 키** 복사
3. 카카오 로그인 → 활성화 ON, Redirect URI 추가:
   `https://gtmpahrfruujzwaddulf.supabase.co/functions/v1/kakao-auth`
4. 동의항목 → **카카오톡 메시지 전송(talk_message)** 사용 설정
5. 플랫폼 → Web 사이트 도메인에 `https://gtmpahrfruujzwaddulf.supabase.co` 와 홈페이지 주소 추가
6. Supabase → Edge Functions → Secrets에 `KAKAO_REST_KEY`, `KAKAO_SETUP_STATE` 입력
   (카카오 앱에서 Client Secret을 켰다면 `KAKAO_CLIENT_SECRET`도)
7. 아래 주소를 브라우저에서 열고 대표님 카카오 계정으로 동의 → "연결 완료"와 테스트 메시지가 오면 끝
   `https://kauth.kakao.com/oauth/authorize?client_id=<REST키>&redirect_uri=https://gtmpahrfruujzwaddulf.supabase.co/functions/v1/kakao-auth&response_type=code&scope=talk_message&state=<KAKAO_SETUP_STATE>`

연결 전에도 신청은 Supabase에 저장됩니다(`client_registrations` 표의 `notify_error` 칸에 사유가 남음).
카카오 토큰은 사용할 때마다 자동으로 갱신되며, 2달 넘게 신청이 한 건도 없으면 7번만 다시 하면 됩니다.

## 거래처 등록 → 마켓봄 엑셀
신청이 들어오면 마켓봄 '거래처 대량 등록 양식'(34행 머리글, 35행 데이터) 엑셀이 자동으로 만들어져
카카오톡 알림의 [마켓봄 엑셀] 버튼으로 받을 수 있습니다. (Storage `biz-certs`에도 보관, 링크는 7일간 유효)
- 거래처아이디 = 사업자등록번호 10자리 (마켓봄 임시 비밀번호 = 끝 6자리), 아이디 메시지 발송 = y
- 배송 유형 = 2(택배배송), 사업자등록증 보유 = 유, 양수자 유형 = 2(소매업자)
- **배송 담당자(필수)**: Supabase Secrets에 `MARKETBOM_DELIVERY_MANAGER` = 마켓봄 '직원 관리'에 등록된 이름을 넣어 주세요. 비어 있으면 업로드 전에 엑셀에서 직접 채워야 합니다.
- 바꾸고 싶으면 Secrets에 `MARKETBOM_DELIVERY_TYPE`, `MARKETBOM_BUYER_TYPE`을 넣으면 됩니다.

## 커뮤니티 (네이버 카페 연동)
1. `index.html`의 `CONFIG.CAFE_URL`에 카페 주소, `CONFIG.CAFE_BOARDS`에 게시판별 주소를 넣으면 버튼이 연결됩니다.
2. **추천 글 고정**: Supabase → Table Editor → `community_picks`에 행 추가
   (board: recipe / oven / frozen / qna / diary / notice, title, url, note). 바로 홈페이지에 "추천"으로 표시됩니다.
3. **자동 최신 글**: developers.naver.com → 애플리케이션 등록 → 사용 API '검색' → Client ID/Secret 발급 →
   Supabase Secrets에 `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`, `CAFE_URL`(예: https://cafe.naver.com/카페아이디) 입력.
   게시판별 키워드(우녹스, 스메그, 광파오븐, 에어프라이어, 데크오븐, 질문, 데코·메뉴, 공지)로 카페 글을 찾아 1시간마다 갱신합니다.
   카페 설정에서 '검색 허용'이 켜져 있고 전체 공개된 글만 잡힙니다. 게시판당 최대 5개.
