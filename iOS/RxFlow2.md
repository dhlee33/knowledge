번역) RxFlow Part 2: In Practice
===
원문: https://twittemb.github.io/swift/coordinator/reactive/rxflow/reactive%20programming/2017/12/09/rxflow-part-2-in-practice/
![RxFlow](https://twittemb.github.io/uploads/RxFlowPart2.png)

몇 주 전에 RxFlow 라는 iOS 프레임 워크를이 블로그에 소개했습니다. 나는 이 프레임 워크를 몇 달 동안 사용해 왔으며 이제 사용할 준비가 되었습니다. 아직 읽지 않았다면 [이 게시물](https://twittemb.github.io/swift/coordinator/rxswift/rxflow/reactive%20programming/2017/11/08/rxflow-part-1-in-theory/)을 살펴 보시기 바랍니다.

요약하면, **RxFlow**는 다음을 목표로 합니다:
- 논리적 섹션으로 네비게이션을 쉽게 할 수 있습니다.
- 뷰컨트롤러에서 네비게이션 코드를 제거합니다.
- 뷰컨트롤러의 재사용을 장려합니다.
- 반응형 프로그래밍 촉진합니다.
- 의존성 주입을 촉진합니다.

용어에 대한 간단한 리마인드:

- **Flow**: 각 Flow는 앱 내의 네비게이션 영역을 정의합니다.
- **Step**: 각 Step는 애플리케이션의 네비게이션 상태입니다. Flow와 Step의 결합은 모든 가능한 네비게이션 동작을 설명합니다.
- **Stepper**: Step을 방출 할 수 있는 모든 것이 될 수 있습니다. Stepper는 Flow 내의 모든 네비게이션 동작을 트리거 할 수 있습니다.
- **Presantable**: 프리젠트 할 수 있는 abstraction입니다. 기본적으로 UIViewController와 Flow는 프리젠트 가능합니다.
- **NextFlowItem**: Coordinator에게 리액티브 메커니즘에 새로운 Step을 만들어 줄 다음 요소가 무엇인지 알려줍니다.
- **Coordinator**: Coordinator의 임무는 Flow와 Step의 결합을 일관된 방식으로 혼합하는 것입니다.

RxFlow는 상속 계층 구조에 코드를 동결하지 않도록 프로토콜 지향 프로그래밍을 사용한다는 것을 명심하는 것도 중요합니다.

[RxFlow repo](https://github.com/RxSwiftCommunity/RxFlow)에서 모든 가능한 네비게이션 타입을 보여주는 데모 앱을 볼 수 있습니다.
![Demo app](https://twittemb.github.io/uploads/versions/demoweavy-mov---x----185-400x---.gif)

## 모든 것이 상태에 관한 것입니다. ##
RxFlow는 주로 반응적인 방식으로 네비게이션 상태 변경을 처리하는 데 사용됩니다. 여러 상황에서 재사용하려면 이러한 상태가 사용자의 현재 네비게이션 Flow를 인식하지 않아야합니다. 따라서 상태는 "이 화면으로 이동하고 싶다"가 아니라 "누군가 또는 어떤 것이 이 동작을 했다"를 의미하고, **RxFlow**는 현재 네비게이션 Flow에 따라 올바른 화면을 선택합니다. **RxFlow**를 사용하면 이 네비게이션 상태를 **Step**이 라고 부릅니다 .

Enum은 **Step**을 설명하는 좋은 방법입니다.

- Enum은 사용하기 쉽고,
- 값은 한 번만 정의 할 수 있습니다(따라서 상태는 고유합니다).
- Swift는 switch문에 가능한 모든 값을 구현하기를 원하기 때문에 사용하기에 안전합니다.
- 한 화면에서 다른 화면으로 가져올 값을 내장 할 수 있습니다.
- 값 타입이기 때문에 참조 공유와 통제되지 않은 전달이 없습니다.

예를들어, 데모 앱에는 네비게이션 가능성을 커버하기 위한 다음과 같은 **Step**이 있습니다.
```swift
import RxFlow

enum DemoStep: Step {
    case apiKey
    case apiKeyIsComplete

    case movieList

    case moviePicked (withMovieId: Int)
    case castPicked (withCastId: Int)

    case settings
    case settingsDone
    case about
}
```

## Flow와 함께 가세요. ##
**RxFlow**를 사용하면 뷰컨트롤러를 프리젠트하거나 푸시하는 등의 모든 탐색 코드가 Flow에 선언됩니다. Flow는 앱의 논리적 네비게이션 섹션을 나타내며, 특정 Step에 결합 될 때, 일부 네비게이션 액션을 트리거합니다.

이것을 위해 Flow는 다음을 구현해야 합니다.
- Flow와 Step에 따라 네비게이션 동작을 수행 하는 `navigate(to:)` 함수
- 이 **Flow**의 네비게이션을 기반으로 할  "**root**" 뷰컨트롤러

아래는 UINavigationController와 네비게이션 스택을 다루는 **Flow**예시 입니다. 이 **Flow**에서, 세가지의 네비게이션 동작이 가능합니다.
```swift
import RxFlow
import UIKit

class WatchedFlow: Flow {

    var root: UIViewController {
        return self.rootViewController
    }

    private let rootViewController = UINavigationController()
    private let service: MoviesService

    init(withService service: MoviesService) {
        self.service = service
    }

    func navigate(to step: Step) -> [NextFlowItem] {
        guard let step = step as? DemoStep else { return NextFlowItem.noNavigation }

        switch step {

        case .movieList:
            return navigateToMovieListScreen()
        case .moviePicked(let movieId):
            return navigateToMovieDetailScreen(with: movieId)
        case .castPicked(let castId):
            return navigateToCastDetailScreen(with: castId)
        default:
            return NextFlowItem.noNavigation
        }
    }

    private func navigateToMovieListScreen () -> [NextFlowItem] {
        let viewModel = WatchedViewModel(with: self.service)
        let viewController = WatchedViewController.instantiate(with: viewModel)
        viewController.title = "Watched"
        self.rootViewController.pushViewController(viewController, animated: true)
        return [NextFlowItem(nextPresentable: viewController, nextStepper: viewModel)]
    }

    private func navigateToMovieDetailScreen (with movieId: Int) -> [NextFlowItem] {
        let viewModel = MovieDetailViewModel(withService: self.service,
                                             andMovieId: movieId)
        let viewController = MovieDetailViewController.instantiate(with: viewModel)
        viewController.title = viewModel.title
        self.rootViewController.pushViewController(viewController, animated: true)
        return [NextFlowItem(nextPresentable: viewController, nextStepper: viewModel)]
    }

    private func navigateToCastDetailScreen (with castId: Int) -> [NextFlowItem] {
        let viewModel = CastDetailViewModel(withService: self.service,
                                            andCastId: castId)
        let viewController = CastDetailViewController.instantiate(with: viewModel)
        viewController.title = viewModel.name
        self.rootViewController.pushViewController(viewController, animated: true)
        return NextFlowItem.noNavigation
    }
}
```
## 네비게이션은 사이드 이팩트 입니다. ##

Functional Reactive Programming을 배울 때 종종 **사이드 이팩트**에 대해 읽습니다. FRP의 목적은 이벤트를 전파하고 그 과정에서 모든 함수를 적용하는 것입니다. 이 함수는 이러한 이벤트를 변형시킬 수 있으며 결국에는(반드시 그런 것은 아니지만) 원하는 기능을 수행 할 코드를 실행합니다(네트워킹, 파일 저장, 경고 표시, ...). 이것들은 **사이드 이팩트**입니다.

**RxFlow**가 리액티브 프로그래밍에 의존하기 때문에, 고유한 개념을 쉽게 식별할 수 있습니다.
- **이벤트**: 방출된 Step입니다.
- **함수**: `navigate(to:)` 함수 입니다.
- **변형**: `navigate(to:)` 함수는 **Step**을 **NextFlowItem**으로 변형합니다.
- **사이드 이팩트**: `navigate(to:)` 안에서 수행되는 네비게이션 액션입니다. (예를들어, `navigateToMovieListScreen()`함수는 새로운 뷰컨트롤러를 네비게이션 스택에 푸쉬합니다.)

## 네비게이션은 NextFlowItem을 만드는 것으로 구성됩니다. ##

기본적으로 **NextFlowItem**은 **Presentable**및 **Stepper**를 보유하는 간단한 데이터 구조입니.

**Presentable**는 **Coordinator**에게 다음에 프리젠트 할 것을 알려주고, **Stepper**는 **Coordinator**에게 **Step**을 방출하기 위해 뭘 해아하는지 알려줍니다.

기본적으로 모든 종류의 뷰컨트롤러가 **Presentable**입니다. 어떤 시점에서 당신이 고유한 **Flow**에 묘사된 완전히 새로운 네비게이션 영역을 시작하고 싶을 것이기 때문에 **Flow** 또한 **Presentable**입니다. 그래서 RxFlow가 그것을 프리젠트 될 수 있는 것으로 판단합니다.
**Coordinator**가 왜 **Presentable**에 대해 알고 있어야 합니까?

**Presentable** 은 프리젠트 할 수 있는 무언가의 추상화입니다. 관련된 **Presentable** 이 display되지 않으면 **Step**을 방출할 수 없기 때문에 **Presentable** 은 **Coordinator**가 구독 할 Reactive observables를 제공 합니다(따라서 **Coordinator**는 **Presentable**의 프리젠테이션 상태를 알고 있습니다). 떄문에 **Presentable**이 아직 완전히 표시되지 않은 상태에서 Step 을 실행 하는 위험 은 없습니다.

**Stepper**는 뷰컨트롤러, 뷰모델, 프리젠터등 어떤 것이든 될 수 있습니다. **Stepper**가 **Coordinator**에 등록되면 RxSwift 서브젝트인 **step** 프로퍼티를 통해 **Step**을 방출 할 수 있습니다. **Coordinator**는 이러한 **Step**을 듣고 **Flow**의 `navigate(to:)` 함수를 호출합니다.

데모 앱의 **Stepper** 예시 입니다.
```swift
import RxFlow
import RxSwift

class WatchedViewModel: Stepper {

    let movies: [MovieViewModel]

    init(with service: MoviesService) {
        // we can do some data refactoring in order to display
        // things exactly the way we want (this is the aim of a ViewModel)
        self.movies = service.watchedMovies().map({ (movie) -> MovieViewModel in
            return MovieViewModel(id: movie.id,
                                  title: movie.title,
                                  image: movie.image)
        })
    }

    public func pick (movieId: Int) {
        self.step.onNext(DemoStep.moviePicked(withMovieId: movieId))
    }
}
```
이 예시에서, 유저가 리스트 안의 무비를 선택할 때 `pick`함수가 호출됩니다. 그 함수는 Rx 스트림으로 `self.step`안에 새 값을 방출합니다.

네비게이션 프로세스를 정리하면 다음과 같습니다.
- **Step**을 인자로 갖는 `navigate(to:)` 함수가 호출됩니다.
- 이 **Step**에 따라 사이드 이팩트로 어떤 네비게이션 코드가 호출됩니다.
- 또한 이 **Step**에 따라, **NextFlowItem**이 생산됩니다. 따라서 **Presantable**과 **Stepper**가 *Coordinator**에 등록됩니다.
- **Stepper**는 새로운 **Step**을 뿜고 반복됩니다.

### Flow와 Step의 조합에 대해 여러 NextFlowItem을 생성하는 것이 왜 괜찮습니까?
###

어플리케이션이 한 번에 여러 네비게이션을 수행하는 것을 금지하지 않기 때문입니다. 예를 들어, 탭 바의 각 항목은 네비게이션 스택으로 이어질 수 있습니다. UITabBarController의 디스플레이를 트리거하는 **Step**이 네비게이션 스택당 하나의 **NextFlowItem**이 될 수 있습니다.

데모 앱을 보고 이 컨셉을 이해할 수 있습니다. UITabBarController를 2 **Flow**와 함께 걸어 놓은 예시입니다.(각 **Flow**는 탭 바 아이템과 연관된 네비게이션 스택을 의미합니다.)
```swift
private func navigationToDashboardScreen () -> [NextFlowItem] {
    let tabbarController = UITabBarController()
    let wishlistStepper = WishlistStepper()
    let wishListFlow = WishlistWarp(withService: self.service,
                                    andStepper: wishlistStepper)
    let watchedFlow = WatchedFlow(withService: self.service)

    Flows.whenReady(flow1: wishListFlow, flow2: watchedFlow, block: { [unowned self]
    (root1: UINavigationController, root2: UINavigationController) in
        let tabBarItem1 = UITabBarItem(title: "Wishlist",
                                       image: UIImage(named: "wishlist"),
                                       selectedImage: nil)
        let tabBarItem2 = UITabBarItem(title: "Watched",
                                       image: UIImage(named: "watched"),
                                       selectedImage: nil)
        root1.tabBarItem = tabBarItem1
        root1.title = "Wishlist"
        root2.tabBarItem = tabBarItem2
        root2.title = "Watched"

        tabbarController.setViewControllers([root1, root2], animated: false)
        self.rootViewController.pushViewController(tabbarController, animated: true)
    })

    return ([NextFlowItem(nextPresentable: wishListFlow,
                      nextStepper: wishlistStepper),
             NextFlowItem(nextPresentable: watchedFlow,
                      nextStepper: OneStepper(withSingleStep: DemoStep.movieList))])
}
```

스테틱 함수인 `Flows.whenReady()`는 ***Flow***를 시작하고 표시 될 준비가되면(즉,이 **Flow**의 첫 번째 화면이 선택 되었을 때) 호출 될 클로저를 가져옵니다.

### Flow와 Step의 조합에 대해 아무런 NextFlowItem을 생성하지 않는 것이 왜 괜찮습니까?
###
내비게이션 흐름이 끝나야하기 때문입니다! 예를 들어 네비게이션 스택의 마지막 화면에서는 더 이상 네비게이션 할 수 없으며 UINavigationController 자체에서 처리하는 백 액션 만 허용합니다. 이 경우 `navigate(to:)` 함수는 **NextFlowItem.noNavigation** 을 반환 합니다.

##Flow에서 일어나는 일은 ... Flow에 머물러 있습니다!##
이미 살펴 보았 듯이 여러 **Flow**를 동시에 네비게이션 할 수 있습니다. 예를 들어 네비게이션 스택안의 화면은 다른 네비게이션 스택을 포함 할 수 있는 팝업을 시작할 수 있습니다. UIKit 관점에서 볼 때 UIViewController 계층 구조는 매우 중요하며 **Coordinator** 내부에서는 이를 망칠 수 없습니다.

따라서 **Flow**가 현재 표시되지 않은 경우(예: 첫 번째 네비게이션 스택이 팝업 아래에있는 경우)에는 **Coordinator**가 방출될 수 있는**Step**을 무시합니다.

좀 더 일반적인 관점에서 볼 때 **Flow**의 컨텍스트에서 방출되는 **Step**은 해당 **Flow**의 컨텍스트에서만 해석됩니다(다른 **Flow**에서 catch 할 수 없습니다).

## 의존성 삽입이 쉬워졌습니다. ##
DI는 **RxFlow**의 주요 목표 중 하나입니다. 기본적으로 서비스, 메니저 등 어떤 것의 구현을 생성자나 함수의 인자로 전달함으로써 의존성 주입을 할 수 있습니다(매개 변수를 통해 할 수도 있습니다.).

**RxFlow**에서는 개발자가 UIViewController, ViewModels, Presenters 등을 인스턴스화 할 때가 필요한 것을 주입 할 수있는 좋은 기회입니다. 다음은 ViewModel에서의 의존성 주입 예제입니다.
```swift
import RxFlow
import UIKit

class WatchedFlow: Flow {

    ...
    private let service: MoviesService

    init(withService service: MoviesService) {
        self.service = service
    }
    ...
    private func navigateToMovieListScreen () -> [NextFlowItem] {
        // inject Service into ViewModel
        let viewModel = WatchedViewModel(with: self.service)

        // injecy ViewMNodel into UIViewController
        let viewController = WatchedViewController.instantiate(with: viewModel)

        viewController.title = "Watched"
        self.rootViewController.pushViewController(viewController, animated: true)
        return [NextFlowItem(nextPresentable: viewController, nextStepper: viewModel)]
    }
    ...
}
```
## 네비게이션 프로세스를 부트 스트랩하는 방법 ##

이제는 **Flow**와 **Step**을 조합하여 네비게이션 동작을 트리거하고 **NextFlowItem**을 생성하는 방법을 알았으므로 해야 할 일이 있습니다. 앱이 시작될 때 탐색 프로세스를 부트 스트랩해야 합니다.

**AppDelegate**에서 모든 일이 발생하며 매우 간단하다는 것을 알 수 있습니다.
- **Coordinator**를 인스턴스화합니다.
- 네비게이션 될 첫 번째 **Flow**를 인스턴스화합니다.
- **Coordinator**에게 첫 번째 **Step**으로 이 **Flow**을 조정하도록 요청합니다.
- 첫 번째 **Flow**가 준비되면 루트를 Window의 rootViewController로 설정합니다.

데모 앱의 예제입니다.
```swift
import UIKit
import RxFlow
import RxSwift
import RxCocoa

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    let disposeBag = DisposeBag()
    var window: UIWindow?
    var coordinator = Coordinator()
    let movieService = MoviesService()
    lazy var mainFlow = {
        return MainFlow(with: self.movieService)
    }()

    func application(_ application: UIApplication,
                     didFinishWithOptions options: [UIApplicationLaunchOptionsKey: Any]?)
                     -> Bool {

        guard let window = self.window else { return false }

        Flows.whenReady(flow: mainFlow, block: { [unowned window] (root) in
            window.rootViewController = root
        })

        coordinator.coordinate(flow: mainFlow,
                               withStepper: OneStepper(withSingleStep: DemoStep.apiKey))

        return true
    }
}
```
### 보너스 ###
**Coordinator**의 두가지 반응형 extension인 **willNavigate**와 **didNavigate**이 있습니다. 예를 들어 AppDelegate에서 구독 할 수 있습니다.

```swift
coordinator.rx.didNavigate.subscribe(onNext: { (flow, step) in
    print ("did navigate to flow=\(flow) and step=\(step)")
}).disposed(by: self.disposeBag)
```
이것은 다음과 같은 로그를 생산합니다.
```
did navigate flow=RxFlowDemo.MainFlow step=apiKeyIsComplete
did navigate flow=RxFlowDemo.WishlistFlow step=movieList
did navigate flow=RxFlowDemo.WatchedFlow step=movieList
did navigate flow=RxFlowDemo.WishlistFlow step=moviePicked(23452)
did navigate flow=RxFlowDemo.WishlistFlow step=castPicked(2)
did navigate flow=RxFlowDemo.WatchedFlow step=moviePicked(55423)
did navigate flow=RxFlowDemo.WatchedFlow step=castPicked(5)
did navigate flow=RxFlowDemo.WishlistFlow step=settings
did navigate flow=RxFlowDemo.SettingsFlow step=settings
did navigate flow=RxFlowDemo.SettingsFlow step=apiKey
did navigate flow=RxFlowDemo.SettingsFlow step=about
did navigate flow=RxFlowDemo.SettingsFlow step=apiKey
did navigate flow=RxFlowDemo.SettingsFlow step=settingsDone
```
이것은 분석 및 디버그에 매우 유용 할 수 있습니다.

이 Reactive Flow Coordinator 패턴이 흥미롭고 유용하다는 것을 알기를 바랍니다. 제 작품에 기고하고 도전 해주시길 바랍니다.

**RxFlow** 에 관한 세 번째이자 마지막 포스트는 모든 반응 메커니즘을 구현하는 데 사용한 팁과 트릭에 관한 것입니다.

계속 지켜봐주세요.

- [번역) RxFlow Part 1: In Theory](https://github.com/ydh1304/knowledge/blob/master/iOS/RxFlow1.md)
