# 3장 함수 정의하기

## 3.1 함수

* 파라미터를 받아 출력값을 반환할 수 있는 코드 블록

### 3.1.1 코틀린 함수의 구조

```kotlin
fun func(param: Int): String {
    return param.toString()
}
```

* 함수 정의를 위해 fun 이라는 키워드 사용
* 파라미터 목록, 반환 타입 정의 가능
    * 파라미터, 반환타입은 항상 타입을 정의해야함
        * 반환 타입이 없으면 Unit(void) 타입 반환으로 간주함
    * 코틀린 함수 파라미터는 무조건 **불변**
* {}로 함수 구현 정의


* return, {} 는 단일 식을 반환한다면 생략 가능
    * 이 경우 반환 타입은 생략 가능

```kotlin
fun identify(param: Int) = param
```

### 3.1.2 위치 기반 인자와 이름 붙은 인자

#### 위치 기반 인자(positional argument)

* 함수 호출 인자가 순서대로 전달되는 것

#### 이름 붙은 인자(named argument)

* 파라미터의 이름을 명시해 인자를 전달하는 것
* 인자의 순서는 중요하지 않음
* 위치 기반 인자와 이름 붙은 인자 함께 사용 가능

```kotlin
fun func(param1: Int, param2: Int) {}

func(param2 = 100, param1 = 150)

func(150, param2 = 100)     // param1은 150
```

### 3.1.3 오버로딩과 디폴트 값

#### 오버로딩

* 오버로딩한 함수의 파라미터 타입은 모두 달라야함
* 자바의 오버로딩 해소 규칙과 비슷한 규칙을 따라 호출함
    1. 파라미터의 개수와 타입을 기준으로 호출할 수 있는 모든 함수를 찾는다
    2. 덜 구체적인 함수를 제외시킨다. 파라미터 타입이 다른 함수의 파라미터 타입의 상위 타입일 경우 덜 구체적인 함수
    3. 후보가 하나로 압축되면 호출할 함수 결정 완료

#### 디폴트 값

```kotlin
fun defaultValue(param: Int = 10) = param
```

* 파라미터 뒤에 변수 초기화 식을 통해 기본값 지정 가능
* 기본 값이 있는 인자를 뒤에 두는 것이 좋은 스타일

### 3.1.4 vararg

* 인자의 개수가 정해지지 않은 함수 파라미터
* vararg는 배열 타입으로 간주됨
    * 배열을 스프레드 연산자(*)를 이용해 인자로 넘길 수 있음
        * 하지만 스프레드 연산자는 배열을 복사함

```kotlin
fun varargFunc(vararg items: Int) {
    items.sort()
}

val arr = intArrayOf(4, 3, 2, 1)
varargFunc(*arr)
```

* 둘 이상을 vararg로 선언하는 것은 금지됨
* vararg 파라미터가 맨 마지막이 아니라면, 이름 붙은 인자를 사용해야 나머지 인자를 전달할 수 있음
    * 마지막에 vararg 파라미터를 위치시키는 것이 좋은 스타일

### 3.1.5 함수의 영역과 가시성

* 코틀린 함수는 세가지로 구분됨
    * 최상위 함수
        * 파일에 직접 선언됨
    * 멤버 함수
        * 어떤 타입의 내부에 선언됨
    * 지역 함수
        * 다른 함수 안에 선언됨

#### 최상위 함수

* 최상위 함수는 공개 함수
* private, internal 키워드로 가시성을 변경할 수 있음
    * private
        * 파일 내부에서만 호출 가능
    * internal
        * 함께 컴파일되는 모듈 내부에서만 호출 가능

#### 지역 함수

* 함수 내부에 지역 함수 정의 가능
* 지역 함수는 선언한 함수 블록 내에서만 호출 가능
    * 블록 내의 변수, 파라미터에 접근 가능
* 지역 함수, 변수는 가시성 변경자를 붙일 수 없음

```kotlin
fun func(param1: Int) {
    fun func2() = param1 + 2
    func2()
}
func2()     // 에러
```

* 코틀린에서는 최상위, 지역 함수를 특별한 클래스를 선언해 호출한다.
    * 지역 함수를 호출할 때마다 클래스 생성 비용이 발생한다.

## 3.2 패키지와 임포트

* 패키지는 관련 있는 선언을 묶는 방법

### 3.2.1 패키지와 디렉터리 구조

* 자바의 패키지 선언과 동일함
* 자바는 디렉터리 구조와 패키지 선언을 일치시켜야 하지만, 코틀린은 그러지 않아도됨
    * 하지만 일치하는 것을 권장함

```kotlin
package a.b.c
```

### 3.2.2 임포트 디렉티브 사용하기

* 자바 문법과 유사함
* as를 이용해 다른 패키지의 같은 식별자를 구분할 수 있음

