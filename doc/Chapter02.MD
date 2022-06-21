# 2장 코틀린 언어 기초

## 2.1 기본 문법

### 2.1.1 주석

```kotlin
// 한줄 주석
/*
여러 줄 주석
여러 줄 주석
/*
여러 줄 주석 안에 또 여러 줄 주석을 넣는 것도 가능
 */
 */
/**
 * KDoc 여러 줄 주석
 * KDoc 여러 줄 주석
 */
```

### 2.1.2 변수 정의하기

```kotlin
val value = 15
val intValue: Int = 15
```

* 세미 콜론은 없어도 된다.
* 값을 초기화할 경우, 타입을 추론해준다.
    * 타입을 지정할 수도 있다.

### 2.1.3 식별자

* 변수, 함수 등 정의된 대상에 붙은 이름
* 하드 키워드는 사용할 수 없다.
* \` 를 감싼 식별자로 어떤 문자열이든 식별자로 사용할 수 있다.
    * 자바와의 키워드 호환성을 위해 제공하는 기능

```kotlin
val `fun` = 1
val `any string` = "abcd"
```

### 2.1.4 가변 변수

* `val` 변수는 불변 변수
* `var` 변수는 가변 변수
* 추론된 타입은 계속 유지된다.

### 2.1.5 식과 연산자

* 모든 식에는 타입이 존재한다.
* 연산에는 우선순위가 존재하므로 우선순위를 잘 알아두어야 한다.

#### 종류

* 리터럴(123, "abc")
* 참조, 함수 호출
* 단항 연산
* 이항 연산

## 2.2 기본 타입

* 자바에는 primitive 타입, 참조 타입의 명확한 구분이 있었지만, 코틀린은 문맥에 따라 구분되어 모호함
* 코틀린 타입은 근본적으로 클래스 정의를 기반으로 만들어짐
    * Int같은 원시 타입도 메서드, 프로퍼티를 제공할 수 있음

### 2.2.1 정수 타입

| 이름    | 크기  | 범위             | 대응하는 자바 타입 |
|-------|-----|----------------|------------|
| Byte  | 1   | -128 ~ 127     | Byte       |
| Short | 2   | -32768 ~ 32767 | Short      |
| Int   | 4   | -2^31 ~ 2^31-1 | Integer    |
| Long  | 8   | -2^63 ~ 2^63-1 | Long       |

* 정수 사이에 _ 문자를 통해 가독성 높이기 가능

```kotlin
val n = 987_654_321
```

* 리터럴의 타입은 초기화되는 크기에 따라 추론됨
* 타입을 지정할 경우 크기보다 큰 값을 대입하면 에러 발생

```kotlin
val tooBig: Short = 100_000 // 에러 발생
```

* 정수 리터럴에 `l`, `L` 을 접미사로 붙이면 Long 타입

```kotlin
val longValue = 10000L
```

* 2진수, 16진수 표현도 가능
* 0으로 시작하는 8진수 리터럴은 지원하지 않음

```kotlin
val binary = 0b101010
val hex = 0xFFFF
val oct = 016    // 에러 
```

* 각 정수 타입에는 최솟값, 최댓값 상수 정의 있음

```kotlin
Short.MIN_VALUE
Long.MAX_VALUE
```

### 2.2.2 부동소수점 수

* Float, Double
* Float, Double은 자동 형변환되지 않음

```kotlin
val pi = 3.14
val quarter = .25   // 정수 부분 없으면 0으로 간주
val one = 1.    // 에러, 소수점 있으며 소수 부분 생략 불가
val two = 2     // 정수

val pi = 31.4E-1        // 과학적 표기법도 허용
val thousand = 1E3      // 과학적 표기법에서는 소수부분 생략 가능

val double = 3.14       // 타입 추론 결과 Double
val float = 3.14f       // 접미사로 f,F 를 붙이면 Float 타입

