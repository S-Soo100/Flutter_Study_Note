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

| Modifier     | construct | extend | implement | mixin | exhaustive |
|--------------|:--------:|:------:|:--------:|:-----:|:---------:|
| **class**    |    ✔     |     ✔  |      ✔  |       |           |
| **base**     |    ✔     |   ✔    |         |       |           |
| **interface**|    ✔     |        |    ✔    |       |           |
| **final**    |    ✔     |        |         |       |           |
| **sealed**   |          |        |         |       |     ✔     |
| **abstract** |          |   ✔    |   ✔     |       |           |
| **mixin**    |    ✔     |     ✔  |     ✔   |     ✔ |           |

([공식문서의 Valid combinations](https://dart.dev/language/modifier-reference))

### 기능의 한줄 설명 
- construct: 클래스 인스턴스화
- extend: 다른 클래스를 부모로 상속
- implement: 인터페이스 메서드 구현
- mixin: 여러 클래스에 공통사용 가능한 기능을 추가
- exhaustive: 본인을 상속한 모든 하위 클래스를 처리하는 `완전성 검사`
