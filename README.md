# Amplitude x Airbridge (Web) 연동 데모

MMP(Airbridge) 어트리뷰션을 Amplitude 로 연결하는 **웹 플랫폼** 연동을 시연하는 단일 HTML 페이지.
킥오프의 "Attribution(MMP) 연동" 파트에서, **대시보드 설정(발표자)** + **웹 식별자 매칭 코드 구현부** 시연용.

## 무엇을 보여주나
- 두 Web SDK 초기화: Amplitude Browser SDK + Airbridge Web SDK
- **핵심: Amplitude device_id 를 Airbridge device alias(`amplitude_device_id`) 로 매칭** (연동의 전제)
- 이벤트 전송 버튼 → Airbridge 이벤트가 V2 연동으로 Amplitude 에 어트리뷰션(User Property)과 함께 전달되는지 확인

## 초기화 순서 (코드 핵심)
```js
1. await amplitude.init(apiKey).promise      // device_id 생성될 때까지 대기
2. const deviceId = amplitude.getDeviceId()  // device_id 획득
3. airbridge.init({ app, webToken, autoStartTrackingEnabled: false })
4. airbridge.setDeviceAlias('amplitude_device_id', deviceId)   // 식별자 매칭 (핵심)
5. airbridge.startTracking()
```
순서가 중요: device_id 를 먼저 얻어 alias 로 묶은 뒤 추적을 시작해야 어트리뷰션이 같은 유저에 붙는다.

## 실행
파일을 바로 열면(file://) 일부 SDK 동작이 제한될 수 있으니 로컬 서버로 연다:
```bash
cd poc/amplitude-airbridge-web
python3 -m http.server 8000
# http://localhost:8000 접속
```

## 사용법
1. 상단에 **Amplitude API Key / Airbridge App Name / Airbridge Web Token** 입력 (대시보드 발급값)
2. **연동 초기화** 클릭 → 식별자 상태에 device_id 와 alias 표시
3. 로그인 / 회원가입 / 장바구니 / 구매 / 커스텀 버튼으로 이벤트 전송
4. 확인:
   - **Amplitude User Lookup**: 이벤트 + 어트리뷰션 User Property(예: `[Airbridge] channel`)
   - **Airbridge Raw Data**: 같은 이벤트와 attributedChannel

## 어트리뷰션 테스트 (웹)
- 어트리뷰션을 만들려면 **트래킹 링크로 이 페이지에 유입**된 세션에서 이벤트를 보내야 함
- 웹은 쿠키/리퍼러 기반 매칭이라 iOS 앱의 ATT/IDFA 이슈가 없다 (앱 대비 설정이 단순)

## 설정값 / 보안
- API Key, Web Token 은 **입력창에서 받음**. 소스에 하드코딩하지 않는다
- Amplitude Browser API Key, Airbridge Web Token 은 원래 클라이언트 노출 값이라 입력 사용 가능
- **Secret Key 류는 절대 웹에 넣지 않는다**

## 알려진 제약 / 확인 필요
- SDK 버전: Amplitude Browser SDK CDN 2.45.10, Airbridge Web SDK latest 로더 사용. 운영은 npm 설치 권장 (Safari 추적방지 등)
- Airbridge 표준 이벤트 카테고리명(signin/signup/addedToCart/order.completed)은 현행 문서 기준. 세부 semanticAttributes 키는 대시보드/문서로 재확인 권장 [요확인]
