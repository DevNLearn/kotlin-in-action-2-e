## 9장

- 코틀린은 직접 작성한 함수를 언어 기능이 호출함으로써 구현되는 경우가 있다
    - `for ... in` 루프의 `Iterable` 구현한 객체
    - `try-with-resource` 의 `AutoCloseable` 구현한 객체
- 이는 특정 함수 이름과 연관되는데, 어떤 클래스 안의 `plus` 이름의 메서드를 정의하면 `+` 연산자를 사용할 수 있다.
- 언어 기능과 미리 정해진 이름의 함수를 연결해 주는 기법을 코틀린에서는 `관례` 라고 부른다.
---

### 이항 산술 연산 오버로딩
- 코틀린에서 관례를 사용하는 단순한 예로 산술 연산자가 있다.
- 코틀린에서는 이항 연산자 (`+`, `-`, `*`, `/`, `%` 등)를 오버로딩할 수 있다. 
- operator 키워드를 사용해서 오버로딩 할 수 있다
```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}

Point(1, 1) + Point (2, 3) = Point (3, 4)
```
- 오버로딩 가능한 이항 산술 연산자는 다음과 같다.

| 식 | 함수 이름 |
|---|-------|
| * | times |
| / | div   |
| % | mod   |
| + | plus  |
| - | minus |


- 연산자 우선순위는 사칙연산과 동일하다.
- 연산자를 정의할 때 두 피연산자가 같은 타입일 필요는 없다.
```kotlin
operator fun Point.times(scale: Double): Point {
return Point((x * scale).toInt(), (y * scale).toInt())
}

Point(1, 2) * 2.0 = Point(2, 4)
```
- 교환법칙을 지원하지 않는다. `(Point * 2.0) != (2.0 * Point)` 이다. 둘다 지원하게 하려면 각각 구현해야한다.
- 코틀린은 표준 숫자 타입에 대해 비트 연산자를 정의하지 않는다. 비트 연산자를 정의할 수도 없다.
- 비트 연산자 함수 목록은 다음과 같다

| 자바에서의 표현식 | 함수 이름 | 기능                    |
|-----------|-------|-----------------------|
| \<\<      | shl   | 왼쪽 시프트                |
| \>\>      | shr   | 오른쪽 시프트(부호 비트 유지)     |
| \>\>\>    | ushr  | 오른쪽 시프트(0으로 부호 피트 설정) |
| &         | and   | 비트 곱                  |
| \|        | or    | 비트 합                  |
| \^        | xor   | 비트 배타 합               |
| ~         | inv   | 비트 반전                 |

---

### 복합 대입 연산자 오버로딩
- `+` 연산자뿐 아니라 그와 관련있는 `+=`, `-=`, `*=`, `/=`, `%=` 등 복합 대입 연산자도 자동으로 함께 지원한다
- 해당 연산자들은 `plusAssign`, `minusAssign` 등의 메서드 이름으로 재정의 할수있으며, 여기에서오 `operator` 키워드를 사용해야 한다.
- `a += b`는 우선 `a.plusAssign(b)`를 호출한다.
- 만약 `plusAssign` 메서드가 정의되어 있지 않으면, `a = a + b`로 대체한다.
```kotlin
class Point(var value: Int) {
    operator fun plusAssign(other: Int) {
        value += other
    }
}

val point = Point(5)
point += 3
println(point.value)  // 출력: 8
```

---
### 단항 연산자 오버로딩
- 피연산자가 하나인 연산자인 단항 연산자도 동일하다. `operator` 키워드를 사용하며 지정된 메서드를 오버로딩하면 된다.
- 다만, 피연산자가 하나이기때문에 따로 파라미터를 받지 않는다.
- 오버로딩할 수 있는 단항 산술 연산자

  | 식        | 함수 이름      |
  |----------|------------|
  | +a       | unaryPlus  |
  | -a       | unaryMinus |
  | !a       | not        |
  | ++a, a++ | inc        |
  | --a, a-- | dec        |

---

