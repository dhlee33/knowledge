Deep Link
==
딥 링크는 단순히 개념일 뿐이다. 특정 콘텐츠에 직접 도달하는 링크를 뜻하며, 우리가 사용하는 대부분 웹 링크 또한 사실 딥 링크 이지만 단지 그렇게 부르지 않을 뿐이다. 대신에 주로 사용자가 앱 내부 콘텐츠에 직접 도달하도록 하는 링크를 의미한다.

# URI Scheme

## 소개
URI(Uniform Resource Identifier)는 URL(Uniform Resource Locator)를 포함하는 개념이다. URI scheme에는 http://와 https://, ftp:// 혹은 feed:// 등이 있다. 모바일 앱은 자신만의 scheme을 myapp:// 과 같이 등록할 수 있다. 

## 한계
URI Scheme은 다음과 같은 상황에 한계를 가진다.
- 앱이 설치되지 않은 경우.
- 두 개 이상의 앱이 myapp://에 응답하려 하는 경우.

많은 개발자들이 위의 문제를 자바스크립트 redirect를 사용하여 첫 번째 문제를 해결했지만, 새로운 iOS가 출시될 때 마다 제대로 작동하지 않는 경우가 있었다.

## 대안
이러한 제한으로 인해 애플은 공식적으로 iOS 9.2 버전부터 딥 링크를 위한 URI 링크를 더는 지원하지 않으며 자바스크립트 리다이렉트를 차단하고 2세대 딥 링크 표준인 **유니버설 링크**를 발표했다. (안드로이드: 앱 링크)

하지만 현실적으로 아직은 URI Scheme 또한 같이 사용된다.

# Universal Link (App Link)

## 소개
유니버설 링크는 별개의 링크이기 보다는 유저가 앱으로 라우팅되는 방식을 제어하는 일부 링크에 적용되는 메커니즘이라고 볼 수 있다. 커스텀 스키마 형식의 딥링크와는 달리, 유니버셜 링크는 일반적으로 웹사이트 URL과 동일한 문자열을 사용한다. 

예를 들어 웹사이트 주소가 www.yoursite.com이라면, 앱으로 연결되는 유니버셜 링크 역시 동일한 www.yoursite.com을 지정할 수 있다. 결과적으로 앱이 유니버설 링크를 지원하고 설치된 상태라면 앱이 열리고, 아니라면 웹 사이트로 이동한다.

## 주의할 점
유니버설 링크를 지원하기 위해, 해당 도매인의 웹 서버와 앱에 각각 별도의 설정이 필요하다. 주의할 점은, 유니버설 링크를 이용할 때에는 사용자가 직접 앱을 실행하는 것이 아니라 외부 URI를 통해 호출되는 상황이므로, 해킹 등에 매우 취약해진다. 따라서 앱에서 특정 URI에만 동작하도록 설정할 필요가 있다.

## 트래킹
앱이 직접 실행되기 때문에, redirect 체인에 기반을 둔 전통적인 마케팅 추적 방식으로는 추적이 힘들다. *방법 업데이트 예정*

Reference
===
- https://www.appsflyer.com/kr/blog/universal-links-uri-schemes-tech-stack-apply-deep-linking/

- https://blog.branch.io/ko/%EC%9C%A0%EB%8B%88%EB%B2%84%EC%84%A4-%EB%A7%81%ED%81%AC-uri-%EC%8A%A4%ED%82%B4-%EC%95%B1-%EB%A7%81%ED%81%AC-%EB%B0%8F-%EB%94%A5-%EB%A7%81%ED%81%AC-%EB%AC%B4%EC%8A%A8-%EC%B0%A8%EC%9D%B4%EA%B0%80/

- https://www.letmecompile.com/universal-link-vs-custom-url-scheme/