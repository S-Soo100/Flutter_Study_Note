
공식문서 : https://dart.dev/language/class-modifiers

클래스 모디파이어(클래스 수정자 혹은 제어자, Class Modifiers)란, class 혹은 mixin이 라이브러리 내/외부로 사용되는 방식을 결정한다.
모디파이어 키워드는 class 혹은 mixin 바로 앞에 배치하며 mixin은 그 자체로 제어자 키워드중 하나이기도 하다.
그리고 이는 클래스의 사용 방식에(프로젝트 내에서 그 클래스가 사용되기를 '원하는'방식에) 직접적으로 관여한다.


| Modifier   | abstract | final | sealed | base | interface | mixin |
|------------|:--------:|:-----:|:------:|:----:|:---------:|:-----:|
| **abstract** |    ✔    |       |        |      |     ✔     |       |
| **final**    |         |   ✔   |        |      |           |       |
| **sealed**   |         |       |   ✔    |      |           |       |
| **base**     |         |       |        |  ✔   |           |       |
| **interface**|    ✔    |       |        |      |     ✔     |       |
| **mixin**    |         |       |        |      |           |   ✔   |
