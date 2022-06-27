# 6장 특별한 클래스 사용하기

## 6.1 이넘 클래스

* 정의된 상수로 이뤄진 집합을 표현하는 클래스
* 상수를 타입 안전하게 다룰 수 있다.
* 이넘 클래스는 내부 클래스, 함수 본문에서는 정의할 수 없다.

```kotlin
enum class WeekDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

### 6.1.1 빠뜨린 부분이 없는 when 식

* 이넘 값을 when에 사용할 경우 else를 생략할 수 있다.

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}

fun rotateClockWise(direction: Direction) = when (direction) {
    Direction.NORTH -> Direction.EAST
    Direction.EAST -> Direction.SOUTH
    Direction.SOUTH -> Direction.WEST
    Direction.WEST -> Direction.NORTH
}
```

* 이넘을 사용해 when식에 else를 생략할 경우 이넘 값 추가에 대한 분기 처리 누락을 컴파일 시점에 알 수 있다.

### 6.1.2 커스텀 멤버가 있는 이넘 정의하기

* 이넘 클래스도 멤버를 포함할 수 있음
    * 멤버는 상수 목록 뒤에 위치해야함
    * 멤버가 존재할 경우 상수 목록 마지막에 세미콜론(;)을 붙여야함

```kotlin
enum class WeekDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    val lowerCaseName get() = name.lowercase()
    fun isWorkDay() = this == SATURDAY || this == SUNDAY
}
```

* 상수 별로 멤버 정의를 포함할 수도 있음
    * 하지만 이런 멤버는 외부에서 접근 불가능

```kotlin
enum class WeekDay {
    MONDAY {
        fun startWork() = println("Work week started")
    },
    TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

fun main() = WeekDay.MONDAY.startWork() // error: unresolved reference: startWork
```

### 6.1.3 이넘 클래스의 공통 멤버 사용하기

* 코틀린의 이넘 클래스는 kotlin.Enum 클래스의 하위 타입
    * 제공하는 프로퍼티, 함수는 자바 Enum의 멤버와 비슷함
    * 공통 프로퍼티
        * ordinal
            * 이넘 값 순서의 인덱스
        * name
            * 이넘 값의 이름
    * 공통 메서드
        * valueOf(String)
            * 문자열에 따른 이넘 값 반환
        * values()
            * 모든 이넘 값 들어있는 배열 반환
* 이넘 제네릭 최상위 메서드
    * enumValues<T>()
    * enumValueOf<T>(String)

## 6.2 데이터 클래스

* 데이터를 저장하기 위한 목적으로 쓰이는 클래스
* 컴파일러가 동등성 비교, String 변환 등의 기본 연산 구현 자동으로 수행해줌
* 구조 분해 선언도 활용 가능
    * 클래스의 프로퍼티를 여러 변수에 나눠 넣는 것

### 6.2.1 데이터 클래스와 데이터 클래스에 대한 연산

```kotlin
data class Name(val property1: Type, val property2: Type...)
```

* 데이터 클래스는 ==, != 연산자로 값에 대한 동등성 비교를 수행할 수 있다.
    * 컴파일러가 equals(), hashCode(), toString() 연산을 정의해준다.
    * 주 생성자 파라미터에 선언한 프로퍼티만 구현에 사용된다.
* 데이터 클래스는 인스턴스를 복사하는 copy() 라는 메서드를 제공한다.
    * copy() 메서드를 통해 인스턴스를 복사하면서, 값도 원한다면 변경할 수 있다.

#### 코틀린 표준 라이브러리의 범용 데이터 클래스

* Pair
    * 두 값을 저장할 수 있는 데이터 클래스
    * first, second 프로퍼티를 가짐
    * `{value} to {value}` 연산자를 통해 Pair를 만들수도 있음
* Triple
    * 세 값을 저장할 수 있는 데이터 클래스
    * first, second, third 프로퍼티를 가짐

### 6.2.2 구조 분해 선언

* 데이터 클래스를 사용하면 하나의 클래스 인스턴스로 각각에 프로퍼티에 해당하는 지역 변수를 선언할 수 있다.

