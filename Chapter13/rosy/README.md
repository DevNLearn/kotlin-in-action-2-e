# 13장 DSL 만들기

DSL은 도메인 특화 언어로 특정 작업이나 영역(도메인)에 최적화된 **전용 언어**를 말한다.

Kotlin에서는 DSL을 사용해 읽기 쉽고, 선언적인 방식으로 코드를 작성할 수 있습니다.

DSL을 통해 일반 함수 호출보다 더 구조적이고 자연어 같은 코드 작성이 가능하다.

이를 가능하게 해주는 핵심 Kotlin 기능:

- **수신 객체 지정 람다 (Lambda with Receiver)**
- **invoke 관례 (invoke Convention)**

이 두 가지를 활용하면 **타입 안전하고 IDE 친화적인 DSL**을 만들 수 있어요.

---

## 13.1 API에서 DSL로

- 프로그래밍에서 중요한 것은 코드를 읽는 사람이 쉽게 이해할 수 있어야 한다.
    - **좋은 API**는 명확하고 간결해야 한다.
    - 불필요한 문법적 잡음은 줄이고, **직관적으로 의도를 전달**해야 한다.
- Kotlin DSL의 핵심 목표
    - **명확하게 이해할 수 있는 코드**
    - **불필요한 문법이나 중복 제거**
- Kotlin의 깔끔한 문법 기능

    | 일반 코드 | Kotlin DSL 스타일 | 사용된 기능 |
    | --- | --- | --- |
    | `StringUtil.capitalize(s)` | `s.capitalize()` | 확장 함수 |
    | `1.to("one")` | `1 to "one"` | 중위 호출 |
    | `set.add(2)` | `set += 2` | 연산자 오버로딩 |
    | `map.get("key")` | `map["key"]` | `get` 관례 |
    | `file.use({ f -> f.read() })` | `file.use { it.read() }` | 람다 축약, 관례 |
    | `sb.append("yes"); sb.append("no")` | `with(sb) { append("yes"); append("no") }` | 수신 객체 람다 |
    | `m.add(1); m.add(2)` | `buildList { add(1); add(2) }` | 빌더 함수 |
    - 이러한 기능들을 조합하면, **언어처럼 자연스러운 API**를 만들 수 있다.

### 도메인 특화 언어(DSL)란?

- 특정 분야(도메인)의 작업을 간결하게 표현할 수 있도록 설계된 언어이다.

| DSL | 목적 |
| --- | --- |
| SQL | 데이터베이스 질의 |
| 정규표현식 | 문자열 패턴 탐색 |
- 기존의 범용 언어는 "어떻게"를 설명하는 코드이다. (명령형 스타일)
- DSL은 "무엇을" 하고 싶은지를 설명한다. (선언형 스타일)
- DSL은 **간결하고 효율적이지만**, 단점도 있다.
    - 문법이 호스트 언어와 다르다 (e.g., Java 안에서 SQL은 문자열로 표현해야 함)
    - 디버깅/자동완성/타입 체크가 어렵다

### Kotlin에서는 DSL도 Kotlin 코드처럼 작성할 수 있다.

- 내부 DSL 방식
    - Kotlin 내부 DSL은 **기존 Kotlin 문법으로 DSL을 만들고 사용할 수 있다.**

### SQL 질의문 만들기

- 외부 DSL 예: SQL

    ```sql
    SELECT Country.name, COUNT(Customer.id)
    FROM Country
    JOIN Customer ON Country.id = Customer.country_id
    GROUP BY Country.name
    ORDER BY COUNT(Customer.id) DESC
    LIMIT 1;
    ```

    - SQL 자체 문법이 필요
    - 문자열로 사용해야 함
    - 타입 체크 불가
- 내부 DSL 예: Kotlin + Exposed 프레임워크

    ```kotlin
    (Country innerJoin Customer)
        .slice(Country.name, Count(Customer.id))
        .selectAll()
        .groupBy(Country.name)
        .orderBy(Count(Customer.id), SortOrder.DESC)
        .limit(1)
    ```

    - 완전한 Kotlin 코드
    - IDE에서 에러, 자동완성, 타입 체크 전부 가능
    - 질의 결과가 Kotlin 객체로 매핑됨

