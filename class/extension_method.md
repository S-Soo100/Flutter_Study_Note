# Dart 언어의 `extension method` 이해하기


한글로 번역하면 `확장 메서드`. 
API, 외부 라이브러리, 내가 쓰는 기본 class들에 커스텀해서 추가 기능을 붙이고 싶다면? </br>
바로 `extension method`를 사용하면 됩니다.</br>
혹시 `something.toString();` 을 사용해 봤다면? 이미 반 쯤은 어떤 느낌인지 알고 있을 것입니다.</br>
</br>
🍀 공식 문서 : https://dart.dev/language/extension-methods
</br>
</br>

---

## 📌 1. `extension method`란?

`extension method`는 이미 존재하는 클래스의 내부 구현을 변경하지 않고 추가 기능을 붙일 수 있는 기능이다. </br>
Dart 언어에서 제공하는 이 기능은 라이브러리나 프레임워크의 클래스를 확장하거나, 기존 코드에 영향 없이 추가적인 기능을 구현하고자 할 때 유용함니다.
</br>
`extension method`의 핵심은 코드 가독성 향상과 재사용성 향상 입니다.
</br>

---

</br>

## 📌 2. `extension method` 사용시 주의사항

- 확장 메서드는 컴파일 시점의 타입 정보를 기반으로 해당 타입을 확장 시킵니다. 이는 정적(static) 성질을 가집니다.
- 그러므로 `dynamic` 타입으로 선언한 변수에는, 이후 코드 내에서 해당 변수의 타입이 명확하더라도 확장 메서드 사용이 불가능합니다.
  - `dynamic`은 동적 타입으로 컴파일시 타입 체크를 하지 않기 떄문입니다.
  - 하지만 `var`타입으로 선언하면 초기화 시점에 타입을 결정하고 동작하기 때문에 확장 메서드 사용이 가능합니다.
- 추가적으로 이러한 정적인 성격 덕분에 static함수 호출과 같이 런타임 시점에서의 동적 디스패치나 추가 검색을 필요로 하지 않아 성능이 뛰어납니다.
  

---
</br>

## 📌 3. `extension method`의 실전 활용

### 예시 코드: `String` 클래스 확장하기

```dart
extension StringExtension on String {
  // 이제 모든 String값에 something.reverse(); 를 사용할 수 있습니다.
  String reverse() {
    return split('').reversed.join('');
  }
}

void main() {
  String text = 'Flutter';
  print(text.reverse());  // rettulF
}
```

- extension StringExtension on String: String 클래스에 reverse()라는 새로운 메서드를 추가하는 확장(extension)을 정의합니다.
- reverse() 메서드: split(), reversed, join() 메서드를 활용해 문자열을 뒤집는 로직을 구현한 메서드입니다.
- 이 확장 메서드를 통해, 기존의 String 클래스에 새로운 기능을 추가할 수 있게 되며, text.reverse()와 같이 직관적으로 사용할 수 있습니다.
</br>
</br>

### 확장 메서드의 getter, setter, operator 사용

- 추가적으로 getter, setter, operator의 정의도 가능합니다(와우!).
- 예를 들어, Point라는 model클래스를 하나 만들고 이 Point의 내부 변수를 get, set하는 메서드와
  Point 2개를 합치는 operator를 만든다고 해보겠습니다.
- 특히 오퍼레이터는 API호출 등에서 여러번 하는 작업을 굉장히 간단하게 만들어 줍니다.
  (모델 클래스를 가공하거나, 합치는 등)
- 이를 통해 코드의 재사용성과 유지보수성을 높일 수 있습니다.
</br>

```dart
class Point {
  int x;
  int y;

  Point(this.x, this.y);
}

extension PointExtension on Point {
  // Getter: 원점(0, 0)으로부터의 거리
  double get distanceFromOrigin {
    return (x * x + y * y).toDouble().sqrt();
  }

  // Setter: 거리 값을 기반으로 위치 설정
  set distanceFromOrigin(double distance) {
    x = (distance / 1.414).round();  // 간단히 대각선 기준으로 값 설정
    y = (distance / 1.414).round();
  }
 // + 연산자 오버로딩: 두 Point 객체의 x, y 값을 더하기
  Point operator +(Point other) {
    return Point(x + other.x, y + other.y);
  }
}

void main() {
  Point p = Point(3, 4);
  print(p.distanceFromOrigin);  // 5.0

  p.distanceFromOrigin = 10;
  print('${p.x}, ${p.y}');  // 7, 7

  Point p1 = Point(3, 4);
  Point p2 = Point(1, 2);

  Point p3 = p1 + p2;  // + 연산자 오버로딩
  print('${p3.x}, ${p3.y}');  // 4, 6
}
```
</br>

