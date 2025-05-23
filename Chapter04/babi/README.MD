## 4장

### 코틀린에서의 클래스와 인터페이스 기본적인 내용

- 자바와 다르게 구현이 있는 메서드도 정의할 수 있다.
- 클래스를 상속 또는 인터페이스를 구현하려면 클래스 뒤에 상속/구현 하려는 클래스를 콜론(`:`) 을 붙이면 된다.
- 메서드 오버라이드시, `override` 변경자가 필수다

```java
// java
interface Clickable {
    void click();
}

public class Button implements Clickable {
    @Override
    public void click(){
        // Todo
    }
}

```


```kotlin
interface Clickable {
    fun click()
}

class Button: Clickable {
    override fun click(){
        // Todo
    }
}
```

---

- 두개이상의 인터페이스를 하나의 클래스에서 구현하고, 동일한 이름의 메서드와 파라미터가 존재하더라도, 구현 클래스에서 명확하다면 에러가 나지 않는다.
- 그러니깐 간단한 코드로 본다면
#### A, B 인터페이스의 구현없는 메서드
```kotlin
// A interface
interface KotInterface1 {
    fun test()
}

// B interface
interface KotInterface2 {
    fun test()
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 구현 필수
    override fun test() {
        print("hi")
    }
}
```
---
#### 한쪽만 구현되어있는 메서드
```kotlin
// A interface
interface KotInterface1 {
    fun test() = print("hi I'm 1")
}

// B interface
interface KotInterface2 {
    fun test()
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 실제로 구현하지 않더라도 `KotInterface1` 인터페이스에서 디폴트 구현이 되어있어서 상관 없음
//    override fun test() {
//        // implements
//    }
}
```
---
#### 양쪽 다 구현 메서드를 사용
```kotlin
// A interface
interface KotInterface1 {
    fun test() = print("hi I'm 1")
}

// B interface
interface KotInterface2 {
    fun test() = print("hi I'm 2")
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 구현하지 않으면 어떤 디폴트 메서드를 사용해야할지 모르기때문에 구현 필수
    override fun test() {
        print("hi")
        
        // 특정 메서드를 지정하고 싶다면 super 를 사용해서 지정할 수 있다
        super<KotInterface1>.test2()
    }
}
```

---

### 변경자들

- 코틀린은 기본적으로 모든 클래스와 메서드는 `final` 을 사용하여 하위 클래스가 오버라이드를 할 수 없다.
  - 클래스를 작성한 사람의 의도와 다른 방식으로 메서드를 오버라이드하는 위험이 존재하여 닫았다고 한다.
- 상속을 허용하기 위해서는 `open` 변경자를 붙여주면 된다.
- 또한 코틀린에서는 `abstract` 클래스는 `abstract` 함수를 만들수 있고, 해당 함수는 반드시 상속받는쪽에서 구현을 해야한다.
- 사실 코드가 보기 편하긴해서, 인터페이스 하나, 클래스 두개로 만들수있는 상황을 만들어봤다.

```kotlin
// 최초 인터페이스
interface FirstInterface {
  fun basicMethod()
  fun secondMethod()
  fun defaultMethod() = print("It's B")
}

// 인터페이스를 구현한 클래스
// open 으로 선언되어있어 상속을 또 받을수 있다
open class SecondClass : FirstInterface {
  override fun basicMethod() = print("Its' basicMethod")

  // 해당 메서드는 더이상 오버라이드 할수 없다.
  final override fun secondMethod() = print("cannot override")

  // 해당 메서드는 디폴트메서드로 지정되어있기에 반드시 오버라이드 하지않아도 된다
  // override fun defaultMethod() = print("Hello~")

  // 해당 클래스에서 생성한 신규 메서드. 하지만 final 로 선언되어 더이상 하위에서 오버라이드 할수 없다.
  fun newMethodA() = print("It's new A")

  // 해당 클래스에서 생성한 신규 메서드. open 으로 열려있어 하위에서 오버라이드 할수 있다.
  open fun newMethodB() = print("It's new B")
}

class ThirdClass : SecondClass {
  // SecondClass 에서 구현되어있어서, 굳이 반드시 오버라이드 하지않아도 된다
  // override fun basicMethod() = print("Its' new new A")

  // SecondClass 에서 구현되지 않았지만, FirstInterface 에서 구현이 열려있기때문에 오버라이드가 가능하다 
   override fun basicMethod() = print("Its' new new A")

  // 여기서는 secondMethod 를 오버라이드 할수 없다.

  // 여기서는 newMethodA 를 오버라이드 할수 없다.

  // 구현체에서 선언된 open 메서드는 하위에서 오버라이드 가능하다.
  override fun newMethodB() = print("yea~")
}
```