```kotlin
data class Person(
    val firstName: String,
    val familyName: String,
    val age: Int
)

fun newPerson() = Person(
    readLine()!!,
    readLine()!!,
    Random.nextInt(100)
)
fun newPersonWithPrompt(): Person {
    println("Input firstName and familyName(press enter after each name):")
    return newPerson()
}

fun main() {
    val person = newPersonWithPrompt()
    val (firstName, familyName, age) = person       // 구조 분해 선언으로 각 지역변수에 person의 프로퍼티 값 대입  


    if (age < 18) {
        println("$firstName $familyName is under-age")
    }
}
```

* 구조 분해 선언은 데이터 클래스 생성자의 정의된 프로퍼티에 위치에 따라 대입된다.
* 사용하지 않는 값이 있을 경우 "_" 으로 생략할 수 있다.
* for 루프에서도 구조 분해 선언을 사용할 수 있다.

```kotlin
val pairs = arrayOf(1 to "one", 2 to "two", 3 to "three")

for ((number, name) in pairs) {
    println("$number: $name")
}
```

* 코틀린 1.1 부터는 람다 파라미터에도 구조 분해를 사용할 수 있다.
* 구조 분해 안에 다른 구조 분해를 포함하는 것은 불가능하다.
* 다른 코틀린 타입도 연산자 오버로딩 관습을 이용해 구조 분해를 지원할 수 있다.

## 6.3 인라인 클래스(값 클래스)

### 6.3.1 인라인 클래스 정의하기

``` kotlin
@JvmInline
value class {Name}(val property1: {Type})
```

* value class 키워드로 인라인 클래스 정의 가능
* JVM 백엔드를 사용할 경우 @JvmInline 애너테이션 붙여야함
* 인라인 클래스의 주생성자에는 불변 프로퍼티를 하나만 선언해야함
    * 런타임에는 클래스의 인스턴스 없이 프로퍼티의 값으로 표현됨
* 인라인 클래스도 함수, 다른 프로퍼티 포함 가능
    * 하지만 상태는 포함할 수 없음
        * 뒷받침하는 필드, lateinit, lazy 등 위임 프로퍼티 사용 불가
* 인라인 클래스는 초기화 블록(init {}) 포함 불가

---

* 원시 값을 포함한 인라인 클래스를 사용해도 박싱하지 않은 값을 사용하지 못하는 경우 인라인되지 않은 형태로 클래스가 사용될 수 있음
* 다른 값으로 캐스팅되지 않은 프로퍼티 타입의 값을 사용할 때만 인라인된다.

```kotlin
fun safeAmount(dollar: Dollar?) = dollar?.amount ?: 0

fun main() {
    println(Dollar(15).amount)      // 인라이닝됨
    println(Dollar(15))             // Any?로 사용되기 때문에 인라이닝되지 않음 println(Any?)
    println(safeAmount(Dollar(15))) // Dollar?로 사용되기 때문에 인라이닝되지 않음
}
```

### 6.3.2 부호 없는 정수

* 코틀린 버전 1.3부터 부호 있는 타입을 기반, 인라인 클래스로 작성된 부호 없는 정수 타입 제공
    * 코틀린 1.5부터 표준 라이브러리 기능으로 정식 도입됨
* 부호 있는 타입앞에 "U" 를 붙인 이름으로 제공
* 값을 표현하려면 리터럴 뒤에 "u", "U" 접미사를 붙인다.
* 부호 있는 타입, 부호 없는 타입은 서로 호환되지 않음
    * to{Type}() 메서드로 반대 타입으로 변환할 수 있음
    * 서로 섞어서 연산을 하는 것도 불가능
* 부호 없는 타입은 단항 부호 반전(-) 연산자 사용 불가능
* 비트 연산, 비교 연산, 범위 타입, 정수 배열 타입도 모두 존재하고 있음
    * 부호 없는 정수 배열 타입은 실험 단계로 @ExperimentalUnsignedTypes 애너테이션을 붙여주거나 컴파일러 실행 시 옵션을 붙여야함
