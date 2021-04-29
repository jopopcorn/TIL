
> 앱 컴포넌트는 개별적으로 비순차적으로 실행될 수 있으며, 운영체제나 사용자가 언제든지 컴포넌트를 제거할 수 있다. 운영체제나 사용자의 이벤트를 직접 제어할 수 없으므로 안드로이드 컴포넌트에 앱 데이터나 상태를 저장해서는 안되며, 컴포넌트가 서로 종속되면 안된다.

<br/>

## 일반 아키텍처 원칙

### 관심사 분리

앱 데이터와 상태를 저장하는데 컴포넌트를 사용할 수 없으므로, 이 원칙을 가장 중요하게 생각해야 한다.

액티비티 혹은 프래그먼트에 모든 코드를 작성하는 실수는 흔히 일어나는데, 이러한 UI 기반의 클래스는 UI 및 OS 상호작용을 처리하는 로직만 포함해야 한다. 관심사 분리를 통해 클래스를 최대한 가볍게 유지하고 수명 주기 관련 문제를 피할 수 있다.

액티비티, 프래그먼트 구현은 소유(own)의 대상이 아니다. 안드로이드 OS와 앱 사이의 계약을 나타내도록 이어주는 클래스일 뿐이다. OS는 사용자 상호작용을 기반으로 Memory Leaks와 같은 시스템 조건으로 인해 언제든지 클래스를 제거할 수 있다. 따라서 클래스에 대한 의존성을 최소화하는 것이 좋다.

### 모델에서 UI 만들기

모델은 앱의 데이터 처리를 담당하는 구성요소로, 앱의 View 객체 및 앱 컴포넌트와 독립되어 있으므로 수명 주기 및 관련 문제의 영향을 받지 않는다.

가급적 지속적인 모델(Persistance Data Model)을 권장하는데, 이유는 다음과 같다.

- AOS에서 리소스를 확보하기 위해 앱을 제거해도 사용자 데이터가 삭제되지 않는다.
- 네트워크 연결이 취약하거나 연결되어 있지 않아도 앱이 계속 작동한다.

데이터 관리 책임이 잘 정리된 모델 클래스를 기반으로 앱을 만들면 테스트도 쉽고 일관성을 유지할 수 있다.
<br/><br/><br/>

## 권장 앱 아키텍처
<img src="https://user-images.githubusercontent.com/35393459/116519872-f4669f80-a90c-11eb-9960-4e19cc743041.png" width="80%"/>

각 컴포넌트가 한 수준 아래의 컴포넌트에만 종속됨을 볼 수 있다. 예를 들어 액티비티/프래그먼트는 ViewModel에만 종속된다. Repository는 여러 개의 다른 클래스에 종속되는 유일한 클래스다. 이 예시에서 Repository는 Persistent Data Model(지속적인 모델)과 Remote Data Source(백엔드 데이터)에 종속된다.

이 설계는 일관되고 즐거운 사용자 환경을 제공한다. 사용자가 앱을 마지막으로 닫은지 몇 분 후 또는 며칠 후에 다시 사용하는지와 관계없이 앱이 로컬에 보존하는 사용자의 정보가 바로 표시된다. 이 데이터가 오래된 경우 앱의 Repository 모듈이 백그라운드에서 데이터 업데이트를 시작한다.
<br/><br/>

### UI 제작

UI는 프래그먼트, 관련 레이아웃 파일로 구성된다. (ex. UserProfileFragment, user_profile_layout.xml)

UI를 도출하려면 데이터 모델에서 다음 데이터 요소가 있어야 한다.

- User ID: 유저의 식별자로, 프래그먼트 인수를 사용하여 이 정보를 프래그먼트에 전달하는 것이 좋다. AOS에서 프로세스를 제거해도 이 정보가 유지되므로, 앱을 다시 시작할 때 아이디를 사용할 수 있다.
- User object: 사용자에 관한 세부정보를 보유하는 데이터 클래스

ViewModel 아키텍처 구성요소에 기반한 UserProfileViewModel을 사용하여 이 정보를 유지한다.

