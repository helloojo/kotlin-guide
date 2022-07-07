# 8장 클래스 계층 이해하기

## 8.1 상속

### 8.1.1 하위 클래스 선언

* 클래스 선언 앞에 open을 붙이면 해당 클래스는 상속 가능
* 클래스 주생성자 뒤에 `: {클래스이름}` 으로 상속을 선언할 수 있다.
* 데이터 클래스는 다른 클래스가 상속할 수 없음
* 인라인 클래스는 다른 클래스 상속 불가, 다른 클래스가 상속하는 것도 불가능
* 객체(object), 동반 객체(companion object)는 다른 클래스 상속 가능
    * 다른 클래스는 객체 상속 불가능
* 메서드 오버라이딩은 상위 클래스의 메서드 앞에 open 키워드를 붙여야함
    * 하위 클래스는 메서드 선언 앞에 override 키워드를 붙여야함

```kotlin
open class Base {
    open fun func() {
    }
    fun func1() {
    }
}

class Derive : Base() {
    override fun func() {
        super.func()
    }
    override fun func1() {        // 에러, func1 은 final 메서드
    }
}

class DeriveDerive : Derive() {
    final override fun func() {       // 오버라이드한 함수에 final 붙이면 더이상 오버라이딩 불가능
        super.func()
    }
}
```

* 프로퍼티도 오버라이드 가능

### 8.1.2 하위 클래스 초기화

* 하위 클래스의 인스턴스를 생성하는 동안 상위 클래스의 초기화 코드 호출 필요함
    * 코틀린에서는 최상위 클래스의 생성자(Any)부터 순서대로 생성자를 호출한다.

```kotlin
open class Base {
    val a: String
    val b: String

    constructor(ab: String) {
        val split = ab.split(" ")
        this.a = split[0]
        this.b = split[1]
    }
}

class Derive : Base {
    val c: String

    constructor(ab: String, c: String) : super(ab) { // 상위 클래스 부생성자 호출
        this.c = c;
    }
}
```

* 코틀린의 상위 클래스 생성자 호출은 반드시 위임 호출로 정의해야 한다.
    * 클래스에 주 생성자가 있으면 부생성자 위임 호출은 불가능 하다.

### 8.1.3 타입 검사와 캐스팅

* is 연산자를 이용 타입 검사 가능
    * null 값은 nullable 타입의 인스턴스로 간주됨
* is 연산자를 사용해 스마트 캐스트도 사용 가능

```kotlin
val objects = arrayOf("1", 2, "3", 4)

var sum = 0

for (obj in objects) {
    if (obj is Int) {
        sum += obj // 여기서는 obj의 타입을 `Int`로 세분화한다
    }
}
println(sum) // 6
```

```kotlin
val objects = arrayOf("1", 2, "3", 4)
var sum = 0

for (obj in objects) {
    when (obj) {
        is Int -> sum += obj            // 여기서 obj는 Int 타입이다
        is String -> sum += obj.toInt() // 여기서 obj는 String 타입이다
    }
}
println(sum) // 10
```

* as 연산자를 사용해서는 타입 캐스트 가능
    * as(안전하지 않은 버전), as?(안전한 버전) 두가지 제공
    * 안전하지 않은 버전은 타입 캐스트 불가능할경우 예외 반환
    * 안전한 버전은 null 반환

```kotlin
val o: Any = 123
println((o as Int) + 1)              // 124
println((o as? Int)!! + 1)           // 124
println((o as? String ?: "").length) // 0
//println((o as String).length)        // java.lang.ClassCastException
```

### 8.1.4 공통 메서드

* kotlin.Any는 코틀린 클래스 계층의 루트
    * 모든 클래스는 Any를 직간접적으로 상속하고 있음
* Any는 equals, hashCode, toString 세개의 공통 메서드를 제공함
* 해당 메서드는 모든 코틀린 클래스에서 오버라이드할 수 있음

## 8.2 추상 클래스와 인터페이스

### 8.2.1 추상 클래스와 추상 멤버

* abstract class 키워드로 정의 가능
* 인스턴스화 할 수 없고, 다른 클래스가 상속할 수 있는 역할이라는 의미
* 추상 클래스도 생성자를 가질 수 있지만, 하위 클래스는 위임 호출로만 호출 할 수 있음

```kotlin
abstract class Entity(val name: String)

class Person : Entity {
    constructor(name: String) : super(name)
    constructor(
        firstName: String,
        familyName: String
    ) : super("$firstName $familyName")
}
```

