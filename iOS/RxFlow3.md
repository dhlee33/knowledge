번역) RxFlow Part 3: Tips and Tricks
===
원문: https://twittemb.github.io/swift/coordinator/reactive/rxswift/reactive%20programming/rxflow/2017/12/22/rxflow-part-3-tips-and-tricks/
- [번역) RxFlow Part 1: In Theory](https://github.com/ydh1304/knowledge/blob/master/iOS/RxFlow1.md)
- [번역) RxFlow Part 2: In Practice](https://github.com/ydh1304/knowledge/blob/master/iOS/RxFlow2.md)

RxFlow의 마지막 여정 입니다. 나는 다음 두 가지 게시물에서 프레임 워크의 모든 주요 기능 / 원칙을 이미 공개했습니다.
- [RxFlow Part 1: In Theory](https://twittemb.github.io/swift/coordinator/rxswift/rxflow/reactive%20programming/2017/11/08/rxflow-part-1-in-theory/)
- [RxFlow Part 2: In Practice](https://twittemb.github.io/swift/coordinator/reactive/rxflow/reactive%20programming/2017/12/09/rxflow-part-2-in-practice/)

Reactive Programming을 다루는 몇 가지 팁과 트릭을 살펴 봅시다.

## UIViewController는 반응적으로 만들었습니다. ##

2부에서 보았듯이, 어떤 시점에서 **Presentable**이 언제 표시 되는지 아닌지를 반응적 방식으로 알 필요가 있었습니다. **Presentable**은 3개의 Observable를 노출합니다.
```swift
/// Observable that triggers a bool indicating if
/// the current Presentable is being displayed
var rxVisible: Observable<Bool> { get }

/// Single triggered when this presentable is displayed
/// for the first time
var rxFirstTimeVisible: Single<Void> { get }

/// Single triggered when this presentable is dismissed
var rxDismissed: Single<Void> { get }
```
**RxFlow**에서, UIViewController는 이 프로토콜을 준수하기 때문에 그들을 반응형으로 만들 수있는 방법을 찾아야 합니다.

훌륭한 프로잭트인 [RxViewController](https://github.com/devxoul/RxViewController)가 많은 도움이 되었기를 바랍니다.

그것은 [이 게시물](https://twittemb.github.io/swift/generics/reactive/rxswift/pattern/name%20space/2017/11/22/versatile-namespace/)에서 설명하는 패턴을 적용하여 UIViewControllers에 대해 Reactive 확장을 제공합니다. 또한 RxCocoa 내장 함수를 사용하여 selector 호출을 관찰 할 수 있습니다. 이 개념을 이해한 후에 직접 UIViewController의 extension을 만들었습니다.
```swift
extension Reactive where Base: UIViewController {

    /// Observable, triggered when the view has appeared for the first time
    public var firstTimeViewDidAppear: Single<Void> {
        return sentMessage(#selector(Base.viewDidAppear)).map { _ in
            return Void()
        }.take(1).asSingle()
    }

    /// Observable, triggered when the view is being dismissed
    public var dismissed: ControlEvent<Bool> {
        let source = sentMessage(#selector(Base.dismiss))
                     .map { $0.first as? Bool ?? false }
        return ControlEvent(events: source)
    }

    /// Observable, triggered when the view appearance state changes
    public var displayed: Observable<Bool> {
        let viewDidAppearObs = sentMessage(#selector(Base.viewDidAppear))
                               .map { _ in true }
        let viewWillDisappearObs = sentMessage(#selector(Base.viewWillDisappear))
                                   .map { _ in false }
        return Observable<Bool>.merge(viewDidAppearObs, viewWillDisappearObs)
    }
}
```

기록하자면, `nextPresentable`은 **Flow**의  `navigate(to :)` 함수에 의해 생성 된 **Presentable**이기 때문에 **Coordinator**가 사용하는 방법입니다. 연관된 **Presentable**의 맨 처음 디스플레이 이후에만 다음 Stepper를 청취합니다.
```swift
nextPresentable.rxFirstTimeVisible.subscribe(onSuccess: { [unowned self, unowned nextPresentable, unowned nextStepper] (_) in
    // we listen to the presentable's Stepper.
    // For each new Step value, we trigger a new navigation process
    // this is the core principle of the whole RxFlow mechanism
    // The process is paused each time the presentable is not currently displayed
    // for instance when another presentable is above it in the VCs hierarchy.
    nextStepper.steps
        .pausable(nextPresentable.rxVisible.startWith(true))
        .asDriver(onErrorJustReturn: NoStep())
        .drive(onNext: { [unowned self] (step) in
            // the nextPresentable's Stepper fires a new Step
            self.steps.onNext(step)
        }).disposed(by: nextPresentable.disposeBag)

}).disposed(by: self.disposeBag)
```
## 멈춥시다. ##
**RxFlow**에 또 다른 주요 원칙이 있습니다: **Flow**에서 일어난 일은 **Flow**안에 머무릅니다. 따라서 **Flow**가 더 이상 뷰 계층의 탑이 아닐 경우에 **Step**의 구독을 "멈출" 방법을 찾아야 했습니다.

RxSwift는 구독을 일시 중지하는 방법을 "즉시" 제공하지 않지만 [RxSwiftCommunity](https://github.com/RxSwiftCommunity)의 프로젝트인 [RxSwiftExt](https://github.com/RxSwiftCommunity/RxSwiftExt)를 사용할 수 있습니다. 이것은 RxSwift에 "[pausable](https://github.com/RxSwiftCommunity/RxSwiftExt#pausable)" 와 같이 많은 새로운 연산자를 추가합니다.

    두 번째 옵저버블 시퀀스의 최신 요소가 참이 아니면 소스 옵저버블 시퀀스의 요소를 일시 중지합니다.

구현을 봅시다.
```swift
extension ObservableType {

    /// Pauses the elements of the source observable sequence based on
    /// the latest element from the second observable sequence.
    /// Elements are ignored unless the second sequence has most recently
    /// emitted `true`.
    /// - Parameter pauser: The observable sequence used to pause the source
    /// observable sequence.
    /// - Returns: The observable sequence which is paused based upon
    /// the pauser observable sequence.
    public func pausable<P: ObservableType> ( _ pauser: P) -> Observable<E>
                                                              where P.E == Bool {
        return withLatestFrom(pauser) { element, paused in
            (element, paused)
            }.filter { _, paused in
                paused
            }.map { element, _ in
                element
        }
    }
}
```
실제로 이것은 3개의 RxSwift 내장 연산자를 조합 한 것입니다.

- withLatestFrom : 주 Observable에 의해 트리거 된 값과 "pauser"(멈춤을 유도하는 Observable)의 마지막 값을 결합합니다.
- filter: "pauser"의 값이 참인 것만 통과시킵니다.
- map: pauser Observable 값을 무시하므로 main Observable의 값만 반환합니다.

다음은 코디네이터가 이것을 이용하는 방법입니다.
```swift
nextStepper
    .steps
    .pausable(nextPresentable.rxVisible.startWith(true))
    .asDriver(onErrorJustReturn: NoStep())
    .drive(onNext: { [unowned self] (step) in
        // the nextPresentable's Stepper fires a new Step
        self.steps.onNext(step)
    }).disposed(by: nextPresentable.disposeBag)
```
매우 직관적입니다. "rxVisible" Observable 값이 false이면 nextStepper의 **Step**이 일시 중지됩니다.

## 저장된 프로퍼티를 가진 프로토콜? ##

RxFlow는 프로토콜 지향 프레임워크이기 때문에 개발자가 여러 프로토콜을 구현하기를 원합니다. 그런 종류의 프레임워크를 구축 할 때 사용자는 이러한 프로토콜을 수행하기 위해 구현해야하는 너무 많은 함수나 프로퍼티로 고생하고 싶지 않아합니다.

프로토콜 extension으로 기본 구현을 제공 할 수 있기 때문에 함수는 문제가 되지 않습니다. 그러나 Swift는 프로퍼티를 extension에 저장할 수 없기 때문에 문제가 됩니다.

예를 들어, **Stepper** 프로토콜을 구현하면 새 **Step**값 을 트리거 할 수 있는 `step` 프로퍼티가 제공 됩니다. 어떻게 했을까요?

다시 한번 RxSwiftCommunity는 큰 도움이 되었습니다. [NSObject-Rx](https://github.com/RxSwiftCommunity/NSObject-Rx)에서 영감을 받았습니다. 이 프로젝트는 RxSwift DisposeBag를 저장하는 NSObject의 extension을 제안합니다. 목표는 NSObject, 특히 UIViewControllers를 확장하는 모든 클래스에 기본 DisposeBag를 제공하는 것입니다. 정확히 내가 필요한 것이지만 프로토콜 extension에 있었습니다. 다음은 Stepper의 코드입니다.

```swift
private var subjectContext: UInt8 = 0

/// a Stepper has only one purpose: emit Steps that correspond to
/// specific navigation states.
/// The state changes lead to navigation actions in the context of
/// a specific Flow
public protocol Stepper: Synchronizable {

    /// the Rx Obsersable that will trigger new Steps
    var steps: Observable<Step> { get }
}

public extension Stepper {

    /// The step in which to publish new Steps
    public var step: BehaviorSubject<Step> {
        return self.synchronized {
            if let subject = objc_getAssociatedObject(self, &subjectContext)
                             as? BehaviorSubject<Step> {
                return subject
            }
            let newSubject = BehaviorSubject<Step>(value: NoStep())
            objc_setAssociatedObject(self,
                                     &subjectContext,
                                     newSubject,
                                     .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
            return newSubject
        }
    }

    /// the Rx Obsersable that will trigger new Steps
    public var steps: Observable<Step> {
        return self.step.asObservable()
    }
}
```
모든 마법은 `step` 컴퓨티드 프로퍼티에서 발생 합니다. "objc_setAssociatedObject"함수를 사용하여 BehaviorSuject에 대한 참조를 저장합니다([이 NSHipster 기사](https://nshipster.com/associated-objects/) 참조). 이 속성에 액세스 할 때마다이 저장된 참조가 검색됩니다 (첫 번째 호출에서는 BehaviorSubject가 만들어지고 subjectContext 참조와 연결됨).

이 트릭에는 단점이 있습니다. 프로토콜은 Struct와 같은 값 타입으로 채택 될 수 있습니다. 즉, 메모리는 힙(참조 유형)이 아닌 스택에서 처리됩니다. 따라서 Struct 인스턴스의 라이프 사이클 및 재사용 가능성은 Swift 런타임에 의해 처리됩니다. 런타임에서 인스턴스를 재사용 할 때 "objc_getAssociatedObject"관련 값이 어떻게되는지 확실하지 않습니다. 안전을 위해, 이런 종류의 프로토콜은 클래스에 의해서만 구현되도록 제약되어야하며, 이것은 모든 것이 힙에서 발생하도록 보장합니다.

### 커뮤니티에 돌려주세요 ###
RxFlow의 주요 기능은 개발자 커뮤니티에서 수행한 작업을 기반으로 합니다. 이것은 당신이 자신의 프로젝트 소스를 열 때 고려해야 하는 것입니다: 당신은 도움이 필요할 것입니다! 나는 커뮤니티에 돌려주는 것이 중요하다고 생각합니다.

RxFlow의 경우 병합 된 2개의 PR을 열 기회가 있었습니다:
- [Rehabilitates the HasDisposeBag protocol](https://github.com/RxSwiftCommunity/NSObject-Rx/pull/49)
- [Add new observables for displayed and dismissed states](https://github.com/devxoul/RxViewController/pull/4)

내 코드가 다른 개발자를 도울 수 있다는 것을 아는 것이 정말 좋았습니다.

## 결론 ##
첫 번째 오픈 소스 프로젝트를 사용할 수 있게 만드는 것은 상당히 어려웠습니다. 다음과 같은 이유 때문에 생각처럼 쉽지 않았습니다.

- 프로젝트로 인도한 모든 아이디어를 모으고 종합하세요(예전 프로젝트의 아이디어, 발생한 문제 및 해결책 등). 코딩을 하기전에 진정하고 그것에 대해 생각해보세요 :-)
- 프로젝트의 복잡성에 따라 적절한 패턴을 고르려고 노력하고, 과도하게 작업하지 마세요.
- 당신이 당신의 코드를 사용할 사람인 것처럼 생각하고 가능한 한 간단하게 하세요(이것이 가장 어려운 일입니다).
- 코드만으로는 프로젝트를 매력적으로 만들기에 충분하지 않기 때문에 좋은 README를 작성하십시오.
- 소스 관리에 프로같이 행동하세요. 아무도 못생겼다고 생각하는 프로젝트에 기여하고 싶지 않습니다 (git CLI는 가장 친한 친구입니다).
- 블로그 기사로 공유하면 똑똑한 사람들로부터 피드백을 얻을 수 있습니다.
- 신념을 지키면 때때로 낙담하게됩니다. 압도 당하면 잠시 휴식을 취하고 뇌를 쉬게 하세요. 아이디어는 나중에 다시 나옵니다.

RxFlow는 CocoaPods와 Cartage의 1.0.1버전으로 출시되었습니다. 나는 이제 다음 몇 가지 게시물에서 내가 이야기 할 새 프로젝트에서 이것을 사용할 것입니다!

RxFlow Github 레포: https://github.com/RxSwiftCommunity/RxFlow

계속 지켜보세요.