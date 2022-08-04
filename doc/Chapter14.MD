# 14장 코틀린 테스팅

## 14.1 코테스트 명세

### 14.1.1 코테스트 시작하기

* io.kotest:kotest-runner-junit5 를 사용해 코테스트 의존성을 추가할 수 있다.
* 인텔리제이 kotest 플러그인으로 코테스트와 통합을 향상시킬 수 있다.

### 14.1.2 명세 스타일

* 명세 스타일에 따라 테스트 코드 구성 방법은 달라진다.

#### StringSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.StringSpec

class NumbersTest : StringSpec({
    "2 + 2 should be 4" { (2 + 2) shouldBe 4 }
    "2 * 2 should be 4" { (2 * 2) shouldBe 4 }
})
```

* 문자열 뒤에 람다를 추가해 테스트를 작성한다.
* shouldBe 연산자를 통해 값의 일치여부를 검사할 수 있다.

#### WordSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.WordSpec

class NumbersTest2 : WordSpec({
    "1 + 2" should {
        "be equal to 3" { (1 + 2) shouldBe 3 }
        "be equal to 2 + 1" { (1 + 2) shouldBe (2 + 1) }
    }
    "Addition" `when` {
        "1 + 2" should {
            "be equal to 3" { (1 + 2) shouldBe 3 }
            "be equal to 2 + 1" { (1 + 2) shouldBe (2 + 1) }
        }
    }
    "Addition2" When {
        "1 + 2" should {
            "be equal to 3" { (1 + 2) shouldBe 3 }
            "be equal to 2 + 1" { (1 + 2) shouldBe (2 + 1) }
        }
    }
})
```

* 테스트를 2, 3계층 구조로 만들 수 있다.
* should를 통해 하나의 그룹을 만들 수 있고, shouldBe로 검증을 할 수 있다.
* when을 사용하면 3단계 계층 구조도 만들 수 있다.

#### FunSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.FunSpec

class NumbersTest3 : FunSpec({
    test("0 should be equal to 0") { 0 shouldBe 0 }
    context("Arithmetic") {
        context("Addition") {
            test("2 + 2 should be 4") { (2 + 2) shouldBe 4 }
        }
        context("Multiplication") {
            test("2 * 2 should be 4") { (2 * 2) shouldBe 4 }
        }
    }
})
```

* 테스트를 계층 구조로 만들 때 사용할 수 있다.
* 테스트 코드는 test 함수 호출로 선언한다.
* context 함수를 이용하면 테스트 그룹을 정의할 수 있다.

#### ExpectSpec

* FunSpec과 동일하지만 test 대신 expect 함수를 사용하고, 최상위에는 테스트코드가 위치할 수 없다.
* 모든 테스트 코드는 context 블록에 속해야 한다.

#### DescribeSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.DescribeSpec

class NumbersTest4 : DescribeSpec({
    describe("Addition") {
        context("1 + 2") {
            it("should give 3") { (1 + 2) shouldBe 3 }
        }
    }
})
```

* describe, context로 그룹을 지을 수 있다.
* it로는 테스트를 정의할 수 있다.

#### ShouldSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.ShouldSpec

class NumbersTest5 : ShouldSpec({
    should("be equal to 0") { 0 shouldBe 0 }
    context("Addition") {
        context("1 + 2") {
            should("be equal to 3") { (1 + 2) shouldBe 3 }
            should("be equal to 2 + 1") { (1 + 2) shouldBe (2 + 1) }
        }
    }
})
```

* FunSpec과 레이아웃은 비슷하다.
* 테스트 블록은 should로 정의하고, context로 테스트 그룹을 지정한다.

#### FreeSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.FreeSpec

class NumbersTest6 : FreeSpec({
    "0 should be equal to 0" { 0 shouldBe 0 }
    "Addition" - {
        "1 + 2" - {
            "1 + 2 should be equal to 3" { (1 + 2) shouldBe 3 }
            "1 + 2 should be equal to 2 + 1" { (1 + 2) shouldBe (2 + 1) }
        }
    }
})
```

* StringSpec 처럼 테스트를 정의한다.
* minus 연산자를 통해 테스트 문맥을 소개한다.

#### FeatureSpec

```kotlin
import io.kotest.core.spec.style.FeatureSpec
import io.kotest.matchers.shouldBe

class NumbersTest7 : FeatureSpec({
    feature("Arithmetic") {
        val x = 1
        scenario("x is 1 at first") { x shouldBe 1 }
        feature("increasing by") {
            scenario("1 gives 2") { (x + 1) shouldBe 2 }
            scenario("2 gives 3") { (x + 2) shouldBe 3 }
        }
    }
})
```

* BDD(행동 주도 개발) 명세 스타일
* feature 로 계층 루트를 만들고 scenario 블록이 구체적인 테스트가 된다.

#### BehaviorSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.BehaviorSpec