### DSL 구조

- 일반 API는 함수 호출만 나열되어 있다.

    ```kotlin
    project.dependencies.add("testImplementation", kotlin("test"))
    project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-core:0.40.1")
    project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-dao:0.40.1")
    ```

    - **구조가 없음**
    - 재사용 어렵고 중복 많음
- DSL은 **구조**를 표현한다.
    - ex) Gradle Kotlin DSL

    ```kotlin
    dependencies {
        testImplementation(kotlin("test"))
        implementation("org.jetbrains.exposed:exposed-core:0.40.1")
        implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
    }
    ```

    - `dependencies` 블록 안에서 `implementation`, `testImplementation`을 사용할 수 있음
    - 구조적, 선언적, 깔끔함

### HTML을 내부 DSL로 만들기

- HTML도 Kotlin DSL로 작성할 수 있다.

    ```kotlin
    fun createSimpleTable() = createHTML().table {
        tr {
            td { +"cell" }
        }
    }
    ```

    - 생성된 HTML

        ```html
        <table>
          <tr>
            <td>cell</td>
          </tr>
        </table>
        ```

- 동적 생성도 가능하다.

    ```kotlin
    fun createAnotherTable() = createHTML().table {
        val numbers = mapOf(1 to "one", 2 to "two")
        for ((num, string) in numbers) {
            tr {
                td { +"$num" }
                td { +string }
            }
        }
    }
    ```

    - 생성된 HTML

        ```html
        <table>
          <tr>
            <td>1</td>
            <td>one</td>
          </tr>
          <tr>
            <td>2</td>
            <td>two</td>
          </tr>
        </table>
        ```

- kotlin으로 HTML을 만드는 이유
    - 타입 안전성을 보장한다.
        - ex) td를 tr안에서만 사용할 수 있다.
    - 동적으로 표의 셀을 생성할 수 있다.

---

## 13.2 구조화된 API 구축: DSL에서 수신 객체 지정 람다 사용

### DSL을 구조화한다는 건?

- 일반 API: 메서드만 나열됨. 구조가 없음.
- DSL: **코드 블록 안에서 어떤 함수가 사용 가능한지를 제한**함으로써, 마치 언어처럼 “문법적인 구조”를 형성.

→ 이 구조를 만들기 위한 핵심 기능이 바로 **수신 객체 지정 람다(Lambda with Receiver)**

### 일반 람다를 받는 buildString 함수

```kotlin
fun buildString(builderAction: (StringBuilder) -> Unit): String {
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

- 이 방식은 람다 본문에 매번 `it.` 을 붙여야 해서 읽기 불편하다.

### 수신 객체 지정 람다를 사용하는 개선된 buildString

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}
fun main() {
	val s = buildString {
    append("Hello, ")
    append("World!")
	}
	println(s)
	// Hello, World!
}
```

- `it.` 없이 곧바로 `append()`를 호출할 수 있어 DSL처럼 자연스럽고 간결해진다.

### 수신 객체 지정 람다란?

- `StringBuilder.() -> Unit` 이런 식으로 **함수 타입에 수신 객체 타입**을 붙인 형태
- 람다 블록 안에서 수신 객체의 함수/프로퍼티를 마치 내 것처럼 직접 호출 가능
- 확장 함수 타입 구조

    ```kotlin
    String.(Int, Int) -> Unit
    ```

    - 수신 객체 타입: `String`
    - 파라미터: `Int, Int`
    - 반환 타입: `Unit`
- 일반 람다와 수신 객체 람다 호출 방식 비교

    | 타입 | 호출 방법 |
    | --- | --- |
    | 일반 람다 `(StringBuilder) -> Unit` | `builderAction(sb)` |
    | 수신 객체 지정 람다 `StringBuilder.() -> Unit` | `sb.builderAction()` |
    - 즉, 람다 자체가 **StringBuilder의 확장 함수처럼 동작**한다.

