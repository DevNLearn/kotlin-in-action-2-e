## 11장

### 제네릭이란?
- 제네릭은 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법.
- 코드의 재사용성을 높이고 타입 안전성을 보장하기 위해 사용
```kotlin
class Box<T>(val value: T)
val boxInt = Box(42)
val boxString = Box("hello")
```

---
- 자바와 동일하게 제네릭 타입의 인스턴스가 만들어질때, 타입 파라미터를 구체적인 타입 인자로 치환한다.
- 타입 인자도 추론할 수 있다
```kotlin
val numbers = listOf(1, 2)
val authors = listOf("babi", "levi")
```
- 대신 빈 리스트를 명시할ㄴ다면, 타입 인자를 추론할 근거가 없기 때문에 반드시 타입 인자를 명시해야 한다
```kotlin
val readers: MutableList<String> = mutableListOf()
val readers = mutableListOf<String>()
```

---
### 제네릭 타입 - 함수와 프로퍼티
- 제네릭 함수는 자신이 타입 파라미터를 받는다.
- 제네릭 함수를 호출할 때는 구체적인 타입으로 타입 인자를 넘겨야 한다
```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
```
- 최초 `<T>` 에서 타입 파라미터 선언
- 이후 타입 파라미터가 수신 객체와 반환 타입에서 사용
- 하지만 대부분 컴파일러에서 타입 인자를 추론할 수 있기 때문에 타입 인자를 명시적으로 기재하지 않아도 상관없다
```kotlin
val letters = ('a'..'z').toList()
print(letters.slice<Char>(0..2))
print(letters.slice(10..13))
```
- 확장 프로퍼티만 제네릭하게 만들 수 있다
- 일반 프로퍼티는 타입 파라미터를 가질 수 없다. -> 클래스 프로퍼티에 여러 타입을 저장할 수 없다
```kotlin
val <T> x: T // ERROR
```

### 제네릭의 선언
- 자바와 동일하게 타입 파라미터를 넣은 홑화살괄호를 클래스나 인터페이스 이름 뒤에 붙인다.
- 타입 파라미터를 이름 뒤에 붙이면 클래스 본문 안에서 다른 일반 타입처럼 사용할 수 있다
```kotlin
interface List<T> {
    operator fun get(index: Int): T // 인터페이스에서 T를 일반 타입처럼 사용
}
```
- 위의 List 를 구현하는 클래스로, 구체적인 타입을 넣을수도, 타입 파라미터로 받은 타입을 넘길수도 있다.
```kotlin
// 구체적인 타입 인자인 'String'을 List 타입 인자로 넘김
class StringList: List<String> {
    override fun get(index: Int): String = TODO()
}

// 제네릭 타입 파라미터 'T'를 List 타입 인자로 넘김
class ArrayList<T>: List<T> {
    override fun get(index:int): T = TODO()
}
```

---

### 제네릭 타입 파라미터 제약
- 타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입인자를 제한하는 기능이다.
- 리스트의 모든 원소의 합을 구하는 `sum` 과 같은 함수가 있다고 가정한다면, `Int`, `Double` 의 타입을 가진 리스트는 가능하지만, `String`, `Boolean` 등의 타입을 가진 리스트는 `sum` 함수를 적용할 수 없다
- 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상계로 지정하면, 해당 제네릭 타입을 인스턴스화 할때 반드시 그 상계 타입이거나, 상계타입의 하위여야 한다
- 제약은 타입 파라미터 이름 뒤에 콜론 `:` 을 표시하고, 상계 타입을 적으면 된다
```kotlin
fun <T : Number> List<T>.sum(): T
```
- 위 `sum` 함수는 `Number`를 확장하기 때문에 가능하다.
```kotlin
fun <T: Comparable<T>> max(first: T, second: T): T{
    return if(first > second) first else second
}
```
- `T`의 상계타입은 `Comparable<T>` 으로, `String` 이 `Comparable<String>`을 확장하기에 해당 함수에 `String`을 사용할 수 있다.

---
- 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우도 있다. 예를들어, 두개 이상의 인터페이스를 구현하게 지정하고 싶다면, 다음과 같이 표현이 가능하다
```kotlin
fun <T> ensureTrailingPeriod(seq: T)
    where T: CharSequence, T: Appendable {  // 타입 파라미터 제약 목록
        // Todo
    }
```

