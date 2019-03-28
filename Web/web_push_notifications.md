Web Push Notifications
========

# 1. Web Push Notification 란?
- 서비스 워커 기반
- 사이트 실시간 업데이트
- Push 는 서버가 서비스 워커에 정보 줄때 발생함
- Notification은 서비스 워커가 유저에게 정보 보여주는 것

#  2. Push 동작
## 2.1 Client Side
- 유저를 push messaging에 subscribe 해야함
- Subscribe 하려면 유저 동의 후 브라우저에서 `PushSubscription` 얻어야함
- `PushSubscription`은 유저에게 푸쉬 보내기 위한 모든 정보를 가지고 있다. 유저 기기의 ID와 비슷함
- `PushSubscription`을 백엔드 서버에 보내야함. 서버에서 이것을 저장하고 푸쉬할 때 사용함
![Alt_text](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/browser-to-server.svg)

## 2.2 푸쉬 메세지 보내기
- 푸쉬 서비스는 네트워크 request를 받고 인증 한 후 적절한 브라우저에 전달한다. 브라우저가 오프라인이면 메세지는 큐에 저장되고 온라인일 때 전달된다.
- 개발자는 각 브라우저의 푸쉬 서비스 종류를 컨트롤 할 수 없지만, 모든 푸쉬 서비스는 같은 API call을 받기 때문에 어떤 푸쉬 서비스인지 고려할 필요 없다.
- `PushSubscription`에 있는 `endpoint` 로 보낸다.

`PushSubscription` example:
```json
{
  "endpoint": "https://random-push-service.com/some-kind-of-unique-id-1234/v2/",
  "keys": {
    "p256dh" :
"BNcRdreALRFXTkOOUHK1EtK2wtaz5Ry4YfYCA_0QTpQtUbVlUls0VJXg7A8u-Ts1XbjhazAkj7I99e8QcYP7DkM=",
    "auth"   : "tBHItJI5svbpez7KI4CCXg=="
  }
}
```
- 푸쉬 메세지 보내면, 푸쉬 서비스는 API call을 받고 메세지를 queue에 저장함. 메세지는 유저의 기기가 온라인되기 전 또는 만료 전까지 저장됨

![Alt_text](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/server-to-push-service.svg)

## 2.3  Web Push Protocol
- 푸쉬 API에 관한 IETF standard
- 특정 해더 필요, data 는 stream of bytes

# 3. 유저 디바이스
- 푸쉬서비스가 메세지를 전달하면, 브라우저가 받고 데이터를 해독한 후 `push` 이벤트를 서비스 워커에게 보낸다
- 서비스워커는 사이트나 브라우저가 안열린 상황에서도 동작 가능
![Alt_text](https://developers.google.com/web/fundamentals/push-notifications/images/svgs/push-service-to-sw-event.svg)

# Reference
<https://developers.google.com/web/fundamentals/push-notifications/>
