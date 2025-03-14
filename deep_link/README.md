https://velog.io/@dmswls5115/Flutter-App-Links-%EC%A0%81%EC%9A%A9%EA%B8%B0-Deep-Link

[Flutter] Deep Link란?
[Flutter] App Links 적용기 (Deep Link)
[Flutter] Universal Link 적용기 (Deep Link)
[Flutter] App Links 적용 에러 모음
[Flutter] Deferred Deep Link 적용 에러 모음
[Flutter] AOS 딥링크로 인해 동일한 앱이 두 번 생성 되는 이슈
[Flutter] 딥링크 진입 유저 뒤로가기 동작을 위한 스택 처리 (app_links 라이브러리)

https://docs.flutter.dev/cookbook/navigation/set-up-app-links
모든 단계는 플러터 공식 문서를 따라가고 있으니 문서를 보면서 참고하면 좋다:)


본격적인 작업이 들어가기 전에 앱 미설치 유저에게 보여줄 웹 페이지가 필요하다. 나는 테스트를 위해 github.io로 간단한 웹페이지를 만들었다.
👉 github.io 빠르게 만드는 방법: https://pages.github.com/

[Android deep link]
1. AndroidManifest 파일에 태그 추가하기
   먼저 android/app/src/main/AndroidManifest.xml파일에 아래처럼 태그를 추가한다.

android:host 부분에 꼭 "example.com" 형태와 동일하게 넣어주셔야 한다!!
<meta-data android:name="flutter_deeplinking_enabled" android:value="true" />
<intent-filter android:autoVerify="true">
<action android:name="android.intent.action.VIEW" />
<category android:name="android.intent.category.DEFAULT" />
<category android:name="android.intent.category.BROWSABLE" />
<data android:scheme="http" android:host="example.com" />
<data android:scheme="https" />
</intent-filter>

2. assetlinks.json 파일 호스팅
1) 웹 프로젝트에 .well-known 폴더 생성하기
   플러터 앱 프로젝트 내에 생성하는 것이 아니라, 웹 프로젝트에 생성해야한다!

루트 디렉토리에 .well-known 폴더를 생성
해당 폴더 안에 assetlinks.json 파일 생성
(apple-app-site-association 은 universal link 만들 때 추가하는 파일이다.)

2) assetlinks.json 파일 생성하기
   [{
   "relation": ["delegate_permission/common.handle_all_urls"],
   "target": {
   "namespace": "android_app",
   "package_name": "com.example.deeplink_cookbook",
   "sha256_cert_fingerprints":
   ["FF:2A:CF:7B:DD:CC:F1:03:3E:E8:B2:27:7C:A2:E3:3C:DE:13:DB:AC:8E:EB:3A:B9:72:A1:0E:26:8A:F5:EC:AF"]
   }
   }]
   package_name
   build.gradle파일에 아래와 같은 형태로 패키지 네임이 있다.
   android {
   namespace "com.exapmle"
   sha256_cert_fingerprints
   공식 문서에 따르면 Release>Setup>App Integrity>App Signing 탭에서 찾을 수 있다.
   엄청 생소해보이지만 그냥 연결해주고 싶은 앱과 디바이스에 설치된 앱이 같은 앱인지 확인하기 위한 지문이라고 생각하면 된다.

🚨 Google Play 앱 서명 키와 업로드 키는 다르니 꼭 서명키로 확인해야 한다.
🚨 서명키가 없다면 아래 명령어를 통해 확인할 수 있다.

adb shell pm get-app-links com.example.app

3. 테스트 해보기
   잊지 말고 웹 프로젝트도 push 를 해주고! 앱 프로젝트는 저장하고, 아래 명령어로 애뮬레이터에서 테스트를 해보자.

adb shell 'am start -a android.intent.action.VIEW \
-c android.intent.category.BROWSABLE \
-d "http://<web-domain>/details"' \
<package name>
앱 미설치 시에 우리가 임시로 만든 웹페이지로 잘 떨어지고, 앱 설치시에 앱이 잘 열린다면 성공이다:)

실기기에서 테스트할 때는 안드로이드는 메모장 앱이 따로 없어서 문자 메시지에 링크를 넣어 테스트를 했다.


💥 assetlinks.json이란?
이 파일은 만약 유저의 디바이스에 앱이 있다면 브라우저 대신 어떤 안드로이드 앱을 열 것인지 유저의 브라우저에 알려주는 역할을 한다.

유저가 딥링크 클릭
안드로이드 운영체제는 해당 도메인에 assetlinks.json 파일이 있는지 확인
이 파일이 존재하고 그 안에 포함된 앱의 SHA-256 지문과 설치된 앱의 지문이 일치하면
이 앱이 해당 도메인과 연결된 신뢰할 수 있는 앱이라고 판단!
그래서 지문이 맞지 않으면 App Link가 실행되지 않는다.

앱이 설치된 경우: 도메인과 앱 간의 신뢰 관계가 설정되었기 때문에, 사용자가 링크를 클릭하면 Android는 해당 링크를 처리하기 위해 앱을 연다.
앱이 설치되지 않은 경우: 신뢰 관계가 없다면, Android는 웹 브라우저에서 해당 링크를 열도록 처리한다.