class NumbersTest8 : BehaviorSpec({
    Given("Arithmetic") {
        When("x is 1") {
            val x = 1
            And("increased by 1") {
                Then("result is 2") { (x + 1) shouldBe 2 }
            }
        }
    }
})
```

* given, when, then 함수로 구분되는 테스트 정의를 할 수 있다.
* and를 통해 여러 테스트 블록을 묶어서 그룹을 추가할 수 있다.

#### AnnotationSpec

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.core.spec.style.AnnotationSpec

class NumbersTest9 : AnnotationSpec() {
    @Test
    fun `2 + 2 should be 4`() {
        (2 + 2) shouldBe 4
    }

    @Test
    fun `2 * 2 should be 4`() {
        (2 * 2) shouldBe 4
    }
}
```

* @Test 애너테이션을 함수에 붙여 테스트를 정의할 수 있다.
* @Ignore 를 통해 테스트를 비활성화 시킬수도 있다.

## 14.2 단언문

#### 14.2.1 매처

* 일반 함수 호출이나, 중위 연산자 형태로 사용할 수 있다.
* 모든 매처의 이름은 shouldBe로 시작한다.
* Matcher 인터페이스를 구현해 커스텀 매처를 정의할 수 있다.
    * Matcher 인터페이스는 MatcherResult를 반환하는 test() 함수를 가진다.

```kotlin
fun beOdd() = object : Matcher<Int> {
    override fun test(value: Int): MatcherResult {
        return MatcherResult(
                value % 2 != 0,
                "$value should be odd",
                "$value should not be odd"
        )
    }
}

fun beOddLength() = beOdd().compose<Collection<*>> { it.size }
```

* 커스텀 매처는 should, shouldNot 등의 확장 함수와 함께 사용될 수 있다.
* Matcher는 and, or, invert 연산도 지원한다.

#### 14.2.2 인스펙터

* 컬렉션 함수에 대한 확장 함수
* forAll()/forNone()
    * 단언문을 모든 원소가 만족하는지, 아닌지 검사한다.
* forExactly(N)
    * 단언문을 N개의 원소가 만족하는지 검사한다.
* forAtLeast(N)/forAtMost(N)
    * 단언문을 최소/최대 N개의 원소가 만족하는지 검사한다.
* forSome()
    * 단언문을 만족하는 원소가 존재하지만, 모든 원소가 만족하지는 않음을 검사한다.

#### 14.2.3 예외 처리

* shouldThrow 를 통해 코드가 발생시키는 예외를 검사할 수 있다.
* assertSoftly 를 사용하면 테스트 중 발생하는 모든 예외를 누적시킨 후 블록이 끝나면 호출한 쪽에 반환한다.
    * 하나의 단언문이 실패해서 모든 테스트가 실행되지 않는 상황을 막을 수 있다.

#### 14.2.4 비결정적 코드 테스트하기

* eventually 를 통해 여러번 시도를 통해 테스트를 통과할 수 있는 비 결정적 코드를 테스트할 수 있다.
* continually 는 단언문이 지정한 기간동안 계속 성립하는지 검사할 수 있다.

```kotlin
import io.kotest.assertions.timing.continually
import io.kotest.assertions.timing.eventually
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import java.io.File
import kotlin.time.Duration
import kotlin.time.ExperimentalTime
import kotlin.time.seconds

@ExperimentalTime
class StringSpecWithEventually : StringSpec({
    "10초 안에 파일의 내용이 단 한줄인 경우가 있어야 함" {
        eventually(10.seconds) {  // Duration.seconds(10)을 권장
            // 주어진 기간동안 파일이 한 줄만 들어있는는 순간이 올 때까지 검사 (최대 10초)
            File("data.txt").readLines().size shouldBe 1
        }
    }
    "10초동안 파일의 내용이 계속 한줄로 유지되야 함" {
        continually(10.seconds) {
            File("data.txt").readLines().size shouldBe 1
        }
    }
})
```

### 14.2.5 속성 기반 테스트

* 술어를 지정하면 임의의 테스트 데이터를 생성해주는 기능 지원
* kotest-property 모듈을 의존 관계에 추가해야함
* checkAll을 사용하면 주어진 인자에 대한 스트림을 생성하고 모든 스트림을 단언문을 통해 검사한다.
* forAll을 사용하면 단언문 대신 boolean 값을 반환하는 람다를 넘길 수 있다.
    * forNone도 사용가능하다.

```kotlin
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.and
import io.kotest.matchers.ints.beLessThanOrEqualTo
import io.kotest.matchers.should
import io.kotest.property.checkAll
import io.kotest.property.forAll
import io.kotest.property.forNone

class NumberTestWithAssertAll : StringSpec({
    "min" {
        checkAll { a: Int, b: Int ->
            (a min b).let {
                it should (beLessThanOrEqualTo(a) and beLessThanOrEqualTo(b))
            }
        }
    }

    "min with expression" {
        forAll { a: Int, b: Int ->
            (a min b).let {
                it <= a && it <= b
            }
        }
    }

    "min (forNone 사용)" {
        forNone { a: Int, b: Int ->
            (a min b).let {
                it > a || it > b
            }
        }
    }
})

infix fun Int.min(n: Int) = if (this < n) this else n
```

#### 속성 기반 테스트 디폴트 생성기