### static helper method를 확장 메서드에서 사용
- static helper 메서드 add(int a, int b)를 정의한 후, 확장메서드 `IntExtension` 확장을 통해 int 타입에 addWithHelper()라는 확장 메서드를 추가했습니다.
- 이 메서드에 내부적으로 static helper method인 add를 호출하여 두 숫자를 더하도록 구현합니다.
- 이러한 예시 구조처럼 확장 메서드에서 '정적 헬퍼 메서드 (static helper method)'를 포함할 수 있습니다.
```dart
class MathUtils {
  // static helper method: 두 값을 더하는 함수
  static int add(int a, int b) {
    return a + b;
  }
}

extension IntExtension on int {
  // 확장 메서드: 두 숫자를 더하기 위해 static helper method 사용
  int addWithHelper(int other) {
    return MathUtils.add(this, other);  // static helper method 호출
  }
}

void main() {
  int a = 10;
  int b = 20;

  // 확장 메서드 호출
  print(a.addWithHelper(b));  // 출력: 30
}
```
- 이를 통해 확장 메서드가 단순히 추가적인 동작을 제공하는 역할을 하도록 하고, 복잡한 계산 로직은 helper 메서드에 맡김으로써 책임을 분리할 수 있습니다.
  

### argument가 있는 경우

```dart
extension StringExtension on String {
  // int times이라는 argument를 받아서 여러 번(times) 문자열을 출력하는 확장 메서드
  String repeat(int times) {
    return this * times;
  }
}

void main() {
  String text = 'Flutter';
  print(text.repeat(3));  // FlutterFlutterFlutter
}
```

- 위 코드처럼 `someString.repeat(3)` 의 형태로 매개변수를 담아서 함수를 사용할 수 있습니다.

</br>


## 📌 4. 충돌 대응

### 같은 클래스에 대한 여러개의 확장들 간의 중복 문제 해결
- 같은 클래스에 대한 extension들에서 중복된 method name을 사용한다면 extension클래스 이름을 명시해서 어떤 확장에서 그 기능을 부르는지 명학하게 할 수 있습니다.
- 예를 들어서, String에 대한 확장 ExtensionA와 ExtensionB에서 change()라는 메서드를 정의했다면, 아래와 같이 구분해서 사용합니다.
```dart
  String a = "a";

  ExtensionA(a).change();
  ExtensionB(a).change();
```
- 이와 같이 특정 확장을 명시적으로 호출함으로써 중복된 메서드 이름 충돌 문제를 해결할 수 있습니다.
  
</br>

### show와 hide의 사용
- show는 특정 요소만 가져오고 나머지는 가져오지 않도록 할 때 사용합니다. 예를 들어, 한 라이브러리에서 특정 함수나 클래스만 필요한 경우에 사용합니다.
```dart
import 'myExtensionA.dart' show change();

void main() {
  String a = "a";
  a.change();
  // a.reverse();  // myExtensionA에서 현재 reverse()는 import되지 않음
}
```
- hide는 특정 요소만 제외하고 나머지는 모두 가져올 때 사용합니다. 라이브러리의 대부분을 사용하고 싶지만, </br> 일부 함수나 클래스는 제외하고 싶을 때 유용합니다.
```dart
// main.dart 파일
import 'myExtensionA.dart' hide reverse;

void main() {
  String a = "a";
  a.change();  // change메서드는 import되었음
  // a.reverse();  // 에러: MyExtensionA import되지 않았음
}
```
  
- 추가적으로, `as` 키워드를 이용해 임의 이름으로 지정해서, `ExA(a).reverse()`로 호출 할 수 있습니다.

---

## 📌 5. Unnamed Extension
- extension class의 이름을 지정하지 않을 수 있습니다.
- 하지만 이 경우, 해당 extension class가 선언된 라이브러리 내에서만 그 확장 기능들을 사용할 수 있습니다.
- 한 파일 내에서만 제한적으로 어떠한 확장 기능을 사용할 수 있게 하려면 `Unnamed Extension(익명 확장 클래스)`를 사용하자!
```dart
extnsion on String {
  // 띄어쓰기를 제거하고 비었는지 확인함
  bool get isBlank => trim.isEmpty; 
}
```

---

## 📌 6. Generic Extension: 제너릭<T> 확장 구현
- Dart에서 확장 메서드는 특정 클래스에 새로운 기능을 추가하는 데 사용되는데, 제너릭을 사용하면 여러 타입에 대해 확장 메서드를 정의할 수 있으며, 다양한 타입에 대해 공통적인 로직을 적용할 수 있으므로, 코드의 중복을 줄이고 유지보수를 쉽게 할 수 있습니다.
  
- 제너릭 확장을 사용할 때 특정 타입에 대해 동작하도록 제약을 걸 수 있습니다. 예를 들면 List는 List인데 num(double, int)만 가지도록 하는 것 처럼 말이죠
- 제너릭 타입에 제약 조건을 추가하면 특정 상위 클래스나 인터페이스에 속하는 타입만 확장할 수 있습니다.
```dart
extension ListAverage<T extends num> on List<T> {
  // 숫자 리스트의 평균을 계산하는 확장 메서드
  double average() {
    if (this.isEmpty) return 0;
    return this.reduce((value, element) => value + element) / this.length;
  }
}

void main() {
  List<int> intList = [1, 2, 3, 4, 5];
  List<double> doubleList = [1.5, 2.5, 3.5];

  print(intList.average());  // 출력: 3.0
  print(doubleList.average());  // 출력: 2.5

  // List<String> stringList = ['a', 'b'];  // 에러: num 타입이 아님
}
```

---

