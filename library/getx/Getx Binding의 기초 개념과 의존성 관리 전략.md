> **주의!** **이미 GetX를 사용하고 있는 프로젝트**에서 **올바른 사용법**을 제시하는 것이 목적입니다. 
절대로 새로운 프로젝트에 GetX 도입을 권장하는 것이 아닙니다.
새 프로젝트에서는 Provider, Riverpod 등의 권장 패턴을 써주세요!

## 개요

Bindings는 인스턴스의 의존성 주입과 라이프사이클을 관리하는 GetX의 핵심 메커니즘입니다. 

**GetX Bindings의 주요 특징:**
- **의존성 주입 관리**: 컨트롤러, 서비스, 저장소 등의 의존성을 자동으로 관리, Get.put()과 Get.find() 사용
- **메모리 최적화**: 페이지 생성 시에만 의존성을 주입하고, 페이지 제거 시 자동으로 메모리에서 해제
- **생명주기 관리**: 라우트 기반으로 의존성의 생명주기를 자동 관리

이 글에서는 GetX 공식 문서의 Bindings 개념과 참여했던 실제 프로젝트에서 어떻게 적용했는지 비교 분석해보겠습니다.

---


## GetX Bindings란?

### 공식 문서 정의([공식문서](https://pub.dev/packages/get/versions/4.7.2))

Bindings는 인스턴스의 의존성 주입과 라이프사이클을 관리하는 GetX의 핵심 메커니즘 이며, 다음과 같은 특징을 가집니다:

- **의존성 주입 관리**: 컨트롤러, 서비스, 저장소 등의 의존성을 자동으로 관리
- **메모리 최적화**: 페이지가 생성될 때만 의존성을 주입하고, 페이지가 제거될 때 자동으로 메모리에서 해제
- **생명주기 관리**: 라우트 기반으로 의존성의 생명주기를 자동 관리, 알아서 켜지고 꺼지고

확실히 '마법같은' 기능을 제공하기로 유명한 만큼, 개발자들이 놓칠 수 있는 부분을 알아서 잡아주는 편리함이 눈에 띄네요.

### 기본 사용법

```dart
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.put<HomeController>(HomeController());
    Get.lazyPut<ApiService>(() => ApiService());
  }
}
```
이런 식으로 HomeBinding을 구성하고, HomePage의 라우트에 같이 묶어둡니다. 그리고 Get.put 메서드에는 주입하는 의존성의 타입을 정확하게 명시해주세요.

```dart
static final routes = [
  GetPage(
    name: HomePage.routeName,
    page: () => const HomePage(),
    binding: HomeBinding()),
  ...
  ];
```

이제 HomePage로 라우팅 할 때 HomeBinding에 붙어있는 의존성들이 줄줄히 인스턴스화 됩니다.
put메서드가 아니라 lazyPut메서드로 주입한건 '필요할 때만 생성'됩니다. 아래에서 또 언급 하겠습니다.


---


## GetMaterialApp과 Bindings의 관계

### GetMaterialApp의 핵심 역할

GetX의 모든 기능은 `GetMaterialApp` 내에서 동작합니다. 이는 일반 Flutter의 `MaterialApp`을 확장한 것으로, GetX의 상태관리, 라우팅, 의존성 주입 등의 기능을 제공합니다. 물론 Binding도 이 안에서 사용해야겠죠?

```dart
class App extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'WheelyX',
      initialRoute: PageRouter.initial,
      getPages: PageRouter.routes,
      initialBinding: InitialBinding(),  // 앱 전체 초기 의존성
      // ... 기타 설정들
    );
  }
}
```

### Bindings의 계층 구조

GetMaterialApp사용 하에 Bindings는 라우터에만 거는게 아니라, 다양한 방식으로 여러곳에 **묶을** 수 있습니다.

1. **InitialBinding**: 앱 시작 시 전역적으로 필요한 의존성 (상시 의존)
2. **Route Binding**: 각 페이지별로 필요한 의존성 (페이지에 의존)

GetMaterialApp 환경에서 Bindings는 다음과 같은 계층 구조로 관리됩니다:

