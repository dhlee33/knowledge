Service Worker
==============
# 1. 서비스 워커란?
## 1.1 소개
- 브라우저가 백그라운드에서 실행하는 스크립트
- 웹페이지와는 별개로 작동
- 웹페이지 또는 사용자 상호작용이 필요하지 않은 기능 지원 가능 
- 푸쉬 알림 및 백그라운드 동기화 등에 사용됨

## 1.2 유의 사항
- 서비스 워커는 자바스크립트 워커이므로 DOM에 직접 액세스 할 수 없다. 대신에 postMessage 인터페이스를 통해 전달된 메시지에 응답하는 방식으로 제어 대상 페이지와 통신할 수 있으며, 해당 페이지는 필요한 경우 DOM 조작 가능
- 서비스 워커는 프로그래밍 가능한 네트워크 프록시이며, 페이지의 네트워크 요청 처리 방법을 제어할 수 있다.
- 사용하지 않을 때는 종료되고 필요할때 재시작하므로 서비스워커의 `onfetch` 및 `onmessage` 핸들러의 전역 상태에 의존할 수 없다. 재시작시 다시 사용해야할 정보가 있으면 IndexedDB API 에 대한 액세스 권한을 가진다.
- 서비스 워커는 Promise를 광범위하게 사용함

# 2. 서비스워커 수명 주기
## 2.1 설치
- 페이지에서 자바스크립트를 이용하여 등록히면 브라우저가 백그라운드에서 서비스 워커 설치 단계를 시작함
- 설치 단계 동안 정적 자산을 캐시함
- 설치 완료되면 활성화 단계가 진행됨
## 2.2 수명주기
![Alt text](https://developers.google.com/web/fundamentals/primers/service-workers/images/sw-lifecycle.png)

# 3. 사전 요구사항
## 3.1 브라우저
- Chrome, Firefox, Opera 지원함
- Microsoft Edge, Safari는 향후 개발 예정
## 3.2 Https
- 배포하려면 서버에 HTTPS 설정 해야함

# 4. 등록
```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      // Registration was successful
      console.log('ServiceWorker registration successful with scope: ', registration.scope);
    }, function(err) {
      // registration failed :(
      console.log('ServiceWorker registration failed: ', err);
    });
  });
}
```
 -  `/sw.js` 에 있는 서비스 워커를 등록함
 - 서비스 워커 위치가 중요함. 워커 위치보다 같거나 하위인 위치의 url페이지에 대해서만 `fetch` 이벤트 처리함

# 5. 설치
```javascript
var CACHE_NAME = 'my-site-cache-v1';
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

self.addEventListener('install', function(event) {
  // Perform install steps
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
```
`install` 콜백 안에서 다음 절차를 수행해야함

1. 원하는 이름으로 캐시를 연다
2. 파일을 캐시한다
3. 필요한 모든 자산이 캐시되었는지 확인

- 모든 파일 캐시 성공하면 설치됨
- 하나라도 실패하면 실패함

# 6. 요청 캐시 및 반환
```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
```

# 7. 업데이트
## 7.1 단계
1. 파일 업데이트하면 브라우저가 백그라운드에서 다시 다운로드함
2. 새 워커가 시작되고 install 이벤트 생성됨
3. 이때 이전 서비스 워커가 아직 현재 페이지 제어하기 때문에 새 워커는 `wating` 상태가 됨
4. 현재 사이트가 닫히면 이전 워커 종료되고 새 워커가 제어권을 갖게됨
5. 제어권을 가지면 `activate` 이벤트가 발생함

## 7.2 캐시 관리
- 설치 단계에서 이전 캐시들을 삭제하면 이전 워커가 망가지므로 `activate` 콜백 에서 캐시 관리를 해야함
```javascript
self.addEventListener('activate', function(event) {

  var cacheWhitelist = ['pages-cache-v1', 'blog-posts-cache-v1'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

# Reference
<https://developers.google.com/web/fundamentals/primers/service-workers/>