- 변경자들을 표로 정리해본다

| 변경자      | 오버라이드 가능?           | 설명                                       |
|----------|---------------------|------------------------------------------|
| final    | 오버라이드할 수 없음(코틀린 기본값) | 클래스 멤버의 기본 변경자                           |
| open     | 오버라이드 가능            | 오버라이드를 가능하게 만들어주는 변경자                    |
| abstract | 반드시 오버라이드 해야함       | 추상 클래스의 멤버에만 붙일수 있고, 추상 멤버에는 구현이 있으면 안된다. |
| override | 오버라이드 가능 여부와는 관계 없음 | 상위 클래스나 인스턴스의 멤버를 오버라이드 하는중.             |


---

- 가시성 변경자는 외부 접근의 제어 역할을 한다. 자바에서도 `public`, `protected`, `private`, `default` 등이 존재한다.
- 기본적으로 코틀린은 메서드에서 아무것도 기재하지않는다면 `public` 이 기본으로 사용된다.
- 하지만 명시적으로 적는걸 강제하고 싶다면 특정 옵션도 제공한다.
  - `Xexplicit- api={strict|warning}` 컴파일러 옵션을 지정하면 명시적 API 모드를 활성화 할 수 있다.
- 코틀린의 가시성 변경자들은 표로 보면 다음과 같다.

| 변경자        | 클래스 멤버             | 최상위 선언           |
|------------|--------------------|------------------|
| public(기본) | 모든 곳에서 접근 가능       | 모든 곳에서 접근 가능     |
| internal   | 같은 모듈 내에서만 접근 가능   | 같은 모듈 내에서만 접근 가능 |
| protected  | 하위 클래스에서 접근 가능(상속) | 최상위 선언할 수 없음     |
| private    | 같은 클래스 내에서만 접근 가능  | 같은 클래스 내에서만 접근 가능 |
- 코틀린에서 internal 변경자로 선언되어있더라도, 자바에서는 internal의 개념이 존재하지 않기 때문에 접근 가능하다.

---

- 봉인된 클래스라는것도 존재하는데, 코틀린 컴파일러에서는 `when`으로 타입을 검사할때 디폴트 분기인 `else`를 붙이도록 강제한다.
- 하지만 무조건 `else`는 없는게 확실한경우라면?
- 예전에 구글 드라이브와 같은 프로젝트를 할때 잘 사용했었던 클래스다. 객체의 종류가 파일과 폴더 두가지밖에 존재하지 않기 때문에 사용하기 적합했다.
- 그리고 해당 클래스를 사용하기위해 몇가지 조건이 있다
  - 하위 클래스들은 봉인된 클래스가 정의된 패키지와 같은 패키지에 속해야한다.
  - 하위클래스가 모두 같은 모듈 안에 위치해야한다.
  - 컴파일 시점에 알수있어야 한다.

```kotlin

sealed class Node(val name: String)

class File(val name: String): Node(name)

class Directory(val name: String): Node(name)

fun isFileNode(node: Node){
    return when (node){
        is File -> true
        is Directory -> false
        // else 가 필요하지 않다!!
        // else -> throw RuntimeException()
    }
}

```

---

### 클래스 생성

#### 주생성자
- 코틀린에서의 클래스는 주 생성자와 부 생성자가 나뉜다.
- 주 생성자로 선언하는 방식은 여러가지가 존재한다

```kotlin
// 1
class User(val name: String)

// 2
class User(val name: String = "babi")

// 3
class User(_name: String){
  val name = _name
}

// 4
class User(_name: String){
  val name: String
  
  init {
      name = _name
  }
}
```

- 그리고 클래스를 정의할때 디폴트 생성자를 자동으로 만들어준다. 
- 상속받을때는 항상 생성자를 호출해야하기때문에 빈 괄호가 들어간다.
- 인터페이스는 생성자가 없으니 괄호가 없다.

---

#### 부생성자
- 일반적으로 코틀린에서는 디폴트 파라미터와 이름붙은 인자로 부생성자를 사용할일이 적다.
- 하지만 다음과 같은 방식으로 부생성자도 만들수있다
```kotlin
class Downloader {
    constructor(url: String?){
        
    }
}
```

---

