Swift 빌드 타임 절약
===
스위프트의 빌드와 컴파일 타임이 길어지는 것에는 많은 이유가 있다. 그 중 가장 중요하고 흔한 이유는 타입 추론이다. 스위프트 컴파일러는 근본적으로 비싼 타입 체킹 과정 때문에 딕셔너리와 어레이를 변환하고 컴파일 할때 특히 느리다.

불행하게도 느린 컴파일 타임을 피할 수 없는 경우가 있지만, 몇몇 툴과 세팅 변경으로 Swift 컴파일과 빌드 타임을 줄일 수 있다.

# 1. Xcode 빌드 타임 diplay
먼저, 컴파일 타임을 비교할 필요가 있다. 이 글에서 추천하는 변경들이 원하는 효과를 가지는지 측정하는 것이 중요하다.

Xcode의 UI안에서 타이머를 enable할 수 있다. 기본 값으로는 보이지 않지만 다음과 같이 커멘드 라인에 실행시키면 앱을 빌드 할 때 마다 시간이 디스플레이 된다.

`defaults write com.apple.dt.Xcode ShowBuildOperationDuration -bool YES`

빌드 타임을 측정하고 싶을 때에는 빌드 폴더를 포함한 당신의 프로잭트를 클린해서 보존된 데이터를 지우는 것이 좋다. 커멘드 라인에서는 다음과 같이 할 수 있다.

`rm -rf ~/Library/Developer/Xcode/DerivedData`

# 2. 느리게 컴파일되는 코드 찾기

Xcode는 긴 컴파일 타임을 유발하는 함수와 식들을 식별하는 내장 기능을 가진다. 컴파일 타임을 제한한 뒤 제한을 넘는 코드베이스 구역을 식별할 수 있다.

프로잭트의 build settings를 열고, `Other Swift Flags`에 다음과 같은 플래그를 추가한다.

- `-Xfrontend -warn-long-function-bodies=100`
- `-Xfrontend -warn-long-expression-type-checking=100`

숫자 100은 함수와 식의 컴파일 타임 밀리세컨드 제한이다. 100ms를 넘는 부분은 warning으로 보여서 코드를 리팩터하고 컴파일 타임을 줄일 기회를 준다.

속도를 향상시키는 방법으로는
- 타입 추론을 줄인다.(추론 가능한 타입도 코드에 명시한다)
- Nil-Coalescing Operator 줄인다. (?? 연산자)
- Ternary Conditional Operator 줄인다. (? 연산자)
- 문자열 합칠 때 + 지양한다. 대신에 string interpolation 사용
- 조건문의 비교값에 연산을 넣지 않는다.
하지만 대부분 코드의 간결함을 해치기 때문에 추천하지 않음

# 3. Active Architecture만 빌드하기

디버그 환경이 선택됐을때는 active architerture만 빌드 해야 한다. 기본값으로 설정 돼 있지만 확인해보는 것이 좋다.

프로잭트 build settings의 `Build Active Architecture Only` 에서 Debug는 `Yes`, Release는 `No`로 선택됐는지 확인한다.

# 4. dSYM 생성을 최적화한다.

크래쉬 리포트를 위해 dSYM 파일을 사용할 수 있다. 이것은 디버거가 걸려 있지 않을 때 특히 유용하다. 하지만 만드는 데에 시간이 걸리므로 Xcode 디버거가 걸려있지 않을 때만 만드는 것이 좋다.

`Debug Information Format`이 Release와 시뮬레이터에서 돌아가지 않는 Debug 빌드에서 항상 dSYM 파일을 만들도록 한다. iOS 시뮬레이터에서는 필요 없다.

# 5. 모듈 최적화

컴파일러가 돌 때 하나의 job이 모든 소스 파일 대신 필요한 소스 파일만 가지고 돌도록 build setting을 설정 할 수 있다. 이것은 병렬성을 줄이지만 중복되는 일을 크게 줄이기 때문에 빌드 타임을 빠르게 할 수 있다.

이것을 구현하기 위해 `Other Swift Flags`에 `-Onone`을 **디버그 환경에만** 추가한다. 추가로 디버그에서 `Optimization Level`을 `Fast, Wholde Module Optiomization`으로 설정한다.

