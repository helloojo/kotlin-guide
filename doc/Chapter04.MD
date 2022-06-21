# 4장 클래스와 객체 다루기

## 4.1 클래스 정의하기

* 클래스 선언은 참조 타입 정의
    * 인라인 클래스라는 개념을 통해 참조 타입이 아닌 클래스 타입 정의 가능

### 4.1.1 클래스 내부 구조

``` kotlin
class {이름} {
    var {field}: {Type} = {value}
    
    fun {method}() = {
    }
}
```

* 클래스 내부에서는 this를 이용해 인스턴스 자체를 참조할 수 있다.
* 코틀린의 클래스 가시성은 기본적으로 public 이다.

### 4.1.2 생성자

* 클래스 인스턴스를 생성할 때 호출되는 함수

```kotlin
class Person(firstName: String, familyName: String) {
    val fullName = "$firstName $familyName"
}
```

* 클래스 이름 뒤에 파라미터 목록으로 생성자를 정의할 수 있다.
    * 클래스 이름에 붙은 파라미터 목록을 주 생성자 선언이라 한다.
    * 초기화 블록을 통해 주 생성자 호출 시 실행할 본문을 정의할 수 있다.
        * 프로퍼티 초기화도 가능
    * 초기화 블록에는 return이 들어갈 수 없다.

```kotlin
class Person(firstName: String, familyName: String) {
    val fullName = "$firstName $familyName"

    init {
        println("Person init")
    }
}
```

* 컴파일러는 모든 프로퍼티가 생성자 블록에서 초기화되는지 확인한다.

```kotlin
class Person(fullName: String) {
    val firstName: String
    val lastName: String

    init {
        val names = fullName.split(" ")
        if (names.size == 2) {      // 프로퍼티가 초기화되지 않는다는 에러 발생
            firstName = names[0]
            lastName = names[1]
        }
    }
}
```

* 주 생성자 파라미터는 초기화 블록 밖에서는 사용할 수 없다.
* 주 생성자 파라미터는 멤버 프로퍼티로 만드는 기능도 제공한다.

```kotlin
class Person(val firstName: String, lastName: String) {
    fun printFirstName() {
        println(firstName)
    }

    fun printLastName() {
        println(lastName)         // 에러 발생
    }
}
```

* 메서드가 정의되지 않은 클래스의 경우 본문을 생략할 수 있다.

```kotlin
class Person(val firstName: String, val familyName: String)
```

* 생성자 파라미터에도 vararg, 기본 값을 사용할 수 있다.

```kotlin
class Person(val firstName: String, val familyName: String = "")
class Room(vararg val persons: Person)
```

* 다른 파라미터를 가지는 생성자를 정의할 수 있다.
    * 이를 부생성자라 한다.
    * 부생성자는 Unit 타입을 반환하는 함수와 마찬가지라 return을 사용할 수 있다.

```kotlin
class Person {
    val firstName: String
    val familyName: String

    constructor(firstName: String, familyName: String) {
        this.firstName = firstName
        this.familyName = familyName
    }

    constructor(fullName: String) {
        val names = fullName.split(" ")
        if (names.size != 2) {
            throw IllegalArgumentException("Invalid Name: $fullName")
        }
        firstName = names[0]
        familyName = names[1]
    }
}
```

* 부생성자가 다른 부생성자에게 프로퍼티 초기화를 위임할 수도 있다.

```kotlin
class Person {
    val fullName: String

    constructor(firstName: String, familyName: String) : this("$firstName $familyName")     // 생성자 위임 호출
    constructor(fullName: String) {
        this.fullName = fullName
    }
}
```

### 4.1.3 멤버 가시성

* public
    * 기본 가시성, 어디서나 멤버를 볼 수 있음
* internal
    * 컴파일 모듈 내부에서만 멤버를 볼 수 있음
* protected
    * 멤버가 속한 클래스와 하위 클래스에서만 멤버를 볼 수 있음
* private
    * 멤버가 속한 클래스에서만 멤버를 볼 수 있음

---

* 주생성자의 가시성을 지정하려면 constructor 키워드를 명시해야 한다.

```kotlin
class Empty private constructor() {     // 해당 클래스는 외부에서 인스턴스화 할 수 없음
}
```

