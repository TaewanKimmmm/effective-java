# Item 06.불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련되다. 특히 불변 객체는 언제든 재사용할 수 있다.

다음 코드는 하지 말아야 할 극단적인 예이다.

`String s = new String("bikini");`

이 문장은 실행될 때마다 String 인스턴스를 새로 만든다. 완전히 쓸데없는 행위다. 생성자에 넘겨진 "bikini" 자체가 이 생성자로 만들어내려는 String과 기능적으로 완전히 똑같다. 이 문장이 반복문이나 빈번히 호출되는 메서드 안에 있다면 쓸데없는 String 인스턴스가 수백만 개 만들어질 수도 있다.

개선된 코드를 보자.

`String s = "bikini";`

이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 나아가 이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.([JLS, 3.10.5](https://docs.oracle.com/javase/specs/jls/se13/html/jls-3.html#jls-3.10.5))

생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.

생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않다. 불변 객체만이 아니라 가변 객체라 해도 사용 중에 변되지 않을 것임을 안다면 재사용할 수 있다.

생성 비용이 아주 비싼 객체도 더러 있다. 이런 '비싼 객체'가 반복해서 필요하다면 캐싱하여 재사용하길 권한다. 예를 들어 `String.matches` 메서드는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복문 안에서 사용하기엔 적합하지 않다. 

```java
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높다. 성능을 개선하려면 필요한 정규표현식을 불변인 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 나중에 이 인스턴스를 재사용하면 된다.

```java
// 값비싼 객체를 재사용해 성능을 개선한다.
private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
```

```java
static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```

이번 아이템을 '객체 생성은 비싸니 피해야 한다'로 오해하면 안 된다. 특히나 요즘의 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.

거꾸로, 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 자체적으로 객체 풀(pool)을 만들지는 말자. 물론 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 재사용하는 편이 낫다. 하지만 일반적으로는 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. 요즘 JVM의 가비지 컬렉터는 상당히 잘 최적화되어서 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.

이번 아이템은 방어적 복사를 다루는 아이템 50과 대조적이다. 이번 아이템이 '기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라'라면, 아이템 50은 '새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라'다. 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자. 방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다.
