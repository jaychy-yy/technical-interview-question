## 점층적 생성자 패턴이란

#### 1. 개요

보통 빌더 패턴(Builder Pattern)을 설명할 때 비교 대상으로 자주 등장하는 디자인 패턴이다.
비교당할 때 빌더 패턴이 우수하다는 의미로 비교당하는데 각각의 장단점이 있어
자바의 API에서 많이 찾아볼 수 있는 디자인 패턴 중의 하나이다.

#### 2. 본문

점층적 생성자 패턴(Telescoping Constructor Pattern)은 필수 매개변수를 가지는 생성자를 필두로,
선택 매개변수를 가지는 생성자를 추가하여 여러 생성자를 가지는 디자인 패턴이다.

다음은 점층적 생성자 패턴의 예제이다.

```java
public class Student {
    private final String name;				// 필수
    private final int age;					// 필수
    private final String gradeClassNumber;	// 필수

    private final String homeAddress;		// 선택
    private final String phoneNumber;		// 선택

    public Student(String name, int age, String gradeClassNumber) {
        this(name, age, gradeClassNumber, null, null);
    }	// 필수 매개변수를 가지는 생성자

    public Student(String name, int age, String gradeClassNumber, String homeAddress) {
        this(name, age, gradeClassNumber, homeAddress, null);
    }	// homeAddress가 추가된 생성자

    public Student(String name, int age, String gradeClassNumber, String homeAddress, String phoneNumber) {
        this.name = name;
        this.age = age;
        this.gradeClassNumber = gradeClassNumber;
        this.homeAddress = homeAddress;
        this.phoneNumber = phoneNumber;
    }	// 모든 매개변수를 가지는 생성자
}
```

필수 매개변수인 `name`, `age`, `gradeClassNumber`를 가지는 매개변수가 존재하고
이 필수 매개변수를 포함하여 선택 매개변수를 추가한 생성자를 추가하여
마치 생성자가 점층적으로 성장하는 생성자를 가지도록 한 디자인 패턴이 점층적 생성자 패턴이다.

#### 3. 장점

1.  객체의 불변성을 유지할 수 있다.
    -   자바빈즈 패턴(JavaBeans Pattern)에서는 객체의 불변성을 지키기 위하여
        freeze() 함수와 같은 함수를 제공해야 하는 불편함이 있다.
    -   빌더 패턴(Builder Pattern)에서는 불변성이 지켜지기에 큰 의미는 없다.
2.  구현이 간단하다.
    -   빌더 패턴에 비하면 간단하다.

#### 4. 단점

1.  점층적 생성자 패턴을 적용한 클래스의 생성자를 호출하는 입장에서는
    매개변수가 맞는지 헷갈리는 경우가 많다.

    -   위 Student 클래스를 인스턴스화 하는 코드는 다음과 같다.

        ```java
        new Student("이진혁", 19, "3417", "경남 김해시");
        ```

        여기서 19가 나이인지 팔 길이인지 알 수 있는 방법은 Student 클래스의 생성자를 확인해야 한다.

2.  매개변수가 많은 경우 수많은 조합이 만들어져, 생성자의 수가 많아지고 클래스가 복잡해진다.

3.  매개변수의 타입이 같은 경우 생성자를 만들 수 없을 수도 있다.

    -   위 Student 클래스의 `homeAddress`를 추가로 받는 생성자와 `phoneNumber`를 추가로 받는 생성자를
        동시에 추가할 수 없다.