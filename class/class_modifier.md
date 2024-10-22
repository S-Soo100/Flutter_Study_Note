# Dart 언어의 `class modifier`와 `fragile base class problem`
<img src="https://github.com/user-attachments/assets/eb9309b7-e6f2-4cd6-a164-d5cc983e04c2" width="70%" alt="banner" align="center"/>

<br/>
클래스 모디파이어(클래스 수정자 혹은 제어자, Class Modifiers)란, class 혹은 mixin이 라이브러리 내/외부로 사용되는 방식을 결정합니다.<br/>
모디파이어 키워드는 class 혹은 mixin 바로 앞에 배치하며 mixin은 그 자체로 제어자 키워드중 하나이기도 하며
클래스의 사용 방식에(프로젝트 내에서 그 클래스가 사용되기를 '원하는'방식에) 직접적으로 관여합니다.
</br>
🍀 공식 문서 : https://dart.dev/language/class-modifiers
</br>
</br>

---

# 빠르게 보는 modifier의 사용과 조합
- Dart의 주요 class modifier에는 abstract, final, sealed, base, mixin, interface 등이 있습니다.
 에기에 class까지 해서 7개 키워드의 각 기능을 정리하면 아래와 같습니다.
생성자, 상속, 구현, 완전성 검사등의 유용한 기능들을 modifier를 사용해서 제한하거나 특정 상황에서만 동작하게 만들 수 있습니다.
enum, typedef, extension, extension type 등에는 사용이 불가능합니다.

| Modifier     | construct | extend | implement | mixin | exhaustive |
|--------------|:--------:|:------:|:--------:|:-----:|:---------:|
| **class**    |    ✔     |     ✔  |      ✔   |        |           |
| **abstract** |          |   ✔    |   ✔      |        |           |
| **final**    |    ✔     |        |          |        |           |
| **base**     |    ✔     |   ✔    |          |        |           |
| **interface**|    ✔     |        |    ✔     |        |           |
| **sealed**   |          |        |          |        |     ✔     |
| **mixin**    |    ✔     |     ✔  |     ✔    |     ✔  |           |

