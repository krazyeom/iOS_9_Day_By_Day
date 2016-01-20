# Day 2 :: User Interface Testing

소프트웨어 응용 프로그램을 개발할 때 자동화된 사용자 인터페이스 테스트는 유용한 도구 입니다. 사용자 인터페이스 테스트는 빠르게 응용 프로그램의 문제를 감지 할수 있으며, 테스트 집합(test suite)의 성공적인 실행은 앱을 릴리즈 하기 전에 개발자에게 신뢰를 제공할 수 있습니다. iOS 플랫폼에서, 자동화된 사용자 인터페이스 테스트는 자바 스크립트로 작성된 UIAutomation을 사용하여 수행됩니다. 이것은 Instruments 라는 별도의 응용 프로그램을 실행하고, 스크립트를 생성하고 실행하는 것을 포함한다. 이런 작업 방법은 아주 고통스러울 만큼 느리고, 익숙해지기에는 많은 시간이 걸립니다.

### UI Testing

애플은 Xcode 7에서 앱의 사용자 인터페이스 테스트를 수행할 수 있는 새로운 방법을 소개했습니다. UI 테스트는 당신이 UI 요소를 찾고 그것들과 상호작용 할 수 있으며, 사용가능한 속성과 상태를 검증할 수 있습니다. UI 테스트는 완벽하게 Xcode 7 테스트 리포트와 통합되었으며, 단위 테스트와 함께 실행 됩니다. XCTest는 Xcode 5 이후 Xcode의 테스트 프레임워크로 통합 되었지만, Xcode 7에서는 새로운 UI 테스트 기능이 포함되어 업데이트 되었습니다. 이것은 당신이 특정 지점에서 당신의 UI 상태를 확인할 수 있도록 문장(assertions)을 만들수 있습니다. 
assertions - (In computer programming, an assertion is a statement that a predicate (Boolean-valued function, a true–false expression) is expected to always be true at that point in the code. If an assertion evaluates to false at run time, an assertion failure results, which typically causes the program to crash, or to throw an assertion exception.)

### Accessibility
UI 테스트가 가능하기 위해서는, 프로임워크에서 UI의 요소별 작업을 수행 할 수 있도록 UI 내부에 있는 다양한 요소들에 접근이 가능해야 한다. 당신이 탭(tap)이나 스와이프(swipe) 테스트를 위한 특정 위치를 지정할 수도 있지만, 그것은 다른 크기의 디바이스나 당신의 UI의 요소들의 위치를 아주 조금만 변경해도 테스트 진행률이 떨어질 것이다. 