```kotlin
import java.lang.Math
import a.Value as aValue
import b.Value as bValue
import kotlin.math.*
```

## 3.3 조건문

* 조건문을 식으로 표현 가능
    * 식으로 표현할 때는 else가 반드시 있어야함

```kotlin
fun func(a: Int, b: Int) = if (a > b) a else b
```

### 3.3.2 범위, 진행, 연산

#### 범위

* 순서가 정해진 값 사이의 수열 표현
* `..` 을 사용한 표현은 닫혀 있음

```kotlin
val chars = 'a'..'h'    // a 부터 h
val oneToTen = 1..10    // 1 부터 10
```

* 어떤 범위에 값이 포함되는지, 안되는지 확인

```kotlin
if (num in 10..99) {        // num 이 10~99 사이에 포함되는지
}
if (num !in 10..99) {       // num 이 10-99 사이에 포함되지 않는지
}
```

* 반만 닫혀 있는 범위 표현

```kotlin
val oneToNine = 1 until 10  // 1 부터 9
```

#### 진행

* 정해진 간격만큼 떨어져 있는 시퀀스
    * .. 을 통해 오름차순 진행
    * downTo를 통해 내림차순 진행
    * step 을 통해 진행 범위 지정 가능

```kotlin
val evenAscending = 2..10 step 2    // 2, 4, 6, 8, 10
val desending = 10 downTo 1         // 10부터 1까지
val oddDesending = 9 downTo 1 step 2// 9, 7, 5, 3, 1
```

* 범위와 진행은 {}Range, {}Progression 으로 정의되어 있음
    * `kotlin.ranges` 패키지에 위치함
* 범위는 동적으로 할당되는 객체라 사용 시 부가 비용 발생할 수 있음
    * 컴파일러에서 어느정도 최적화는 해준다.

### 3.3.3 when 문과 여럿 중에 하나 선택하기

```kotlin
fun func(n: Int): Boolean {
    when {
        n in 0..9 -> return true
        else -> return false
    }
}
fun func1(n: Int): Boolean = when {
    n in 0..9 -> true
    else -> false
}
fun func2(n: Int): Boolean = when (n) {
    in 0..9 -> true
    else -> false
}
fun func3(n: Int): Boolean {
    return when {
        n in 0..9 -> true
        else -> false
    }
}
fun func4(n: Int): Boolean {
    when (n) {
        in 0..9 -> return true
        else -> return false
    }
}
```

* 다양한 방식으로 표현 가능

## 3.4 푸르

### 3.4.1 while과 do-while 루프

* 자바와 동일한 구조

```kotlin
while (true) {
}

do {
} while (true)
```

### 3.4.2 for 루프와 이터러블

```kotlin
val a = intArrayOf(1, 2, 3, 4)
var sum = 0
for (i in a) {
    sum += i        // 1+2+3+4
}

val s = "abcde"

for (c in s) {
    print(c + 1)        // bcdef 출력
}

for (i in a.indices step 2) {
    a[i] *= 2       // [2, 2, 6, 4]
}
```

* 코틀린 타입은 for 루프를 사용하기 위해서는 iterator() 함수만 지원하면 됨

### 3.4.3 루프 제어 흐름 변경하기: break와 continue

* break
    * 즉시 루프 종료
* continue
    * 현재 루프 이터레이션 종료, 다시 조건 검사 실행

### 3.4.4 내포된 루프와 레이블

* 중첩된 루프에서는 break, continue는 가장 안쪽의 루프에서만 실행됨
* 레이블을 이용해 중첩된 루프에서 제어 흐름 변경 가능

```kotlin
fun indexOf(subarray: IntArray, array: IntArray): Int {
    outerLoop@ for (i in array.indices) {
        for (j in subarray.indices) {
            if (subarray[j] != array[i + j]) continue@outerLoop
        }
        return i
    }
    return -1
}
```

### 3.4.5 꼬리 재귀 함수

* 꼬리 재귀 함수
    * 재귀 호출 다음에 아무 동작도 수행하지 않는 함수
* 꼬리 재귀 함수에 tailrec을 붙이면 재귀함수를 비재귀적으로 변환해준다.

## 3.5 예외 처리

* 자바와 아주 비슷함

### 3.5.1 예외 던지기

* throw 를 이용해 예외 객체 던질 수 있음

### 3.5.2 try 문으로 예외 처리하기

* try-catch 문으로 예외 처리 가능
* catch가 선언된 순서대로 예외 타입을 검사하기 때문에 상위 타입의 예외가 앞에 위치해야함
* 자바처럼 `catch (NullPointerException | IllegalArgumentException e)` 구문으로 여러 예외 타입 처리는 미지원
* 코틀린은 검사, 비검사 예외는 없다
* finally 구문으로 예외 처리 이후 수행할 작업 선언 가능