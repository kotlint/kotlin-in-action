# 람다로 프로그래밍

- 람다는 다른 함수에 넘길 수 있는 작은 코드 조각이다.

람다는 공통 코드 구조를 라이브러리 함수로 쉽게 뽑아낼 수 있다.(Kotlin 라이브러리는 람다를 아주 많이 사용한다.)

### 람다 소개 : 코드 블록을 함수 인자로 넘기기

- 무명 내부 쿨래스로 리스너 구현하기
~~~java
button.setOnClickListener (new OnClickListener() {
    @Override
    public coid onClick(View view){
        /*클릭시 수행할 동작*/
    }
});
~~~

- 람다식 구현
~~~java
button.setOnClickListener {/*클릭시 수행할 동작*/}
~~~

### 람다와 컬렉션

### 람다식 문법

### 현재 영역에 있는 변수에 접근

### 멤버 참조

### 컬렉션 함수형 API

### 필수적인 함수 : filter 와 map

### all, any, count, find : 컬렉션에 술어 적용

### groupBy : 리스트를 여러 그룹으로 이뤄진 맵으로 변경

### flatMap 과 flatten : 중첨된 컬렉션 안의 원소 처리

### 지연 계산(lazy) 컬렉션 연산

### 시퀀스 연산 실행 : 중간 연산과 최종 연산

### 시퀀스 만들기

### 자바 함수형 인터페이스 활용

### 자바 메서드에 람다를 인자로 전달

### SAM 생성자 : 람다를 함수형 인터페이스로 명시적으로 변경

### 수신 객체 지정 람다: with 와 apply

### apply 함수