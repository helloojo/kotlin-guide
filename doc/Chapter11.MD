# 11장 도메인 특화 언어

## 11.1 연산자 오버로딩

* 코틀린 내장 연산자에 새로운 의미를 부여할 수 있게 해주는 언어 기능
* operator fun 키워드로 확장 함수, 멤버 함수를 정의해 연산자 오버로딩을 할 수 있다.

### 11.1.1 단항 연산

| 식   | 의미             |
|-----|----------------|
| +e  | e.unaryPlus()  |
| -e  | e.unaryMinus() |
| !e  | e.not()        |

### 11.1.2 증가와 감소

| 식   | 의미    |
|-----|-------|
| ++  | inc() |
| --  | dec() |

### 11.1.3 이항 연산

| 식       | 의미                  |
|---------|---------------------|
| a + b   | a.plus(b)           |
| a - b   | a.minus(b)          |
| a * b   | a.times(b)          |
| a / b   | a.div(b)            |
| a % b   | a.rem(b)            |
| a .. b  | a.rangeTo(b)        |
| a in b  | b.contains(a)       |
| a !in b | !b.contains(a)      |
| a < b   | a.compareTo(b) < 0  |
| a <= b  | a.compareTo(b) <= 0 |
| a > b   | a.compareTo(b) > 0  |
| a >= b  | a.compareTo(b) >= 0 |

### 11.1.4 중위 연산

* to 같은 중위 연산은 infix 키워드를 함수 앞에 붙여야 한다.

```kotlin
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

### 11.1.5 대입

| 식      | 의미             | 의미               |
|--------|----------------|------------------|
| a += b | a = a.plus(b)  | a.plusAssign(b)  |
| a -= b | a = a.minus(b) | a.minusAssign(b) |
| a *= b | a = a.times(b) | a.timesAssign(b) |
| a /= b | a = a.div(b)   | a.divAssign(b)   |
| a %= b | a = a.rem(b)   | a.remAssign(b)   |

### 11.1.6 호출과 인덱스로 원소 찾기

* invoke() 함수를 정의해 함수처럼 값을 사용할 수 있다.

```kotlin
operator fun <K, V> Map<K, V>.invoke(key: K) = get(key)

val m = mapOf("I" to 1, "V" to 5, "X" to 10)
println(m("V")) // 5
println(m("L")) // null
```

* invoke를 활용해 동반 객체를 팩토리로 만드는 방법을 사용할 수 있다.

```kotlin

class Rational private constructor(
    val sign: Int,
    val num: Int,
    val den: Int
) {
    companion object {
        fun of(num: Int, den: Int = 1): Rational {
            return Rational(sign, numAbs / gcd, denAbs / gcd)
        }
    }
}

operator fun Rational.Companion.invoke(num: Int, den: Int = 1) =
    of(num, den)

fun main() {
    val r = Rational(1, 2)
}
```

* 인덱스 연산자는 get, set(대입) 함수 호출로 변환된다.

### 11.1.7 구조 분해

* componentN() 함수를 멤버 함수, 확장 함수로 정의해 구조 분해를 정의할 수 있다.

### 11.1.8 이터레이션

```kotlin
operator fun <T> TreeNode<T>.iterator() = children.iterator()
```

* iterator 함수를 정의해 for 루프에서 타입을 사용할 수 있다.

## 11.2 위임 프로퍼티

### 11.2.1 표준 위임들

#### lazy

* by lazy 키워드로 지연 계산 프로퍼티를 표현할 수 있다.
* 기본적인 lazy 구현은 스레드 안전한 구현
* 다른 lazy 버전을 사용할 수도 있다.

```kotlin
val text by lazy { File("README.md").readText() }

private val lock = Any()
val text2 by lazy(lock) { File("README.md").readText() }    // 동기화 객체 지정 가능

val myValue by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {    // 기본 lazy 구현, 한번에 한 스레드만 프로퍼티를 초기화할 수 있다.
    println("Initializing myValue")
    123
}

val myValue2 by lazy(LazyThreadSafetyMode.PUBLICATION) {    // 초기화 함수가 여러 번 호출될 수 있지만, 처음 도착하는 결과가 프로퍼티의 값이 되도록 프로퍼티 접근을 동기화한다.
    println("Initializing myValue")
    123
}

