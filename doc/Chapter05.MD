# 5장 고급 함수와 함수형 프로그래밍 활용하기

## 5.1 코틀린을 활용한 함수형 프로그래밍

* 함수형 언어는 함수를 일급 시민 값으로 취급
    * 함수를 타입의 값과 똑같이 취급함

### 5.1.1 고차 함수

```kotlin
fun aggregate(numbers: IntArray, op: (Int, Int) -> Int): Int {
    var result = numbers.firstOrNull()
        ?: throw IllegalArgumentException("Empty array")

    for (i in 1..numbers.lastIndex) result = op(result, numbers[i])

    return result
}

fun sum(numbers: IntArray) =
    aggregate(numbers, { result, op -> result + op })

fun max(numbers: IntArray) =
    aggregate(numbers, { result, op -> if (op > result) op else result })
```

* op 파라미터가 람다식을 타입으로 정의한다.
* sum, max 함수에서 람다식을 정의해 aggregate가 다양한 동작을 하도록 정의할 수 있다.
* 람다식은 파라미터, 식으로 표현된다

``` kotlin
{ param1, param2... -> 식 }
```

* 파라미터의 타입은 컴파일러가 문맥으로부터 추론해준다.

### 5.1.2 함수 타입

* 함수처럼 쓰일 수 있는 값들을 표시하는 타입

``` kotlin
(param1, param2, ...) -> 반환타입
```

* 반환값이 없는 함수도 반환 타입을 명시해야 한다.
    * Unit을 사용한다.
* 함수 타입은 함수처럼 사용하거나, invoke 메서드를 호출해서 사용할 수 있다.
* 코틀린 1.4 부터는 코틀린 인터페이스 앞에 fun을 붙이면 자바의 단일 추상 메서드로 취급한다.

```kotlin
fun interface StringConsumer {
    fun accept(s: String)
}

fun main() {
    val consume = StringConsumer { s -> println(s) }

    consume.accept("Hello")
}
```

* 함수 타입의 파라미터의 괄호는 필수이다.
* 함수 타입도 nullable로 지정할 수 있다.

```kotlin
fun measureTime(action: (() -> Unit)?): Long {
    val start = System.nanoTime()

    action?.invoke()

    return System.nanoTime() - start
}
```

* 함수 타입을 다른 함수 타입에 내포시켜 고차 함수 타입도 정의할 수 있다.

```kotlin
fun main() {
    val divisor: (Int) -> (Int) -> Int = { n -> { i -> i / n } }

    val diviedByTwo = divisor(2)

    val diviedByFive = divisor(5)

    println(diviedByTwo(10)) // 5
    println(diviedByFive(10)) // 2
}
```

* 함수 타입의 파라미터 목록에 이름을 포함시킬 수도 있다.
    * 문서화 목적

```kotlin
fun aggregate(numbers: IntArray, op: (resultSoFar: Int, nextValue: Int) -> Int)
```

### 5.1.3 람다와 익명 함수

#### 함수형 타입의 값 만드는 방법

##### 람다식 사용

``` kotlin
{ param1, param2... -> 식 }
```

* 람다식이 함수의 마지막 파라미터일 경우, 인자가 위치하는 괄호 밖에 람다를 위치시킬 수 있음
    * 코틀린 권장 스타일
* 람다에 인자가 없으면 화살표 생략 가능

```kotlin
fun printTime(print: () -> Unit) {
    print()
}

printTime() { println("${Date()}") }
```

* 인자가 하나인 람다는 it 라는 특별한 키워드로 파라미터 생략, 접근 가능

```kotlin
fun check(s: String, condition: (Char) -> Boolean): Boolean {
    for (c in s) {
        if (!condition(c)) return false
    }
    return true
}

fun main() {
    println(check("Hello") { c -> c.isLetter() }) // true
    println(check("Hello") { it.isLowerCase() }) // false
}
```

* 코틀린 1.1 부터는 사용하지 않는 람다 파라미터를 _ 로 표현 가능하다.

##### 익명함수 사용

```kotlin
fun sum(numbers: IntArray) = aggregate(numbers, fun(result, op) = result + op)
```

* 함수 문법과 거의 똑같음
* 차이점
    * 이름을 지정하지 않음
    * 파라미터 추론 가능할 경우 타입 지정하지 않아도 됨
    * 익명함수는 식이기 때문에 다른 변수에 대입하는 것처럼 값으로 사용 가능
