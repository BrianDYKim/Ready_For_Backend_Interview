### 1. Dependency Injection에 대해서 아는대로 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

우선 Dependency Injection에 대해서 설명하기 이전에 Association 관계, Dependency 관게에 대해서 설명하고 시작해야할 것 같습니다.

1️⃣ **Association Relationship**
![](./img/association.png)

2️⃣ **Dependency Relationship**
![](./img/dependency.png)

Association 관계의 경우 A라는 클래스가 B라는 객체를 직접 소유하는 관계를 예로 들 수 있습니다.

~~~kotlin
class A {
    private val b: B = B()
}

class B {
    ...
}
~~~

해당 Association 관계의 경우 B 객체를 A가 직접 소유하기 때문에 **둘 사이의 관계는 강하게 결합되어 있습니다.** 
다시 말하면, B의 코드가 변경되면 A로 변경이 전파될 가능성이 존재한다는 것입니다. 이는 **고수준 모듈이 저수준 모듈의 구현에 영향을 강하게 받는다는 소리이기 때문에 객체지향적으로 좋지 못합니다.**

다음으로 Dependency 관계에 대해서 설명을 하자면, A라는 클래스는 B라는 클래스의 객체를 직접 소유하는 것이 아닌, Interface 등의 **변경 가능성이 거의 없는** 것에 의존하는 것을 의미합니다.

~~~kotlin
class A {
    private lateinit var b: B
    
    constructor(b: B): this(b)
    
    ...
}

interface B {
    ...
}

class BImpl: B {
    ...
}

fun main(args: Array<String>): Unit {
    val a = A(BImpl())
}
~~~

위의 방식처럼, 실제 객체를 생성자에 선언하는 것이 아닌, 인터페이스를 생성자의 인자로 제공하여 생성자는 컴파일 시점에 Interface에 의존하고, 런타임에는 A 객체는 실제 인스턴스에 의존하게 만드는 방식입니다.

이러한 방식은 장점이 아래와 같습니다.

1. A 클래스의 로직은 B의 구현체 로직과 디커플링된다.
2. 고수준의 모듈 구현이 저수준의 모듈 구현에 덜 영향을 받게된다.
3. 전략패턴에 부합한다. 즉, 필요에 따라서 B 인터페이스를 구현하는 객체를 A에 대신 끼울수 있다는 것이다.

이는 객체지향이 지향하는 바인 OCP, DIP에 부합하게됩니다.

Spring에서 제공하는 DI의 방식은 크게 3가지인데요, 생성자 주입, 수정자 주입, 필드 주입이 있습니다. 이에 대해서는 나중에 설명을 드리겠습니다.

</div>
</details>