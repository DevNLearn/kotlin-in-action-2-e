# 13. DSL 만들기
## 1. API에서 DSL로: 표현력이 좋은 커스텀 코드 구조 만들기

### 도메인 특화 언어

범용 프로그래밍 언어와 특정 영역(도메인)에 초점을 맞추고 최적화된 언어

### 내부 DSL은 프로그램의 나머지 부분과 매끄럽게 통합된다.

- 외부 DSL과 다르게 내부 DSL은 범용 언어로 작성된 일부며, 범용 언어와 동일한 문법을 사용한다.-

외부 DSL

```sql
// SQL
SELECT Country.name, Count(Customer.id)
FROM Country INNER JOIN Customer
  ON Country.id = CUstomer.country_id
GROUP BY Country.name
ORDER BY COUNT(Customer.id) DESC
LIMIT 1
```

내부 DSL

```kotlin
// Exposed 프레임워크
(Country innerJoin CUstomer)
    .slice(Country.name, Count(Customer.id))
    .selectAll()
    .groupBy(Country.name)
    .orderBy(Count(Customer.id), order = SortOrder.DESC)
    .limit(1)
```

### DSL의 구조

- DSL은 중위 호출이나 연산자 오버로딩에 의존한다.
- 코틀린 DSL에서는 보통 람다를 내포시키거나 메서드 호출을 연쇄시키는 방식으로 구조를 만든다.
- DSL에서는 여러 함수 호출을 조합해서 연산을 만든다.
- 타입 검사기는 여러 함수 호출이 바르게 조합됐는지 검사한다.
- 함수 이름은 보통 동사, 함수 인자는 명사 역할을 한다.
- DSL 구조는 매 함수 호출 시마다 반복하지 않고 재사용할 수 있다.

```kotlin
// DSL 구조
dependencies { // 람다 내포 구조
    testImplementation(kotlin("test"))
	  implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
}
```

명령-질의 API 통해 표현할 수 있지만 코드 중복이 발생한다.

```kotlin
// 명령-질의 API
project.dependencies.add("testImplementation", kotlin("test"))
project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-core:0.40.1")
project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-dao:0.40.1")
```

### 내부 DSL로 HTML 만들기

```kotlin
import kotlinx.html.stream.createHTML
import kotlinx.html.*
fun createSimpleTable() = createdHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
```

직접 HTML 텍스트로 작성하지 않고 코틀린 코드로 HTML을 만드는 장점은 아래와 같다.

- 코틀린 버전은 타입 안정성을 보장한다.
- 일반 코틀린 코드라서 원하는 대로 사용할 수 있다.
- 동적으로 생성할 수 있다.

```kotlin
import kotlinx.html.stream.createHTML
import kotlinx.html.*
fun createSimpleTable() = createdHTML().table {
    val numbers = mapOf(1 to "one", 2 to "two")
    for ((num, string) in numbers) {
        tr {
            td { +"$num"}
            td { +string}
        }
    }
}
```

## 2. 구조화된 API 구축: DSL에서 수신 객체 지정 람다 사용

### 수신 객체 지정 람다와 확장 함수 타입

```kotlin
// 람다를 인자로 받는 buildeString() 정의
fun buildString(
    builderAction: (StringBuilder) -> Unit // 함수 타입인 파라미터를 정의
): String {
    val sb = StringBuilder()
    builderAction(sb)
    return sb.toString()
}

fun main() {
    val s = buildString {
        it.append("Hello, ")
        it.append("World!")
    }
    
    println(s)
    // Hello, World!
}
```

위 코드는 람다 본문에서 매번 `it`을 사용해 StringBuilder 인스턴스를 참조해야 한다.

```kotlin
// 수신 객체 지정 람다를 파라미터로 받는 buildString() 정의
fun buildString(
    builderAction: StringBuilder.() -> Unit // 수신 객체가 지정된 함수 타입 파라미터 선언
): String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}

fun main() {
    val s = buildString {
        this.append("Hello, ")
        append("World!")
    }
    println(s)
    // Hello, World!
}
```

수신 객체 지정 람다를 인자로 넘기며 람다 안에서 `it` 을 사용하지 않아도 된다.

파라미터를 선언할 때 함수 타입 대신 확장 함수 타입을 사용한다.

`(StringBuilder) → Unit`을 `StringBuilder.() → Unit` 으로 바꾸며 수신 객체 타입을 사용했다.

수신 객체 지정 람다를 사용 장점

- 람다 안에서 수신 객체를 `this`로 간결하게 참조 할 수 있다
- 확장 함수처럼 수신 객체의 멤버에 직접 접근할 수 있다.
- `this`  생략으로 DSL 처럼 읽힌다.

수신 객체 지정 람다를 변수에 저장하여 사용 할 수도 있다.