* 익명 함수도 반환 타입 지정가능
    * 함수 본문이 블록 선언일 경우 반환 타입이 Unit이 아닌 이상 명시적으로 지정해야함
* 익명 함수는 람다처럼 인자 목록 바깥에 위치는 불가능

### 5.1.4 호출 가능 참조

* 이미 정의된 함수를 고차 함수에 값처럼 넘기는 방법
    * :: 연산자를 활용한다.

```kotlin
fun isCapitalLetter(c: Char) = c.isUpperCase() && c.isLetter()

fun main() {
    println(check("Hello", ::isCapitalLetter)) // false
}
```

* 호출 가능 참조는 최상위, 지역 함수부터, 클래스의 생성자, 인스턴스의 함수까지 모두 지정 가능
* 호출 가능 참조는 오버로딩 함수를 구분할 수 없음으로 반드시 타입을 지정해주어야 한다.
* 프로퍼티에 대한 호출 가능 참조를 만드는 것도 가능

```kotlin
class Person(var firstName: String, var familyName: String)

fun main() {
    val person = Person("John", "Doe")
    val readName = person::firstName.getter     // 게터 참조
    val writeFamily = person::familyName.setter // 세터 참조

    println(readName())        // John
    writeFamily("Smith")
    println(person.familyName) // Smith
}
```

* 현재 지역 변수에 대한 호출 가능 참조는 불가능

### 5.1.5 인라인 함수와 프로퍼티

* 고차 함수, 함수값 사용시 객체 생성에 대한 비용이 발생한다.
* 이런 런타임 비용을 줄이기 위해 고차 함수를 호출하는 부분을 본문으로 대체하는 inline 기법을 제공한다.
* 함수 앞에 inline 키워드를 붙여 사용할 수 있다.

```kotlin
inline fun indexOf(numbers: IntArray, condition: (Int) -> Boolean): Int {
    for (i in numbers.indices) {
        if (condition(numbers[i])) return i
    }

    return -1
}

fun main() {
    println(indexOf(intArrayOf(4, 3, 2, 1)) { it < 3 }) // 2
}

// 실제로는 아래와 같은 함수 본문으로 실행됨
fun main() {
    val numbers = intArrayOf(4, 3, 2, 1)
    var index = -1

    for (i in numbers.indices) {
        if (numbers[i] < 3) {
            index = i
            break
        }
    }

    println(index)
}
```

* 인라인 함수를 사용하면 컴파일 코드 크기가 커지지만, 성능을 높일 수 있다는 이점이 있다.
* 코틀린의 inline 구문은 컴파일러가 최적화를 위해 사용하는 구문이 아닌 반드시 본문에 포함시키겠다는 선언이다.
* 인라인 함수는 별도의 객체가 아니므로 다른 변수에 저장되거나 인라인 함수가 아닌 함수에 전달될 수 없음
* 인라인 함수는 널이 될 수 있는 타입의 인자도 받을 수 없음
    * noinline 변경자를 통해 특정 람다를 인라인 하지 않도록 지정 가능

```kotlin
inline fun forEach3(a: IntArray, noinline action: ((Int) -> Unit)?) {
    if (action == null) return
    for (n in a) action(n)
}
```

* 공개 인라인 함수에 비공개 멤버를 사용하는 것은 불가능
    * 캡슐화 위반 가능

```kotlin
class Person4(
    private val firstName: String,
    private val familyName: String
) {
    inline fun sendMessage(message: () -> String) {
        println("$firstName $familyName: ${message()}") // 공개 인라인 함수에 비공개 멤버를 사용함
    }
}
```

* 코틀린 1.1부터는 프로퍼티 접근자도 인라인 가능
* 프로퍼티 자체에도 inline 선언 가능
    * 이 경우 getter, setter 모두 인라인

### 5.1.6 비지역적 제어 흐름

* 고차 함수 사용 시 return 처럼 제어 흐름을 중단하고 싶을 때 문제가 발생할 수 있다.

```kotlin
fun forEach(a: IntArray, action: (Int) -> Unit) {
    for (n in a) action(n)
}

fun main() {
    forEach(intArrayOf(1, 2, 3, 4)) {
        if (it < 2 || it > 3) return // 에러 발생
        println(it)
    }
}
```

* 람다를 종료시키는 것이 의도지만, 실제로는 return 은 가장 안쪽 함수부터 반환하는 동작을 하기 때문에 main을 반환하는 코드가 된다.
* 익명 함수를 사용하면 반환이 가능하다.

