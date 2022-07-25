# 12장 자바 상호 운용성

## 12.1 자바 코드를 코틀린에서 사용하기

### 12.1.1 자바 메서드와 필드

* 대부분의 자바 메서드를 코틀린 함수처럼 노출 가능
* 캡슐화되지 않은 필드를 코틀린 프로퍼티처럼 사용 가능

### 12.1.2 Unit과 Void

* 자바 void 함수는 코틀린에서 Unit을 반환하는 함수로 보임

### 12.1.3 합성 프로퍼티

* 자바에는 합성 프로퍼티는 없음
* 코틀린 컴파일러는 자바 접근자를 일반적인 프로퍼티처럼 쓸 수 있게 노출시켜준다.
* 접근자는 관습을 따라가야한다.
    * 게터는 파라미터가 없는 메서드, 메서드 이름이 get으로 시작해야 한다.
    * 세터는 파라미터가 하나만 있는 메서드, 메서드 이름이 set으로 시작해야 한다.
* 게터만 있으면 불변 프로퍼티 처럼 동작한다.

### 12.1.4 플랫폼 타입

* 자바는 널 가능 타입이 구분되지 않아 코틀린 컴파일러는 자바 코드에서 전달된 객체가 널인지 여부에 대해 가정할 수 없다.
* 코틀린 컴파일러는 자바 타입을 명확한 널 가능성이 지정되지 않은 타입인것 처럼 취급한다.
    * 이를 플랫폼 타입이라 한다.
* 플랫폼 타입은 널이 될 수 있는 타입이기도 하고 널이 될 수 없는 타입이기도 한다.
* 플랫폼 타입은 모든 문맥에서 사용할 수 있다.
    * 하지만 플랫폼 타입을 널이 될 수 없는 타입에서 사용할 경우 NPE가 발생할 수도 있다.
* 플랫폼 타입은 변수에 대입하거나, 함수에서 반환하면 타입이 전파될 수 있다.

```kotlin
import java.math.BigInteger

fun Int.toBigInt() = BigInteger.valueOf(toLong())       // BigInteger! 반환 타입

val num = 123.toBigInt()    // BigInteger! 타입
```

```kotlin
import java.math.BigInteger

fun Int.toBigInt(): BigInteger = BigInteger.valueOf(toLong())   // BigInteger 반환 타입

val num = 123.toBigInt()    // BigInteger 타입
```

* 타입을 명시적으로 지정해 플랫폼 타입을 강제로 지정할 수 있다.
* 코틀린은 자바 컬렉션 타입도 플랫폼 타입을 쓴다.
    * 자바는 불변/가변 컬렉션이 구분되지 않기 때문

### 12.1.5 널 가능성 애너테이션

* 자바에서는 널 안정성을 보장하는 방법은 애너테이션을 사용하는 것
* 코틀린 컴파일러도 일부 애너테이션을 지원한다.
    * @Nullable, @NotNull (org.jetbrains.annotations 패키지)
    * @Nullable, @NotNull 변종들 (안드로이드 SDK)
    * @Nonnull 등의 JSR-350 널 가능성 애너테이션 (javax.annotation 패키지)
* NotNull 같은 애너테이션이 붙으면 코틀린 컴파일러는 플랫폼 타입을 널이 될수 없는 타입으로 취급한다.

### 12.1.6 자바/코틀린 타입 매핑

#### 원시 타입

| 자바 타입          | 코틀린 타입 |
|----------------|--------|
| byte/Byte      | Byte   |
| short/Short    | Short  |
| int/Integer    | Int    |
| long/Long      | Long   |
| char/Character | Char   |
| float/Float    | Float  |
| double/Double  | Double |

#### 내장 클래스

* Object => Any 를 제외한 java.lang 패키지의 내장 클래스의 일부도 코틀린 클래스로 매핑된다.
* Cloneable
* Comparable
* Enum
* Annotation
* CharSequence
* String
* Number
* Throwable

#### 컬렉션 타입

* 코틀린 표준 컬렉션 타입은 java.util 의 컬렉션 타입으로 매핑된다.
* 자바에서 코틀린 방향으로 매핑은 플랫폼 타입을 만들어낸다.

#### 제네릭 타입

* extends => 공변 프로젝션
* super => 반공변 프로젝션
* 로우 타입 => 스타 프로젝션

#### 원시 타입 배열

* 자바의 원시 타입 배열은 코틀린의 특화된 코틀린 배열 클래스로 매핑된다.
    * ex) int[] => IntArray

#### 12.1.7 단일 추상 메서드 인터페이스

* 단일 추상 메서드 인터페이스(SAM)는 코틀린 함수 타입처럼 작동한다.
* 하지만 컴파일러가 문맥을 파악할 수 없을 경우가 있는데, 그럴 땐 가장 구체적인 시그니쳐를 선택한다.

```kotlin
import java.util.concurrent.ScheduledThreadPoolExecutor

fun main() {
    val executor = ScheduledThreadPoolExecutor(5)
    val future = executor.submit { 1 + 2 }  // 암시적으로 Runnable

    println(future.get())       // null
    executor.shutdown()
}
```

```kotlin
import java.util.concurrent.ScheduledThreadPoolExecutor

fun main() {
    val executor = ScheduledThreadPoolExecutor(5)
    val future = executor.submit(Callable { 1 + 2 })

    println(future.get())       // 3
    executor.shutdown()
}
```