### 표준 함수 apply와 with

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String =
    StringBuilder().apply(builderAction).toString()
```

- apply

    ```kotlin
    inline fun <T> T.apply(block: T.() -> Unit): T {
        block()
        return this
    }
    ```

    - 수신 객체로 람다를 호출하고, 다시 그 객체를 반환한다.
- with

    ```kotlin
    inline fun <T, R> with(receiver: T, block: T.() -> R): R {
        return receiver.block()
    }
    ```

    - 수신 객체로 람다를 호출하고, 람다의 결과를 반환한다.

| 항목 | `apply` | `with` |
| --- | --- | --- |
| 람다 수신 객체 | 현재 객체 (`this`) | 명시적으로 지정된 객체 (`receiver`) |
| 반환값 | 객체 자신 (`this`) | 람다의 결과값 |
| 사용 목적 | **객체 구성 후 그대로 반환** | **결과를 얻는 작업 후 결과만 반환** |
| 일반 형태 | `obj.apply { ... }` | `with(obj) { ... }` |
| 체이닝 | 매우 적합 | 덜 적합 (반환값이 달라서) |

### HTML DSL로 테이블 만들기

```kotlin
fun createSimpleTable() = createHTML().table {
    tr {
        td { +"cell" }
    }
}
```

- `table`, `tr`, `td`, `+”cell”` 전부 일반 함수
- 각 블록은 수신 객체 지정 람다
- `+` 연산자는 `UnaryPlus` 오버로딩으로 정의됨 → 셀 안에 문자열 추가

### 이름 결정 규칙

```kotlin
table {
    tr {
        td { +"cell" }
    }
}
```

- `table` 블록의 수신 객체는 `TABLE`
- 그 안에서만 `tr()`을 사용할 수 있음
- `tr` 안에서는 `td()`만 사용 가능
- 밖에서는 사용 불가

→ 이름 해석 범위가 각 블록(태그)에 따라 제한됨 → HTML 구조와 동일한 문법이 된다.

### 수신 객체를 명시적으로 쓰면?

```kotlin
fun createSimpleTable() = createHTML().table {
    this@table.tr {
        this@tr.td {
            +"cell"
        }
    }
}
```

- `this@table` : `TABLE`
- `this@tr` : `TR`
- `this@td` : `TD`

이런 명시가 가능한 이유는 수신 객체가 중첩되어 있기 때문이다.

### 수신 객체가 중첩될 경우의 문제

```kotlin
createHTML().body {
    a {
        img {
            href = "..." // img 수신 객체에서 a의 href를 쓰면 문제 발생
        }
    }
}
```

- 중첩된 람다 안에서 바깥 람다의 수신 객체를 참조하는 일이 발생할 수 있다.
- 혼동 위험 ‼️

### DSL 마커: @DslMarker

- Kotlin은 이런 혼동을 막기 위해 `@DslMarker`를 제공한다.

    ```kotlin
    @DslMarker
    annotation class HtmlTagMarker
    
    @HtmlTagMarker
    open class Tag
    ```

    - 같은 DSL 마커가 붙은 수신 객체끼리는 **블록 안에서 하나만 암시적으로 사용 가능**
    - 외부 수신 객체는 접근 불가 → 실수 방지
    - 이제 img 람다에서 a 수신 객체를 사용할 수 없다.
        - 두 타입 모두 `@HtmlTagMarker` 가 붙어있어 사용하려고 하면 아래와 같은 오류가 발생함.
          >  var href: String can't be called in this context by implicit receiver.


| 어노테이션 | 역할 | 예시 |
| --- | --- | --- |
| `@DslMarker` | **메타 어노테이션**→ DSL 마커 전용 어노테이션을 만들 때 사용 | `@DslMarker`를 사용해 `@HtmlTagMarker`를 정의함 |
| `@HtmlTagMarker` | DSL에서 **실제로 사용하는 어노테이션**→ 수신 객체 간의 경계를 구분해줌 | `@HtmlTagMarker`가 붙은 클래스끼리는 암시적 수신 객체 충돌 방지 |

### 추상화와 재사용을 가능하게 해주는 코틀린 빌더

- HTML DSL도 **일반 코드처럼 함수로 추상화하고 재사용**할 수 있다.
- 책 요약 리스트를 만들기

    ```kotlin
    fun BODY.listWithToc(block: ListWithToc.() -> Unit) {
        val toc = ListWithToc().apply(block)
        ul {
            for ((i, entry) in toc.entries.withIndex()) {
                li { a("#$i") { +entry.first } }
            }
        }
        for ((i, entry) in toc.entries.withIndex()) {
            h2 { id = "$i"; +entry.first }
            p { +entry.second }
        }
    }
    ```

    - 도우미 클래스

        ```kotlin
        @HtmlTagMarker
        class ListWithToc {
            val entries = mutableListOf<Pair<String, String>>()
            fun item(title: String, body: String) {
                entries += title to body
            }
        }
        ```

    - 사용 예시

        ```kotlin
        fun buildBookList() = createHTML().body {
            listWithToc {
                item("The Three-Body Problem", "The first book tackles...")
                item("The Dark Forest", "The second book starts with...")
                item("Death's End", "The third book contains...")
            }
        }
        ```


→ DSL 안에서도 **코드 분리, 재사용, 추상화가 자유롭게 가능**

---

## 13.3 invoke 관례를 사용해 더 유연하게 블록 내포시키기

### `invoke` 관례란?

- 클래스에 `operator fun invoke(...)`를 정의하면, 객체를 **함수처럼** 사용할 수 있다.
- `greeter("World")`처럼 객체 뒤에 괄호를 붙이면 `greeter.invoke("World")`로 컴파일된다.

### 클래스에 `invoke` 메서드 정의하기

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

fun main() {
    val greeter = Greeter("Servus")
    greeter("Dmitry")  // → Servus, Dmitry!
}
```