```kotlin
fun main() {
    forEach(intArrayOf(1, 2, 3, 4), fun(it: Int) {
        if (it < 2 || it > 3) return
        println(it)
    })
}
```

* 람다로 제어 흐름을 반환하고 싶다면 레이블을 사용해야 한다.

```kotlin
val action: (Int) -> Unit = myFun@{
    if (it < 2 || it > 3) return@myFun
    println(it)
}

// 고차 함수의 인자로 넘길 경우 함수 이름으로 레이블 사용 가능
forEach(intArrayOf(1, 2, 3, 4)) {
    if (it < 2 || it > 3) return@forEach
    println(it)
}
```

* 람다를 인라인 할 경우에는 return을 사용할 수 있음

```kotlin
inline fun forEach(a: IntArray, action: (Int) -> Unit) {
    for (n in a) action(n)
}

fun main() {
    forEach(intArrayOf(1, 2, 3, 4)) {
        if (it < 2 || it > 3) return // main에서 반환됨
        println(it)
    }
}
```

* 인라인이 될 수 있는 람다를 받는 고차 함수가 지역 함수, 지역 클래스의 멤버에서 람다를 호출하는 것은 금지됨
    * 람다의 return 과 람다를 실행해주는 함수가 다른 실행 스택 프레임에 위치하기 때문에
    * crossinline 변경자를 통해 이런 동작 허용 가능

```kotlin
inline fun forEach(a: IntArray, action: (Int) -> Unit) = object {
    fun run() {
        for (n in a) {
            action(n) // error: can't inline 'action' here: it may contain non-local returns.
        }
    }
}

inline fun forEach(a: IntArray, crossinline action: (Int) -> Unit) = object {
    fun run() {
        for (n in a) {
            action(n)
        }
    }
}
```

* break, continue은 아직 비지역적 제어 흐름으로 사용 불가능

```kotlin
fun forEach(a: IntArray, op: (Int) -> Unit) {
    for (n in a) {
        op(n)
    }
}

fun main() {
    while (true) {
        forEach(intArrayOf(1, 3, 4, 5)) {
            if (it < 2 || it > 3) break     // 에러 발생
            println(it)
        }
    }
}
```

## 5.2 확장

* 코틀린은 멤버인 것 처럼 쓸 수 있는 함수, 프로퍼티를 클래스 바깥에서 선언할 수 있음
* 확장 사용 시 클래스 변경 없이 새로운 기능으로 기존 클래스 확장 가능

### 5.2.1 확장 함수

* 어떤 클래스의 멤버인 것처럼 호출할 수 있지만, 실제로는 멤버가 아닌 함수
* `클래스이름.함수이름` 으로 정의

```kotlin
fun String.truncate(maxLength: Int): String {
    return if (length <= maxLength) this else substring(0, maxLength)
}
```

* 확장 함수는 기존 타입의 캡슐화를 깰 수 없음
    * 비공개 멤버에 접근 불가능
* 클래스 본문 안에 확장 함수 정의할 경우 수신 객체의 멤버인 동시에 확장 함수가 되어 비공개 멤버에 접근 가능

```kotlin
class Person(val name: String, private val age: Int) {
    // Ok: age에 접근할 수 있음
    fun Person.showInfo() = println("$name, $age")
}
```

* 확장 함수는 호출 가능 참조로도 지정 가능
* 클래스 멤버와 확장 함수의 시그니처가 같다면 멤버 함수가 우선적으로 선택된다.
* 지역 확장 함수도 정의 가능
* 다른 패키지의 최상위 확장 함수 호출 시 반드시 확장 함수 임포트 필요함
* nullable 타입도 확장 함수 정의 가능
    * nullable 타입에 확장 함수를 정의함으로서 안전한 호출 연산자 없이 기능 사용 가능

## 5.3 확장 프로퍼티

* 프로퍼티 이름 앞에 수신 객체 타입 지정 시 확장 프로퍼티도 사용 가능

```kotlin
val IntRange.leftHalf: IntRange
    get() = start..(start + endInclusive) / 2

fun main() {
    println((1..3).leftHalf) // 1..2
    println((3..6).leftHalf) // 3..4
}
```

* 왼쪽 범위의 절반을 계산하는 확장 프로퍼티 정의
* 확장 프로퍼티는 인스턴스에 상태를 추가할 수 때문에 뒷받침하는 필드를 쓰는 것이 불가능하다.
    * 초기화 불가능
    * 접근자에서 field 사용 불가능
    * lateinit으로 정의 불가능
