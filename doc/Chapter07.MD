# 7장 컬렉션과 I/O 자세히 알아보기

## 7.1 컬렉션

* 엘리먼트들로 이뤄진 그룹을 저장하기 위해 설계된 객체
* 컬렉션을 조작하는 모든 연산은 인라인 함수
    * 함수 호출, 람다 호출 등의 부가 비용은 발생하지 않음

### 7.1.1 컬렉션 타입

* 배열, 이터러블(Iterable), 시퀀스(Sequence), 맵(map)
* 컬렉션 타입은 제네릭 타입

#### 이터러블

* `Iterable\<T\>` 로 표현됨
* 즉시 계산되는 상태가 있는 컬렉션
    * 상태가 있음 => 원소를 필요할 때 생성하는 제너레이터 함수를 유지하지 않고 저장한다.
    * 즉시 계산 => 컬렉션을 최초로 생성할 때 원소가 초기화된다.
* `iterator()` 메서드 제공
    * 원소를 순회할 수 있는 메서드
    * for 루프에서 `iterator()` 메서드를 통해 원소를 순회할 수 있음
* 코틀린 컬렉션은 불변, 가변 컬렉션을 구분한다.
    * 불변 컬렉션은 생성한 후 내용을 바꿀 수 없음
    * 불변 컬렉션을 사용하면 공변성을 성립시킬 수 있음
        * T가 U의 하위 타입이라면 `Iterable<T>`는 `Iterable<U>`의 하위타입

#### 컬렉션, 리스트, 집합

* 이터러블의 하위 분류
    * Collection 인터페이스 타입
    * MutableCollection 인터페이스 타입

##### Collection 상속 클래스

* 리스트
    * 인덱스를 통한 원소 접근이 가능한 순서가 정해진 원소의 컬렉션
* 집합
    * 유일한 원소들로 이뤄진 컬렉션
    * HashSet. LinkedHashSet, TreeSet 등

#### 시퀀스

* iterator() 메서드를 제공하는 타입
* 시퀀스는 Iterable과는 다르게 지연 계산을 가정함
    * 객체 초기화 시 원소를 초기화하지 않고 요청에 따라 원소를 계산함
    * 시퀀스는 대부분 상태를 가지고 있지 않음
* 시퀀스 구현은 내부적이라 외부에서 직접 사용할 수 없음
    * 특별한 함수를 이용해 시퀀스를 만들어야 함

#### 맵

* 유일한 키와 값 쌍으로 이뤄진 집합
* 맵은 Collection의 하위 타입이 아님
    * 맵의 원소를 컬렉션처럼 사용 가능
* HashMap, LinkedHashMap, TreeMap 등

### 7.1.2 Comparable과 Comparator

* Comparable
    * 자연적인 순서(natural order) 지원
    * 동일한 타입의 다른 인스턴스와 순서를 비교할 수 있는 `compareTo()` 메서드 포함
        * 수신 객체가 전달받은 인자보다 크면 양수, 작으면 음수, 같으면 0 반환
    * <, > 연산 사용 가능
* Comparator
    * 클래스를 여러 방법으로 비교해야 할 때 사용하는 클래스
    * 같은 타입의 인자를 두 개 받는 `compare()` 메서드를 제공
        * compareTo()와 동일한 비교 결과 반환
    * 람다를 통해 비교기 작성 가능

### 7.1.3 컬렉션 생성하기

* 표준 컬렉션 클래스의 생성자를 호출해서 생성 가능

```kotlin
val list = ArrayList<String>()
list.add("red")
list.add("green")
println(list) // [red, green]

val hashSet = HashSet<Int>()
hashSet.add(12)
hashSet.add(21)
hashSet.add(12)
println(hashSet) // [12, 21]

val map = java.util.TreeMap<Int, String>()
map[20] = "Twenty"
map[10] = "Ten"
println(map) // {10=Ten, 20=Twenty}
```

