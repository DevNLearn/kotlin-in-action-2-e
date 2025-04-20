# 5. 람다를 사용한 프로그래밍

람다식을 통해 더 간결하고 선언적인 방식으로 코드를 구성하는 방법을 설명하며, 멤버 참조, 고차 함수, 컬렉션 API와의 통합 방식, 그리고 with/apply 같은 유용한 스코프 함수의 활용을 다룬다.

---

## 5.1 람다식과 멤버 참조

람다(lambda)는 다른 함수에 전달할 수 있는 작은 코드 블록이다. 코틀린은 람다를 통해 매우 간결한 문법을 제공하며, 특히 컬렉션을 다룰 때 유용하다.

### 자바와 비교한 코드 간결성

```kotlin
button.setOnClickListener(object: OnClickListener {
    override fun onClick(v: View) {
        println("I was clicked")
    }
})

// 코틀린에서는
button.setOnClickListener {
    println("I was clicked")
}

```

### 람다식 문법

```kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1, 2)) // 3

```

- 중괄호 `{}`로 감싸고, `->`로 매개변수와 본문을 구분한다.

### `run` 사용 이유

run은 코드 블록을 감싸 즉시 실행할 수 있는 함수로, 표현식처럼 사용되며 불필요한 변수 선언 없이 짧은 로직을 작성할 때 유용하다.

```kotlin
run { println(42) } // 42

```

### 람다 최적화 예시: 컬렉션 최대값 찾기

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.maxByOrNull { it.age })
println(people.maxByOrNull(Person::age)) // 멤버 참조

```

- 인자가 하나일 때는 `it` 사용 가능
- `Person::age` 같은 형태는 **멤버 참조**이며 가독성을 높여줌

### 멤버 참조란?

멤버 참조는 단일 메서드 또는 프로퍼티를 람다처럼 참조할 수 있는 문법으로, 람다보다 더 간결하게 표현 가능하다.

```kotlin
val getAge = Person::age
val seb = Person("Seb", 26)
println(getAge(seb))       // 26
val sebAge = seb::age
println(sebAge())          // 26

```

### 생성자 참조

```kotlin
val createPerson = ::Person
val p = createPerson("Alice", 29)

```

생성자 참조를 통해 클래스 인스턴스를 생성하는 람다를 만들 수 있다.

> 📌 람다 표현 정리 과정 예시
>
> 1. people.maxByOrNull({ p: Person -> p.age })
> 2. 괄호 밖으로 람다 이동: people.maxByOrNull() { p: Person -> p.age }
> 3. 타입 생략: people.maxByOrNull { p -> p.age }
> 4. it 활용: people.maxByOrNull { it.age }
> 5. 멤버 참조로 축약: people.maxByOrNull(Person::age)

---

## 5.2 람다를 자바 메서드의 파라미터로 전달하기

> **람다와 리스너 등록 해제하기**
리스너처럼 상태를 유지하거나 해제해야 하는 경우에는 람다식보다는 명시적인 참조가 더 적합하다. 람다는 매 호출마다 새로운 인스턴스를 만들 수 있기 때문에, 동일한 리스너로 등록과 해제를 하려면 변수로 분리해두는 것이 좋다.
>

코틀린은 자바의 함수형 인터페이스(SAM: Single Abstract Method)에 람다를 직접 전달할 수 있다.

```kotlin
// 자바 코드
void delayComputation(int delay, Runnable computation);

// 코틀린 호출
postponeComputation(1000) { println(42) }

```

- 코틀린은 컴파일 시 람다를 Runnable 객체로 변환
- 객체를 명시적으로 선언하지 않아도 됨

### 변경 가능한 변수 캡처하기

**캡처된 변수란?** 람다가 바깥 변수에 접근하면 해당 변수는 **캡처(capture)** 되어 클로저(closure)로 유지된다.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    responses.forEach {
        if (it.startsWith("4")) clientErrors++
        else if (it.startsWith("5")) serverErrors++
    }
    println("$clientErrors client errors, $serverErrors server errors")
}

```

자바는 final 변수만 캡처 가능하지만, 코틀린은 var도 허용하며 내부적으로 참조 래퍼(Ref)를 만들어 처리함

> 📌 캡처된 변수는 함수가 끝나도 람다 안에서 살아남는다. 이처럼 람다가 외부 상태를 유지할 수 있다는 특성이 클로저의 핵심이다.
>

---

## 5.3 SAM 변환을 코틀린에서 사용: `fun interface`

코틀린 1.4부터 SAM 인터페이스를 직접 정의하려면 fun interface 키워드를 사용함

```kotlin
fun interface IntPredicate {
    fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }
println(isEven.accept(3)) // false

```

- 이를 통해 람다를 곧바로 함수형 인터페이스 구현으로 사용할 수 있다

> 📌 자바 코드에서도 쉽게 사용 가능
코틀린에서 정의한 fun interface는 자바에서도 SAM 타입처럼 사용할 수 있어, 코틀린과 자바의 상호 운용성이 강화된다.
>

---

## 5.4 수신 객체 지정 람다: `with`, `apply`, `also`

수신 객체를 명시하지 않고 this로 멤버에 접근할 수 있어 객체 초기화, 빌더 구성 등에 유용하다.

### `with`

- 첫 번째 인자를 수신 객체로 사용하며, 결과를 반환함

```kotlin
fun alphabet(): String {
    val sb = StringBuilder()
    return with(sb) {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\nNow I know the alphabet!")
        toString()
    }
}

```

### `apply`

- 수신 객체 자체를 반환하며, 객체 구성에 적합함

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()

```

### `also`

- `it`으로 수신 객체를 받아, 로깅이나 디버깅 시 유용

```kotlin
val list = mutableListOf("apple", "banana")
    .also { println("Original list: $it") }

```

> 📌 차이점 요약
>
> - `with`: 외부에서 객체 넘겨받고 마지막 식 결과 반환
> - `apply`: 객체를 구성하고 자신 반환
> - `also`: 디버깅 용도로 현재 객체를 it으로 넘겨 처리

---

## 5.5 람다 안에서 변수 캡처하기

람다는 자신이 정의된 외부 함수의 지역 변수에 접근할 수 있으며, 이를 "변수를 캡처했다"고 한다.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    responses.forEach {
        if (it.startsWith("4")) clientErrors++
        else if (it.startsWith("5")) serverErrors++
    }
    println("$clientErrors client errors, $serverErrors server errors")
}

```

캡처된 변수는 함수가 끝나도 살아남을 수 있으며, 이는 클로저(closure)를 형성한다.
코틀린은 이런 캡처를 위해 내부적으로 변수에 래퍼 객체를 씌운다 (예: Ref<T> 클래스)

---

## 요약

- **람다**는 코드 블록을 값처럼 다룰 수 있게 해주는 문법이다.
- **멤버 참조**를 통해 람다를 간결하게 표현할 수 있으며, 생성자나 프로퍼티도 참조 가능하다.
- **수신 객체 지정 람다**(`run`, `with`, `apply`, `also`)는 객체 구성, 초기화, 빌더 패턴을 더욱 간결하게 구현하게 해준다.
- **람다의 변수 캡처** 기능은 상태 공유나 누적 계산에서 매우 유용하다.
- **SAM 변환**은 자바와의 상호 운용성을 높이며, `fun interface`를 통해 코틀린에서도 직접 정의 가능하다.

> 코틀린의 람다는 단순한 축약 문법을 넘어서, 선언적이고 함수형 스타일의 강력한 프로그래밍 도구이다.
>