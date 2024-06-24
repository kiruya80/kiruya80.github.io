# flutter_inappwebview

나중에 빌드는
flutter build web --web-renderer html --release
또는
flutter build web --release --web-renderer html

Android Studio 에서는 상단에 Chrome(web)을 선택한 상태에서 main.dart 항목에 Edit Configuration을 클릭한 다음
Additional run args 에 --web-render html 을 추가한 다음 apply 또는 OK 버튼을 누르면 된다.
 
flutter build web --web-renderer html
flutter build web --web-renderer canvaskit

=====
https://github.com/pichillilorenzo/flutter_inappwebview/issues/1479
https://inappwebview.dev/docs/intro/#setup-web

flutter_inappwebview 사용시 에러
 > TypeError: Cannot set properties of undefined (setting 'nativeCommunication')
index.html 헤더에 아래 추가
<head>
  <!-- ... -->
    <script type="application/javascript" src="/assets/packages/flutter_inappwebview_web/assets/web/web_support.js" defer></script>
  <!-- ... -->
</head>
=====