CocoaPods를 사용중이라면 다음 코드를 Podfile 끝에 추가시켜서 모든 의존성들을 최적화 할 수 있다.
```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      if config.name == 'Debug'
        config.build_settings['OTHER_SWIFT_FLAGS'] = ['$(inherited)', '-Onone']
        config.build_settings['SWIFT_OPTIMIZATION_LEVEL'] = '-Owholemodule'
      end
    end
  end
end
```

# 6. 서드파티 의존성
서드파티 의존성들을 다루는 가장 흔한 방법은 CocoaPods를 이용하는 것이다. 이것은 사용하기 쉽지만 빌드 타임에는 최고의 옵션이 아니다.

다른 대안으로 Carthage가 있다. CocoaPods보다 사용하기 어렵지만 빌드 속도를 향상시킨다. Carthage는 외부 의존성을 새로운 것이 추가될 때에만 빌드 한다. 새로운 것을 추가하지 않았다면 외부 의존성을 빌드할 필요가 없다.

# 7. Xcode의 새 빌드 시스템
Xcode 9 에서 애플은 새로운 빌드 시스템을 소개했다. 새로운 빌드 시스템의 주요 장점 중 하나는 빠른 빌드 타임이다. Xcode 10 에서 현재 기본값이다.

새로운 빌드 시스탬을 쓰기 위해서는 **File** 메뉴의 **Workspace Settings** 또는 **Project Settings** 에서 설정해야한다.

# 8. 빌드 동시성
Xcode 9.2에서 Swift 빌드 작업을 병렬로 수행하는 기능을 소개했다. 다음과 같은 코드를 커멘드 라인에 입력한다.

```
defaults write com.apple.dt.Xcode BuildSystemScheduleInherentlyParallelCommandsExclusively -bool NO
```

몇몇 프로잭트는 다른 것보다 큰 이득을 얻는다(어떤 사례의 경우 최대 40%). 당신이 큰 RAM을 가지고 있지 않으면 더 느려질 수도 있다. 이런 경우 동시성을 disable할 수 있다.

```
defaults delete com.apple.dt.Xcode BuildSystemScheduleInherentlyParallelCommandsExclusively
```

# 9. Swift의 의존성 이해
Swift는 Objective-C 와 다르게 header file이 없다. 따라서 변경사항이 생겼을 때 스위프트 컴파일러가 어떤 파일을 다시 컴파일 해야하는지 아는 방식을 이해하는 것이 중요하다. 스위프트는 같은 소스코드 파일 안에 오브잭트를 여러개 정의하는 것을 허용한다(되도록 안하는 것이 좋다). 

Swift 컴파일러는 `struct`, `enum` 또는 `class`와 같은 싱글 엔티티를 추적하는 것이 아니라 파일을 추적한다. 여러개의 엔티티를 하나의 파일에 정의했을 경우 같은 파일에 엔티티의 추가, 삭제가 생기거나 함수 바디 외부의 단순한 변경사항이 있을 경우 전체 파일을 다시 컴파일한다.(함수 내부에 정의된 객체는 영향 받지 않는다)

새로운 정의를 위해 새로운 파일을 만드는 것은 컴파일 속도를 빠르게 할 뿐 아니라 정의된 위치를 찾는 것 또한 쉽게 해준다.

# 10. Objective-C 인터페이스 제한
Swift와 Objective-C 사이의 의존성을 위해 Bridging Header와 Generated Header가 존재한다. 하나의 스위프트 파일이 변경될 경우 그것을 사용하는 모든 Objective-C 파일이 다시 컴파일된다.

Objective-C 에 노출되는 프로퍼티나 함수를 `private` 으로 정의하는 것이 좋은 방법이다. 이것은 캡슐화에도 좋다.

Swift4 마이그래이션 시에 자동으로 생성되는 `Swift3 @objc Inference`는 NSObject의 하위 클래스들 안의 모든 속성이 Objective-C에 노출되게 한다. 이 설정을 `Default`가 되게 한 후 @objc를 사용하는 것이 좋다.

# 결과
위 10가지 방법을 적용 시킨 후의 시간 변화를 측정한다.

프로잭트의 기존 빌드타임: 102.910s
- Carthage 적용 전: 86.742s


