[ios deep link]
Android App Link 적용에 이어 이번엔 iOS Univesal Link를 적용해보겠습니다:)

딥링크에 대한 이해가 먼저 필요하다면 이 블로그를 먼저 확인하세요!
👉 App Links 적용기 (Deep Link란?)

먼저 xcode를 열어 환경 설정을 해줍니다.

1. FlutterDeepLinkingEnabled 키 생성


1) Xcode -> Runner → info 로 이동하여 Information Property List를 클릭해 + 버튼을 눌러 새로운 값을 추가합니다.
2) FlutterDeepLinkingEnabled 키를 만들고, Boolean 으로 설정한 후 Yes로 업데이트 합니다.


2. Associated Domains 추가
   이번엔 앱과 웹을 연결해줍니다.

1) Ruuner → Signing & Capabilities로 이동하여 Associated Domains를 클릭합니다.
2) +를 누르고 applinks: web domain.web domain 형식에 맞춰 도메인을 넣어줍니다.
   (https:// 는 넣지 않도록 주의합니다!)



3. apple-app-site-association 파일 생성 및 호스팅
   App Link를 적용할 때 applinks.json 파일을 생성했던 것과 마찬가지로 Universal Link에는 apple-app-site-assoiciation 파일을 생성해야 합니다.

AASA 파일이라고 불리는 이 파일은 도메인과 앱 간의 신뢰를 설정하는 데 사용됩니다. 파일이 올바르게 설정되어 있어야 iOS가 해당 도메인의 링크를 앱으로 전달하게 되는거에요!

{
"applinks": {
"apps": [],
"details": [
{
"appID": "S8QB4VV633.com.example.deeplinkCookbook",
"paths": ["*"]
}
]
}
}
앱 프로젝트가 아닌 웹 프로젝트에 생성해주어야 해요.

appID는 team id.bundle id 로 구성되어 있어요.
bundle id는 xcode에서, team id는 개발자 계정에서 찾을 수 있습니다.
문제 없이 잘 만들어졌다면 직접 경로를 아래처럼 생성해 브라우저에 입력했을 때 파일 다운로드가 이루어집니다.
https://example.com/.well-known/apple-app-site-association
만약 웹에 접근할 수 없거나 404 에러가 뜬다면 리포지토리 루트에 .nojekyll이라는 파일을 만들고, 그 안에 아무 내용도 넣지 않고 커밋 및 푸시를 하면 됩니다. (이런 설정을 하면 특수문자로 시작하는 폴더명의 접근이 가능하게 해줄거에요)


4. 테스트하기
   시뮬레이터에서 테스트할 때는 아래 명령어를 실행하면 테스트가 가능합니다.

xcrun simctl openurl booted https://<web domain>/details
실기기에서 테스트 할때는 메모 앱이 없어서 일정알림 앱을 활용해서 링크를 생성해 테스트를 했어요.
앱 설치 상태에서는 앱이, 앱 미설치 상태에서는 웹으로 이동한다면 잘 되는거에요:)

💁🏻‍♀️ 에러 기록
저는 계속 웹만 실행되고 앱이 켜지지 않았어요.. AASA 파일에 문제가 있다고 생각되어 아래처럼 여러가지 방법으로 원인을 찾아보았어요.

1. AASA 파일 validation 체크하기
   https://branch.io/resources/aasa-validator

내가 생성한 AASA 파일에 문제가 없는지 체크를 할 수 있는 사이트에요. 만약 문제가 있다면 아래 이미지처럼 어떤 부분에 문제가 있는지 확인할 수 있어요.


서버에서 400 이상을 반환하는 것은 웹 서버가 AASA 파일을 제대로 인식하지 못하기 떄문이에요!
웹 서버가 자동으로 AASA 파일의 Content-Type을 application/json으로 인식할 수 있도록 헤더 설정이 필요합니다.

.htaccess 파일을 웹 프로젝트 루트 디렉토리 바로 아래에 생성하고, 아래 내용을 넣어 push를 해줍니다.

<FilesMatch "apple-app-site-association">
ForceType application/json
</FilesMatch>
그리고 다시 확인해보면 이제 AASA 파일의 문제는 사라집니다. 명령어를 통해서도 체크가 가능해요! (200ok가 나오면 문제 해결!)


curl --http1.1 -i https://example.com/.wellknown/apple-app-site-association



2. AASA 파일과 도메인 매칭 여부 확인해보기
   혹시 그래도 에러가 나는 분들은 아래 명령어를 통해 내가 생성한 AASA 파일의 데이터가 도메인과 잘 맞는지 확인해 볼 수도 있어요.

sudo swcutil verify -d angela-jung.github.io -j ./example.json -u https://angela-jung.github.io
매칭이 잘 된 상태라면 아래와 같은 응답을 받을 수 있습니다.

{ s = applinks, a = CN47329R73.com.daysenter.ios.daysnovel, d = angela-jung.github.io }: Pattern "https://angela-jung.github.io" matched.

3. 다시 한번 설정 살펴보기..
   마지막으로 저는 Associated Domains에 추가할 때 실수로 https를 넣은 걸 확인했고.. 수정을 하고 나니 잘 실행되었습니다!