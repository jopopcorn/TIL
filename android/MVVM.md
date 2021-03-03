# MVVM

- View와 ViewModel의 연결을 최소화하기 위한 패턴
- 1990년 마틴 파울러에 의해 나온 MVP에서 파생된 패턴
- Model에서 데이터가 변경되면 ViewModel을 거쳐 View로 전달되도록 하는데 Android에서는 LiveData나 RxJava 등을 통해 구현할 수 있음

### View

- 화면에 표현되는 레이아웃에 대해 관여함
- 기본적으로 비즈니스 로직을 배제하지만 UI와 관련된 로직을 수행할 수 있음
- View는 ViewModel을 옵저빙하고 있다가 상태 변화가 전달되면 화면을 갱신해야 함
- View는 화면 정보 변화를 ViewModel에 전달
- View는 XML, Activity/Fragment가 아님!

### ViewModel

- 뷰에 연결할 데이터와 명령으로 구성되어 있으며 변경 알림을 통해 뷰에게 상태 변화를 전달함
- 전달받은 상태 변화를 화면에 반영할지는 View가 선택하도록 함
- 명령은 UI를 통해서 동작하도록 함