- `operator` 키워드는 관례 함수를 정의할 때 사용한다.
- `invoke`는 함수 호출 연산자 `()`를 대체하는 관례이다.

### DSL에서 `invoke`를 왜 사용할까?

- DSL에서는 아래처럼 두 가지 방식으로 객체를 사용할 수 있게 만들고 싶을 때가 많다.

    ```kotlin
    // (1) 일반 메서드 호출
    dependencies.implementation("org.jetbrains.exposed:exposed-core:0.40.1") 
    
    // (2) 블록 구조 DSL 호출
    dependencies { 
        implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    }
    ```

    - 이 두 가지를 동시에 지원하려면 `invoke` 관례가 필요하다.


### DSL에서의 예제: DependencyHandler

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
```

- 이제 다음 코드가 가능해진다.

    ```kotlin
    val deps = DependencyHandler()
    
    deps.implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    
    deps {
        implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    }
    ```

    - 이렇게 하면 사용자는 그냥 자연스럽게 DSL처럼 `deps { ... }`로 사용할 수 있다.
- DSL을 블록 구조로 표현할 수 있다.
- 객체 하나로 다양한 호출 방식 지원 가능
- 가독성이 좋아지고, 선언적인 느낌이 강해진다.

---

## 13.4 실전 코틀린 DSL

### kotest란?

- https://github.com/kotest/kotest
- Kotlin에 최적화된 테스트 프레임워크로, **자연어처럼 읽히는 DSL 기반 테스트 코드**를 작성할 수 있게 해준다.

### Kotest DSL 단언문

```kotlin
val s = "kotlin".uppercase()
s should startWith("K")
```

- 마치 자연어처럼 읽히는 이 코드는 Kotest의 DSL 덕분에 가능한 문법이다.

### 내부 동작 예시

- 중위 함수 정의

    ```kotlin
    infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)
    ```

    - 이 함수는 **`T` 타입의 객체**에 대해 `should`라는 **중위 호출(infix call)** 을 사용할 수 있게 한다.
    - `should`는 `Matcher<T>` 타입의 인자를 받아 `matcher.test(this)`를 호출한다.
    - 여기서 `this`는 `should` 앞에 있는 값 (`s`)이며, 즉 `matcher`의 `test` 메서드로 검증을 위임하는 역할이다.

  → 예를 들어 `s should startWith("K")`는 내부적으로 `startWith("K").test(s)`가 된다!

- Matcher 인터페이스

    ```kotlin
    interface Matcher<T> {
        fun test(value: T)
    }
    ```

    - 이 인터페이스는 **"검증 로직"을 담은 객체**를 정의하기 위한 기반이다.
    - `T`는 제네릭 타입으로, 어떤 타입이든 사용할 수 있게 한다.
    - `test()` 메서드는 값을 받아서 그 값이 기대 조건을 만족하는지 확인하고, 만족하지 않으면 예외를 던지는 방식으로 구현된다.
- `startWith()` Matcher 구현

    ```kotlin
    fun startWith(prefix: String): Matcher<String> =
        object : Matcher<String> {
            override fun test(value: String) {
                if (!value.startsWith(prefix)) {
                    throw AssertionError("$value does not start with $prefix")
                }
            }
        }
    ```

    - `startWith("K")`처럼 문자열이 특정 접두사로 시작하는지 확인하는 **Matcher**를 정의한 함수다.
    - 익명 객체로 `Matcher<String>`을 구현하며, `test()`에서 `startsWith` 검사를 수행한다.
    - 만약 문자열이 주어진 prefix로 시작하지 않으면, `AssertionError`를 발생시켜 테스트 실패를 나타낸다.

### 날짜 DSL

```kotlin
val now = Clock.System.now()
val yesterday = now - 1.days
val afterFiveHours = now + 5.hours
```

- 여기서 `1.days`, `5.hours`는 원래 Kotlin에 없던 문법이지만, **확장 프로퍼티**를 활용해서 DSL처럼 사용할 수 있게 만든 것이다.

```kotlin
val Int.days: Duration
    get() = this.toDuration(DurationUnit.DAYS)