> ViewModel은 Fragment 같은 특정 UI 구성요소에 관한 데이터를 제공하고 모델과 커뮤니케이션하기 위한 데이터 처리 비즈니스 로직을 포함한다.  
> 예를 들어 ViewModel은 데이터를 로드하기 위해 다른 구성요소를 호출하고 사용자 요청을 전달하여 데이터를 수정할 수 있다.  
> ViewModel은 UI 컴포넌트에 관해 알지 못하므로 Configuration 변경(예: 기기 회전 시 액티비티 재생성)의 영향을 받지 않는다.

<br/><br/>
### 데이터 가져오기

방법 1. 백엔드와 통신하기 위해 WebService라는 인터페이스를 정의할 수 있다.

```kotlin
interface Webservice {
   @GET("/users/{user}")
   suspend fun getUser(@Path("user") userId: String): User
}
```

이 설계 방법은 효과가 있지만, 이를 사용하면 앱이 커지면서 유지관리가 어려워질 수 있으며, ViewModel 클래스에 너무 많은 책임을 부여해서 관심사 분리 원칙을 위반하게 된다.

또한 ViewModel의 범위는 Activity 또는 Fragment 수명 주기에 연결되어 있으므로, 관련 UI 객체의 수명 주기가 끝나면 Webservice의 데이터가 손실된다. 이 동작은 바람직하지 않은 사용자 환경을 만든다.

<br/>

방법 2. Repository 모듈 사용하여 데이터 작업을 처리한다. (권장)

```kotlin
class UserRepository {
   //Webservice 인스턴스를 사용하여 사용자 데이터를 가져옴
   private val webservice: Webservice = TODO()
   suspend fun getUser(userId: String) =
       // This isn't an optimal implementation because it doesn't take into
       // account caching. We'll look at how to improve upon this in the next
       // sections.
       webservice.getUser(userId)
}
```

Repository 모듈은 데이터 작업을 처리한다. 또한 깔끔한 API를 제공하므로 나머지 앱에서 데이터를 간편하게 가져올 수 있으며, 데이터가 업데이트될 때 어디에서 데이터를 가져올지 및 어떤 API를 호출할지 알고 있다. 레포지토리는 Persistant Data Model, Web Service, Cache 등 다양한 데이터 소스 간 중재자(Mediator)로 간주할 수 있다.

Repository 모듈은 불필요해 보일 수 있지만 앱의 나머지 부분에서 데이터 소스를 추출하는 중요한 용도로 사용된다. 이제 ViewModel이 데이터를 가져오는 방법을 알지 못하기 때문에, 여러 개의 서로 다른 데이터 가져오기 구현을 통해 얻은 데이터를 뷰모델에 제공할 수 있다.

<br/>

**💡 구성 요소 간 종속성 관리?**

위의 UserRepository 클래스에서 사용자의 데이터를 가져오려면 Webservice의 인스턴스가 필요하다. 인스턴스를 간단히 만들 수 있지만 그렇게 하려면 Webservice의 종속성도 알아야 한다. 또한 UserRepository는 Webservice가 필요한 유일한 클래스가 아닐 수도 있다. 이러한 상황에서 Webservice의 참조가 필요한 각 클래스에서 클래스와 종속성을 구성하는 방법을 알아야 하므로 코드를 복제해야 한다. 클래스별로 새로운 Webservice를 만들면 앱의 리소스 소모량이 너무 커질 수도 있다.

다음 설계 패턴을 통해 이 문제를 해결할 수 있다.

