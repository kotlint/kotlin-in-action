# 클래스, 객체, 인터페이스

### 클래스 초기화 : 주 생성자와 초기 블록

~~~kotlin
class User(val nickname: String)
~~~

코틀린에서는 java 와 다르게 클래스의 생성자를 파라미터로 가질수 있다. 

위와 같이 쿨래스 이름 뒤에 오는 **(val nickname: String)** 의 형태를 주생성자라 한다.

- 주 생성자의 목적
  - 생성자 파라미터를 지정
  - 지정한 생성자 파라미터에 의해 초기화 되는 프로퍼티를 정의
    
- 명시적인 표현

~~~kotlin
class User constructor(_nickname: String) {

    val nickname: String = _nickname

    init { // 초기화 블록
        nickna = _nickna
    }
}
~~~

위 코드와같이 작성할 수 있지만, 밑과 같은 코드로 간소화 시킬수 있다.

~~~kotlin
class User constructor(_nickname: String) {

    val nickname: String = _nickname

}
~~~

## 키워드 constructor 와 init{ }
- constructor 키워드는 주 생성자나 부 생성자를 정의할 떄 사용
- init{ } 키워드는 초기화 블록을 시작


~~~kotlin
class User (_nickname: String) {

    val nickname: String = _nickname

}
~~~