### 디자인 패턴을 사용하는 이유

0. 객체지향적 프로그래밍을 유지시키기 위함.
1. 개발자끼리의 암묵적 약속을 형성 할 수 있음 (즉, 패턴에 맞게 설계된 코드라면 특별한 코드 설명 없이 개발자들은 패턴을 인식 할 수 있음.)

### 디자인 패턴 사용 시 주의사항

0. 프로그래밍에서 디자인 패턴은 '수단'이 되어야지, 그 자체로 '목적'이 되어서는 안됨.
ex) 계산기를 만들 때 : plus, minus 같은 기능 구현 후, 리팩토링으로 팩토리 메서드 사용 --> OK!
                      팩토리 메서드를 지키기 위해 plus, minus의 일부 기능 삭제 --> NEVER!