```kotlin
// 수신 객체 지정 람다를 변수에 저장하기
val appendExcl: StringBuilder.() -> Unit = 
    { this.append("!") }
    
fun main() {
    val stringBuilder = StringBuilder("Hi")
    stringBuilder.appendExcl()
    println(stringBuilder)
    // Hi!
    println(buildString(appendExcl))
    // !   
}
```

- appendExcl을 확장 함수처럼 호출 할 수 있다.
- appendExcel을 인자로 넘길 수 있다.

`apply` 와 `with`  같은 표줌 함수를 결합하면 간결하게 사용 할 수 있다.

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String =
    StringBuilder().apply(builderAction).toString()
```

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}
inline fun <T,R> with(receiver: T, block: T.() -> R): R =
    receiver.block()
```

### 수신 객체 지정 람다를 HTML 빌더 안에서 사용

코틀린 HTML 빌더 규현이 어떻게 동작하는지 살펴본다.

```kotlin
// 코틀린 HTML 빌더를 사용해 간단한 HTML 표 만들기
fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell"}
        }
		 }
```

`table`, `tr`, `td` 각 함수는 고차 함수로, 수신 객체 지정 람다를 인자로 받는다.

각 수신 객체 지정 람다가 이름 결정 규칙을 바꾼다. 각 람다는 자신의 본문에서 호출될 수 있는 함수들을 새로 추가한다.

```kotlin
fun createSimpleTable(): String = createHTML()
    .table {
        tr {
            td {
                +"cell"
            }
        }
    }
```

table 함수에 넘겨진 람다 밖에서는 tr 함수를 찾을 수 없고, td 함수는 tr 안에서만 접근 가능하고 단항 덧셈 연산자는 td 태그 안에서만 접근 가능하다.

```kotlin
// HTML 빌더를 위한 태그 클래스 정의
open class Tag

class TABLE : Tag {
    fun tr(init : TR.() -> Unit)
}

class TR : Tag {
    fun td(init : TD.() -> Unit)
}
class TD : Tag
```

- TABLE, TR, TD는 모두 유틸리티 클래스다
- 모두 Tag를 확장 했다.
- 각 클래스에는 자신의 내부에 들어갈 수 있는 태그를 생성하는 메서드가 들어 있다.
- 각 함수의 init 파라미터 타입은 모두 확장 함수 타입이다.

HTML 빌더 호출의 수신 객체를 명시할 수도 있다.

```kotlin
fun createSimpleTable() = createHTML().table {
    this@table.tr {
        (this@tr).td {
            +"cell"
        }
    }
}
```

내포 깊이가 깊은 구조에서는 어떤 식의 수신 객체가 무엇인지 분명하지 않아 @DslMaker 어노테이션을 사용해 내포된 람다에서 외부 람다의 수신 객체에 접근하지 못하게 제한할 수 있다.

```kotlin
@DslMaker
annotation class HtmlTagMaker

@HtmlTagMaker
open class Tag
```

### 코틀린 빌더: 추상화와 재사용을 가능하게 해준다.

코틀린 내부 DSL을 사용하면 반복되는 내부 DSL 코드 조각을 새 함수로 묶어 재사용할 수 있다.

```kotlin
fun builderBookList() = createHTML().body {
    ul {
        li { a("#1") { +"The Three-Body Problem" }}
        li { a("#2") { +"The Dark Forest" }}
        li { a("#3") { +"Death's End" }}
    }
    h2 { id = "1"; + "The Three-Body Problem" }
    p { +"The first book tackles..." }

    h2 { id = "2"; + "The Dark Forest" }
    p { +"The second book starts with..." }
    
    h2 { id = "3"; + "Death's End" }
    p { +"The third book contains..." }
}
```

```kotlin
// 도우미 함수를 사용
fun buildBookLIst() = createHTML().body {
    listWithToc {
        item("The Three-Body Problem", "The first book tackles...")
        item("The Dark Forest", "The second book starts with...")
        item("Death's End", "The third book contains...")
    }
}
```

```kotlin
@HtmlTagMaker
class LISTWITHTOC {
    val entries = mutableListOf<Pair<String, String>>()
    fun item(headline: String, body: String) {
        entries += headline to body
    }
}
```

```kotlin
fun BODY.listWithToc(block: LISTWITHTOC.() -> Unit) {
    val listWithTOc = LISTWITHTOC()
    listWithTOc.block()
    ul {
        for ((index, entry) in listWithToc.entries.withIndex()) {
            li { a("#$index") { +entry.first }}
        }
    }
    for ((index, entry) in listWithTOc.entries.withIndex()) {
        h2 { id = "$index"; +entry.first }
        p { +entry.second }
    }
}
```

## 3. invoke 관례를 사용해 더 유연하게 블록 내포시키기

invoke 관례를 사용하면 어떤 커스텀 타입의 객체를 함수처럼 호출할 수 있다.

### invoke 관례를 사용해 더 유연하게 블록 내포시키기

operator 변경자가 붙은 invoke 메서드 정의가 들어있는 클래스의 객체를 함수처럼 호출할 수 있다.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