* 가변 길이 인자를 갖는 함수를 이용해서도 컬렉션 인스턴스 생성 가능
    * `emptyList()`, `emptySet()`
    * `listOf()`, `setOf()`
    * `listOfNotNull()`
    * `mutableListOf()`, `mutableSetOf()`
    * `arrayListOf()`
    * `hashSetOf()`, `linkedSetOf()`, `sortedSetOf()`
    * `emptyMap()`
    * `mapOf()`
    * `mutableMapOf()`
    * `hashMapOf()`, `linkedMapOf()`, `sortedMapOf()`

#### 시퀀스 생성

* `sequenceOf()` 메서드를 이용해 시퀀스 생성 가능
* 컬렉션 인스턴스의 `asSequence()` 메서드로도 시퀀스 생성 가능
* 제너레이터 함수를 이용해서도 시퀀스 생성 가능
    * 제너레이터는 파라미터가 없는 함수를 인자로 받고, 그 함수가 널을 반환할 때까지 계속해서 시퀀스 원소를 생성함

```kotlin
val numbers = generateSequence { println("Input a number:");readLine()?.toIntOrNull() }
// 숫자가 아닌 입력을 받거나 입력이 끝날때까지 원소 반환하는 시퀀스
```

* 코틀린 1.3부터 특별한 빌더를 이용해 시퀀스 생성 가능
    * `sequence()`
        * SequenceScope가 수신 객체 타입인 확장 람다를 받는 메서드
        * yield()
            * 원소를 하나 시퀀스에 추가한다.
        * yieldAll()
            * 저장한 이터레이터, 이터러블, 시퀀스에 들어있는 모든 원소를 시퀀스에 추가한다.
        * 두 함수를 이용해 시퀀스 구성 가능

```kotlin
val numbers = sequence {
    yield(0)
    yieldAll(listOf(1, 2, 3))
    yieldAll(intArrayOf(4, 5, 6).iterator())
    yieldAll(generateSequence(10) { if (it < 50) it * 3 else null })
}
println(numbers3.toList()) // [0, 1, 2, 3, 4, 5, 6, 10, 30, 90]
```

* 시퀀스 빌더는 유예 가능 계산이라는 코틀린 기능의 예시다.

#### 컬렉션 변환

* `to{Type}()` 메서드를 이용해 컬렉션 사이의 변환을 수행할 수 있음
* `as{Type}()` 메서드를 사용하면 원본 컬렉션의 뷰를 생성해줌

### 7.1.4 기본 컬렉션 연산

* 이터레이션
    * iterator() 메서드 사용
    * 컬렉션의 원소를 순회할 수 있음
    * for 루프에서 사용 가능
    * forEach() 확장 함수도 사용 가능
        * 인덱스를 참조하려면 forEachIndexed() 함수 사용

```kotlin
intArrayOf(1, 2, 3).forEach { println(it * it) }
listOf(10, 20, 30).forEachIndexed { i, n -> println("$i: ${n * n}") }
```

#### 리스트, 집합

```kotlin
val list = listOf(1, 2, 3)

println(list.isEmpty())                 // false
println(list.size)                      // 3
println(list.contains(4))               // false
println(2 in list)                      // true     contains 대체 가능
println(list.containsAll(listOf(1, 2))) // true
```

* MutableCollection은 원소 조작 메서드 제공

```kotlin
val list = arrayListOf(1, 2, 3)
list.add(4)                     // 원소 하나 추가: [1, 2, 3, 4]
list.remove(3)                  // 원소 하나 제거: [1, 2, 4]
list.addAll(setOf(5, 6))        // 합집합: [1, 2, 4, 5, 6]
list.removeAll(listOf(1, 2))    // 차집합: [4, 5, 6]
list.retainAll(listOf(5, 6, 7)) // 교집합: [5, 6]
list.clear()                    // 모든 원소 제거: []

list += 4               // 원소 추가
list -= 3               // 원소 제거
list += setOf(5, 6)     // 원소 추가
list -= listOf(1, 2)    // 원소 제거
```

* 리스트는 배열 처럼 인덱스를 이용한 접근 가능