```
My App Service
├── InitialBinding (앱 전체)
│   ├── AuthStore (permanent: true)
│   ├── FCM Controller (permanent: true)
│   └── Repository Layer (fenix: true)
└── Route Bindings (페이지별)
    ├── HomeBinding
    ├── AuthBinding
    └── TrainingBinding
```


---


## InitialBinding - 앱 전체 초기 의존성
주로 앱 전체에서 공유되는 핵심 서비스들을 여기에 걸어둡니다.

```dart
class InitialBinding extends Bindings {
  @override
  void dependencies() {
    // FCM 푸시메세지 관리자
    Get.put<FcmFirebaseController>(
      FcmFirebaseController(messaging: FirebaseMessaging.instance),
      permanent: true,  // 앱 종료까지 유지
    );
    
	// 항상 관리해야 하는 중요한 상태
    Get.put<AuthStore>(AuthStoreImpl(), permanent: true);
    Get.put<HistoryStore>(HistoryStoreImpl(), permanent: true);

    // Repository 계층
    Get.lazyPut<AuthRepository>(
      () => AuthRepositoryImpl(authDio: Get.find<AuthDio>()),
      fenix: true,  // 사용 후에도 메모리에 유지
    );
  }
}
```
이렇게 선언후 위의 GetMaterilaApp의 initialBinding에 넣어주면 됩니다.
프로젝트에서 InitialBinding에 포함시켜야 할 주요 의존성은 대부분 전역에서 관리해야 하는 객체들로, 아래처럼 정리할 수 있을 것 같습니다.

**포함해야 할 의존성**:

- 앱 생명주기와 일치하는 서비스
- 전역 상태 관리 객체
- 시스템 서비스 (푸시 알림, 센서 연결 등)
- 인증 관련 서비스

**제외해야 할 의존성**:

- 페이지별 ViewModel
- 초기화 비용이 높은 서비스
- 조건부로만 필요한 기능


여기서 짚어봐야 할 점이 있습니다. 바로 **Get.put**과 **lazyPut**이라는 인스턴스의 **두 개의 의존성 생산 전략**과, 이들과 함께 붙는 **permanent, fenix 파라미터** 입니다.
잠시 체크하고 가겠습니다.

### Get.put() vs Get.lazyPut()
생성 시점과 메모리 비용 효율성을 고려해서 어떻게 생성할 건지를 결정하는 메서드인 만큼, 잘 골라서 사용해야 합니다. 

| 구분        | Get.put()        | Get.lazyPut()     |
| ----------- | ---------------- | ----------------- |
| 생성 시점   | 즉시 생성        | 첫 사용 시 생성   |
| 메모리 사용 | 높음 (즉시 점유) | 효율적 (필요시만) |
| 적용 대상   | 필수 서비스      | 선택적 서비스     |
| 초기화 시간 | 앱 시작 시 부담  | 분산된 부담       |

### permanent vs fenix 심화 이해
permanent의 경우는 말 그대로 true로 하는 경우 어떤 상황에도 종료되지 않고 항시 인스턴스가 유지되도록 해주는 기능입니다.
앱이 종료되기 전 까지 절대로 해제되지 않고 메모리에 상주시키며, 항상 접근이 가능한 기능입니다.

그렇다면 fenix(피닉스)는 lazyPut에 붙어서 사용 할 때마다 부활하는 특성을 보여주는 기능으로, 사용 후 메모리에서 해제되지만 재사용 시 기존 인스턴스를 그대로 활용하는 기능입니다.

 좀 유치하지만 **"불사조(Phoenix)" 패턴**이라고 부르던데.. 아래와 같은 동작방식을 가지니 알아두면 좋을 것 같습니다.
 
 **동작 방식:**

1. **첫 번째 Get.find**: 인스턴스가 생성되고 메모리에 저장됨
2. **사용 완료**: 일반적으로 `lazyPut`은 사용 후 메모리에서 해제됨
3. **재사용 시**: `fenix: true`가 있으면 **기존 인스턴스를 재활용**
4. **메모리 효율성**: 사용하지 않을 때는 메모리에서 해제되지만, 재사용 시 빠른 접근 가능

