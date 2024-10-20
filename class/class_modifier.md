# Dart 언어의 `class modifier` 이해하기
<img src="https://github.com/user-attachments/assets/fa72a18f-cec1-43f1-ab4f-d3623e1e30bb" width="70%" alt="banner_ext" align="center"/>

클래스 모디파이어(클래스 수정자 혹은 제어자, Class Modifiers)란, class 혹은 mixin이 라이브러리 내/외부로 사용되는 방식을 결정합니다.
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