```kotlin
val list = listOf(1, 4, 6, 2, 4, 1, 7)

println(list.get(3))         // 2
println(list[2])             // 6
//println(list[10])            // java.lang.ArrayIndexOutOfBoundsException
println(list.indexOf(4))     // 1
println(list.lastIndexOf(4)) // 4
println(list.indexOf(8))     // -1
```

* 가변 리스트는 인덱스를 이용해 원소 변경도 가능

```kotlin
val list = arrayListOf(1, 4, 6, 2, 4, 1, 7)
val segment = list.subList(2, 5) // [6, 2, 4, 1]
// subList는 원본 컬렉션의 래퍼 뷰를 생성하기 때문에 원본 컬렉션의 변화가 반영된다.
list[3] = 0
println(segment[1])  // 0
segment[1] = 8
println(list[3])    // 8
```

#### Map

* Map은 키를 활용해 값을 얻는 메서드, 키, 값의 컬렉션을 반환하는 메서드 등을 제공한다.

```kotlin
val map = mapOf(1 to "I", 5 to "V", 10 to "X", 50 to "L")

println(map.isEmpty())              // false
println(map.size)                   // 4
println(map.get(5))                 // V
println(map[10])                    // X
println(map[100])                   // null
println(map.getOrDefault(100, "?")) // ?
println(map.getOrElse(100) { "?" }) // ?
println(map.containsKey(10))        // true
println(map.containsValue("C"))     // false
println(map.keys)                   // [1, 5, 10, 50]
println(map.values)                 // [I, V, X, L]
println(map.entries)                // [1=I, 5=V, 10=X, 50=L]
```

* MutableMap은 변경 연산도 지원한다.

```kotlin
val map = sortedMapOf(1 to "I", 5 to "V")

map.put(100, "C")            // {1=I, 5=V, 100=C}
map[500] = "D"               // {1=I, 5=V, 100=C, 500=D}
map.remove(1)                // {5=V, 100=C, 500=D}
map.putAll(mapOf(10 to "X")) // {5=V, 10=X, 100=C, 500=D}
map += 50 to "L"             // {5=V, 10=X, 50=L, 100=C, 500=D}
map += mapOf(
    2 to "II",
    3 to "III"
)                  // {2=II, 3=III, 5=V, 10=X, 50=L, 100=C, 500=D}
map -= 100                   // {2=II, 3=III, 5=V, 10=X, 50=L, 500=D}
map -= listOf(2, 3)          // {5=V, 10=X, 50=L, 500=D}

```

### 7.1.5 컬렉션 원소에 접근하기

* 코틀린 표준 라이브러리에는 개별 컬렉션 원소에 대한 접근을 편리하게 해주는 확장함수 포함

#### first(), last()

* 컬렉션의 첫번째, 마지막 원소 반환
* 컬렉션이 비어있으면 NoSuchElementException 발생
* firstOrNull(), lastOrNull() 을 사용하면 예외를 발생시키지 않고 원소가 없으면 널 반환
* 조건을 만족하는 첫번째, 마지막 원소를 찾기 위해 람다식을 넘길수도 있음

```kotlin
println(listOf(1, 2, 3).first { it > 2 })      // 31
println(listOf(1, 2, 3).lastOrNull { it < 0 }) // null
```

#### single()

* 싱글턴 컬렉션의 원소 반환
* 컬렉션이 비어있거나, 원소가 두 개 이상이면 예외 발생
* singleOrNull() 함수로 예외 대신 널 반환 가능

#### elementAt()

* 인덱스를 사용해 컬렉션 원소 접근 가능
* 배열, 이터러블, 시퀀스 등에 모두 적용 가능
    * 임의 접근 컬렉션이 아닌 경우 인덱스 값에 따른 탐색 비용 발생
* 인덱스가 잘못된 경우 예외 발생
* elementOrNull() => 인덱스가 잘못된 경우 널 반환
* elementAtOrElse() => 인덱스가 잘못된 경우 지정한 람다식 계산 결과 반환

#### 배열, 리스트 구조 분해

* 배열, 리스트를 구조분해해 최대 5개의 원소 추출 가능
* 컬렉션의 원소의 수보다 더 많은 원소를 구조분해 하려하면 예외 발생