val Int.hours: Duration
    get() = this.toDuration(DurationUnit.HOURS)
```

- `Int` 타입에 `days`/`hours` 확장 프로퍼티를 추가
- `1.days` → `Duration` 객체가 반환됨

- 코드가 **읽기 쉬움**
- 선언적이며 **사람이 직관적으로 이해 가능**
- 비즈니스 로직을 **자연어 수준으로 표현 가능**

### 멤버 확장 함수: SQL을 위한 내부 DSL

- 멤버 확장은 **클래스 안에 정의된 확장 함수**를 말한다.
    - 이 함수는 **어떤 타입을 확장하면서**, 동시에 **클래스의 멤버처럼 동작**한다.
- 익스포즈드 프레임워크 (Exposed)
    - SQL을 Kotlin DSL로 작성할 수 있게 해주는 라이브러리이다.
    - 테이블 선언

        ```kotlin
        object Country : Table() {
            val id = integer("id").autoIncrement()
            val name = varchar("name", 50)
            override val primaryKey = PrimaryKey(id)
        }
        ```

        - `id`, `name`은 각각 정수, 문자열 컬럼
        - `id`는 `autoIncrement()`를 통해 자동 증가 컬럼으로 지정
        - `autoIncrement()`는 `Column<Int>`에 대한 **멤버 확장 함수**이다.
- 멤버 확장을 사용하는 이유

    ```kotlin
    fun Column<Int>.autoIncrement() // 그냥 확장 함수로 정의
    ```

    - 이렇게 하면 아무 곳에서나 `autoIncrement()`를 호출할 수 있어 **제어가 어렵다.**

    ```kotlin
    class Table {
        fun Column<Int>.autoIncrement(): Column<Int> {
            // Table 내부에서만 호출 가능!
        }
    }
    ```

    - `Table` 안에서만 쓸 수 있음 → **맥락이 명확해짐**!
- 실제 SQL로 변환!
    - 위의 `Country` 객체는 다음과 같은 SQL로 변환된다.

        ```sql
        CREATE TABLE IF NOT EXISTS Country (
            id INT AUTO_INCREMENT NOT NULL,
            name VARCHAR(50) NOT NULL,
            CONSTRAINT pk_Country PRIMARY KEY (id)
        )
        ```

    - 즉, Kotlin DSL 코드 → SQL 문으로 자연스럽게 변환된다.
- 멤버 확장 함수의 제약이 중요한 이유
    - `autoIncrement()`는 **숫자 타입(`Column<Int>`)**에서만 호출되어야 함
    - 문자열 컬럼에서 호출하면 **컴파일 에러** 발생

  → 타입 안정성을 DSL로 표현하는 매우 좋은 방법!

- 익스포즈드에서 두 테이블 조인하기

    ```kotlin
    val result = (Country innerJoin Customer)
        .select { Country.name eq "USA" }
    
    result.forEach {
        println(it[Customer.name])
    }
    ```

    - 이 코드는 다음과 같은 SQL과 같다.

        ```sql
        SELECT * FROM Country
        INNER JOIN Customer
        WHERE Country.name = 'USA'
        ```

    - 여기도 멤버 확장이 쓰인다.
        - `eq()`는 `Column<T>`에 대한 확장 함수지만, 단독으로는 호출할 수 없고 **SqlExpressionBuilder** 안에서만 사용 가능하다.

        ```sql
        object SqlExpressionBuilder {
            infix fun <T> Column<T>.eq(value: T): Op<Boolean> {
                // ...
            }
        }
        ```

        - `select { ... }` 안에서는 `SqlExpressionBuilder`가 **암시적 수신 객체**로 사용됨

      → `Country.name eq "USA"`가 가능한 이유!

- 멤버 확장의 장점

    | 특징 | 설명 |
    | --- | --- |
    | 맥락 제어 | 필요한 컨텍스트(`Table`, `SqlBuilder`) 안에서만 호출 가능 |
    | 타입 안전성 | 잘못된 타입에서 쓰이면 컴파일 에러 발생 |
    | API 설계 유연성 | 의미 있는 곳에서만 기능 노출 가능 |
- 멤버 확장도 여전히 멤버다.
    - **외부에서 확장 불가능**

      → 기존 클래스(`Table`)를 수정하지 않으면 새로운 멤버 확장 추가 불가

    - 해결 아이디어

        ```kotlin
        context(Table)
        fun Column<Int>.autoIncrement(): Column<Int> {
            // Table과 Column 양쪽의 속성 접근 가능
        }
        ```

    - 콘텍스트 수신 객체(`context receiver`)를 활용하면 해결 가능성 있다.
    - 하지만 현재는 실험적 기능 (→ Kotlin KEE 문서 참고)

---

## 요약

- 내부 DSL은 여러 메서드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 있는 설계 패턴이다.
- 수신 객체 지정 람다는 람다 본문 안에서 메서드를 결정하는 방식을 재정의함으로써 여러 요소를 내포시킬 수 있는 구조를 만들 수 있다.
- 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수 타입이다. 람다를 파라미터로 받아 사용하는 함수는 람다를 호출하면서 람 다에게 수신 객체를 제공한다.
- 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면 코드를 추 상화하고 재활용할 수 있다.
- 원시 타입에 대한 확장을 정의하면 기간 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다.
- invoke 관례를 사용하면 임의의 객체를 함수처럼 다룰 수 있다.
- kothinx.html 라이브러리는 HIML 페이지 생성 내부 DSL을 제공한다. 또한 애플리케이션에 맞게 그 내부 DSL을 확장할 수 있다.
- 코테스트 라이브러리는 단위 테스트에서 읽기 쉬운 단언문을 지원하는 내부 DSL을 제공한다.
- 익스포즈드 라이브러리는 데이터베이스를 다루기 위한 내부 DSL을 제공한다.