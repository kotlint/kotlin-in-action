# Best Practice in Kotlin

- __Table of Contents__
  - [임시 구조체(Ad-Hoc Structs)](https://github.com/kotlint/kotlin-in-action/new/main#%EC%9E%84%EC%8B%9C-%EA%B5%AC%EC%A1%B0%EC%B2%B4ad-hoc-structs)
  - [if 대신 as + smart cast + elvis operator 를 응용하자](https://github.com/kotlint/kotlin-in-action/new/main#if-%EB%8C%80%EC%8B%A0-as--smart-cast--elvis-operator-%EB%A5%BC-%EC%9D%91%EC%9A%A9%ED%95%98%EC%9E%90)

## 임시 구조체(Ad-Hoc Structs)

- listOf, mapOf 및 infix 함수는 모두 JSON 과 같은 구조체를 빠르게 구성하는 데 사용할 수 있다.
- 이러한 임시 구조체는 테스트 코드를 작성할 때 유용하다. 테스트를 위한 JSON 객체를 유연하게 생성할 수 있다.

```kotlin
val customer = mapOf(
        "name" to "Clair Grube",
        "age" to 30,
        "languages" to listOf("german", "english"),
        "address" to mapOf(
                "city" to "Leipzig",
                "street" to "Karl-Liebknecht-Straße 1",
                "zipCode" to "04107"
        )
)
```

## if 대신 as + smart cast + elvis operator 를 응용하자

```kotlin
// Don't
if (service !is CustomerService) {
    throw IllegalArgumentException("No CustomerService")
}
service.getCustomer()
```

```kotlin
// Do
service as? CustomerService ?: throw IllegalArgumentException("No CustomerService")
service.getCustomer()
```