```kotlin
val list = listOf(1, 2, 3)

val (x, y) = list        // 1, 2
//val (a, b, c, d) = list  // java.lang.ArrayIndexOutOfBoundsException
```

### 7.1.6 컬렉션에 대한 조건 검사

#### all()

* 컬렉션의 모든 원소가 주어진 조건식을 만족하면 true 반환
    * 모든 컬렉션 객체에 함수 적용 가능

```kotlin
println(listOf(1, 2, 3, 4).all { it < 10 })     // true
println(listOf(1, 2, 3, 4).all { it % 2 == 0 }) // false
println(
    mapOf(1 to "I", 5 to "V", 10 to "X")
        .all { it.key == 1 || it.key % 5 == 0 }
)                                               // true

val seq = generateSequence(1) { if (it < 50) it * 3 else null }

println(seq.all { it % 3 == 0 }) // false
println(seq.all { it == 1 || it % 3 == 0 }) // true
```

#### none()

* 컬렉션에 주어진 조건식을 만족하는 원소가 하나도 없을 경우 true 반환

```kotlin
println(listOf(1, 2, 3, 4).none { it > 5 }) // true
println(
    mapOf(1 to "I", 5 to "V", 10 to "X").none { it.key % 2 == 0 }
)                                           // false

// 1, 3, 9, 27, 81로 이뤄진 시퀀스 만들기
val seq = generateSequence(1) { if (it < 50) it * 3 else null }
println(seq.none { it >= 100 })             // true
```

#### any()

* 컬렉션의 원소 중 하나라도 주어진 조건식을 만족하면 true 반환

#### 빈 컬렉션, 무한 시퀀스

* 빈 컬렉션의 all(), none()의 결과는 true, any()는 false 이다.
* 무한 시퀀스에 all(), none(), any()를 적용할 경우 실행이 끝나지 않을 수 있음
* any(), none()은 파라미터를 전혀 받지 않는 버전 존재하고 있음
    * 컬렉션 객체가 비어있는지 여부 검사

```kotlin
println(emptyList<String>().any())  // false
println(emptyList<String>().none()) // true
println(listOf(1, 2, 3).any())      // true
println(listOf(1, 2, 3).none())     // false
```

### 7.1.7 집계

#### count()

* 컬렉션의 원소 개수 반환
* 원소 개수가 Int.MAX_VALUE 보다 크면 예외 반환
    * 무한 시퀀스에서의 count를 호출할 경우
* 조건 람다식을 적용하는 오버로딩 버전 존재하고 있음
    * 주어진 조건을 만족하는 원소 개수 반환

```kotlin
println(listOf(1, 2, 3, 4).count { it < 0 })      // 0
println(listOf(1, 2, 3, 4).count { it % 2 == 0 }) // 2
println(
    mapOf(1 to "I", 5 to "V", 10 to "X").count { it.key == 1 }
)                                                 // 1
```

#### sum()

* 수로 이뤄진 배열, 이터러블, 시퀀스의 산술 합계 반환
* sumOf를 이용해 원소 타입을 수로 변환할 수 있는 컬렉션도 합계 게산 가능

````kotlin
println(listOf(1, 2, 3, 4).sum())           // 10
println(doubleArrayOf(1.2, 2.3, 3.4).sum()) // 6.9

// 1, 3, 9, 27, 81의 합계 구하기
val seq = generateSequence(1) { if (it < 50) it * 3 else null }

println(seq.sum()) // 121

println(listOf(1, 2, 3, 4).sumOf { it / 4.0 })   // 2.5
println(arrayOf("1", "2", "3").sumOf { it.toInt() }) // 6

// X, XX, XXX, XXXX, XXXXX
val seq1 = generateSequence("X") {
    if (it.length >= 5) null else it + "X"
}

println(seq1.sumOf { it.length })                     // 15
````

#### average()

* 수로 이뤄진 배열, 이터러블, 시퀀스의 산술 평균 반환
    * 반환 타입은 Double
* 컬렉션이 비어있을 경우 Double.NaN 반환