#### 인터페이스의 프로퍼티
- 인터페이스에서 선언된 프로퍼티는 상속받은 하위 클래스에서 오버라이드 해줘야한다.
- 예를들면 다음처럼 볼 수 있다
```kotlin
// 인터페이스에 선언된 프로퍼티
interface User {
    val nickname: String
}

// 1
class PrivateUser(override val nickname: String) : User

// 2
class SubscribingUser(val email: String) : User {
    override val nickname: String
    get() = email.substringBefore('@')
}

// 3
class SocialUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId)
}

fun getFacebookName(accountId: Int) = "kodee$accountId"
```
- 위 예제에서 2번은 getter 로 호출할때 마다 `substringBefore` 이라는 함수를 호출하게 되고, 3번은 초기화시 계산하여 필드에 저장해서 불러오는 방식이다
- 예전에 2장했을때 메서드를 사용하지 커스텀 getter 를 사용하는 이유에 대해 토론한적이 있었는데 하나같이 메서드를 사용한다고 했었다. 하지만 커스텀 getter의 차이점은 다음과 같다
  - 예외를 던지지 않음
  - 계산 비용이 적게 듬 (최초 실행한 결과가 캐싱)
  - 객체 상태가 바뀌지 않으면 항상 같은 결과

---

### 데이터 클래스

- 자바에서는 롬복 어노테이션인 `@EqualsAndHashCode` 를 통해 `equals`, `hashCode` 를 구현할수 있다
- 하지만 코틀린에서는 어노테이션 같은걸 붙이지 않더라도 데이터 클래스는 자동으로 `equals`, `hashCode`, `toString`을 생성해준다

---

#### equals
- 두개의 객체가 동등한지 검사하기 위해 사용된다.
- 하지만 오버라이드를 하지 않는다면, 두개의 객체가 가르키는곳이 동일한 주소인지 검사를 하게된다.
- 통상적으로 해당 함수를 오버라이드하여 객체의 멤버가 같다면 동일한 객체로 인지하도록 한다.

---


#### hashCode
- 실행중에 객체의 유일한 Integer를 반환한다.
- Object 클래스는 heap에 저장된 객체의 메모리 주소를 반환하도록 되어있다


---

#### equals 그리고 hashCode 
- 동일한 객체는 동일한 메모리 주소를 갖는다는 의미.
- `equals()` 가 오버라이드 된다면 `hashCode()` 또한 오버라이드 해야한다.
- 서로의 관계를 정리하면 다음과 같다
  - `equals()` 에 사용된 객체가 수정되지 않았다면, `hashCode()`는 항상 동일한 정수값을 반환해야한다.
  - 두 객체가 `equals()` 에 의해 동일하다면, 두 객체는 `hashCode()` 값도 일치해야한다
  - 두 객체가 `equals()` 에 의해 동일하지 않다면, 두 객체는 `hashCode()` 값은 일치하지 않아도 된다.

---

#### 구현

- 단순히 구현으로 클래스 앞에 `data` 를 붙이면 위에서 설명한 모든게 다 설정된다!
```kotlin
// equals(), hashCode() 재정의 완료!
data class Person(val name: String, val age: Int)
```

---

#### Copy? 
- 코틀린에서는 불변의 `val`를 권장한다고 한다. 
- 하지만 객체가 변경이되어야한다면 `var`를 붙이는것보다 `copy` 메서드를 사용할수 있다!
- 당연하게도 원본과는 다른 생명주기를 가지며, 복사본의 값을 바꾸거나 제거하더라도 원본에는 영향을 미치지 않는다. 코틀린이 지향하는 `val`을 게속 사용할수 있다!

---

> 코틀린의 데이터 클래스 vs 자바의 레코드
> - 둘 다 필드 생성과 동시에 생성자 정의를 하여 간결함
> - toString, hashCode, equals 메서드 자동생성 
> - 레코드는 불변 값으로 이루어져있음. 데이터 클래스는 무조건 불변은 아님.
> - 레코드는 copy 같은 메서드가 없음
> - 레코드는 상위 클래스를 확장할 수 없음

---

#### 클래스의 위임

- 코틀린에서는 분명 상속을 기본적으로 막혀있다고 했다.
- 하지만 불가피하게 추가적인 동작을 추가해야할 때가 있는데, 일반적으로 데코레이터 패턴을 사용한다.
- 이때, 새로 정의하는 기능은 데코레이터 메서드로 재정의하고, 기존 기능이 그대로 필요한 부분은 기존 클래스의 메서드에 요청을 전달하게 된다.

```kotlin
class DelegatingCollection<T>(
        innerList: Collection<T> = mutableListOf<T>()
): Collection<T> by innerList {
  var size = 0
  
  override fun add(element: T): Boolean {
    size++
    return innerList.add(element)
  }
}
```