fun main() {
    val x by lazy(LazyThreadSafetyMode.NONE) { 1 + 2 }  // 프로퍼티 접근을 동기화하지 않는다. 스레드 안전을 보장할 수 없다.
    println(x) // 3
}
```

#### notNull

* 프로퍼티 초기화를 미루면서 널이 아닌 프로퍼티를 정의할 수 있게 해주는 위임
* lateinit 프로퍼티와 기본적으로 의미가 동일하다.
    * lateinit 키워드가 더 간결하고 성능도 좋아 lateinit을 사용하는 것이 낫다.
    * 하지만 원시 타입은 lateinit을 사용하지 못하기 때문에 notNull을 사용해야한다.

#### obserable

* 프로퍼티 값이 변경될 때 통지를 받을 수 있다.
* 프로퍼티의 초기값, 변경이 될 때마다 실행될 람다를 인자로 받는다.

```kotlin
class Person(name: String, val age: Int) {
    var name: String by observable(name) { property, old, new ->
        println("Name changed: $old to $new")
    }
}
```

#### vetoable

* 프로퍼티 값을 변경하기 전에 조건을 검사할 수 있다.
* 프로퍼티의 초기값과, Boolean을 반환하는 람다를 인자로 받는다.
    * 초기값은 람다로 검사되지 않는다.

```kotlin
var password: String by vetoable("password") { _, _, new ->
    if (new.length < 8) {
        println("Password should be at least 8 characters long")
        false
    } else {
        println("Password is Ok")
        true
    }
}
```

#### map 인스턴스를 프로퍼티 값으로 위임 사용

* by 키워드를 통해 프로퍼티 값 접근을 위임할 map 인스턴스를 지정해 사용핦 수 있다.
* 프로퍼티에 접근할 때는 map에서 값을 가져와 캐스팅해서 반환한다.
    * 이 때 타입 안전성이 깨질 수 있다.

```kotlin
class CartItem(data: Map<String, Any?>) {
    val title: String by data
    val price: Double by data
    val quantity: Int by data
}

fun main() {
    val item = CartItem(
        mapOf(
            "title" to "Laptop",
            "price" to 999.9,
            "quantity" to 1
        )
    )

    println(item.title)    // Laptop
    println(item.price)    // 999.9
    println(item.quantity) // 1
}

```

```kotlin
class MutableCartItem(data: MutableMap<String, Any?>) {
    var title: String by data
    var price: Double by data
    var quantity: Int by data
}

fun main() {
    val item = MutableCartItem(
        mutableMapOf(
            "title" to "Laptop",
            "price" to 999.9,
            "quantity" to 1
        )
    )

    println(item.title)    // Laptop
    println(item.price)    // 999.9
    println(item.quantity) // 1
    item.title = "My Laptop"
    println(item.title)    // My Laptop
}
```

### 11.2.2 커스텀 위임 만들기

* 커스텀 위임을 만들기 위해 연산자 함수를 정의하는 타입이 필요하다.
    * 프로퍼티 값을 읽고 쓰는 방법을 구현하는 함수

#### 읽기(getValue)

* 두가지 파라미터를 받는다.
    * receiver
        * 수신 객체 값이 들어있다.
    * property
        * 프로퍼티 선언을 표현하는 리플렉션이 들어있다.

```kotlin
class CachedProperty<in R, out T : Any>(val initializer: R.() -> T) {
    private val cachedValues = HashMap<R, T>()

    operator fun getValue(receiver: R, property: KProperty<*>): T {
        return cachedValues.getOrPut(receiver) { receiver.initializer() }
    }
}

fun <R, T : Any> cached(initializer: R.() -> T) = CachedProperty(initializer)

class Person(val firstName: String, val familyName: String)

val Person.fullName: String by cached { "$firstName $familyName" }

fun main() {
    val johnDoe = Person("John", "Doe")
    val harrySmith = Person("Harry", "Smith")

    // johnDoe  수신 객체에 최초 접근. 값을 계산해 캐시에 담음
    println(johnDoe.fullName)  // John Doe

    // harrySmith 수신 객체에 최초 접근. 값을 계산해 캐시에 담음
    println(harrySmith.fullName) // Harry Smith

    // johnDoe 수신 객체에 재접근. 캐시에서 값을 읽음
    println(johnDoe.fullName)  // John Doe

    // harrySmith 수신 객체에 재접근. 캐시에서 값을 읽음
    println(harrySmith.fullName) // Harry Smith
}
```

* `kotlin.properties.ReadOnlyProperty` 인터페이스를 사용하면 읽기 전용 커스텀 위임을 편리하게 정의할 수 있다.
    * getValue 연산자를 추상화한 버전을 정의해준다.

#### 쓰기(setValue)

* 세가지 파라미터를 받는다.
    * receiver
        * 수신 객체 값
    * property
        * 프로퍼티 선언을 표현하는 리플렉션
    * newValue
        * 프로퍼티에 저장할 새 값

```kotlin
class FinalLateinitProperty<in R, T : Any> {
    private lateinit var value: T
    operator fun getValue(receiver: R, property: KProperty<*>): T {
        return value
    }
    operator fun setValue(
        receiver: R,
        property: KProperty<*>,
        newValue: T
    ) {
        if (this::value.isInitialized) throw IllegalStateException(
            "Property ${property.name} is already initialized"
        )
        value = newValue
    }
}