---
### Null이 될수 있는 타입 인자 제외시키기
- 아무것도 상계를 정하지 않은 타입 파라미터는 `Any?` 를 상계로 정한 파라미터와 같다.
```kotlin
class Processor<T> {
    fun process(value: T){
        value?.hashCode()   // value 는 null이 될 수 있다.
    }
}
```
- 하지만 코틀린에서 null이 될 수 없돋록 보장하려면 `Any`를 상계하면 된다
```kotlin
class Processor<T: Any> {
    fun process(value: T){
        value.hashCode()   // value 는 null이 될 수  djqt다.
    }
}
```

---

### 타입 검사와 캐스팅의 한계
- 코틀린 제네릭 타입 인자 정보는 런타임에 지워진다
- 다음과 같은 예시에서 실행중 내부 데이터타입을 추론하는 로직에서는 에러가 발생한다
```kotlin
fun printList(list: List<Any>) {
    when (list) {
        // Error 발생
        is List<String> -> println("String")
        is List<Int> -> println("Integer")
    }
}
```

- 타입 파라미터가 2개 이상이라면 모든 타입 파라미터에 스타 프로젝션(`*`)을 포함하면 된다.
- 컴파일러에서 경고가 나긴하지만, 문제없이 컴파일 된다
```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int>
            ?: throw RuntimeException()
    println(intList.sum())
}

printSum(listOf(1, 2, 3)) // 정상
printSum(setOf(1, 2, 3)) // 리스트가 아니기때문에 IllegalArgumentException 발생
printSum(listOf("a", "b")) // 리스트는 맞지만, 타입 체크는 안되어 ClassCastException 발생
```

---

### 실체화된 타입 파라미터를 사용하는 함수는 타입 인자를 실행 시점에 언급할 수 있다
```kotlin
fun <T> isA(value: Any) = value is T
// Error
```
- 위 예시에서 제네릭 타입은 런타임에선 타입 정보가 사라지는 타입 소거 문제가 있다.
- 하지만 인라인 함수의 타입 파라미터는 실체화되고, 실행 시점에 인라인 함수의 실제 타입 인자를 알수있게 된다.
```kotlin
inline fun <reified T> isA(value: Any) = value is T

println(isA<String>("Hello"))  // true 출력
println(isA<Int>("Hello"))     // false 출력
```
- 실체화된 타입 파라미터를 활용하는 간단한 예로, `filterIsInstance`를 사용해보자.
- 반환 타입이 일치하는 원소들만 추려낼 수 있다
  - 컴파일러는 인라인 함수의 본문을 구현한 바이트코드를 그 함수가 호출되는 모든 지점에 삽입한다. 
  - 컴파일러는 실체화된 타입 인자를 사용해 인라인 함수를 호출하는 각 부분의 정확한 타입 인자를 알 수 있다.
  - 만들어지는 바이트코드는 타입 파라미터가 아닌, 구체적인 타입을 사용하므로 타입 소거의 영향을 받지 않는다.
```kotlin
inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
    val result = mutableListOf<T>()
    for (element in this) {
        if (element is T) {  // reified 덕분에 T의 타입 확인 가능
            result.add(element)
        }
    }
    return result
}

val mixedList = listOf("a", 1, "b", 2, "c", 3)
val strings = mixedList.filterIsInstance<String>()
val numbers = mixedList.filterIsInstance<Int>()

println(strings)  // ["a", "b", "c"]
println(numbers)  // [1, 2, 3]
```

---

### 클래스 참조를 타입 파라미터로 대신하여 java.lang.Class 파라미터 피하기
- 자바에서는 런타임에 클래스 정보를 얻으려면 `java.lang.Class` 객체를 함수의 인자로 넘겨줘야 한다.
- 표준 자바 API 인 `ServiceLoader`를 사용해 서비스를 읽으려면 다음과 같이 호출한다

```kotlin
import java.util.ServiceLoader

val serviceImpl = ServiceLoader.load(Service::class.java)
```
- `::class.java` 구문은 `java.lang.Class` 참조를 얻는 방법을 보여준다.
- 실체화된 타입 파라미터를 사용해 서비스를 읽을 클래스를 `loadService`의 타입 파라미터로 지정한다면 다음과 같다
```kotlin
val serviceImpl = loadService<Service>()
```

- `loadService` 함수는 다음과 같이 정의할 수 있다

```kotlin
import java.util.ServiceLoader

inline fun <reified T> loadService() {
    return ServiceLoader.load(T::class.java)
}
```

---
- 프로퍼티 접근자에 대해 게터 세터의 커스텀 구현을 만들 수 있다.
- 제네릭 타입에 대해 프로퍼티 접근자를 정의하면 프로퍼티를 `inline`으로 표시하고 파라미터를 `reified`로 하면 타입 인자에 쓰인 구체적인 클래스를 참조할 수 있다
---

