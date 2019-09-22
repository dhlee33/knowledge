Automatic Reference Counting
===
정확히 짚고 가기 위해서 개념들을 정리해보려 한다.

### 어느시점에 작동하는가?
**컴파일 시점**에 작동. 빌드할 때 특정 개체의 레퍼런스 카운트를 추적하여, 0이 되는 시점에 자동으로 release코드를 넣어준다.

### 참조 키워드
- strong(default): RC를 1 증가시킴. 클로저 안에서 참조 될 때의 기본값도 strong이기 때문에 RC 1증가
- weak: RC 증가시키지 않음. 따라서 optional 값에만 사용 가능
- unonwed: RC를 증가시키지 않지만, 참조한 객체가 항상 있다고 가정하기 때문에 optional이 아님. 따라서 객체의 라이프사이클이 명확하고 개발자가 제어 가능할 경우에 사용하면 코드가 간결해짐

### 강한 순환 참조
객체들이 서로를 강하게 참조하고 있어서 절대 RC가 0이 되지 않는 상태. 특히 closure에서 외부 객체를 참조할 때 많이 나타남. capture list에 weak 또는 unowned사용해서 해결 가능.

### Checking reference cycle
- Xcode Memory Graph 이용해서 Live Object 들을 확인하고, Leak 된 Object 를 체크.
- Instrument 의 Leak 도구를 이용하여 체크.
- deinit 을 활용하여 로깅코드를 통해 체크.

### Escaping Closure
함수의 인자로 전달받은 closure가 함수의 라이프사이클 내에서 끝나지 않고 외부에 전달될 때는 escaping 키워드 사용해야함. 따라서 순환참조 조심해야함.