fun <R, T : Any> finalLateInit() = FinalLateinitProperty<R, T>()

var message: String by finalLateInit()

fun main() {
    message = "Hello"
    println(message) // Hello
    message = "Bye"  // Exception: Property message is already initialized
}
```

* `kotlin.properties.ReadWriteProperty` 인터페이스를 사용하면 읽기와 쓰기에 대한 커스텀 위임을 편리하게 정의할 수 있다.
    * getValue, setValue 연산자를 추상화한 버전을 정의해준다.

---

* getValue, setValue는 확장 함수로도 정의할 수 있어, 이미 정의된 타입도 쉽게 위임 객체로 바꿀 수 있다.

#### 위임 인스턴스화 제어

* 코틀린 1.1부터는 provideDelegate() 함수를 통해 위임 인스턴스화를 제어할 수 있다.
    * 프로퍼티의 수신 객체, 리플렉션 객체를 파라미터를 받아, 위임 객체를 돌려주는 함수이다.
* 이 함수를 통해 위임 생성 전 대상 프로퍼티를 검증해 예외를 던질 수도 있다.

```kotlin


import kotlin.reflect.*
import kotlin.reflect.full.getExtensionDelegate
import kotlin.reflect.jvm.isAccessible

class Person(val firstName: String, val familyName: String)

@Target(AnnotationTarget.PROPERTY)
annotation class NoCache

class CachedPropertyProvider<in R, out T : Any>(
    val initializer: R.() -> T
) {
    operator fun provideDelegate(
        receiver: R?,
        property: KProperty<*>
    ): CachedProperty<R, T> {
        if (property.annotations.any { it is NoCache }) {
            throw IllegalStateException("${property.name} forbids caching")
        }
        return CachedProperty(initializer)
    }
}

class CachedProperty<in R, out T : Any>(val initializer: R.() -> T) {
    private val cachedValues = HashMap<R, T>()
    operator fun getValue(receiver: R, property: KProperty<*>): T {
        return cachedValues.getOrPut(receiver) { receiver.initializer() }
    }
}

fun <R, T : Any> cached(initializer: R.() -> T) = CachedPropertyProvider(initializer)       // 위임 팩토리 반환

val Person.fullName: String by cached { "$firstName $familyName" }  // 위임 적용 전 provideDelegate 실행

fun main() {
    println(
        Person::fullName
            .apply { isAccessible = true }
            .getExtensionDelegate()!!::class.qualifiedName
    ) // CachedProperty
}
```

### 11.2.3 위임 표현

* 런타임에 위임은 별도의 필드에 저장되고, 프로퍼티에 대한 접근자가 자동으로 생성된다.
    * 접근자는 위임에 있는 적절한 메서드를 호출해준다.
* 리플렉션 API를 사용하면 getDelegate() 를 통해 위임 값에 접근할 수 있다.
* 확장 프로퍼티의 경우 getExtensionDelegate() 함수를 통해 위임을 얻을 수 있다.

## 11.3 고차 함수와 DSL

### 11.3.1 중위 함수를 사용해 플루언트 DSL 만들기

* https://github.com/gilbutITbook/080250/blob/master/Chapter11/chapter11_2.kts

### 11.3.2 타입 안전한 빌더 사용하기
### 11.3.3 @DslMarker

* https://github.com/gilbutITbook/080250/blob/master/Chapter11/Form.kt

* @DslMarker를 이용해 암시적 수신 객체의 영역을 제한할 수 있다.
  * 특정 애너테이션에 DSL을 구분하는 마커 애너테이션을 정의하는데 사용된다.
  * 예제에서 `LayoutDsl` 애너테이션
