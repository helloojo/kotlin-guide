# 9장 제네릭스

## 9.1 타입 파라미터

### 9.1.1 제네릭 선언

* 제네릭 선언을 위해서는 타입 파라미터를 추가하면 된다.
* 코틀린에서는 다양한 방식의 타입 추론을 지원한다.

```kotlin
// 생성자에 타입 인자 지정해 인스턴스 생성
val map = HashMap<Int, String>()
val list = arrayListOf<String>()

// 타입을 명시해 타입인자(<>) 생략
val map2: Map<Int, String> = HashMap()

// arrayListOf()에 전달된 인자의 타입(모두 String)으로부터 타입 인자를 추론할 수 있음
val list2 = arrayListOf("abc", "def")
```

* 상위 클래스가 제네릭 타입일 경우 컴파일러가 생성자 위임 호출의 타입 추론을 지원해주지 못해, 타입 인자를 명시해야 한다.

```kotlin
open class DataHolder<T>(val data: T)

// 실제 타입을 상위 타입의 타입 인자로 넘김
class StringDataHolder(data: String) : DataHolder<String>(data)

// 타입 인자를 상위 타입의 타입 인자로 넘김
class TreeNode<T>(data: T) : DataHolder<T>(data) {}
```

* 프로퍼티, 함수에 타입 파라미터를 사용하는 것도 가능
    * 프로퍼티는 확장 프로퍼티만 타입 파라미터를 가질 수 있음
        * 클래스 멤버 프로퍼티는 타입 인자로 여러 값을 제공하는 것이 불가능하기 때문
* 객체 선언에는 타입 파라미터 추가 금지됨

### 9.1.2 바운드와 제약

* 타입 파라미터는 기본적으로 어떤 타입이든 들어갈 수 있음
* 별도의 선언으로 타입 파라미터의 바운드를 제약할 수 있음

```kotlin
fun <T : Number> TreeNode<T>.average(): Double {    // T를 Number의 하위 타입으로 제약
    var count = 0
    var sum = 0.0
    walkDepthFirst {  // 깊이 우선으로 노드를 방문하면서 함수 수행
        count++
        sum += it.toDouble()
    }
    return sum / count
}
```

* where 키워드를 통해 타입 파라미터에 더 복잡한 제약을 넣을 수있음

```kotlin
interface Named {
    val name: String
}

interface Identified {
    val id: Int
}

class Registry<T> where T : Named, T : Identified {     // T는 Named, Identified를 모두 구현해야한다.
    val items = ArrayList<T>()
}
```

### 9.1.3 타입 소거와 구체화

* 타입 파라미터가 실제 타입을 대신할 수 있는 것은 아님
    * 런타임 시 타입 파라미터에 대한 정보는 코드에서 소거된다.
* 코틀린은 제네렉 타입에 타입 인자가 반드시 포함되어야 하기 때문에, 제네릭 타입 인자는 관심이 없고, 타입만 검사하고 싶다면 `list is List<*>` 같은 방법을 사용하면 된다.
* 타입 파라미터 정보를 런타임까지 유지하려면 인라인한 함수를 통해 우회하는 방법이 있다.
    * reified 키워드를 활용할 수 있다.

```kotlin
inline fun <reified T> isInstanceOf(obj: Any?) = obj is T

fun main() {
    val str = "ABC"
    if (isInstanceOf<String>(str)) {
        println("$str is String")
    }
}
```

* reified 키워드를 사용한 구체화된 타입 파라미터도 생성자를 호출하거나, 동반 객체 멤버에 접근하는 등의 연산은 지원하지 않는다.

## 9.2 변성

* 타입 파라미터가 달라질 때 제네릭 타입의 하위 관계가 달라지는 것을 설명하는 것
    * `Array<String>`은 `Array<Any>` 의 하위타입이 아니지만, 불변 컬렉션은 타입 파라미터의 하위 관계가 유지된다.
* 변성을 사용하면 타입 안전성을 지키면서 API의 유연성을 향상시킬 수 있다.

### 9.2.1 변성: 생산자와 소비자 구분

* 무공변
    * 타입 파라미터의 하위 타입 관계가 성립해도 제네릭 타입 사이에는 하위 타입 관계가 생기지 않는다는 것
* 공변
    * 타입 파라미터의 상하위 타입 관계에 따라 제네릭 타입에서도 상하위 타입 관계가 생기는 것