즉, 인스턴스를 새로 만들지 않고, 기존 인스턴스가 잠깐 죽어있다가 부활한다는 의미입니다.

#### AuthRepository에 왜 lazyPut과 fenix를 사용했을까요?
항시 켜놓아도(put과 permanent를 통해) 상관없을지 모르지만, repository계층의 특성을 생각하면 lazyPut이 적합하다고 판단했습니다.
하지만 인스턴스는 쓰던 놈을 계속 써야 하죠. 정리하자면 아래와 같은 이유가 있습니다.
- **네트워크 연결 재사용**: HTTP 클라이언트(Dio) 연결을 재활용하여 성능 향상
- **토큰 관리**: 인증 토큰이 저장된 Repository 인스턴스를 재사용
- **메모리 효율성**: 사용하지 않을 때는 메모리에서 해제, 필요할 때만 유지
- **빠른 응답**: 재사용 시 새로운 인스턴스 생성 시간 절약

실제 프로젝트에서의 활용 예시도 볼까요?

```dart
// AuthDio - HTTP 클라이언트 (토큰 관리)
Get.lazyPut<AuthDio>(
  () => AuthDio(locale: Get.locale, dio: Dio(), storage: FlutterSecureStorage()),
  fenix: true,
);

// AuthRepository - 인증 관련 API 호출
Get.lazyPut<AuthRepository>(
  () => AuthRepositoryImpl(authDio: Get.find<AuthDio>()),
  fenix: true,
);
```

이렇게 `fenix: true`를 사용하면 **메모리 효율성과 성능을 동시에 확보**할 수 있습니다.
주로 Repository, API 클라이언트같은** 무거운 객체**, 네트워크 연결, 데이터베이스 연결 같은** 초기화 비용이 높은 객체**, 특정 기능에서만 **가끔 사용하는 객체** 등을 등록해주시면 됩니다.


---


## RouteBinding - 라우트 기반 의존성 관리

Route Bindings는 각 페이지별로 필요한 의존성을 관리하는 핵심 메커니즘입니다. InitialBinding에서 전역 서비스를 관리했다면, Route Bindings는 페이지 생명주기와 연결된 컨트롤러와 서비스를 관리합니다.


### 기본 구현 패턴

#### 1. 단일 ViewModel Binding

```dart
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.put<HomeViewModel>(HomeViewModel(
      authRepository: Get.find<AuthRepository>(),
      authStore: Get.find<AuthStore>(),
    ));
  }
}
```

**특징**:

- 페이지 진입 시 ViewModel 생성
- 페이지 종료 시 자동으로 메모리에서 해제
- InitialBinding의 전역 의존성 활용

#### 2. 다중 의존성 Binding
여러 기능을 하나의 ViewModel에 묶어서 사용하는 경우 다중 의존성 Binding기법을 사용해주세요.

제 프로젝트에서는 운동을 수행할 때, 유저 정보, 히스토리(운동 기록)정보, 블루투스 연결, 운동을 수행하는 트레이닝 서비스 등등 많은 기능이 필요했습니다.

```dart
class TrainingBinding extends Bindings {
  @override
  void dependencies() {
    // ViewModel 주입
    Get.put<BasicTrainingViewModel>(BasicTrainingViewModel(
      authRepository: Get.find<AuthRepository>(),
      historyRepository: Get.find<HistoryRepository>(),
      bluetoothStore: Get.find<BluetoothStoreImpl>(),
    ));

    // 페이지별 서비스 주입
    Get.put<TrainingService>(TrainingService());
    Get.put<TimerService>(TimerService());

    // Lazy 의존성 주입
    Get.lazyPut<SensorDataProcessor>(() => SensorDataProcessor());
  }
}
```

### Route Bindings 심화 패턴

#### 1. 조건부 Bindings 

Get.arguments를 통해 매개변수를 전달 받아서 의존성을 조건부로 주입합니다. 