* 주로 사용하는 타입에 대한 디폴트 생성기를 제공한다.
* 모든 생성기는 Gen 추상 클래스를 상속한다.
* Arb
    * 엣지 케이스와 무한한 난수 샘플을 생성해준다.
    * arbitrary 확장 함수를 통해 쉽게 구현할 수 있다.
* Exhaustive
    * 주어진 공간에 속한 모든 데이터를 생성해준다.
    * exhaustive 확장 함수를 통해 구현할 수 있다.

## 14.3 픽스처와 설정

### 14.3.1 픽스처 제공하기

* 픽스처
    * 테스트 시작 시 환경, 자원을 초기화하고, 테스트 종료 시 정리해야하는 테스트 환경
* TestListener 인터페이스를 구현해 픽스처를 지정할 수 있다.
* beforeTest는 테스트마다 실행되고, beforeSpec은 명세가 인스턴스화 될 때 실행된다.
* prepareSpec, finalizeSpec은 개별 테스트 Spec에 픽스처를 추가할 경우 호출되지 않는다.
* ProjectConfig 타입에 리스너를 등록하면 프로젝트 단위의 리스너도 지정할 수 있다.
* autoClose 함수를 통해 AutoCloseable 인터페이스를 구현한 자원을 할당하면 테스트 종료 시 자원을 자동으로 해제해준다.

```kotlin
import io.kotest.core.config.AbstractProjectConfig
import io.kotest.core.listeners.*
import io.kotest.core.listeners.ProjectListener
import io.kotest.core.spec.Spec
import io.kotest.core.spec.style.FunSpec
import io.kotest.core.test.TestCase
import io.kotest.core.test.TestResult
import io.kotest.matchers.shouldBe
import kotlin.reflect.KClass

object SpecLevelListener : TestListener {
    override val name: String = "SpecLevelListener"

    override suspend fun prepareSpec(kclass: KClass<out Spec>) {
        println("PrepareSpec(in SpecLevelListener): ${kclass.qualifiedName}")
    }

    override suspend fun beforeSpec(spec: Spec) {
        println("BeforeSpec: ${spec.materializeRootTests().joinToString { it.testCase.displayName }}")
    }

    override suspend fun beforeTest(testCase: TestCase) {
        println("BeforeTest: ${testCase.displayName}")
    }

    override suspend fun afterTest(testCase: TestCase, result: TestResult) {
        println("AfterTest: ${testCase.displayName}")
    }

    override suspend fun afterSpec(spec: Spec) {
        println("AfterSpec: ${spec.materializeRootTests().joinToString { it.testCase.displayName }}")
    }

    override suspend fun finalizeSpec(kclass: KClass<out Spec>, results: Map<TestCase, TestResult>) {
        println("FinalizeSpec(in SpecLevelListener): ${kclass.qualifiedName}")
    }
}

class NumbersTestWithFixture1 : FunSpec() {
    init {
        context("Addition") {
            test("2 + 2") {
                2 + 2 shouldBe 4
            }
            test("4 * 4") {
                4 + 4 shouldBe 8
            }
        }
    }

    override fun listeners() = listOf(SpecLevelListener)
}

class NumbersTestWithFixture2 : FunSpec() {
    init {
        context("Multiplication") {
            test("2 * 2") {
                2 * 2 shouldBe 4
            }
            test("4 * 4") {
                4 * 4 shouldBe 16
            }
        }
    }

    override fun listeners() = listOf(SpecLevelListener)
}

object MyProjectListener : ProjectListener, TestListener {
    override val name: String = "MyProjectListener"

    override suspend fun beforeProject() {
        println("Before project")
    }

    override suspend fun afterProject() {
        println("After project")
    }

    override suspend fun prepareSpec(kclass: KClass<out Spec>) {
        println("PrepareSpec: ${kclass.qualifiedName}")
    }

    override suspend fun finalizeSpec(kclass: KClass<out Spec>, results: Map<TestCase, TestResult>) {
        println("FinalizeSpec: ${kclass.qualifiedName}")
    }
}

object ProjectConfig : AbstractProjectConfig() {
    override fun listeners() = listOf(MyProjectListener)
}
```

### 14.3.2 테스트 설정

#### 테스트 설정

* 명세 함수(context, when, given 등)의 config() 함수를 통해 여러 테스트 설정을 지정할 수 있다.
    * 테스트 실행 횟수(invocation)
    * 테스트 실행 스레드 개수(threads)
    * 테스트 비활성화 여부(enabled)
    * 타임아웃(timeout)
* defaultConfig 함수를 명세 클래스에 오버라이드해 테스트케이스의 설정을 일괄로 변경할 수 있다.

#### 테스트 격리 모드

* 테스트 중 테스트케이스 인스턴스를 공유하는 방법 지정 가능
* isolcationMode 프로퍼티를 오버라이드해서 사용한다.
* SingleInstance
    * 테스트 케이스의 인스턴스는 하나만 만들어진다.
    * 기본 동작
* InstancePerTest
    * 테스트 블록이 실행될 때마다 인스턴스가 만들어진다.
* InstancePerLeaf
    * 개별 테스트 블록을 실행하기 전 테스트가 인스턴스화 된다.