* 확장 프로퍼티는 반드시 getter, 가변일 경우 setter까지 정의해야 한다.
* 확장 프로퍼티도 위임을 사용할 수 있다.
    * 하지만 위임식이 수신 객체에 접근할 수는 없다.
    * 딱히 사용시 이점은 없음
    * 객체(object) 정의에서는 유용할 수 있음

## 5.4 동반 확장

* 동반 객체(companion object)
    * 바깥 클래스의 이름을 통해 멤버에 접근할 수 있는 객체
* 동반 객체를 확장할 수도 있음
    * 확장 함수, 프로퍼티 모두 정의 가능

```kotlin
fun IntRange.Companion.singletonRange(n: Int) = n..n

fun main33() {
    println(IntRange.singletonRange(5))           // 5..5
    println(IntRange.Companion.singletonRange(3)) // 3..3
}
```

### 5.4.1 람다와 수신 객체 지정 함수 타입

* 람다, 익명 함수도 확장 수신 객체를 활용할 수 있음
    * 수신 객체 지정 함수 타입(functional type with receiver)이라는 타입으로 표현됨

```kotlin
fun aggregate(numbers: IntArray, op: Int.(Int) -> Int): Int {       // Int 타입의 수신 객체 지정 함수 타입 정의
    var result = numbers.firstOrNull()
        ?: throw IllegalArgumentException("Empty array")

    for (i in 1..numbers.lastIndex) result = result.op(numbers[i])    // Int 타입의 result가 확장된 op를 호출할 수 있다.

    return result
}

fun sum(numbers: IntArray) = aggregate(numbers) { op -> this + op }

fun sum(numbers: IntArray) = aggregate(numbers, fun Int.(op: Int) = this + op)      // 익명 함수도 사용 가능
```

* 수신 객체가 있는 함수 값을 호출할 경우, 첫번째 파라미터의 수신 객체를 넣어 일반 함수로 호출도 가능

```kotlin
fun aggregate(numbers: IntArray, op: Int.(Int) -> Int): Int {
    var result = numbers.firstOrNull()
        ?: throw IllegalArgumentException("Empty array")

    for (i in 1..numbers.lastIndex) {
        result = op(result, numbers[i]) // 비 확장 함수 호출
    }

    return result
}
```

* 수신 객체 람다는 DSL 같은 API 구축 시 강력한 도구 제공함

## 5.5 수신 객체가 있는 호출 가능 참조

* 수신 객체가 있는 함수의 호출가능 참조는 수신 객체 타입을 :: 앞에 붙여 사용할 수 있다.

```kotlin
fun aggregate(numbers: IntArray, op: Int.(Int) -> Int): Int {
    var result = numbers.firstOrNull()
        ?: throw IllegalArgumentException("Empty array")

    for (i in 1..numbers.lastIndex) result = result.op(numbers[i])

    return result
}

fun Int.max(other: Int) = if (this > other) this else other
fun max(a: Int, b: Int) = if (a > b) a else b

fun main() {
    val numbers = intArrayOf(1, 2, 3, 4)
    println(aggregate(numbers, Int::max))  // 수신 객체가 있는 확장함수 호출 가능 참조
    println(aggregate(numbers, ::max))     // 비확장 함수로 암시적 변환해서 사용도 가능하다.
}
```

* 비확장 함수에 수신 객체가 있는 확장 함수의 호출 가능 참조를 대입하는 것도 가능하다.

```kotlin
fun aggregate(numbers: IntArray, op: (Int, Int) -> Int): Int {      // op는 비확장 함수 타입
    var result = numbers.firstOrNull()
        ?: throw IllegalArgumentException("Empty array")

    for (i in 1..numbers.lastIndex) result = op(result, numbers[i])

    return result
}

fun Int.max(other: Int) = if (this > other) this else other

fun main() {
    println(aggregate(intArrayOf(1, 2, 3, 4), Int::max))  // 4        // 수신 객체가 있는 확장함수 전달
}
```

* 클래스 멤버이면서 확장 함수로 정의된 함수에 대한 호출 가능 참조는 지원하지 않음

### 5.5.1 영역 함수

* 식을 계산한 값을 문맥 내부에서 임시로 사용할 수 있도록 해주는 함수
* 지역 변수를 선언하지 않고, 식의 값이 있는 암시적인 영역을 정의해 코드를 단순화하는데 사용한다.

#### run과 with 함수