* 추상 클래스의 추상 멤버도 정의 가능
    * 각 멤버 정의 앞에는 abstract 키워드가 붙어야함
* 추상 클래스를 상속하는 클래스는 추상 멤버를 모두 오버라이드 해야한다.

### 8.2.2 인터페이스

* 추상 메서드, 프로퍼티를 포함하지만, 인스턴스, 생성자를 만들 수 없는 타입
* interface 키워드로 정의한다.
* 인터페이스는 클래스, 다른 인터페이스의 상위 타입이 될 수 있음
* 인터페이스의 추상 멤버, 프로퍼티에 구현을 추가해 기본 구현을 정의할 수도 있음
    * 상태를 가지거나, 초기화 코드, 위임이 붙은 프로퍼티는 정의 불가
* 인터페이스는 추상 타입이지만, 추상 클래스처럼 생성자 정의는 불가능
* 인터페이스는 다중 상속 가능

### 8.2.3 봉인된 클래스와 인터페이스

* 제한된 범위에서 상속을 허용하는 클래스
    * 내포된 클래스, 객체, 같은 파일 안에서 최상위 클래스로 정의되어야 봉인된 클래스 상속 가능
    * 코틀린 1.5부터는 같은 패키지에 있는 봉인된 클래스, 인터페이스도 상속 가능
* 클래스 정의 앞에 sealed 키워드를 붙여 선언 가능
* 봉인된 클래스는 인스턴스 생성 불가능
    * 생성자의 가시성 기본값 private(변경하면 컴파일 에러 발생)
* 봉인된 클래스를 상속한 하위 클래스가 open class 인 경우, 해당 클래스는 상속할 수 있음
* 코틀린 1.1 부터는 봉인된 클래스가 다른 클래스 상속도 가능

### 8.2.4 위임

* 코틀린 클래스는 기본적으로 final
* 상속이 불가능한 클래스를 확장, 변경이 필요할 경우 위임 패턴을 사용할 수 있다.
* 상위 인터페이스 이름 뒤에 by 키워드를 붙여 위임할 인스턴스를 지정해 위임을 사용할 수 있다.

```kotlin
interface PersonData {
    val name: String
    val age: Int
}

open class Person(
    override val name: String,
    override val age: Int
) : PersonData

class Alias(
    private val realIdentity: PersonData,
    private val newIdentity: PersonData
) : PersonData by newIdentity {     // PersonData 의 name, age는 기본적으로 newIdentity의 구현을 따라간다.
    override val age: Int get() = realIdentity.age    // 새로운 구현을 override할 수 있다.
}

fun main() {
    val valWatts = Person("Val Watts", 30)
    val johnDoe = Alias(valWatts, Person("John Doe", 25))
    println(johnDoe.age) // 30
}
```

```kotlin
class Alias(
    private val realIdentity: PersonData,
    newIdentity: PersonData         // 필드가 아닌 생성자 파라미터로도 위임 사용가능
) : PersonData by newIdentity       // 컴파일러는 위임 값을 저장하는 필드를 자동으로 생성함
```

```kotlin
class Alias(
    private val realIdentity: PersonData
) : PersonData by newIdentity { // error: unresolved reference: newIdentity
    val newIdentity = Person("John Doe", 30)      // 클래스 본문에 정의된 프로퍼티는 위임 사용 불가
}
```

* 다음과 같이 객체 식과 조합해 다른 구현을 제공할 수도 있다.

```kotlin
fun PersonData.aliased(newIdentity: PersonData) =
    object : PersonData by newIdentity {
        override val age: Int get() = this@aliased.age      // age는 PersonData의 수신 객체의 age 반환
    }

fun main() {
    val valWatts = Person("Val Watts", 30)
    val johnDoe = valWatts.aliased(Person("John Doe", 25))

    println("${johnDoe.name}, ${johnDoe.age}") // John Doe, 30
}
```

* 인터페이스 멤버를 구현할 때만 위임 사용 가능

```kotlin
class Alias(
    private val realIdentity: PersonData,
    private val newIdentity: PersonData
) : Person by newIdentity // error: only interfaces can be delegated to

```

* 위임을 사용하면 객체 합성, 상속의 이점 활용 가능
* 코틀린에서는 상속보다는 합성을 사용하도록 권장하고 있다.
