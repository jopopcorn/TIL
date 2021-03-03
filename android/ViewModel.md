### ViewModel을 시작하기 전에..
> 💡 [MVVM](https://github.com/jopopcorn/TIL/blob/master/android/MVVM.md)은 AAC의 ViewModel과 연관성이 없다.
> AAC ViewModel이 없어도 MVVM을 충분히 구현할 수 있다.


## ViewModel (AAC ViewModel)

- Activity/Fragment의 Lifecycle 의존성을 낮추기 위한 것
- LiveData는 Repository로부터 데이터 변화 반응/적용
- ViewModel은 LiveData로부터 View에 필요한 데이터를 관리

**즉, AAC ViewModel은 View를 가지고 있는 (관리하고 있는) 용도**


### 특징

- Android Jetpack 구성 요소 중 하나로, 본래 ViewModel이라는 이름은 소프트웨어 설계 개발 디자인 패턴 중 하나인 MVVM(Model-View-ViewModel) 디자인 패턴으로부터 파생되었다.
- 수명 주기를 고려하여 UI 관련 데이터를 저장하고 관리하도록 설계된 클래스
- 화면 회전과 같이 [`Configuration`](https://developer.android.com/guide/topics/resources/runtime-changes?hl=ko)을 변경할 때도 데이터를 유지할 수 있다.


ViewModel을 사용하려면 아래와 같이 Lifecycle 종속성을 추가해야 한다.

```java
dependency{
	...
	implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
}
```

> UI 컨트롤러(Activity, Fragment, ...)에 과도한 책임을 할당하면 다른 클래스로 작업이 위임되지 않고, 단일 클래스가 혼자서 앱의 작업을 모두 처리하려고 할 수 있다. 
> 또한 이런 방법으로 UI 컨트롤러에 과도한 책임을 할당하면 테스트가 훨씬 더 어려워진다.
> 따라서 UI 컨트롤러 로직에서 뷰 데이터 소유권을 분리하는 방법이 훨씬 쉽고 효율적이다.

**💡 ViewModel은 뷰, Lifecycle 또는 활동 컨텍스트 참조를 포함하는 클래스를 참조해서는 안 된다!**

### ViewModel의 Lifecycle
- 일반적으로 시스템에서 Activity 객체의 onCreate() 메서드를 처음 호출할 때 ViewModel을 요청한다. 시스템은 Activity 기간 내내(예: 기기 화면이 회전될 때) onCreate() 메서드를 여러 번 호출할 수 있다. ViewModel이 처음 요청되었을 때부터 Activity가 끝나고 폐기될 때까지 ViewModel은 존재한다.
- 즉, ViewModel은 Activity의 lifecycle이 시작되고 완료될 때까지 지속된다. 따라서 ViewModel에서 복잡한 데이터를 캐슁할 수 있다. 순환 후 Activity가 다시 생성되면 캐시 데이터가 이미 있는 것과 동일한 ViewModel을 사용한다.  
- Configuration 변경이 (예:화면 회전) 발생할 때 액티비티가 다시 시작 되는 것을 확인할 수 있다. 하지만 ViewModel은 여전히 메모리 상에 남아있는다. 이는 Activity 내부에서 Configuration 변경과 무관하게 유지 되는 NonConfigurationInstances 객체를 따로 관리하기 때문이다.
- Activity의 finish() 호출등에 의해 액티비티가 생명주기가 종료됨에 따라 내부의 LifecycleEventObserver를 통해 ViewModel도 onCleared() 콜백 메서드를 호출하고 종료된다.
- 데이터를 검색하려면 비동기식 호출을 해야 하는 경우도 많다. ViewModel이 없다면 호출이 끝나기도 전에 Activity가 중지되어 메모리 누수가 발생할 수 있다. ViewModel에서 비동기식으로 호출하면 결과가 ViewModel로 다시 전달되며, Activity가 삭제되었는지 여부는 중요하지 않으므로 메모리 누수를 걱정할 필요가 없다.
