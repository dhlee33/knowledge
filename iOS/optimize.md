iOS 최적화
===
[애플 공식 문서](https://github.com/apple/swift/blob/master/docs/OptimizationTips.rst#writing-high-performance-swift-code) 요약

## Build Setting
Build Setting 에서 최적화 레벨, compilation mode등 설정 가능.
기본값 써도 됨

## Dynamic Dispatch
클레스는 메소드 및 프로퍼티 접근에 동적 디스페치를 사용한다. 상속 가능하기 때문에 어떤 것을 호출하는지 컴파일러가 모르기 때문아다. 간접호출에 드는 오버헤드 + 컴파일러 최적화 못함 때문에 직접호출보다 느려진다.

막기 위해 `final`, `private`, `fileprivate` 키워드를 잘 사용하자.

## Container Type
Array에서는 값타입을 사용하자. 참조타입을 사용하면 어레이 내부에 참조와 관련된 트래픽을 추가로 들고 있어야 한다.
참조타입이 필요하고 NSArray로 브릿징이 필요 없는 경우에는 기본 Array 대신에 `ContinuousArray`를 쓰면 된다.

## Unchecked operations
Swift는 정수 연산시에 overflow를 자동으로 해결한다. 하지만 overflow가 발생할 일이 없는 코드에서 쓸데없이 overflow를 체크하느라 오버헤드가 일어난다. 그럴때는 `+` 대신에 `&+` 사용하면 된다.

## Protocols
프로토콜을 클레스만 사용한다면, 해당 프로토콜에 명시하자. 런타임에 해당 프로토콜이 클레스인지 구조체인지 몰라서 두개 다 다응하는 오버헤드 없어진다.

```swift
protocol Example: AnyObject {}
```