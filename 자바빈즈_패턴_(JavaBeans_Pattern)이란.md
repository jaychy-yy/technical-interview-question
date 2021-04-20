## 자바빈즈 패턴이란?

#### 1. 개요

보통 빌더 패턴(Builder Pattern)이 얼마나 우수한지를 설명하기 위해서 등장하는 디자인 패턴이다.

#### 2. 본문

자바빈즈 패턴(JavaBeans Pattern)은 JSP 때부터 내려온 자바빈(Java Bean)처럼
setter를 클래스에 추가함으로써 생성자 이후 setter로 필드 값을 채워넣으며
객체를 완성하는 디자인 패턴이다.

다음은 자바빈즈 패턴의 예제이다.

```java
public class Student {
    private final String name;				// 필수
    private final int age;					// 필수
    private final String gradeClassNumber;	// 필수

    private String homeAddress;				// 선택
    private String phoneNumber;				// 선택
    
    public Student(String name, int age, String gradeClassNumber) {
        this.name = name;
        this.age = age;
        this.gradeClassNumber = gradeClassNumber;
    }
    
    // homeAddress, phoneNumber의 setter
}
```

이렇게 필수 매개변수만 가지고 있는 생성자를 둔 뒤 setter로 선택 매개변수를 추가로 저장할 수 있다.

```java
Student student = new Student("이진혁", 19, "3417");
student.setHomeAddress("경남 김해시");
student.setPhoneNumber("010-1111-1111");
```

#### 3. 장점

1.  점층적 생성자 패턴(Telescoping Constructor Pattern)에 피하여
    매개변수에 의미를 담을 수 있다.

#### 4. 단점

1.  객체를 완성하는 동안 객체의 일관성이 보장되지 않는다.
    
    -   `setHomeAddress` 함수를 실행하기 전까지는 완성된 객체가 아니다.
    
2.  객체의 불변성을 유지할 수 없다.
    -   객체를 완성한 뒤에도 `setter`는 여전히 사용 가능하기 때문에
        내용이 변경될 여지가 존재한다.
        
    -   대안으로는 `freeze`와 같은 객체를 완성한 뒤 `setter`를 사용하지 못하게 하는 방법도 있다.
    
        ```java
        public class A {
            private String a;
            private int b;
        
            private boolean freeze = false;
        
            public A() {}
        
            public void setA(String a) {
                if (isFreeze()) {
                    throw new AssertionError("완성된 객체는 불변성을 유지해야 합니다.");
                }
                this.a = a;
            }
        
            public String getA() {
                return a;
            }
        
            public void setB(int b) {
                if (isFreeze()) {
                    throw new AssertionError("완성된 객체는 불변성을 유지해야 합니다.");
                }
                this.b = b;
            }
        
            public int getB() {
                return b;
            }
        
            public boolean isFreeze() {
                return freeze;
        }
        
            public void freeze() {
                this.freeze = true;
            }
        }
        ```
        
        객체를 완성한 뒤 `freeze()` 메소드를 사용하면 객체를 얼려, 불변성을 유지시킬 수 있다.
    
3.  객체 하나를 완성하기 위하여 여러 메소드를 호출해야 한다.