* run()
    * 확장 람다를 받는 확장 함수
    * 람다의 결과를 반환한다.
    * 보통 객체의 상태를 설정한 후, 객체를 대상으로 어떤 결과를 만들어내는 람다를 호출하는데 사용한다.

```kotlin
class Address {
    var zipCode: Int = 0
    var city: String = ""
    var street: String = ""
    var house: String = ""

    fun post(message: String): Boolean {
        println("Message for {$zipCode, $city, $street, $house}: $message\nInput OK or other string")
        return readLine() == "OK"
    }
}

fun main() {
    val isReceived = Address().run {
        // Address 인스턴스를 this로 사용할 수 있다
        zipCode = 123456
        city = "London"
        street = "Baker Street"
        house = "221b"
        post("Hello!") // 반환값
    }

    if (!isReceived) {
        println("Message is not delivered")
    }
}
```

* run이 없으면 Address 인스턴스를 담을 변수를 추가해야 하지만, run으로 인해 그런 변수가 없어도 된다.
    * 함수의 다른 부분에서는 Address 인스턴스에 접근할 수 없다.
    * 지역 변수의 가시성을 세밀하게 제어할 수 있다.
* 반환 결과의 타입은 Unit일수도 있다.

---

* with()
    * with는 run과 비슷하지만 확장 함수 타입이 아니기 때문에 식을 with의 인자로 전달해야 한다.
    * 문맥 식의 멤버 함수와 프로퍼티에 대한 호출을 묶어 동일한 영역에서 실행할 경우에 사용한다.

```kotlin
class Address(val city: String, val street: String, val house: String) {
    fun asText() = "$city, $street, $house"
}

fun main() {
    val message = with(Address("London", "Baker Street", "221b")) {
        "Address: $city, $street, $house"
    }
    println(message)


    // with를 사용하지 않으면 이렇게 구현해야함
    val addr = Address("London", "Baker Street", "221b")
    val message2 = "Address: ${addr.city}, ${addr.street}, ${addr.house}"
    println(message2)
}
```

#### 문맥이 없는 run

* 코틀린 표준 라이브러리는 run을 오버로딩한 함수도 제공
    * 문맥 식이 없고, 람다의 값을 반환만 하는 역할

```kotlin
fun main43() {
    val address = run {
        println("Input city, street, house(press 'enter' after input each):")
        val city = readLine() ?: return
        val street = readLine() ?: return
        val house = readLine() ?: return
        Address(city, street, house)
    }

    println(address.asText())
}
```

* 위 예제처럼 하나의 인스턴스 초기화 식을 표현하기 위해 사용할 수 있다.

#### let 함수

* run과 비슷하지만 확장 함수 타입의 람다를 받지 않고 인자가 하나인 함수 타입의 람다를 받음
    * 문맥 식의 값은 람다의 인자로 전달된다.
    * let은 람다의 반환 값을 반환한다.
    * 외부 영역에 새로운 변수를 도입하는 일을 피하고 싶을 때 사용한다.

```kotlin
class Address(val city: String, val street: String, val house: String) {
    fun post(message: String) {}
}

fun main45() {
    Address("London", "Baker Street", "221b").let {
        // 이 안에서는 it 파라미터를 통해 Address 인스턴스에 접근할 수 있음
        println("To city: ${it.city}")
        it.post("Hello")
    }
}
```

* let은 널 안전성 검사에도 사용할 수 있음

```kotlin
fun readInt() = try {
    println("Input int:")
    readLine()?.toInt()
} catch (e: NumberFormatException) {
    null
}

fun main47(args: Array<String>) {
    println("Input int:")
    val index = readInt()
    // val arg = if (index != null) args.getOrNull(index) else null     // let을 안쓰면 if 문으로 null 검사를 해야함
    val arg = index?.let { args.getOrNull(it) }
    if (arg != null) {
        println(arg)
    }
}
```

#### apply/also 함수

* apply()
    * 확장 람다를 받는 확장 함수
    * 자신의 수신 객체를 반환함
    * 객체의 상태를 설정하는데 사용한다.

```kotlin
class Address {
    var city: String = ""
    var street: String = ""
    var house: String = ""

    fun post(message: String) {}
}

fun main() {
    println("Input string:")
    val message = readLine() ?: return

    Address().apply {   // this로 Address 인스턴스 참조 가능
        city = "London"
        street = "Baker Street"
        house = "221b"
    }.post(message)
}
```

* also()
    * 인자가 하나 있는 람다를 파라미터로 받는 apply와 비슷한 함수