### 4.1.4 내포된 클래스

* 코틀린은 다른 클래스도 멤버로 가질 수 있음

```kotlin
class Person(val id: Id, val age: Int) {
    class Id(val firstName: String, val familName: String)
}

fun main() {
    val id = Person.Id("A", "B")
    val person = Person(id, 25)
}
```

* 내포된 클래스도 가시성을 지정할 수 있다.
* 코틀린의 기본 내포된 클래스는 외부 클래스 인스턴스의 참조를 가지지 않는다.
* 참조를 가지려면 inner 키워드를 내포된 클래스 선언 앞에 붙여야 한다.(내부 클래스)

### 4.1.5 지역 클래스

* 함수 본문에서도 클래스 정의 가능
* 지역 클래스는 함수의 선언에 접근할 수 있다.
* 지역 클래스를 이용해 함수 내부의 변수의 값을 바꿀 수도 있다.
    * 하지만 이는 컴파일 시 특별한 래퍼 객체를 사용하기 때문에 부가적인 비용이 발생한다.
* 지역 클래스도 클래스의 여러 멤버를 포함할 수 있지만, 내포된 클래스를 사용할 때는 반드시 내부(inner) 클래스로 선언해야 한다.
    * 지역 클래스의 변수 가시성 규칙과 혼동을 일으킬 수 있어 내포된 클래스 허용 X

## 4.2 널 가능성

* null
    * 아무것도 참조하지 않는 경우
    * 코틀린에서는 null 발생 가능성이 있는 참조 타입을 구분할 수 있다.

### 4.2.1 널이 될 수 있는 타입

* 코틀린의 모든 참조 타입은 널이 될 수 없는 타입
* 컴파일 단계에서 널이 전달되는지 검사해준다.
* 파라미터 타입 뒤에 ? 를 붙여 널이 될 수도 있는 타입을 정의할 수 있다.
    * nullable type

```kotlin
val value: String? = null
```

```kotlin
val value: String = "ABC"
val nullable: String? = value       // nullable 타입에 null이 될 수 없는 타입 대입은 가능
val value2: String = nullabe        // nullable 타입을 null이 될 수 없는 타입에 대입하는 것은 불가능
```

* 원시 타입도 nullable 타입이 존재한다.
    * 원시 타입의 nullable은 박싱한 타입이 된다.
* nullable 타입은 기존 타입의 프로퍼티, 메서드를 제공하지 않는다.
    * 값이 null일 경우 프로퍼티, 메서드를 사용할 수 없기 때문에

### 4.2.2 널 가능성과 스마트 캐스트

```kotlin
fun isLetterString(s: String?): Boolean {
    if (s == null) return false
    if (s.isEmpty()) return false
    for (ch in s) {
        if (!ch.isLetter()) return false
    }
    return true
}
```

* 조건문을 이용해 널 가능성을 처리할 수 있음
* 스마트 캐스트라는 기능
    * 코틀린에서는 조건문을 이용해 널 검사를 수행하면 반대 코드 흐름은 타입이 널이 아니라는 사실을 안다.
* 스마트 캐스트는 when, 루프, 지연 논리 연산 등에서도 동작한다.
* 코드 흐름 중 값이 바뀔 경우 스마트 캐스트는 동작하지 않는다.
* 객체의 가변 프로퍼티는 언제든 변할 수 있기에 스마트 캐스트가 절대 적용되지 않는다.

### 4.2.3 널 아님 단언 연산자

* !! 연산자를 이용해 변수가 널이 아니라는 것을 의미할 수 있음
* !!를 사용했을때 변수가 널이라면 NPE가 발생한다.

### 4.2.4 안전한 호출 연산자

* 널이 될 수 있는 타입의 값은 프로퍼티, 메서드를 사용할 수 없음
* 안전한 호출 연산을 사용하면 프로퍼티, 메서드 사용가능

```kotlin
fun readInt() = readLine()?.toInt()
```

* ?. 을 이용해서 안전한 호출 가능
* 만약 객체가 null 이라면 그냥 null을 반환해준다.
* 해당 연산자를 사용해 반환하는 값은 nullable type 이다.

### 4.2.5 엘비스 연산자