- 실체화된 타입 파라미터는 유용하지만 몇가지 제약이 있으며, 일단 다음과 같은 경우는 실체화된 파라미터를 사용할 수 있다
  - 타입 검사와 캐스팅
  - 코틀린 리플랙션 API
  - `java.lang.Class` 얻기
  - 다른 함수를 호출할 때 타입 인자로 사용
- 하지만 다음과 같은 일은 할 수 없다
  - 타입 파라미터 클래스의 인스턴스 생성
  - 타입 파라미터 클래스의 동반 객체 메서드 호출
  - 실체화된 타입 파라미터를 요구하는 함수를 호출하며, 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
  - 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 `reified`로 지정

---
### 클래스, 타입, 하위 타입
- 타입 사이의 관계를 논의하려면 하위 타입의 개념을 알아야 한다.
- 아까 상계타입에서 나왔던 개념인데, A 값이 필요한 모든 장소에 어떤 타입 B 를 넣어도 아무 문제가 없다면, 타입 B는 타입 A의 하위 타입이다.
- `Int` 는 `Number`의 하위타입이지만, `String`의 하위타입은 아니다. 그리고 반대의 개념이 상위타입이다.
- 어떤 타입이 다른 타입의 하위 타입인지 검사는 다음처럼 할 수 있다

```kotlin
fun test(i: Int) {
    val n: Number = i   // Int가 Number의 하위 타입이어서 컴파일된다.
    
    fun f(s: String) {}
    f(i)    // Int가 String의 하위 타입이 아니라 컴파일되지 않는다.
}
```


- 기본적으로 하위 타입은 하위 클래스와 근본적으로 같다
- `Int` 클래스는 `Number` 의 하위 클래스 이므로, `Int`는 `Number`의 하위 타입이다
- `String`은 `CharSequence`의 하위 타입이다
- 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이다.
- 하지만 `MutableList<String>`, `MutableList<Any>` 와 같은 데이터타입은 서로 하위타입이 아니다.


---

### 공변성은 하위 타입 관계를 유지한다
- 공변성은 제네릭 타입에서 타입 인자의 하위 타입 관계를 그대로 유지하도록 하는 기능을 뜻한다.
- `Int`가 `Number`의 하위 타입이라고 할 때, 만약 `List<Int>`를 `List<Number>`로 취급할 수 있다면, 공변적이라고 한다.
- 코틀린에서는 공변성을 표현할 때 타입 파라미터 앞에 out 키워드를 사용한다.
```kotlin
interface Producer<out T> {
    fun produce(): T
}
```
- 공변으로 지정하지 않은 다음 예시가 있다고 가정하자
```kotlin

open class Animal {
    fun feed() {}
}

class Herd<T: Animal> { // 공변으로 지정되지 않음
    val size: Int get() = Todo()
}

fun feedAll(animals: Herd<Animal>) {
    for(i in 0..<animals.size) {
        animals[i].feed()
    }
}
```

- 다음 예시에서 그대로 사용하게된다면 에러가 발생한다
```kotlin
class Cat: Animal() {
    //
}

fun takeCareOfCats(cats: Herd<Cat>) {
    feedAll(cats) // 컴파일 오류 발생
}
```

- `Herd` 클래스의 `T`타입 파라미터에 대해 아무 변성도 지정하지 않았기 때문에 고양이 무리는 동물 무리의 하위 클래스가 아니다.
- 명시적 타입 캐스팅을 사용할 수 있겠지만, 공변적 클래스로 만들면 손쉽게 해결할 수 있다
```kotlin
class Herd<out T: Animal> { // T는 공변적이 되었다
    
}

fun takeCareOfCats(cats: Herd<Cat>) {
    feedAll(cats) // 캐스팅이 필요하지 않음
}
```

---

- 파라미터 T 에 붙은 out 키워드는 다음 두가지를 의미한다
  - 하위 타입 관계가 유지된다
  - T를 (함수의 반환 타입) 아웃 위치에서만 사용할 수 있다
```kotlin
interface Transformer<T> {
    fun transform(t: T) : T
}
```
- `(t: T)` 에서의 `T`는 인 위치
- 함수의 `: T` 는 아웃 위치

- 공변성은 읽기전용 메서드로 `T`를 반환하는 메서드만 정의한다. (아웃 위치에서 사용)
- 생성자 파라미터는 인 아웃 어느 쪽도 아니다.

---

### 반공변성은 하위 타입을 뒤집는다
- 반공변성은 공변성의 정 반대라 할 수 있다
```kotlin

interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int {}
}
```