```kotlin
println(listOf(1, 2, 3, 4).average())           // 2.5
println(doubleArrayOf(1.2, 2.3, 3.4).average()) // 2.3000000000000003

// 1, 3, 9, 27, 81의 평균 구하기
val seq = generateSequence(1) { if (it < 50) it * 3 else null }

println(seq.average())                             // 24.2
```

#### minOrNull(), maxOrNull()

* 비교 가능한 타입의 값이 들어있는 배열, 이터러블, 시퀀스의 최솟값, 최댓값 계산
* 최댓값, 최솟값 연산은 빈 컬렉션의 경우 널 반환

```kotlin
println(intArrayOf(5, 8, 1, 4, 2).minOrNull())               // 1
println(intArrayOf(5, 8, 1, 4, 2).maxOrNull())               // 8
println(listOf("abc", "w", "xyz", "def", "hij").minOrNull()) // abc
println(listOf("abc", "w", "xyz", "def", "hij").maxOrNull()) // xyz

// 1, -3, 9, -27, 81
val seq = generateSequence(1) { if (it < 50) -it * 3 else null }

println(seq.minOrNull()) // -27
println(seq.maxOrNull()) // 81
```

#### minByOrNull(), maxByOrNull()

* 비교 가능한 값으로 원소를 변환하는 함수를 제공해 최대, 최소 값을 반환할 수 있는 함수

```kotlin
class Person(
    val firstName: String,
    val familyName: String,
    val age: Int
) {
    override fun toString() = "$firstName $familyName: $age"
}

fun main() {
    val persons = sequenceOf(
        Person("Brook", "Watts", 25),
        Person("Silver", "Hudson", 30),
        Person("Dane", "Ortiz", 19),
        Person("Val", "Hall", 28)
    )

    println(persons.minByOrNull { it.firstName })  // Brook Watts: 25
    println(persons.maxByOrNull { it.firstName })  // Val Hall: 28
    println(persons.minByOrNull { it.familyName }) // Val Hall: 25
    println(persons.maxByOrNull { it.familyName }) // Brook Watts: 28
    println(persons.minByOrNull { it.age })        // Dane Ortiz: 19
    println(persons.maxByOrNull { it.age })         // Silver Hudson: 30
}
```

#### minWithOrNull(), maxWithOrNull()

* Comparator 를 전달받아 비교를 수행해 최대, 최소 값을 반환하는 함수

#### joinToString()

* 컬렉션 원소를 문자열로 엮는 함수
    * 기본 형식은 원소의 구분 문자열로 ", " 를 사용하는 것
* 람다식을 이용해 출력되는 문자열의 형식을 지정할 수 있음

```kotlin
println(listOf(1, 2, 3).joinToString()) // 1, 2, 3

println(listOf(1, 2, 3).joinToString { it.toString(2) }) // 1, 10, 11
// 컬렉션 원소 이진수 형식으로 출력
```

* 다양한 파라미터도 지정가능
    * seperator
        * 구분 문자열(기본값: ", ")
    * prefix, postfix
        * 접두, 접미 문자열 지정(기본값: "")
    * limit
        * 최대 출력 원소 개수(기본값: -1(제한 없음))
    * truncated
        * limit가 양수인 경우, 컬렉션의 원소를 모두 표현하지 못할 때 출력하는 접미사(기본값: "...")

#### joinTo()

* 문자열을 생성하는 대신 파라미터로 전달받은 Appendable 객체 뒤에 덧붙여줌
    * StringBuilder -> Appendable의 하위 타입

```kotlin
fun main() {
    val builder = StringBuilder("joinTo: ")
    val list = listOf(1, 2, 3)

    println(list.joinTo(builder, separator = "|")) // joinTo: 1|2|3
}
```

#### reduce(), fold()

* 파라미터가 두 개인 함수를 전달받는다.
    * 첫번째 인자는 누적된 값
    * 두번째 인자는 컬렉션의 현재 값
* 최초에는 컬렉션의 첫번째 값이 누적된 값으로 설정되고, 함수의 본문을 수행한 이후 결과가 계속해서 누적값이 된다.
* 컬렉션이 비어있으면 예외를 던진다.