* SAM 변환은 인터페이스만 적용되고, 메서드가 하나뿐인 클래스에는 적용할 수 없음
* 코틀린 인터페이스도 SAM 변환을 사용할 수 없음
    * fun interface 로 SAM 변환 사용 가능

#### 12.1.8 자바를 코틀린으로 변환하는 변환기 사용하기

* 인텔리J 플러그인에서 자바 소스 파일을 코틀린 코드로 변환해주는 도구 제공

## 12.2 코틀린 코드를 자바에서 사용하기

### 12.2.1 프로퍼티 접근

* 코틀린 프로퍼티는 자바에서 직접 접근할 수 없음
* 컴파일된 바이트코드에서는 프로퍼티는 접근자 메서드로 표현되기 때문에 자바에서 접근자로 프로퍼티 접근 가능
    * 게터는 `get{프로퍼티이름}()`
    * 세터는 `set{프로퍼티이름}(value)`

```kotlin
class Person(var name: String, val age: Int)
```

* 위 코틀린 클래스는 자바 관점에서 아래처럼 보인다.

```java
public class Person {
    @NotNull
    public String getName() {
    }

    public void setName(@NotNull String value) {
    }

    public int getAge() {
    }
}
```

* 프로퍼티 이름이 is 로 시작할 경우 코틀린 컴파일러는 세터를 is를 set 으로 바꿔 제공한다.
* 코틀린 프로퍼티에 뒷받침하는 필드가 필요하면, @JvmField 애너테이션을 프로퍼티 앞에 넣으면 자바에서 접근할 수 있다.
    * 해당 애너테이션이 붙어있으면 접근자는 생성되지 않고 프로퍼티와 동일한 가시성으로 만들어진다.
* @JvmField는 추상 프로퍼티, 열린(open) 프로퍼티에 적용할수는 없다.
* 객체(object)의 프로퍼티에 대해 @JvmField 를 적용하면 정적 필드를 만들어낸다.

### 12.2.2 파일 퍼사드와 최상위 선언

* 코틀린에서는 최상위 선언을 자주 사용함
* 자바에서는 모든 메서드는 어떤 클래스에 속해야하기 때문에 코틀린 컴파일러는 최상위 함수, 프로퍼티를 파일 퍼사드 클래스에 넣는다.
    * 퍼사드 클래스 이름은 소스코드 이름 뒤에 Kt 를 덧붙인 이름
* 퍼사드 클래스는 자바 관점에서 정적 메서드, 정적 프로퍼티를 제공한다.
    * 자바 코드에서 퍼사드 클래스를 인스턴스화할 필요는 없다.
* @JvmName 파일 수준 애너테이션을 통해 퍼사드 클래스의 이름을 지정할 수 있다.
* @JvmMultifileClass 파일 수준 애너테이션을 통해 여러 파일의 최상위 선언을 한 클래스로 모을 수 있다.
    * 합치려는 파일에 대한 애너테이션을 붙이고, @JvmName으로 동일한 클래스 이름을 지정하면 됨
* 코틀린 클래스에서는 퍼사드 클래스에 접근할 수 없다.

### 12.2.3 객체와 정적 멤버

* 코틀린 객체 선언은 INSTANCE 필드가 있는 일반적인 클래스
* @JvmStatic 애너테이션을 이용해 객체 함수, 프로퍼티 접근자를 정적 메서드로 제공할 수 있다.

### 12.2.4 노출된 선언 이름 변경하기

* @JvmName 으로 파일 퍼사드 클래스 이름만 바꿀 수 있는 것은 아님
* 함수, 프로퍼티 접근자 이름도 변경할 수 있다.
    * 자바에서 금지된 선언의 시그니처 충돌을 막기 위함

```kotlin
class Person(val firstName: String, val familyName: String)

val Person.fullName
    get() = "$firstName $familyName"

fun getFullName(person: Person): String {
    return "${person.familyName}, ${person.firstName}"
}   // 컴파일 에러 발생
```

* JVM 상에서 프로퍼티, 함수가 똑같은 시그니처를 갖기에 컴파일 에러 발생
* @JvmName으로 이름을 변경할 수 있다.

### 12.2.5 오버로딩한 메서드 생성하기

* 코틀린 함수는 디폴트 값 존재함
* 자바에서는 디폴트 값 개념이 없어 사용할 수 없음
* @JvmOverloads 애너테이션을 사용하면 기존 코틀린 함수 외의 오버로딩된 함수 추가 제공
* @JvmOverloads 로 생성된 함수는 코틀린에서는 호출할 수 없음

### 12.2.6 예외 선언하기

* 코틀린은 검사/비검사 예외 구분 없음
* 자바는 검사 예외 목록을 함수에 추가해야 함
* 코틀린 선언에서 발생하는 검사 예외를 자바에서 처리할 경우 문제가 발생한다.
    * 자바는 선언되지 않은 검사 예외를 try, catch로 처리하는 것을 금지하기 때문에
* @Throws 애너테이션을 이용해 검사 예외를 선언할 수 있다.
* 자바에서는 검사 예외를 선언한 것처럼 보인다.

### 12.2.7 인라인 함수

* 코틀린 인라인 함수는 자바에서 일반 메서드로 노출된다.
* 자바에서도 인라인 함수를 호출할 수는 있지만, 자바 코드로 함수 본문이 인라인되진 않는다.
* 구체화한 타입 파라미터가 있는 제네릭 인라인 함수의 경우, 자바에서 호출하는 것이 불가능하다.

### 12.2.8 타입 별명

* 자바에서 타입 별명을 참조하는 선언은 모두 원래 타입으로 보임