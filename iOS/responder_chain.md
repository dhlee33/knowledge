Responder Chain
===

## UIResponder
responder chain이란 이벤트를 처리할 수 있는 응답자 객체들이 연결된 chain 이다. responder chain은 앱의 구조에 따라 동적으로 형성된다.

iOS 앱은 응답자 객체를 이용해 이벤트를 받고 처리한다. 응답자 객체인 `UIResponder`의 서브 클래스로는 `UIView`, `UIViewController`, `UIApplication`등이 있다.

응답자 객체는 이벤트 데이터를 받으면 반드시 처리하거나 다른 객체에게 넘긴다. 앱에서 이벤트가 발생하면 앱은 가장 적절한 응답자 객체에 이를 넘기고, 그 객체를 *first responder* 라고 한다.

first responder는 이벤트를 처리하거나, responder chain을 따라 이벤트를 전달한다.
chain의 마지막까지 처리되지 않은 이벤트는 버려진다.

이벤트별 타입과 first responder는 다음과 같다.

Event Type | First Responder
---- | ----
Touch Events | Touch가 발생한 뷰
Press Events | Press가 발생한 뷰
Shake-motion Events	| 사용자 혹은 UIKit이 지정한 객체
Remote-control Events |	사용자 혹은 UIKit이 지정한 객체
Editing Menu Messages |	사용자 혹은 UIKit이 지정한 객체

## UIEvent
이벤트는 UIEvent 객체 형태로 응답자 객체에 전달된다. 이벤트를 처리하기 위해서는
```swift
func touchesBegan(Set<UITouch>, with: UIEvent?)
```
과 같은 매소드를 정의해주면 된다.

이벤트 객체는 최초 생성 후에 재사용된다. 즉, 처음 만들어진 이벤트 객체가 다음번에 발생한 이벤트를 전달하는 데에도 쓰인다. 이벤트가 연속적이지 않아도 해당한다.

## Reference
https://baked-corn.tistory.com/129