```kotlin
fun main() {
    println("Input string:")
    val message = readLine() ?: return

    Address().also {    // it는 Address 인스턴스
        it.city = "London"
        it.street = "Baker Street"
        it.house = "221b"
    }.post(message)
}
```

### 5.5.2 클래스 멤버인 확장

* 확장 함수는 클래스 멤버로 선언할 수 있음
* 클래스 내부에서 확장 함수, 프로퍼티를 선언할 경우 수신 객체가 두개가 됨
* 확장 수신 객체(extension receiver)
    * 확장 정의의 수신 객체 타입의 인스턴스
    * `this@확장함수이름` 으로 참조한다.
    * 클래스 멤버의 확장 함수의 `this` 키워드는 확장 수신 객체를 의미한다.
* 디스패치 수신 객체(dispatch receiver)
    * 확장이 포함된 클래스 타입의 인스턴스
    * `this@클래스이름` 으로 참조한다.

```kotlin
class Address(val city: String, val street: String, val house: String)

class Person(val firstName: String, val familyName: String) {
    fun Address.post(message: String) {
        // 암시적 this: 확장 수신 객체(Address)
        val city = city
        // 한정시키지 않은 this: 확장 수신 객체(Address)
        val street = this.city
        // 한정시킨 this: 확장 수신 객체(Address)
        val house = this@post.house
        // 암시적 this: 디스패치 수신 객체(Person)
        val firstName = firstName
        // 한정시킨 this: 디스패치 수신 객체(Person)
        val familyName = this@Person9.familyName

        println("From $firstName, $familyName at $city, $street, $house:")
        println(message)
    }

    fun test(address: Address) {
        // 디스패치 수신 객체: 암시적
        // 확장 수신 객체: 명시적
        address.post("Hello")
    }
}
```

* 확장 함수나 내부 클래스의 인스턴스로도 확장 수신 객체, 디스패치 수신 객체가 전달될 수 있다.

```kotlin
class Address(val city: String, val street: String, val house: String)

class Person(val firstName: String, val familyName: String) {
    fun Address.post(message: String) {}
    inner class Mailbox {
        fun Person.testExt(address: Address) {
            address.post("Hello")
        }
    }
}

fun Person.testExt(address: Address) {
    address.post("Hello")
}
```

```kotlin
class Address(val city: String, val street: String, val house: String) {
    fun test(person: Person) {
        // person.post("Hello") 수신 객체가 Address이기에 post 호출 불가능
        with(person) {      // this가 Person이 되어 post 호출 가능
            post("Hello")
        }
        person.run {        // run도 가능
            post("HELLO")
        }
    }
}

class Person(val firstName: String, val familyName: String) {
    fun Address.post(message: String) {}
}
```

```kotlin
class Address(val city: String, val street: String, val house: String)

class Person(val firstName: String, val familyName: String) {
    fun Address.post(message: String) {}
}

fun main() {
    with(Person("John", "Watson")) {        // 클래스 외부에서도 이렇게 호출 가능
        Address("London", "Baker Street", "221b").post("Hello")
    }
}
```

* 수신 객체의 영역은 자신이 포함된 선언 내부로 제한하는 것이 좋다.

```kotlin
class Address(val city: String, val street: String, val house: String)

class Person(val firstName: String, val familyName: String) {
    // Person 클래스 밖에서는 쓸 수 없음
    private fun Address.post(message: String) {}
    fun test(address: Address) = address.post("Hello")
}
```

* 디스패치와 확장 수신 객체가 같은 타입인 경우는 혼란이 생기기 쉬우므로 피해야한다.

```kotlin
class Address(val city: String, val street: String, val house: String) {
    fun Address.post(message: String) {}
}
```

* 동반 객체 안에 선언된 확장 함수를 이용해 최상위 확장 처럼 사용할 수 있다.

```kotlin
import Person.Companion.parsePerson

class Person private constructor(val firstName: String, val familyName: String) {
    companion object {
        fun String.parsePerson(): Person? {     // String의 확장 함수
            val names = split(" ")      // String의 함수 호출
            return if (names.size == 2) Person(names[0], names[1]) else null        // Person 생성자 호출, private 생성자를 가지는 Person을 String으로 생성할 수 있다
        }
    }
}

fun main() {
    // Person.Companion 인스턴스가 암시적으로 공급됨
    println("John Doe".parsePerson()?.firstName) // John
}
```

* 하지만... 최상위 확장을 우선적으로 사용하는 것이 좋다.
    * 더 간단하고 읽기 좋은 코드를 만들 수 있음