- 종속성 주입(DI): 종속성 주입을 사용하면 클래스가 자신의 종속성을 구성할 필요 없이 종속성을 정의할 수 있다. 런타임 시 다른 클래스가 이 종속 항목을 제공해야 한다.
- [서비스 로케이터 패턴](https://en.wikipedia.org/wiki/Service_locator_pattern): 클래스가 자신의 종속 항목을 구성하는 대신 종속 항목을 가져올 수 있는 레지스트리를 제공한다.

이 패턴은 코드를 복제하거나 복잡성을 추가하지 않아도 종속 항목을 관리하기 위한 명확한 패턴을 제공하므로 코드를 확장할 수 있다. 또한 이러한 패턴을 사용하면 테스트 및 프로덕션 데이터 가져오기 구현 간에 신속하게 전환할 수 있다.

종속 항목 삽입 패턴을 따르고 Android 앱에서 Hilt 라이브러리를 사용하는 것이 좋다. Hilt는 종속 항목 트리를 따라 이동하여 객체를 자동으로 구성하고 종속 항목의 컴파일 시간을 보장하며 Android 프레임워크 클래스의 종속 항목 컨테이너를 만든다.


<br/>

### ViewModel - Repository 연결

```kotlin
@HiltViewModel
class UserProfileViewModel @Inject constructor(
   savedStateHandle: SavedStateHandle,
   userRepository: UserRepository
) : ViewModel() {
   val userId : String = savedStateHandle["uid"] ?:
          throw IllegalArgumentException("missing user id")

   private val _user = MutableLiveData<User>()
   val user = LiveData<User> = _user

   init {
       viewModelScope.launch {
           _user.value = userRepository.getUser(userId)
       }
   }
}
```

<br/>

**데이터 캐시**

UserRepository 구현은 Webservice 객체 호출을 추출하지만 하나의 데이터 소스에만 의존하기 때문에 유연성이 떨어진다. 

UserRepository 구현에서 발생하는 중요한 문제는 백엔드가 데이터를 가져온 후 어디에도 보관하지 않는다는 점이다. 따라서 사용자가 Fragment를 떠났다가 다시 돌아오면 데이터가 변경되지 않았어도 앱에서 데이터를 다시 가져와야 한다.

이 설계는 다음과 같은 이유로 최적의 방법이 아니다.

- 귀중한 네트워크 대역폭을 낭비한다.
- 새 쿼리가 완료될 때까지 사용자가 기다려야 한다.

이러한 단점을 해결하기 위해 메모리에 UserRepository 객체를 캐시하는 User에 새로운 데이터 소스를 추가한다.

```kotlin
// @Inject tells Hilt how to create instances of this type
// and the dependencies it has.
class UserRepository @Inject constructor(
   private val webservice: Webservice,
   // Simple in-memory cache. Details omitted for brevity.
   private val userCache: UserCache
) {
   suspend fun getUser(userId: String): User {
       val cached: User = userCache.get(userId)
       if (cached != null) {
           return cached
       }
       // This implementation is still suboptimal but better than before.
       // A complete implementation also handles error cases.
       val freshUser = webservice.getUser(userId)
       userCache.put(userId, freshUser)
       return freshUser
   }
}
```

<br/>

**데이터 지속 (Persistant Data Model)**

현재 구현 방식을 사용하면 사용자가 기기를 회전하거나 앱에서 나갔다가 즉시 돌아오는 경우 저장소가 메모리 내 캐시에서 데이터를 가져오기 때문에 기존 UI가 즉시 표시된다.

그런데 사용자가 앱에서 나갔다가 몇 시간 뒤 Android OS에서 프로세스를 종료한 후에 다시 돌아오면 어떻게 될까? 이 상황에서 현재 구현에 의존하면 네트워크에서 데이터를 다시 가져와야 한다. 이렇게 데이터를 다시 가져오는 프로세스는 사용자 환경을 저해할 뿐 아니라 귀중한 모바일 데이터를 소비한다는 점에서 낭비를 불러온다.

이 문제는 웹 요청을 캐시하여 해결할 수 있지만, 이로 인해 중요한 문제가 새로 발생한다. 친구 목록 가져오기와 같은 다른 유형의 요청에서 동일한 사용자 데이터가 표시되면 어떻게 될까? 앱에서 일치하지 않는 데이터를 표시하여 혼란스러워진다. 예를 들면 사용자가 친구 목록과 단일 사용자를 서로 다른 시간에 요청한 경우 앱에서 동일한 사용자 데이터의 두 가지 다른 버전을 표시할 수 있다. 이렇게 되면 앱에서 일치하지 않는 데이터를 병합할 방법을 알아야 한다.

이 상황은 지속 모델을 사용하여 적절하게 해결될 수 있다. 여기에서 [Room](https://developer.android.com/training/data-storage/room) 지속성 라이브러리가 필요하다. Room에 대한 학습은 나중에..

<br/>

+) 단일 소스 저장소 (Remote Data Source)

다양한 REST API 엔드포인트는 흔히 동일한 데이터를 반환한다. 예를 들어 백엔드에 친구 목록을 반환하는 다른 엔드포인트가 있다면, 2개의 다른 API 엔드포인트에서 서로 다른 세분화 수준을 사용하여 동일한 사용자 객체를 제공할 수 있다. 일관성 확인 없이 `UserRepository`가 `Webservice` 요청으로부터 응답을 있는 그대로 반환했다면 저장소의 데이터 버전과 형식이 가장 최근에 호출된 엔드포인트에 종속되므로 UI에서 혼란스러운 정보를 표시할 수 있다.

그렇기 때문에 `UserRepository` 구현에서는 데이터베이스에 웹 서비스 응답을 저장한다. 그러면 데이터베이스 변경 시 활성 *LiveData* 객체에 콜백이 트리거된다. 이 모델을 사용하면 데이터베이스가 단일 소스 저장소 역할을 하며 앱의 다른 부분은 `UserRepository`를 사용하여 데이터베이스에 액세스한다. 디스크 캐시 사용 여부와 상관없이 저장소에서 데이터 소스를 나머지 앱의 단일 소스 저장소로 지정하는 것이 좋다.

<br/>

**진행 중인 작업 표시**

당겨서 새로고침과 같은 일부 사용 사례에서는 현재 진행 중인 네트워크 작업이 있음을 사용자에게 UI로 표시하는 게 중요하다. 다양한 이유로 데이터가 업데이트될 수 있으므로 실제 데이터와 UI 작업을 분리하는 것이 좋다. 예를 들어 친구 목록을 가져왔다면 LiveData<User> 업데이트를 트리거하여 프로그래밍 방식으로 동일한 사용자를 다시 가져올 수도 있다. UI 관점에서 보면 진행 중인 요청이 있다는 사실은 User 객체의 다른 데이터 부분과 유사한 다른 데이터 포인트에 불과하다.

데이터 업데이트 요청의 출처와 관계없이 다음 전략 중 하나를 사용하여 UI에 일관성 있는 데이터 업데이트 상태를 표시할 수 있다.

- `LiveData` 유형의 객체를 반환하도록 `getUser()`를 변경한다. 이 객체에는 네트워크 작업 상태가 포함된다. 
- `UserRepository` 클래스에 `User`의 새로고침 상태를 반환할 수 있는 다른 공개 함수를 제공한다. 데이터 가져오기 프로세스가 당겨서 새로고침 같은 명시적인 사용자 작업에서 시작되었을 때 UI에 네트워크 상태를 표시하려는 경우 이 옵션이 더 적합하다.

<br/><br/>

## 권장 사항
다음 권장사항은 필수는 아니지만, 경험에 의하면 권장사항을 따르는 경우 장기적으로 더 강력하고, 테스트 및 유지관리가 쉬운 코드베이스를 만들 수 있다.

#### 1. Activity, Service, Broadcast Receiver와 같은 앱의 진입점을 데이터 소스로 지정하지 말기  
#### 2. 앱의 다양한 모듈 간 책임이 잘 정의된 경계를 만들기
#### 3. 각 모듈에서 가능하면 적게 노출하기
#### 4. 각 모듈을 독립적으로 테스트하는 방법을 고려하기
#### 5. 다른 앱과 차별되도록 앱의 고유한 핵심에 초점을 맞추기
#### 6. 가능한 한 관련성이 높은 최신 데이터를 보존하기
#### 7. 하나의 데이터 소스를 단일 소스 저장소로 지정하기