- 위 예제는, `T` 타입의 값을 사용하기만 하고, 반환하지 않는다. 그래서 `T` 앞에는 `in` 키워드가 붙어야 한다.
- 어떤 클래스에 대해 타입 B가 타입 A의 하위 타입일때, `Consumer<A>`가 `Consumer<B>`의 하위 타입인 관계가 성립하면, 제네릭 클래스는 타입 인자 `T`에 대해 반공변이다.
- `in`이라는 키워드는 키워드가 붙은 타입이 클래스 메서드 안으로 전달되어 메서드에서 사용된다는 뜻이다.
 
 | 공변성                                      | 반공변성                                    | 무공변성                  |
  |------------------------------------------|-----------------------------------------|-----------------------|
  | Producer<out T>                          | Consumer<in T>                          | MutableList<T>        |
  | 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지된다          | 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다          | 하위 타입 관계가 성립하지 않는다    |
  | Producer<Cat>은 Producer<Animal>의 하위 타입이다 | Consumer<Animal>은 Consumer<Cat의 하위 타입이다 |                       |
  | T를 아웃 위치에서만 사용할 수 있다                     | T를 인 위치에서만 사용할 수 있다.                    | T를 아무 위치에서나 사용할 수 있다. |


---

### 사용 지점 변성을 사용해 타입이 언급되는 지점에서 변성 지정
- 클래스 선언과 동시에 변성을 지정하면 해당 클래스를 사용하는 모든 장소에 변성 지정자가 영향을 끼치게 되고, 이걸 `선언 지점 변성`이라고 부른다
- 타입 파라미터가 있는 타입으로 사용할 때마다 그 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지 명시해야하는데, 이걸 `사용 지점 변성`이라고 부른다
- 선언 지점 변성을 하면, 변성 변경자를 한번만 표시하면 되어 코드가 더 간결해진다.
- 예로, `Stream.map` 메서드는 다음과 같이 정의되어있다
```java
public interface Stream<T> {
    <R> Strema<R> map(Function<? super T, ? extends R> mapper);
}
```

---
- 다음 예시는 컬렉션의 원소를 다른 컬렉션으로 복사한다.
- 두 컬렉션 모두 무공변 타입이지만, 원본 컬렉션에서는 읽기만하고 다른 컬렉션은 쓰기만 한다.
```kotlin
fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}
```

---


- 다음 예시에서는 `source` 파라미터가 호출만 되고 있으므로 `out` 키워드를 붙여 공변으로 만든다.
- 이는 `T` 타입을 `in` 위치에 있는 메서드를 호출하지 않는다는 것을 뜻한다.

```kotlin
fun <T> copyData(source: MutableList<out T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}
```
```kotlin
fun <T> copyData(source: MutableList<T>, destination: MutableList<in T>) {
    for (item in source) {
        destination.add(item)
    }
}
```

- 코틀린의 `MutableList<out T>`는 자바의 `MutableList<? extends T>`와 동일하다
- 코틀린의 `MutableList<in T>` 는 자바의 `MutableList<? super T>` 와 동일하다

---

### 스타 프로젝션
- 앞에서 타입 검사와 캐스트에 설명할 때, 제네릭 타입 인자 정보가 없음을 표현하기 위해 스타 프로젝션을 사용한다고 했다.
- 먼저, `MutableList<*>` 는 `MutableList<Any?>` 와 같지않다.
- 정해진 구체적인 타입의 원소를 담지만, 타입을 모른다는 뜻일뿐 아무 원소나 다 담아도 된다는 뜻이 아니다.

---
- 자바에서의 와일드카드로는 `?` 에 대응한다.
- 타입 인자에 대한 정보가 중요하지 않을때 사용할 수 있겠다.

```kotlin
fun printFirst(list: List<*>) {
    if (list.isNotEmpty()) {    // isNotEmpty() 함수는 제네릭 타입 파라미터를 사용하지 않는다.
        println(list.first())   // first() 에서는 Any? 를 반환하지만 그걸로도 충분하다.
    }
}
```

---
### 타입별명
- 여러 제네릭 타입이 조합되어있다면, 타입의 의미를 찾기가 힘들 수 있다
- `List<(String, Int) -> String>` 타입 컬렉션의 목적이 무엇인지 확인하기도 힘들고, 함수형 타입을 여러 곳에서 반복사용을 피하고 싶다
- 그때 `typealias` 키워드 뒤에 별명 선언을 할 수 있다
```kotlin
typealias NameCombiner = (String, String, String, String) -> String

val authorsCombiner: NameCombiner = { a, b, c, d -> "$a et al." }
```
- 타입 별명을 도입하면 코드를 읽을 때 좀 더 쉽게 이해가 되도록, 함수형 타입에 새로운 맥락을 부여할 수 있다.
- 