```kotlin
println(intArrayOf(1, 2, 3, 4, 5).reduce { acc, n -> acc * n })  // 120
println(listOf("a", "b", "c", "d").reduce { acc, s -> acc + s }) // abcd
```

* reduceIndexed() 를 이용해 인덱스를 이용할 수도 있다.
    * 하지만 첫번째 원소는 무조건 처리된다는 점을 유의해야 한다.

* fold()를 이용해 누적값의 초기값을 지정할 수 있다.
    * 사용법은 reduce와 동일하다.
* 반대 방향부터 계산을 수행하는 reduceRight(), foldRight(), reduceRightIndexed(), foldRightIndexed() 함수도 존재한다.

### 7.1.8 걸러내기

#### filter()

* 원소를 유지해야 하는 경우 true, 아닐 경우 false를 반환하는 람다식을 전달받는 함수
* 배열, 이터러블, 맵, 시퀀스에 적용가능하고 반환 타입은 각각 정해져 있다.
    * 배열, 이터러블 => List
    * 맵 => Map
    * 시퀀스 => Sequence

```kotlin
// List: [green, blue, green]
println(
    listOf("red", "green", "blue", "green").filter { it.length > 3 }
)

// List: [green, blue]
println(setOf("red", "green", "blue", "green").filter { it.length > 3 })

// List: [green, blue, green]
println(
    arrayOf("red", "green", "blue", "green").filter { it.length > 3 }
)

// List: [2, 4]
println(byteArrayOf(1, 2, 3, 4, 5).filter { it % 2 == 0 })

// Map: {X=10, L=50}
println(
    mapOf("I" to 1, "V" to 5, "X" to 10, "L" to 50)
        .filter { it.value > 5 }
)

// Sequence
val seq = generateSequence(100) {
    if (it != 0) it / 3 else null
}.filter { it > 10 }

// 리스트로 변환: [100, 33, 11]
println(seq.toList())
```

* 맵의 경우 키, 값만 걸러내고 싶다면 filterKeys(), filterValues() 함수 사용 가능
* filterNot()으로 조건 부정해 걸러낼 수 있음
* filterIndexed()로 인덱스도 사용 가능
* 표준 라이브러리에서는 일반적인 조건을 사용해 걸러내는 함수 제공
    * filterNotNull()
        * 널이 아닌 원소만 컬렉션으로 만들어 반환
    * filterIsInstance()
        * 특정 타입의 원소만 컬렉션으로 만들어 반환
* filterTo() 메서드를 이용해 가변 컬렉션에 걸러내기 결과 추가 가능

#### partition()

* 컬렉션을 조건을 만족하는 부분 컬렉션과 만족하지 않는 부분 컬렉션의 쌍으로 만들어주는 함수

```kotlin
val (evens, odds) = listOf(1, 2, 3, 4, 5).partition { it % 2 == 0 }
println(evens) // [2, 4]
println(odds) // [1, 3, 5]
```

### 7.1.9 변환

#### map()

* 원본 컬렉션의 원소를 변환하는 함수를 전달받는 함수
* mapIndexed() 로 인덱스 사용 가능
* mapNotNull(), mapIndexedNotNull()
    * 널이 아닌 값만 선택해 변환하는 함수
* mapKeys(), mapValues()
    * 맵의 키, 값만 변환한 맵을 반환하는 함수
* mapTo() 함수로 변환 결과를 가변 컬렉션의 원소로 추가 가능

```kotlin
println(setOf("red", "green", "blue").map { it.length }) // [3, 5, 4]
println(listOf(1, 2, 3, 4).map { it * it })                // [1, 4, 9, 16]
println(byteArrayOf(10, 20, 30).map { it.toString(16) }) // [a, 14, 1e]
```

#### flatMap()

* 컬렉션의 각 원소를 컬렉션으로 변환한 다음, 변환된 컬렉션을 차례로 이어붙여 하나의 컬렉션으로 합치는 함수