// 타입별 특별한 상수 정의 제공
Double.MAX_VALUE        // 가장 큰 유한값 
Double.MIN_VALUE        // 가장 작은 유한값
Double.POSITIVE_INFINITY// 양의 무한대
Float.NEGATIVE_INFINITY // 음의 무한대
Float.NaN               // 숫자가 아닌 값
```

### 2.2.3 산술 연산

* +(단항), -(단항), +(덧셈), -(뺄셈), *(곱셈), /(나눗셈), %(나머지) 연산 모두 제공
* 정수 타입은 몫 연산(`floorDiv()`), 나머지 연산(`mod()`) 메서드 제공(코틀린 1.5 이후)
* ++, -- 단항 연산 제공

### 2.2.4 비트 연산

* Int, Long 타입이 지원
    * 코틀린 1.1부터는 Byte, Short 타입 or, xor, inv 지원
* `shl`(왼쪽 시프트), `shr`(오른쪽 시프트), `ushr`(부호 없는 오른쪽 시프트), `and`(비트 곱), `or`(비트 합), `xor`(비트 배타합), `inv()`(비트 반전)
    * inv는 메서드(`15.inv()`)

### 2.2.5 문자 타입 Char

* 유니코드 한 글자 표현(2 바이트)
* 문자를 이용한 몇가지 연산 가능
    * 덧셈, 뺏셈, ++, --

```kotlin
val char = 'a'
val an = '안'
```

### 2.2.6 수 변환

* `to{Type}()` 형식의 메서드
* 코틀린은 범위가 큰 타입에 작은 타입을 대입할 수 없음

```kotlin
val n: Int = 100
val l: Long = n     // 에러 발생
```

### 2.2.7 불 타입과 논리 연산

* Boolean 타입 제공(true, false)
* 수 타입과는 다른 타입, 수로 변환 불가능

#### 불 연산

* !
    * 논리 부정
* or, and, xor
    * 즉시 계산되는 논리합, 논리곱, 논리배타합
* ||, &&
    * 지연 계산 방식의 논리합, 논리곱

#### 비교와 동등성

* ==, !=, <, <=, >, >= 제공
* ==, != 는 같은 타입일 때만 허용

## 2.3 문자열

* String 타입
* 코틀린 문자열은 불변

### 2.3.1 문자열 템플릿

* 기본 코틀린 문자열 정의

```kotlin
val string = "hello, kotlin"
```

* 코틀린 식을 문자열에 넣을 수 있음
    * ${} 사용하면 됨

```kotlin
val name = "AYJ"
val hello = "Hello $name, Today is ${Date()}"
```

* 여러 줄을 이스케이프 문자 없이 표현하는 방법도 있음

```kotlin
val multiline = """
    Hello
    Hi
    Nice to Meet You!
""".trimIndent()
```

* `trimIndent()` 메서드를 이용해 들여쓰기 제거 가능

### 2.3.2 기본 문자열 연산

```kotlin
"Hello".length      // 문자열의 길이
"Hello".lastIndex   // 문자열의 마지막 문자의 인덱스

"Hello"[0]          // 'H'

"Hello" + " " + "World"     // 문자열 연결 연산 가능

"Hell" == "Hell"        // ==, != 이용해 문자열 비교 가능

"aaa" > "abc"       // 사전 순서 비교 가능

"true".toBoolean()  // 수 타입, 불 타입으로 변환 가능
"1233".toShort()
```

## 2.4 배열

### 2.4.1 배열 정의하기

* 기본 타입 Array<T> 제공
    * 하지만 모든 타입을 박싱함
* ByteArray, IntArray 등 원시 타입을 위한 배열 제공

```kotlin
val emptyArr = emptyArray<String>()
val strArr = arrayOf("abc", "abcd")

val size = 15
val sizeArr = Array(size)

val boolArr = BooleanArray(2)
val charArr = charArrayOf('A', 'B')
val longArr = longArrayOf(123L, 234L)

val arrInit = IntArray(10) { it }       // it는 인덱스, 0 ~ 9 까지로 초기화
```

### 2.4.2 배열 사용하기

* size, lastIndex 제공
* [] 를 이용해 인덱스의 값 접근, 변경 가능
* 배열 타입의 변수는 참조
    * 다른 배열 대입 시 같은 집합 공유함
    * 복사하려면 `copyOf()` 메서드 사용한다.
* 타입이 다른 배열은 대입할 수 없음
* 배열 생성 이후 길이를 바꿀 수 없음

```kotlin
val newArray = intArrayOf(1, 2, 3) + 4      // [1, 2, 3] 에 4 추가해 새로운 배열([1, 2, 3, 4]) 생성
```

* 배열 비교는 참조 비교
    * 내용을 비교하려면 `contentEquals()` 메서드 사용