* 널을 대신할 디폴트 값을 지정할 수 있는 연산자

```kotlin
fun func(value: String?) = value ?: "Default"
```

* 안전한 호출 연산자(?.)와 엘비스 연산자를 조합하면 유용함

```kotlin
val n = readLine()?.toInt() ?: 0
```

* return, throw 등의 식을 엘비스 연산자 우항에 넣을 수도 있다.

```kotlin
class Name(val firstName: String, val familyName: String?)
class Person(val name: Name?) {
    fun describe(): String {
        val currentName = name ?: return "Unknown"
        return "${currentName.firstName} ${currentName.familyName}"
    }
}
```

## 4.3 단순한 변수 이상인 프로퍼티

### 4.3.1 최상위 프로퍼티

* 프로퍼티도 최상위에 정의 가능
    * 전역 변수, 상수 역할을 할 수 있다.
* 최상위 프로퍼티는 가시성(public/internal/private)을 지정할 수 있다.

### 4.3.2 늦은 초기화

* 어떤 프로퍼티는 클래스 인스턴스 생성 이후, 프로퍼티가 사용되는 시점에 초기화돼야 할 수 있음
    * 프로퍼티를 null로 초기화하고 사용할 때 초기화하면 되지만, 널 처리를 해줘야하는 번거로움이 존재함
* lateinit을 이용하면 늦은 초기화되는 프로퍼티를 지정할 수 있음

```kotlin
class Content {
    lateinit var text: String

    fun loadFile(file: File) {
        text = file.readText()
    }
}
```

* 초기화되지 않은 lateinit 프로퍼티에 접근할 경우 예외를 발생시킨다.
* lateinit 조건
    * 프로퍼티는 가변 프로퍼티(var)여야 한다.
    * 타입은 널이 아닌 타입이어야 하고 원시 값을 표현하는 타입이 아니어야 한다.
    * lateinit을 사용하면 초기화 식을 지정해 값을 바로 대입할 수 없다.
* 코틀린 1.2부터 최상위 프로퍼티와 지역 변수에서도 늦은 초기화를 사용할 수 있다.

### 4.3.3 커스텀 접근자 사용하기

* 코틀린 프로퍼티는 변수와 함수의 동작을 하나의 선언에 조합할 수 있다.

#### getter

```kotlin
class Person(val firstName: String, val familyName: String) {
    val fullName: String
        get(): String {
            return "$firstName $familyName"
        }
}
```

* fullName 프로퍼티를 읽으면 자동으로 get 함수가 호출된다.
* fullName 프로퍼티는 뒷받침하는 필드(backing field)가 없기 때문에, 클래스 인스턴스에서 메모리를 차지하지 않고, 접근마다 get 함수를 실행하며 값을 계산하는 방식이다.
* `field` 키워드를 사용하는 접근자를 사용할 경우 뒷받침하는 필드(backing field)가 생성된다.

```kotlin
class Person(val firstName: String, val familyName: String, age: Int) {
    val age: Int = age
        get(): Int {
            println("Accessing Age")
            return field
        }
}
```

* 커스텀 접근자를 통해 프로퍼티에 대한 접근을 커스텀화 할 수 있다.

#### setter

* 가변 프로퍼티는 set(value) 라는 커스텀 접근자를 추가로 선언할 수 있다.

```kotlin
class Person(val firstName: String, val familyName: String) {
    var age: Int? = null
        set(value) {
            if (value != null && value <= 0) {
                throw IllegalArgumentException("Invalid Age: $value")
            }
            field = value
        }
}

val person = Person("A", "B")
person.age = 100        // 커스텀 세터 호출
```

* setter의 파라미터는 프로퍼티의 타입과 같다.
* 프로퍼티 초기화는 바로 값을 필드에 쓰기 때문에 setter를 호출하지는 않는다.
* 프로퍼티 접근자에 별도의 가시성 변경자를 붙일 수도 있다.

```kotlin
class Person(name: String) {
    var lastChanged: Date? = null
        private set     // Person 안에서만 접근 가능, 단순 값 대입은 set, get 키워드만 정의할 수 있다.
}
```

### 4.3.4 지연 계산 프로퍼티와 위임

* 프로퍼티를 처음 읽기 전까지 계산을 미뤄두고 싶은 경우 사용한다.