```kotlin
// [a, b, c, d, e, f, g, h, i]
println(setOf("abc", "def", "ghi").flatMap { it.asIterable() })

// [1, 2, 3, 4]
println(listOf(1, 2, 3, 4).flatMap { listOf(it) })

// [1, 1, 2, 1, 2, 3]
Array(3) { it + 1 }.flatMap { 1..it }
```

#### flatten()

* 원소가 컬렉션인 모든 컬렉션에 적용 가능
* 각각의 컬렉션을 이어붙인 하나의 컬렉션 반환
* flatMapTo()를 이용해 가변 컬렉션의 원소로 추가 가능

```kotlin
println(
    listOf(listOf(1, 2), setOf(3, 4), listOf(5)).flatten()    // flatMap { it } 와 동일함
) // [1, 2, 3, 4, 5]

println(Array(3) { arrayOf("a", "b") }.flatten()) // [a, b, a, b, a, b]
```

#### associateWith()

* 원본 컬렉션의 원소를 맵의 키로, 변환 함수의 결과를 값으로 하는 맵을 반환하는 함수

#### associateBy()

* 원본 컬렉션의 원소를 맵의 값으로, 변환 함수의 결과를 키로 하는 맵을 반환하는 함수

#### associate()

* 컬렉션의 원소를 사용해 키와 값을 만들어낸 맵을 반환하는 함수

### 7.1.10 하위 컬렉션 추출

#### slice()

* 정수 범위를 사용해 부분 리스트를 추출할 수 있음
* 배열을 추출하려면 sliceArray() 사용

#### take(), drop()

* take
    * 이터러블, 배열에서 원소를 주어진 개수만큼 앞에서 추출
    * takeLast()를 사용해 뒤에서부터 추출 가능
* drop
    * 주어진 개수만큼 원소를 제거한 나머지 원소 반환
    * dropLast()를 사용해 뒤에서부터 제거 가능
* take/drop 연산은 람다식을 전달해 조건을 만족하지 못할때까지 원소를 추출, 제거 가능

#### chunked()

* 코틀린 1.2에 추가
* 이터러블, 시퀀스를 주어진 개수를 넘지 않는 작은 리스트로 나눠줌
* 최대 청크 크기를 인자로 받아 부분 리스트를 만들거나, 람다식을 전달받아 청크를 계산한 결과를 하나의 리스트로 반환할 수 있음

#### windowed()

* 코틀린 1.2에 추가
* 일정한 간격으로 청크를 연속적으로 얻어낸 슬라이딩 윈도우 반환
* 파라미터를 통해 슬라이딩 윈도우 생성 규칙 지정가능
    * step
        * 서로 인접한 윈도우의 첫번째 원소 사이의 거리(기본값: 1)
    * partialWindows
        * 컬렉션의 마지막 부분에서 지정한 윈도우 크기보다 작은 크기의 윈도우를 포함시킬지 여부(기본값: false)

```kotlin
// 0, 1, 4, 9, 16, 25
val list1 = List(6) { it * it }

// [[0, 1, 4], [1, 4, 9], [4, 9, 16], [9, 16, 25]]
println(list1.windowed(3))

// 1, 3, 9, 27, 81, 243
val seq = generateSequence(1) { if (it > 100) null else it * 3 }

// [[1, 3, 9], [3, 9, 27], [9, 27, 81], [27, 81, 243]]
println(seq.windowed(3).toList())

// 0, 1, 4, 9, 16, 25
val list2 = List(6) { it * it }

println(list2.windowed(3) { it.sum() }) // [5, 14, 29, 50]
```

* zipWithNext()를 이용해 원소가 두 개뿐인 윈도우 생성 가능
    * Pair의 시퀀스, 리스트 반환

### 7.1.11 순서

#### sorted()

* 원소가 비교가능한 배열, 이터러블, 시퀀스에 적용 가능
    * 정렬된 새로운 컬렉션을 반환함
* 오름차순으로 원소 정렬
    * sortedDescending()으로 내림차순 정렬 가능