### 동등성 연산자
- 코틀린에서 `==` 연산자 호출을 `equals` 메서드 호출로 컴파일 한다. 
- data class 에서는 자동으로 생성해준다
- 내부에서 `null` 검사까지 있기 때문에 `null` 가능성이 있는 값에도 적용할 수 있다.
- `equals` 함수는 항상 `override`가 붙어있다. 다른 연산자 오버로딩 관례와 달리 `equals` 는 `Any`에 정의된 메서드라 `override`가 필요하다.
- `Any` 의 `equals` 에는 `operator`가 붙어있지만 그 메서드를 오버라이드하는 메서드 앞에는 자동으로 상위 클래스의  `operator` 지정이 적용된다.
- `Any`에서 상속받은 `equals`가 확장 함수보다 우선순위가 높아서, `equals` 를 확장함수로 정의할 수 없다.
```kotlin
class Point(val x: int, val y: Int) {
    override fun equals(obj: Any?): Boolean {
        if(obj === this) return true
        if(obj !is Point) return false
        return obj.x == x && obj.y == y
    }
}

fun main() {
    // true
    println(Point(1, 2) == Point(1, 2))
    // true
    println(Point(1, 2) != Point(5, 5))
    // false
    println(null == Point(1, 2))
}
```

---


### 순서 연산자
- 코틀린에서 두 객체를 비교(`<`, `<=`, `>`, `>=`) 하기위해서 `compareTo` 를 사용한다.
```kotlin
data class Person(val name: String, val age: Int) : Comparable<Person> {
    override operator fun compareTo(other: Person): Int {
        return this.age - other.age
    }
}

val alice = Person("Alice", 30)
val bob = Person("Bob", 25)

// true
println(alice > bob)
// false
println(alice < bob)
```
- `Comparable`의 `compareTo` 에도 `operator` 변경자가 붙어있기 때문에, 하위 클래스에서 오버라이드 할때 함수 앞에 `operator` 키워드를 붙이지 않아도 된다.
- `compareTo는` 반드시 정수값을 반환해야 하며, 다음과 같은 규칙이 있다 
  - 0 반환 → 두 객체가 동일
  - 양수 반환 → 호출 객체가 크다
  - 음수 반환 → 호출 객체가 작다

---

### 인덱스로 원소 접근 get, set
- 코틀린에서 맵에 접근할때, 각괄호(`[]`) 를 사용한다
```kotlin
val value = map[key]
```

- 동일한 연산자를 사용하여 변경 가능한 맵에서 값을 변경할 수 있다
```kotlin
mutableMap[key] = newValue
```
---

- `get` 메서드를 만들고, `operator` 변경자를 붙인다면, 각괄호를 사용한 접근이 재정의된다!
```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("")
    }
}

fun main() {
    val p = Point(10, 20)
    // 10
    println(p[0])
    // 20
    println(p[1])
    // IndexOutOfBoundsException
    println(p[2])
}
```

- 파라미터를 두개 이상 받을수 있고, `Int` 가 아닌 타입도 사용할 수 있다!
```kotlin
operator fun Point.get(row: Int, col: Int): Int {
    // Todo
}

operator fun Point.get(idx: String): Int {
    // Todo
}

fun main(){
    val p = Point(10, 20)
  
    println(p["hello"])
  
    println(p[1][2])
}
```

---

#### 컬렉션에 들어있는지 검사 : in 관례
- 동일한 메커니즘으로, `contains` 메서드명으로 `in` 연산자를 사용하도록 구현할 수 있다.
- `operator` 키워드를 추가하고, `contains` 메서드로 구현해보자
```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)
operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x..<lowerRight.x &&
            p.y in upperLeft.y..<lowerRight.y
}

fun main() {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    // true
    println(Point(20, 30) in rect)
    // false
    println(Point(5, 5) in rect)
}
```

- in 오른쪽의 객체는 `contains` 의 수신객체가 되고, in 왼쪽 객체는 `contains` 메서드에 인자로 전달된다
- `a in c` -> `c.contains(a)`

---

