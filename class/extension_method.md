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
</br>

## 📌 2. `extension method`의 실전 활용

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


##📌 충돌 대응

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
  
</br>