* 공변, 무공변의 구분은 제네릭 타입이 자신의 타입 파라미터를 취급하는 방법에 달려있다.
    1. T 타입의 값을 반환하는 연산만 제공하고 T 타입의 값을 입력으로 받는 연산은 제공하지 않는 제네릭 타입인 생산자
        * 생산자 X는 A가 B의 하위타입이면 제네릭 타입 `X<A>`는 `X<B>`의 하위 타입이 된다.(공변적)
    2. T 타입의 값을 입력으로 받기만 하고 결코 T 타입의 값을 반환하지 않는 제네릭 타입인 소비자
        * 소비자 X는 A가 B의 하위타입이면 제네릭 타입 `X<B>`는 `X<A>`의 하위 타입이 된다.(반공변적)
    3. 위 두가지 경우에 해당하지 않는 나머지 타입들
        * 이 경우는 타입 안전성을 깨지 않고는 하위 타입 관계를 유지할 수 없음

### 9.2.2 선언 지점 변성

* 기본적으로 타입 파라미터는 무공변으로 취급된다.
* 타입 파라미터 선언 지점에서 out/in 키워드를 통해 공변적(생산자), 반공변적(소비자) 타입 파라미터를 표현할 수 있다.
    * out 키워드를 사용할 경우 타입 파라미터는 out 위치(ex. 반환 타입)에서만 사용되어야 한다.
    * in 키워드를 사용할 경우 타입 파라미터는 in 위치(ex. 파라미터)에서만 사용되어야 한다.

### 9.2.3 프로젝션을 사용한 사용 지점 변성

* 무공변 타입을 문맥에 따라 사용 지점에서 out/in 키워드를 붙여 공변적, 반공변적으로 선언할 수 있다.
    * 이를 프로젝션이라 한다.
* 메서드 파라미터 선언에서 프로젝션을 활용할 수 있다.
    * 프로젝션을 통해 타입의 유연함을 제공할 수 있다.
* 프로젝션을 사용할 경우 제네릭 타입 인자에 해당하는 선언 지점 변성은 의미가 사라진다.
    * 타입 인자의 변성과 프로젝션이 일치하지 않으면 컴파일 오류가 발생한다.

### 9.2.4 스타 프로젝션

* 타입 인자가 타입 파라미터의 바운드 안에서 어떤 타입이나 될 수 있다는 사실을 표현한다.
    * 제네릭 타입의 하위 타입 중 어떤 것이든 허용한다는 뜻

```kotlin
// List의 원소 타입은 `Any?`에 의해 제한되므로 아무 리스트나 가능함
val anyList: List<*> = listOf(1, 2, 3)

// 자기 자신과 비교가능한 아무 객체나 가능(T : Comparable<T> 바운드에 의해)
val anyComparable: Comparable<*> = "abcde"
```

* 스타 프로젝션을 사용하면 타입 인자가 중요하지 않거나, 알려져 있지 않은 제네릭 타입을 간결하게 표현할 수 있다.
* 타입 파라미터에 바운드가 둘 이상 있다면 * 으로 프로젝션을 대신할 수 없다.
    * 컴파일러가 정확한 타입 바운드를 유추하지 못하기 때문에

```kotlin
interface Named {
    val name: String
}

interface Identified {
    val id: Int
}

class Registry<T> where T : Named, T : Identified

// Registry의 타입 파라미터의 바운드는 Named와 Identified의 교집합이다
// 스타프로젝션은 타입 파라미터의 바운드를 무시하기 때문에 컴파일러가 바운드를 유추하지 못한다.
var registry: Registry<*>? = null

//println(registry?.id ?: "")    // error: unresolved reference: id 
//println(registry?.name ?: "")  // error: unresolved reference: name
```

## 9.3 타입 별명

* 기존 타입의 이름을 대신할 수 있는 새로운 이름을 사용하는 것
* typealias 키워드를 사용해 선언한다.

```kotlin
typealias IntPredicate = (Int) -> Boolean
typealias IntMap = HashMap<Int, Int>
```

* 내포된 클래스를 가리키는 타입 별명을 선언할 수도 있다.

```kotlin

sealed class Status {
    object Success : Status()
    class Error(val message: String) : Status()
}

typealias StSuccess = Status.Success
typealias StError = Status.Error
```

* 제네릭 타입 파라미터도 포함해 선언도 가능하다.

```kotlin

typealias ThisPredicate<T> = T.() -> Boolean
typealias MultiMap<K, V> = Map<K, Collection<V>>
```

* private, internal 등의 가시성을 선언할 수도 있다.

```kotlin
private typealias MyMap = Map<String, String> // 현재 파일 내부에서만 볼 수 있음
```

* 타입 별명은 최상위에서만 선언할 수 있다.
* 제네릭 타입 별명은 타입 제약, 바운드를 선언할 수는 없다.
* 타입 별명과 기존 타입을 자유롭게 바꿔 사용하는 것도 가능하다.