```kotlin
val text by lazy {
    File("data.txt").readText()
}
```

* text에 접근하기 전까지는 블록이 실행되지 않는다.
* text에 접근하는 순간 블록이 실행된다.
* lazy는 위임 객체를 통해 프로퍼티를 구현하게 해주는 위임 프로퍼티 기능의 한 종류
    * 위임 객체는 by 뒤에 위치함
    * 코틀린이 정의한 규약을 만족하는 객체를 반환하는 임의의 식이 된다.
* lazy 프로퍼티는 초기화된 이후에는 변경되지 않는다.
* 또한 스레드 안전하다.
* 위임 프로퍼티는 스마트 캐스트를 사용할 수 없다.

## 4.4 객체

### 4.4.1 객체 선언

* 코틀린은 싱글턴 패턴 내장
* object라는 키워드로 싱글턴 객체 선언 가능

```kotlin
object Application {
    val name = "My Application"
    override fun toString() = name
    fun exit() {}
}
```

* object 객체의 인스턴스는 하나이고, 스레드 안전하다.
* 싱글턴 클래스가 실제 로딩되는 시점에 초기화가 수행된다.
* 싱글턴 클래스도 다른 클래스와 동일하게 멤버 함수, 프로퍼티를 가질 수 있지만, 생성자는 없다.
* 싱글턴 클래스의 내부 클래스에는 inner가 붙을 수 없다.
    * inner가 붙으면 바깥 클래스의 참조를 가지게되는데, 싱글턴 클래스의 인스턴스는 단 하나라 의미가 없음

``` kotlin
import Application.name
import Application.exit

println(name)
exit()

import Application.*        // 모든 멤버 임포트는 불가능, 공통 메서드 정의를 덮어씌울 수 있기 때문에
```

### 4.4.2 동반 객체

* 내포 객체는 자신을 둘러싼 클래스의 멤버에 접근할 수 있음
    * 팩토리 디자인 패턴을 쉽게 구현할 때 활용 가능

```kotlin
class Application private constructor(val name: String) {
    object Factory {
        fun create(args: Array<String>): Application? {
            val name = args.firstOrNull() ?: return null
            return Application(name)
        }
    }
}

fun main(args: Array<String>) {
    val app = Application.Factory.create(args)
}
```

* 하지만 매번 Application.Factory.create 이름을 붙여 호출해야함
* 동반 객체를 사용하면 이런 문제를 해결할 수 있음
* 동반 객체의 멤버는 상위 클래스의 이름으로 접근 가능

```kotlin
class Application private constructor(val name: String) {
    companion object Factory {
        fun create(args: Array<String>): Application? {
            val name = args.firstOrNull() ?: return null
            return Application(name)
        }
    }
}

fun main(args: Array<String>) {
    val app = Application.create(args)
}
```

* 동반 객체는 이름을 생략할 수도 있음
    * 훨씬 더 권장하는 방식
* 동반 객체는 한 클래스에 둘 이상 존재할 수 없음
* companion을 최상위 객체, 다른 객체에 내포된 객체에 붙이는 것은 금지됨

### 4.4.3 객체 식

* 명시적인 선언 없이 객체를 바로 생성할 수 있는 특별한 식

```kotlin
fun main() {
    fun midPoint(xRange: IntRange, yRange: IntRange) = object {
        val x = (xRange.first + xRange.last) / 2
        val y = (yRange.first + yRange.last) / 2
    }

    val midPoint = midPoint(1..5, 2..6)

    println("${midPoint.x}, ${midPoint.y}") // (3, 4)
}
```

* 객체 식의 타입은 익명 객체 타입
* 익명 객체 타입은 지역 선언, 비공개 선언에만 전달 가능

```kotlin
fun midPoint(xRange: IntRange, yRange: IntRange) = object {
    val x = (xRange.first + xRange.last) / 2
    val y = (yRange.first + yRange.last) / 2
}

fun main() {
    val midPoint = midPoint2(1..5, 2..6)

    println("${midPoint.x}, ${midPoint.y}")   // Error: x와 y를 찾을 수 없음
}
```

* 객체 식도 자신을 둘러싼 코드 영역의 변수 접근, 변경 가능
