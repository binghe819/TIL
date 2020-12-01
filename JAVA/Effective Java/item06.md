> 본 자료는 [Effective Java 3/E]()를 바탕으로 작성되었습니다.

# 아이템 6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.

특히 불변 객체는 언제든 재사용할 수 있다.

<br>

## 문자열

```java
String s = new String("bikini"); // 따라 하지 말 것!
```

**이 문장은 실행될 때마다 `String`인스턴스를 새로 만든다. 완전 쓸데 없는 행위다.**

이 문장이 반복문이나 빈번히 호출되는 메서드 안에 있다면 쓸데없는 `String` 인스턴스가 수백만 개 만들어질 수도 있다.

아래와 같이 개선해보자.

```java
String s = "bikini";
```

이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 `String` 인스턴스를 사용한다. (String Pool)

**같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.**

<br>

## static 팩토리 메서드

생성자 대신 static 팩토리 메서드를 제공하는 불변 클래스에서는 **static 팩토리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.** 

* 생성자 - 호출할 때마다 새로운 객체를 만든다.
* **static 팩토리 메서드 - 새로운 객체를 만들지 않고 캐싱해둔 객체를 반환할 수 있다.**

예를 들어, `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` static 메서드를 사용하는 것이 좋다.

`Boolean`은 `true`, `false` 밖에 없기 때문에 캐싱해둔 것을 반환하는 것이 좋기 때문이다.

<br>

## 생성하기 무거운 객체

만드는데 메모리나 시간이 오래 걸리는 객체 즉 **"비싼 객체"를 반복적으로 만들어야 한다면 캐시해두고 재사용할 수 있는지 로겨하는 것이 좋다.**

```java
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
```

**`String.matches`가 가장 쉽게 정규 표현식에 매치가 되는지 확인하는 방법이긴 하지만 성능이 중요한 상황에서 반복적으로 사용하기에 적절하지 않다.**

**`String.matches`는 내부적으로 `Pattern`이라는 객체를 새로 만들어 쓴다.**

이 객체를 만들려면 정규 표현식으로 유한 상태 기계로 컴파일 하는 과정이 필요하다. 즉 **비싼 객체다.**

성능을 개선하려면 `Pattern` 객체를 만들어 재사용하는 것이 좋다.

```java
public class RomanNumber {
    
    private static final Pattern ROMAN 
        = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

이렇게 `Pattern` 객체인 `ROMAN`을 재사용하므로써 성능을 개선할 수 있다.

하지만 **이 코드도 문제가 있다. 만약 `isRomanNumeral` 메서드가 호출되지 않는다면, `ROMAN`은 필요없이 만든셈이 된다.**

게으른 초기화(lazily initializing)([아이템 83]())를 사용해서 최적화 할 수 있지만 추천하지 않는다고 한다.

<br>

## 어댑터

불변 객체인 경우에 안정하게 재사용하는 것이 매우 명확하다 하지만 몇몇 경우에 분명하지 않은 경우가 있다. 오히려 반대로 보이기도 한다. 어댑터를 예로 들면, 어댑터는 인터페이스를 통해서 뒤에 있는 객체로 연결해주는 객체라 여러개 만들 필요가 없다.

<br>

책에서는 `Map`인터페이스의 `keySet`을 예로 든다.

`Map` 인터페이스가 제공하는 `keySet`은 `Map`이 뒤에 있는 `Set` 인터페이스의 뷰를 제공한다. 

**`keySet`을 호출할 때마다 새로운 객체가 나올거 같지만 사실 같은 객체를 리턴하기 때문에 리턴 받은 `Set` 타입의 객체를 변경하면, 결국에 그 뒤에 있는 `Map`객체를 변경하게 된다.**

```java
public class Main {

    public static void main(String[] args) {

        Map<Integer, String> students = new HashMap<>();
        students.put(1, "홍길동");
        students.put(2, "자바");

        Set<Integer> test1 = students.keySet();
        Set<Integer> test2 = students.keySet();
        System.out.println(test1.size()); // 2

        test1.remove(1);
        System.out.println(students.size()); // 1 (줄어들면 안되는 줄어든다.)
        System.out.println(test2.size()); // 1
    }
}
```

즉, `Map`은 내부적으로 `keySet`을 가지고 있으며, **얕은 복사로 인해 문제가 발생할 수 있다.**

**이 문제를 해결하는 방법은 객체를 생성하는 깊은 복사이다.** 

즉, **꼭 객체를 생성하지 않는 것이 좋은 것만 아니라는 것.**

<br>

## 오토박싱

불필요한 객체를 생성하는 또 다른 방법은 오토박싱이 있다.

오토박싱은 프로그래머가 프리미티브 타입과 박스 타입을 섞어 쓸 수 있게 해주고 박싱과 언박싱을 자동으로 해준다.

```java
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++) {
            sum += i;
        }
        return sum;
    }
```

위 코드에서 `sum` 변수의 타입을 `Long`으로 만들었기 때문에 불필요한 `Long` 객체를 2^31개 만큼 만들게 된다..

불필요한 오토박싱을 피하려면 박스 타입보다는 프리미티브 타입을 사용해야 한다.

<br>

## 객체를 생성하는 것이 좋을 때도 있다

**이번 아이템을 "객체 생성은 비싸니 피해야 한다"라고 오해하면 안된다.**

방어적 복사(깊은 복사와 유사)가 필요한 상황에서는 객체를 재사용하는 것보다는 새로 만들어야 한다.

만약 방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어 지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다. (VO객체를 떠올리면 될 듯하다.)