fun main() {
    val bavarianGreeter = Greeter("Servus")
    bavarianGreeter("Dmitry")
    // Servus, Dmitry!
}
```

invoke 메서드 정의를 통해 Greeter 인스턴스를 함수처럼 호출할 수 있다.

### DSL의 invoke 관례: 그레이들 의존관계 선언

내포된 블록 구조를 허용하면서 함수 호출 구도도 함께 제공하는 API를 만들 수 있다.

```kotlin
dependencies.implementation("org.jetbrains.exposed:exposed-core:0.40.1")
dependencies {
	  implementation("org.jetbrains.exposed:exposed-core:0.40.1")
}
```

첫 번째 : dependencies 변수에 대해 implementation 메서드를 호출

두 번째 : dependencies 안에 람다를 받는 invoke 메서드를 정의해서 사용

```kotlin
class DependencyHandler {
    fun implementation(coordinate: String) {
        println("Added dependency on $coordinate")
    }
    operator fun invoke(
        body: DependencyHandler.() -> Unit) {
        body()
        }
}

fun main() {
    val dependencies = DependencyHandler()
    dependencies.implementation("org.jetbrains.kotlinx:kotlinx-corutines-core:1.8.0")
    // Added dependenct on org.jetbrains.kotlinx:kotlinx-corutines-core:1.8.0
    dependencies {
        implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.5.0")
    }
    // Added dependenct on org.jetbrains.kotlinx:kotlinx-datetime:0.5.0
}
```

## 4. 실전 코틀린 DSL

### 중위 호출 연쇄시키기: 테스트 프레임워크의 should 함수

DSL을 깔금하게 만들려면 코드에 쓰이는 기호의 수를 줄여야 한다. 람다 호출이나 중위 함수 호출이 간결함을 제공한다.

```kotlin
import io.kotest.matchers.should
import io.kotest.matchers.string.startWith
import org.junit.jupiter.api.Test

class PrefixTest {
    @Test
    fun testKPrefix() {
        val s = "kotlin".uppercase()
        s should startWith("K")
    }
}
```

이런 문법을 DSL에서 사용하려면 should 함수 선언 앞에 infix 변경자를 붙여야 한다.

```kotlin
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)
```

should 함수는 Matcher의 인스턴스를 요구한다. Matcher는 값에 대한 단언문을 표현하는 제네릭 인터페이스다.

```kotlin
interface Matcher<T> {
    fun test(value: T)
}
fun startWith(prefix: String): Matcher<String> {
    return object : Matcher<String> {
        override fun test(value: String) {
            if(!value.startsWith(prefix)) {
                throw AssertionError("$value dose not start with $prefix")
            }
        }
    }
}
```

### 원시 타입에 대해 확장 함수 정의하기: 날짜 처리

```kotlin
import kotlin.time.DurationUnit
val Int.days: Duration
    get() = this.toDuration(DurationUnit.DAYS)
val Int.hours: Duration
    get() = this.toDuration(DurationUnit.HOURS)
```

### 멤버 확장 함수: SQL을 위한 내부 DSL

DSL 설계에서 확장 함수가 중요한 역할을 한다. 클래스안에서 확장함수와 확장 프로퍼티를 선언할 수 있다. 그렇게 정의한 확장 함수나 확장 프로퍼티는 그들이 선언된 클래스의 멤버인 동시에 그들이 확장하는 다른 타입의 멤버이며 이런 함수나 프로퍼티를 `멤버 확장` 이라고 부른다.

```kotlin
object Country : Table() {
    val id = integer("id").autoIncrement()
    val name = varchar("name", 50)
    override val primaryKey = PrimaryKey(id)
}

fun main() {
    val db = Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
    transaction(db) {
        SchemaUtils.create(Country)
    }
}
```

## 5. 요약

- 내부 DSL은 여러 메서드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 잇는 설계 패턴이다.
- 수신 객체 지정 람다는 람다 본문 안에서 메서드를 결정하는 방식을 재정의함으로써 여러 요소를 내포시킬 수 있는 구조를 만들 수 있다.
- 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수타입이다.
- 람다를 파라미터로 받아 사용하는 함수는 람다를 호출하면서 람다에게 수신 객체를 제공한다.
- 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면 코드를 추상화하고 재활용할 수 있다.
- 원시 타입에 대한 확장을 정의하면서 기간 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다.
- invoke 관례를 사용하면 임의의 객체를 함수처럼 다룰 수 있다.
- kotlinx.html 라이브러리는 HTML 페이지 생성 내부 DSL을 제공한다. 또한 애플리케이션에 맞게 그 내부 DSL을 확장할 수 있다.
- 코테스트 라이브러리는 단위 테스트에서 읽기 쉬운 단언문을 지원하는 내부 DSL을 제공한다.
- 익스포즈드 라이브러리는 데이터베이스를 다루기 위한 내부 DSL을 제공한다.