#### 범위 만들기: rangeTo, rangeUntil
- 범위를 만들기 위해 `..` 구문을 사용할수 있고, 해당 연산자는 `rangeTo()` 메서드로 구현할 수 있다
- 하지만 어떤 클래스가 `Comparable` 인터페이스를 구현하면 `rangeTo`를 정의할 필요가 없다.
```kotlin
fun main() {
    val now = LocalDate.now()
    val vacation = now..now.plusDays(10)
  
    // true
    println(now.plusWeeks(1) in vacation)
    // false
    println(now.plusWeeks(2) in vacation)
}
```
- `rangeTo` 연산자는 다른 산술 연산자보다 우선순위가 낮다. 혼동을 피하기 위해 괄호로 인자를 감싸주면 좋다
```kotlin
fun main() {
    val n = 9
    // 0..10
    println(0..(n + 1))
}
```
- 범위 연산자는 우선순위가 낮아 범위의 메서드를 호출하려면 범위를 괄호로 둘러싸야 한다.
```kotlin
fun main() {
    val n = 9
    (0..n).forEach { println(it) }
}
```

---

#### 자신의 타입에 대한 루프 수행: iterator
- 코틀린의 for 루프는 범위 검사와 똑같이 `in` 연산자를 사용한다.
- 하지만 `for (x in list)` 와 같은 문장은 `list.iterator()` 를 호출하여 이터레이터를 얻은 다음, 이터레이터에 대해 `hasNext()`와 `next()` 를 반복하는 식으로 변환된다.
- 그리고 이것또한 `iterator` 메서드를 확장 함수로 정의할 수 있다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object : Iterator<LocalDate> {
          var current = start
          override fun hasNext(): Boolean {
            TODO("Not yet implemented")
          override fun next(): LocalDate {
            TODO("Not yet implemented")
          }
        }
      }
```

---

#### components 함수를 통한 구조 분해 선언
- data 클래스의 주 생성자에 들어있는 프로퍼티에 대해 자동으로 초기화시켜준다.
- 그러니깐 클래스 내부의 프로퍼티를 바로 사용할수 있는데, 예전에 했었다. 
```kotlin
fun main(){
    val p = Point(10, 20)
    val (x, y) = p
  
    // 10
    println(x)
    // 20
    println(y)
}
```
- 일반 변수 선언과 비슷해보이지만, 괄호로 여러 변수를 묶은게 차이점으로 볼 수 있다.
- 내부에서 구조 분해 선언은 관례를 사용하는데, 각 변수를 초기화하기위해 `componentN`이라는 함수를 호출한다.
```kotlin
class Point(val x: Int, val y: Int){
    operator fun component1() = x
    operator fun component2() = y
}
```
- `Pair` 나 `Triple` 클래스와 비슷하지만, 내부 원소의 의미를 각각 알지 못하는 아쉬움이 있다.

---
- 변수 선언이 들어갈 수 있는곳은 어디든 구조 분해 선언을 사용할 수 있다.
- 루프 안에서도 구조 분해 선언을 사용할 수 있다
```kotlin
fun entries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
```
위의 코드는 실제로 다음과 동일하게 컴파일 된다
```kotlin
for (entry in map.entries){
    val key = entry.component1()
    val value = entry.component2()
}
```

---

#### 구조 분해 값 무시 : _
- 데이터 클래스에서 일부 변수만 필요하다면?
- 다음과 같은 `Person` 클래스가 있다고 가정하자
```kotlin
data class Person(
        val firstName: String,
        val lastName: String,
        val age: Int,
        val city: String,
)
```
- 구조 분해선언을 하게 된다면 다음 네가지 변수로 받을 수 있다.
```kotlin
fun testMethod(p: Person){
    val (firstName, lastName, age, city) = p
  println("$firstName, $lastName, $age")
}
```
- 하지만 마지막 변수인 `city` 가 필요없어 삭제하고 싶다면?
```kotlin
fun testMethod(p: Person){
    val (firstName, lastName, age) = p
    println("$firstName, $lastName, $age")
}
```
- 앞에서 볼수있듯, `component1`, `component2`... 이름과는 상관없이 가져오기 때문에 뒤에 변수를 사용하지 않는다면, 구조분해시 삭제를 하면 된다
- 하지만 중간 변수를 사용하지 않아 삭제하고싶다면?
```kotlin
fun testMethod(p: Person){
    // 사용하지 않는 변수는 `_` 으로 구조 분해. 실제로 `_` 변수를 호출하면 에러 발생
    val (firstName, _, age) = p
    println("$firstName, $age")
}
```