```dart
class HistoryDetailBinding extends Bindings {
  @override
  void dependencies() {
    // 기본 ViewModel
    Get.put<HistoryDetailViewModel>(HistoryDetailViewModel(
      historyRepository: Get.find<HistoryRepository>(),
    ));

    // 조건부 서비스 주입
    final args = Get.arguments as HistoryDetailArgs?;
    if (args?.isGameHistory == true) {
      Get.put<GameAnalysisService>(GameAnalysisService());
    }

    if (args?.needsComparison == true) {
      Get.lazyPut<ComparisonService>(() => ComparisonService());
    }
  }
}
```

혹은 Get.parameters를 사용해도 됩니다.

```dart
class CourseBinding extends Bindings {
  @override
  void dependencies() {
    // 트레이닝 코스를 판별
    final courseId = Get.parameters['courseId'];

    // 특정 트레이닝 코스별로 기능 구분
    Get.put<CourseViewModel>(CourseViewModel(
      courseId: courseId,
      courseRepository: Get.find<CourseRepository>(),
    ));
    Get.lazyPut<CourseAnalyzer>(() => CourseAnalyzer(courseId: courseId));
  }
}
```


#### 2. 계층형 Bindings

`BaseBinding`을 만들어서 코드의 중복을 줄이고, 공통 사용 의존성을 쉽게 관리합니다. `Bindings`도 클래스기 때문에 당연히 유효한 전략입니다.

```dart
// 공통 기능을 담당하는 Base Binding
abstract class BaseTrainingBinding extends Bindings {
  void registerCommonDependencies() {
    Get.put<SensorService>(SensorService());
    Get.put<AudioService>(AudioService());
  }
}

// 구체적인 트레이닝별 Binding
class ATrainingBinding extends BaseTrainingBinding {
  @override
  void dependencies() {
  	// 공통 의존성 등록 딸깍
    registerCommonDependencies();

    Get.put<ATrainingViewModel>(ATrainingViewModel(
      sensorService: Get.find<SensorService>(),
      audioService: Get.find<AudioService>(),
    ));
  }
}

class BTrainingBinding extends BaseTrainingBinding {
  @override
  void dependencies() {
  	// 공통 의존성 등록 딸깍
    registerCommonDependencies();

    Get.put<BTrainingViewModel>(BTrainingViewModel(
      sensorService: Get.find<SensorService>(),
      audioService: Get.find<AudioService>(),
    ));

    // B 트레이닝에만 붙는 서비스
    Get.put<SpecialTimerService>(SpecialTimerService());
  }
}
```
---

## 마무리 및 GetX Bindings 사용 시 주의사항

마무리로 Bindings사용 전략을 정리하고, 권장 패턴과 안좋은 패턴을 알아보고 끝내겠습니다.

1. **과도한 전역 의존성 지양**: InitialBinding에는 정말 필요한 것만 등록
2. **타입 안전성**: `Get.put<Type>()`에서 타입을 명시적으로 선언
3. **메모리 관리**: `permanent`와 `fenix` 옵션을 적절히 활용
4. **의존성 순서**: 의존성 간의 순서를 고려하여 등록


#### 권장 패턴

```dart
// ✅ 좋은 예
class WellStructuredBinding extends Bindings {
  @override
  void dependencies() {
    // 1. 필수 서비스부터
    Get.put<CoreService>(CoreService());

    // 2. ViewModel 등록
    Get.put<PageViewModel>(PageViewModel(
      coreService: Get.find<CoreService>(),
    ));

    // 3. 옵셔널 서비스는 lazy로
    Get.lazyPut<OptionalService>(() => OptionalService());
  }
}
```

#### 비 권장 패턴
```dart
// ❌ 피해야 할 예
class PoorBinding extends Bindings {
  @override
  void dependencies() {
    // 타입 명시 없음
    Get.put(SomeController());

    // 불필요한 permanent 사용
    Get.put(TemporaryService(), permanent: true);

    // 의존성 순서 무시
    Get.put(DependentService(Get.find<NotYetRegistered>()));
  }
}
```