- 여기서 `add()` 메서드만 오버라이드하였고, 나머지 모든 메서드들은 `innerList` 에 위임했다.
---

#### object 

- 클래스를 정의하는 동시에 객체를 생성한다는 공통점이 있다.
  - 객체 선언 (`object declaration`) : 싱글톤 정의하는 방법
  - 동반 객체 (`companion object`) : 클래스와 관련이 있지만, 호출하기 위해 클래스의 객체가 필요하지 않은 메서드와 팩토리 메서드를 담을 때 쓰인다. 여기 멤버에 접근할 때는, 동반 객체가 포함된 클래스의 이름을 사용한다
  - 객체 식 : 자바의 익명 내부 클래스 대신 사용

#### 객체 선언
- 다음과같이 선언되어있는 객체가 있다고 가정하자

```kotlin
object Payroll {
    val allEmployees = arrayListOf()
  
    fun calculate(){
        for(employee in allEmployees){
            // Do some
        }
    }
}
```

- 일반 클래스와 달리 싱글톤 객체는 객체 선언문이 있는 위치에서 생성자 호출없이 즉시 만들어지기 때문에, 생성자 정의가 필요없다.
- 객체 선언을 하며 제공하고 싶은 초기 상태가 있다면 객체의 본문에서 직접 제공할 필요가 있다.
- 변수와 동일하게 객체 이름 뒤에 체이닝방식으로 메서드나 프로퍼티에 접근할 수 있다.
  - `Payroll.allEmployees`
  - `Payroll.calculate()`

---

#### 동반 객체
- 코틀린 클래스는 정적인 멤버가 없다 (자바에서의 `static` 키워드)
- 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체선언을 활용한다. 하지만 최상위 함수는 `private`으로 표시된 클래스 비공개 멤버에 접근할 수 없다. (팩토리 메서드에서는 `private` 멤버에 접근할 수 있어야 생성이 가능;;)
- 클래스 객체에 `companion` 표시를 붙이면, 객체 멤버에 접근시 자신을 감싸는 클래스의 이름을 통해 직접 사용할 수 있다
- 동반 객체는 자신에 대응하는 클래스에 속하기 때문에, 해당 클래스의 인스턴스는 **동반 객체의 멤버에 접근할 수 없다.**

```kotlin
class MyClass {
    companion object {
        fun callMe() {
            println("hi")
        }
    }
}

fun main() {
    // "hi"
    MyClass.callMe()
  
    val myObject = MyClass()
    // Error!
    myObject.callMe()
}
```

---


#### 팩토리 메서드 만들기
- 코틀린에서는 부 생성자를 선언하는 방법도 있지만, 팩토리 메서드로 대신하는 방법도 있다
```kotlin
// 주 생성자를 private 으로 선언하여 클래스 밖에서는 선언하지 못하도록 함
class User private constructor(val name: String) {
    companion object {
        // 팩토리 메서드
        fun of(name: String){
            User(name)
        }
    }
}

fun main() {
    val user =  User.of("babi")
}
```

---

#### 인라인 클래스

- 책에서 예제로 환율에 대해 나와있다. `addExpense(expense: Int)` 메서드가 있고, 파라미터로 미국 달러의 센트를 넘긴다고 가정했을때, 호출하는 쪽에서 달러 센트가 아닌 원화, 엔화 등으로 넘길수도 있는 노릇이다.
- 이때, 가장 쉽게 클래스를 하나 생성하여 파라미터를 `Int` 가 아닌 다른 클래스를 사용하는 것으로 방지할 수 있다.
```kotlin
class UsdCent(val amount: Int)

fun addExpense(expense: UsdCent){
    // Do Some
}
```
- 이렇다면 함수에 잘못된 의미를 전달할 위험이 아주 줄어들지만, 해당 함수를 호출할 때 마다 `UsdCent` 객체가 새로 생성되고 버려지기에 GC 에 의해 수많은 객체들이 제거될것이다.
```kotlin
@JvmInline
value class UsdCent(val amount: Int)
```
- 위와같이 선언한것만으로, 실행 시점에서 `UsdCent` 인스턴스는 감싸진 내부 프로퍼티로 대체되어 객체 생성의 비용을 줄이게 된다.
- 인라인으로 표시하려면 클래스가 프로퍼티를 하나만 가져야하며, 프로퍼티는 주 생성자에서 초기화되어야 한다.
- 인라인 클래스는 클래스 계층에 참여하지 않으므로, 다른 클래스를 상속 하거나 상속 받을 수 없다.
- 하지만 인터페이스를 상속하여 메서드를 정의, 계산된 프로퍼티를 제공할 수 있다.