([공식문서의 Valid combinations](https://dart.dev/language/modifier-reference))

### 기능의 한줄 설명 
- construct: 클래스 인스턴스화
- extend: 다른 클래스를 부모로 상속
- implement: 인터페이스 메서드 구현
- mixin: 여러 클래스에 공통사용 가능한 기능을 추가
- exhaustive: 본인을 상속한 모든 하위 클래스를 처리하는 `완전성 검사`

#### 각 기능들이 왜 중요한가요?
- 클래스 간의 관계와 동작을 정의하기 때문입니다.
 객체 지향 프로그래밍(OOP) 에서 예를들어, 클래스의 확장과 구현은 OOP의 핵심 원리인 `재사용성`과 `유연성`을 실현하게 해줍니다.<br/><br/>
- 확장(extends)은 기존 클래스를 상속하는 과정을 통해 이루어지는데,<br/> 매번 기본 기능을 추가할 필요가 없어 코드의 재사용성을 올려주며, 기본 기능의 수정을 상위클래스에서 하면 상속자들에게 모두 반영되기 때문에 유지보수성이 좋다고 할 수 있습니다.
(물론 여기서 발생되는 다른 문제들 또한 존재합니다)
 <br/><br/>
- 구현(implements)은 구현자에게 인터페이스에 정의된 메서드 구현을 강제합니다. <br/>이는 OOP에서 **다형성(polymorphism)**을 지원하고, 일관된 동작을 제공하는 데 매우 유용합니다.
<br/>이로 인해 클래스의 구성이 달라도 인터페이스가 같기 때문에 에러 없이 상호작용 할 수 있습니다. <br/>이는 상속과는 다른 방식으로 유연한 설계를 가능하게 해줍니다.


# 각 modifier의 원리와 사용구조

## 1. abstract(추상 클래스)
- abstract는 공통된 동작이나 기능을 여러 클래스에 제공하는 목적으로 사용됩니다.<br/>또한 클래스가 직접 인스턴스화될 수 없도록 만들며, 반드시 상속을 통해 구현해야 하는 메서드들을 정의하는 데 사용되며 각 하위 클래스는 필요에 따라 이를 확장하거나 수정할 수 있습니다.
- **하지만 모든 메서드를 추상화 해야 하는것은 아니며**, 구현 메서드나 값 또한 포함할 수 있습니다.
- interface와 함께 사용할 수도 있습니다. 이 경우 굉장히 `pure`한 인터페이스를 만들 수 있습니다.

```dart
abstract class Animal {
  void makeSound(); // 추상 메서드
  void move() { // 구현 메서드
    print("move");
  }
}

class Dog extends Animal {
  @override
  void makeSound() {
    print('Woof!');
  }
}

```

## 2. final
- 변수에 사용하는 final 키워드와 달리 이는 **상속을 방지**하는 용도로 사용합니다.<br/>
더이상 mixin이나 class가 상속되지 않도록 고정하며, abstract가 아니고 class로 만들었다면 인스턴스화는 가능합니다.
```dart
final class Robot {
  void turnOn() {...}
}

// class Gundam extends Robot {} // ERROR!! : Robot은 상속이 불가능합니다.
Robot myRobot = Robot(); // 인스턴스화는 가능합니다.
```

## 3. sealed
- sealed 는 Dart 3.0에서 도입된 키워드로, 내 클래스를 같은 파일 안에서만 상속이 가능하도록 합니다.<br/>
즉, sealed 클래스를 상속하는 모든 하위 클래스는 같은 파일에 있어야 합니다.<br/>이를 통해 상속 계층을 더 엄격하게 관리할 수 있습니다.
- sealed 클래스를 상속받는 클래스는 enum과도 비슷하다고 생각 할 수 있습니다. 모든 하위 클래스를 포괄적으로 처리할 수 있는 `완전성(exhaustive)`을 제공하기 때문입니다.
  <br/>하위 클래스의 범위가 특정 파일 내(현재 파일 내)로 제한되기 때문에, 모든 확장 가능성을 알고 이를 처리할 수 있게 됩니다. 이를 통해 switch 구문이나 패턴 매칭에서 모든 하위 클래스를 반드시 처리할 수 있습니다.
- 정리하면 sealed는 코드의 안전성과 유지보수성을 극대화하는 기능입니다.
- 외부에서 직접 상속할 수 없는 클래스를 만들 때, sealed와 final이라는 두 가지 선택지가 주어집니다.<br/>이에 대한 비교는 별도의 글에서 다루도록 하겠습니다.

(sealed 예시코드는 역시 도형이 맛있다.)
```dart
sealed class Shape {}

class Circle extends Shape {}
class Square extends Shape {}

Shape shape = getShape();

switch (shape) {
  case Circle:
    // Circle 처리
    break;
  case Square:
    // Square 처리
    break;
  // 모든 하위 클래스를 처리하므로 오류 방지 가능
}

```

## 4. base
- 클래스 또는 믹스인 구현의 상속을 강제하는 모디파이어 입니다.
- base 클래스는 상속받을 수는 있지만 구현(implement)하거나 mixin사용은 불가능합니다.(명시적인 상속 제한이라 부릅니다.)
- 파일 경계를 넘어 상속이 가능하며, 상속받는 모든 클래스는 base, final, sealed중 하나의 키워드를 표시해야 합니다.
```dart
-----a.dart
base class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}
-----b.dart
import 'a.dart';

// Can be constructed.
Vehicle myVehicle = Vehicle();

// Can be extended.
base class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// ERROR: Can't be implemented.
base class MockVehicle implements Vehicle {
  @override
  void moveForward() {
    // ...
  }
}
```

## 5. interface
- 본래 Dart에서는 Java등과 다르게 interface라는 별도의 키워드가 없었으며 모든 클래스는 자동으로 인터페이스 역할을 했습니다.<br/>interface 키워드는 Dart 3.0 업데이트와 함께 추가되어 이를 통해 명시적으로 인터페이스로 사용할 클래스를 정의할 수 있습니다. 
- 이 키워드는 클래스 자체를 인터페이스로 사용하도록 구성하여 implements를 통해 내부 메서드들을 구현하도록 강제합니다.
- 상속을 방지하여 내부 로직이 의도치 않게 변경되는 것을 방지하고 `fragile base class problem`를 줄일 수 있습니다.(이 또한 다른 글에서 다루겠습니다)
- 추상 메서드가 아니라 구현 메서드도 내부에 포함할 수 있습니다.
```dart
interface class Vehicle {
  void moveForward(int meters) {
    // ...
  }
}

// 생성자 사용 가능
Vehicle myVehicle = Vehicle();

// ERROR!!: 상속 불가
class Car extends Vehicle {
  int passengers = 4;
  // ...
}

// 구현 가능
class MockVehicle implements Vehicle {
  @override
  void moveForward(int meters) {
    // ...
  }
}
```

## 6. mixin(믹스인)
- 상속, 구현과는 별개로 여러 클래스 위계에서 재사용 될 수 있는 코드를 만듭니다. <br/>특정 클래스 상속이 불가능하며 생성자 선언도 불가능합니다. 마찬가지로 자체 인스턴스 화도 불가능합니다.
- 기능을 재사용하고 코드 중복을 줄이는데 큰 도움을 주며,<br/>여러 클래스에서 사용할 내용을 하나의 mixin에서 한 번에 제공하도록 설계되어 있습니다.
- with 키워드로 클래스 명 혹은 부모 클래스 명 뒤에 기재하며, 복수 사용이 가능합니다.
- 추상 메서드/추상 변수를 가질 수 있으며, 믹스인 사용시 꼭 이를 구현해야 합니다.
- mixin class로 구성해서 상속과 믹스인 두 가지 용도로 사용하는 것 또한 가능합니다.
```dart
mixin class Both {}

class UseAsMixin with Both {}
class UseAsSuperclass extends Both {}
```

# modifier combinations
- modifier는 여러개를 조합해서 붙일 수 있습니다. 그중 아래 표의 조합만이 가능하며, 이는 클래스의 기능을 더 명확하게 제한해줍니다.
- 아까보다 조금 더 긴 표를 보겠습니다. 아래 조합만 valid하며, 표기되지 않은 조합(mixin abstract등)은 사용이 불가능 합니다.
- 이 조합을 통해 우리는 각각의 클래스가 상속, 구현, 믹스인이 가능한지 불가능한지를 정의합니다.<br/>
abstract interface class등을 통해 구현만 가능한 `pure`한 인터페이스를 정의하는 것도 가능합니다.
  <br/>




| Class Type                    |Construct | Extend | Implement | Mixin | Exhausive |
|-------------------------------|------------|---------------|------------------|--------------|--------|
| `class`                       | ✔          | ✔             | ✔                |              |        |
| `base class`                  | ✔          | ✔             |                  |              |        |
| `interface class`             | ✔          |               | ✔                |              |        |
| `final class`                 | ✔          |               |                  |              |        |
| `sealed class`                |            |               |                  |              | ✔      |
| `abstract class`              |            | ✔             | ✔                |              |        |
| `abstract base class`         |            | ✔             |                  |              |        |
| `abstract interface class`    |            |               | ✔                |              |        |
| `abstract final class`        |            |               |                  |              |        |
| `mixin class`                 | ✔          | ✔             | ✔                | ✔            |        |
| `base mixin class`            | ✔          | ✔             |                  | ✔            |        |
| `abstract mixin class`        |            | ✔             | ✔                | ✔            |        |
| `abstract base mixin class`   |            | ✔             |                  | ✔            |        |
| `mixin`                       |            |               | ✔                | ✔            |        |
| `base mixin`                  |            |               |                  | ✔            |        |


# Fragile Base Class Problem

## 문제 정의 
확장(extends)기능은 코드를 재사용할 수 있는 좋은 기능이나, 클래스 간의 의존 관계를 만들기 때문에 가끔 문제를 일으키고는 합니다.<br/>
**Fragile Base Class Problem**은 객체 지향 프로그래밍에서 발생하는 객체 지향 프로그래밍 시스템의 근본적인 아키텍처 문제로,<br/>
**`부모 클래스`의 변경이 `자식 클래스`에 side-effect를 발생시키는 현상을 말합니다.**

### 예시 코드 1
```dart
class Parent {
  void methodA() {
    print("A");
    methodB();  // methodB 호출
  }
  
  void methodB() { 
    print("B");
  }
}

class Child extends Parent {
  @override
  void methodB() {
    super.methodB();  // 부모의 methodB도 호출
    print("Child's B");
  }
}
```
가장 쉬운 예시로 위 코드를 보겠습니다.<br/>
이 예시 코드에서 만약 부모 클래스의 methodA가 수정되어 methodB를 두 번 호출하도록 변경된다면 자식 클래스의 동작도 예상치 못하게 변경됩니다.<br/>
예를 들어 Child.methodA();를 호출한다면 Child는 알지 못하는 변경에 의해 본인의 methodB를 두 번 부르게 됩니다.
<br/>
또는 이렇게 생각할 수 있습니다. <br/>
"Child의 methodB를 override 할 때, super.methodB를 호출하지 않고 새롭게 정의해서 쓰면 되는게 아닌가?"<br/>
이 경우 만약 Parent 클래스의 methodB를()가 중요한 검증 로직을 포함하고 있었다면 이를 호출하지 않으므로써 필수적인 알고리즘 단계를 건너뛰는 오류를 야기할 수 있으며, 이는 "계약 위반(Contract Violation)"이라는 새로운 문제가 발생하는 것 입니다.<br/>
<br/>
### 예시 코드 2
```dart
class Parent {
  void validateBalance() {
    originMethod();
    checkNetwork();  // 나중에 추가된 필수 네트워크 체크
    print("checkNetwork후에 잔액 검증 하는 메서드");
  }
  
  void checkNetwork() { // 새로 추가된 필수 검증 메서드
    print("네트워크 상태 확인하는 메서드");
  }
}
```
조금 더 구체적인 예시는 "부모 클래스에 새로운 의존성이 추가되었을 때" 발생하는 문제 입니다.<br/>
Parent 클래스에 checkNetwork라는 새로운 의존성이 생겨버리면 Parent를 상속하는 많은 클래스들은 이를 알 수 없습니다.<br/>
이에 따라서 Child 클래스에서 side-effect가 발생하기 쉽습니다.<br/>

## 해결 방안

### 1. 템플릿 메서드 패턴 사용
```dart
abstract class Parent {
  // 템플릿 메서드 - final로 선언하여 오버라이드 방지
  final void methodA() {
    step1(); // 하위 클래스에서 구현
    if (shouldExecuteStep2()) {  // 선택적 기능을 제공하는 hook메서드
      step2();  // 하위 클래스에서 구현
    }
    _commonStep(); // 공통 기능은 접근 불가능하도록 
  }
  
  void step1(); // 추상 메서드 - 반드시 구현해야 함
  void step2() { // 선택적으로 구현 가능한 메서드 (훅 메서드)
    print("기본 step2 실행"); // 기본 구현을 제공하고 사용시 추가 구현
  }

  bool shouldExecuteStep2() { // 훅 메서드 - 실행 여부 결정
    return false;  // 기본값
  }

  void _commonStep() {  // private 메서드 - 변경 불가능한 공통 기능
    print("변경 불가능한 공통 기능 실행");
  }
}

// 구현 클래스 1
class Child1 extends Parent {
  @override
  void step1() {
    print("Child1의 step1 구현");
  }
}

// 구현 클래스 2
class Child2 extends Parent {
  @override
  void step1() {
    print("Child2의 step1 구현");
  }
  
  @override
  void step2() {
    print("Child2의 커스텀 step2 구현");
  }
    
  @override
  bool shouldExecuteStep2() {
    return true;  // step2 실행하도록 구현
  }
}
```
템플릿 메서드 패턴은 알고리즘의 구조를 메서드에 정의하고, 일부 단계를 하위 클래스에서 구현할 수 있도록 하는 패턴입니다.<br/> 
알고리즘의 뼈대는 상위 클래스에서 정의하고, 일부 단계의 구현은 하위 클래스에 위임하는 형태입니다.<br/>
"템플릿"을 통해 알고리즘의 구조를 명확하게 하고 하위 클래스에서 수정 가능한 부분이 명확히 정의하기 때문에<br/>
공통 기능의 무결성이 보장되며 확장은 용이하되 기존 구조는 깨지지 않습니다.
(패턴 자체에 대해서는 다른 글에서 더 자세히 다뤄보겠습니다)

### 2. 컴포지션 사용
```dart
class WithdrawValidator {
  void validate() {
    print("잔액 검증");
  }
}

class BankAccount {
  final WithdrawValidator validator; // 컴포지션으로, 상속이 아니라 내부변
  
  BankAccount(this.validator);
  
  void withdrawMoney() {
    validator.validate();
    processingWithdraw();
  }
}
```
컴포지션은 "has-a" 관계를 만드는 것으로, 상속("is-a" 관계) 대신 인스턴스를 내부에 포함시켜 사용하는 방식입니다.<br/>
상속과 달리 결합도가 매우 낮아 내부 구현 변경에 '덜'취약하며 테스트가 용이한 장점이 있습니다. 


### 3. 인터페이스 정의
```dart
abstract class BalanceValidator {
  void validate();
}

class StandardValidator implements BalanceValidator {
  @override
  void validate() {
    print("표준 잔액 검증");
  }
}

class CustomValidator implements BalanceValidator {
  @override
  void validate() {
    print("커스텀 잔액 검증");
  }
}
```
BalanceValidator라는 부모 클래스의 구현을 포기하고 "인터페이스로 사용"하여 상세 구현을 모두 자식 클래스에서 한다면 해당 문제는 해결됩니다.

#### 정리
상속에서 오는 고질적인 문제인 Fragile Base Class Problem를 피하기 위해서는 대신 위와 같은 설계 패턴을 사용하여 더 안전하고 유연한 구조를 만드는 것이 좋습니다.