* 배열은 sortedArray(), sortedArrayDescending() 함수 사용
* sortedBy(), sortedWith()를 이용해 순서를 제공 가능
    * sortedBy()는 원소를 비교가능한 타입의 값으로 변환하는 함수를 인자로 제공
        * sortedByDescending() 함수로 내림차순 정렬7
    * sortedWith()는 원소 타입의 Comparator를 제공

#### sort()

* 가변 리스트, 배열에서 사용가능
* 원본 컬렉션을 변경하는 inplace 정렬 수행

```kotlin
val array = intArrayOf(4, 0, 8, 9, 2).apply { sort() }
println(array.contentToString()) // [0, 2, 4, 8, 9]

val list = arrayListOf("red", "blue", "green").apply { sort() }
println(list)                    // [blue, green, red]
```

#### reversed()

* 이터러블, 배열을 역순으로 뒤집어주는 함수
* 배열은 reversedArray() 사용
* reverse()로 가변 리스트, 배열의 원본 변경 가능

#### asReversed()

* 새로운 리스트를 반환하지만 원본에 대한 래퍼로 원본이 변경되면 그대로 반영됨

#### shuffled()

* 원본의 원소를 임의의 순서로 재배치한 새로운 리스트 반환
* shuffle()로 원본 변경 가능

## 7.2 파일과 I/O 스트림

* 코틀린 표준 라이브러리는 JDK의 I/O 관련 클래스를 쉽게 사용할 수 있게 해주는 확장 함수, 프로퍼티 제공

### 7.2.1 스트림 유틸리티

#### 전체 컨텐츠 읽기

```kotlin
fun InputStream.readBytes(): ByteArray
fun Reader.readText(): String
fun Reader.readLines(): Line<String>
```

* 위 3개의 확장 메서드를 이용해 스트림의 모든 컨텐츠 읽기 가능
* readLines()는 값을 반환하며 스트림을 닫아줌

#### 버퍼 스트림 이터레이션

```kotlin
FileInputStream("data.bin").buffered().use {
    var sum = 0
    for (byte in it) sum += byte
}

FileReader("data.bin").buffered().use {
    for (line in it.lineSequence()) println(line)
}
```

* Reader 인스턴스는 forEachLine(), useLines() 메서드 이용해 줄 단위 접근 가능
* copyTo() 함수를 사용해 다른 스트림으로 데이터 전달도 가능
* use() 함수를 이용하면 명시적으로 정리해야 하는 스트림, 다른 자원을 안전하게 처리할 수 있음
    * Closable을 구현해야 호출 가능

### 7.2.2 스트림 생성

* bufferedReaders(), bufferedWriter() 확장 함수를 이용해 BufferedReader, BufferedWriter 인스턴스 쉽게 생성 가능
    * reader(), writer() 확장 함수도 존재함
* printWriter() 함수는 PrintWriter 인스턴스 반환
* 각 reader, writer 함수에 문자 집합 지정 가능
* inputStream(), outputStream()을 통해서는 이진 데이터 처리 가능
* 다양한 형식의 {}InputStream(), {}OutputStream() 확장 함수 제공
* Reader, Writer 클래스는 buffered() 함수로 버퍼를 제공하기도 한다.

### 7.2.3 URL 유틸리티

* URL 주소로부터 데이터를 읽어오는 함수 제공
    * fun URL.readText(): String
    * fun URL.readBytes(): ByteArray

### 7.2.4 파일 콘텐츠 접근하기

* 코틀린 표준 라이브러리는 파일 콘텐츠를 읽을 수 있는 함수 제공
* readText(), readLines(), writeText(), appendText() 등 제공
* 이진 파일은 readBytes(), writeBytes(), appendBytes() 등 함수 제공
* forEachLines(), useLines()로 파일을 부분씩 읽을 수도 있음

### 7.2.5 파일 시스템 유틸리티

* deleteRecursively()
    * 파일, 디렉터리를 하위 파일까지 쉽게 삭제 가능
* copyTo()
    * 파일의 내용을 다른 파일로 복사하고 복사본을 가리키는 객체 반환
    * copyRecursively()로 디렉터리 단위 복사 가능
* walk()
  * 깊이 우선 디렉터리 구조 순회
