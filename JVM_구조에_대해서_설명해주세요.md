## JVM Structure

### 1. 개요

`JVM`은 `Java Virtual Machine`의 약자로 자바 바이트코드를 실행할 수 있는 가상 환경이라고 할 수 있다.

![JVM Structure](https://www.javainterviewpoint.com/java-virtual-machine-architecture-in-java/jvm-architecture/)

`JVM` 구조를 그림으로 정리하면 위와 같이 정리할 수 있을 것 같다.
처음 봤을 때는 굉장히 복잡해보이는데 모두 알고 보면 자연스러운 과정이라는 것을 알 수 있다.

### 2. Class Loader

클래스 파일(`.class` 파일, `Byte Code`)을 받으면 이를 토대로 클래스를 로딩하는 일을 담당한다.
`Class Loader`가 하는 일은 세 가지로 나누어진다.

#### 2-1. Loading

`Class Loader`가 하는 전반적인 클래스 로딩과 관련된 일들은 모두 여기서 처리한다.
하지만 클래스 로딩을 하는 `Class Loader`는 크게 세 가지로 나누어진다.

##### 1) Kind of Class Loader

1.  Bootstrap Class Loader
    -   자바에 기본적으로 내장되어 있는 클래스들을 로딩하는 `Class Loader`이다.
        이러한 내장 클래스들은 `rt.jar`라는 `jar`파일에 담겨져있다.
2.  Extension Class Loader
    -   `$JAVA_HOME/lib/ext` 디렉토리에 존재하는 클래스와 `java.ext.dirs` 시스템 변수에 저장된
        클래스들을 로딩하는 `Class Loader`이다.
3.  Application Class Loader
    -   애플리케이션 수준에 존재하는 클래스들을 로딩하는 `Class Loader`이다.
        개발자가 직접 작성한 클래스들이 여기에 해당된다.
    -   `System Class Loader`라고도 한다.

이외에도 개발자가 직접 `Class Loader`를 구성하였을 경우 `Custom Class Loader`를 만들 수 있다.

##### 2) Priority of Class Loader

1.  `Class Loader Cache`
    -   `Class Loader`는 처음부터 모든 클래스를 로딩하지 않는다.
        클래스가 필요할 때마다 요청에 의해 클래스를 찾아, 로딩하는데
        이미 로딩된 클래스를 다시 로딩할 필요는 없으므로 클래스를 캐싱해둔다.
2.  `Bootstrap` > `Extension` > `Application(System)` > `Custom` (`Bootstrap`이 우선순위가 높음)
    -   `Class Loader`는 우선순위가 높은 클래스를 로딩한다.
        만약 `Bootstrap Class Loader`와 `Application Class Loader`에 같은 클래스가 있다면
        `Bootstrap Class Loader`에 있는 클래스를 로딩한다는 뜻이다.
        이러한 알고리즘을 `위임 계층 알고리즘 (Delegation Hierarchy Algorithm)` 이라고 한다.
    -   하지만 `Class Loader`는 `Custom` 즉, 우선순위가 낮은 `Class Loader`부터 탐색하는데
        이는 클래스를 찾으면 바로 로딩하도록 구현된 `WAS`가 탄생할 수 있도록 했다.
3.  `Class Not Found Exception`
    -   `Bootstrap Class Loader`에서도 클래스를 찾지 못하면
        클래스를 찾을 수 없다는 에러를 발생시킨다.

##### 3) Feature of Class Loader

1.  `Limit Visibility`
    -   하위에 있는 `Class Loader`는 상위에 있는 `Class Loader`를 알 수 있지만
        상위에 있는 `Class Loader`는 하위에 있는 `Class Loader`를 알 수 없다.
2.  `Unload Not Possiable`
    -   `Class Loader`는 로딩될 순 있지만 다시 제거될 순 없다.
3.  `Name Space`
    -   `Class Loader`는 클래스를 [패키지명] + [클래스명]으로 `Full name`을 저장하고
        각각의 `Class Loader`마다 다른 영역을 가지므로
        같은 이름의 클래스가 로딩되더라도 다른 클래스로 구분지을 수 있다.

#### 2-2. Linking

`Linking`에서는 바이트코드를 검증하고 준비하는 과정을 거친다.

##### 1) Verify

바이트 코드가 정상적인지 확인한다.

##### 2) Prepare

정적 변수에게 메모리를 할당하고 기본 값을 저장한다.

>   여기서 기본값이란 기본 타입에게는 0, 참조 타입에게는 null을 의미한다.

##### 3) Resolve

심볼릭 메모리 참조를 실제 메모리 참조로 변경한다.

>   클래스 파일은 다른 레퍼런스에 대한 참조을 알 수 없기 때문에
>   심볼릭 메모리 참조로 대체하고 있다.
>   이제 `Class Loader`에 의해 메모리에 올라온 클래스를 참조할 수 있으므로
>   참조하도록 변경하는 것이다.

#### 2-3. Initialization

`Initialization` 과정에서는 `Prepare` 과정에서 기본 값을 넣었던 정적 변수에 실제 값을 저장하고,
`static initializer`를 실행한다.

### 3. Runtime Data Area

`Class Loader`가 로딩한 클래스들은 `Runtime Data Area`의 저장공간에서 관리된다.
`Runtime Data Area`는 크게 다섯 가지로 나눌 수 있다.

#### 3-1. Method Area

모든 스레드가 공유하는 영역으로, `Class Loader`에서 로딩한 인터페이스, 클래스의
`static variable`, `static method`, `static class`에 대한 정보를 관리하며,
그 중 `Runtime Constant Pool`에서는 모든 메소드와 필드의 메모리 참조 주소 테이블을 관리하여
메소드 참조시 `Runtime Constant Pool`을 통해 매핑한다.

#### 3-2. Heap Area

모든 스레드가 공유하는 영역으로, 클래스의 인스턴스를 저장하는 공간이다.
`Heap Area`는 크게 세 구역으로 나누어진다.

##### 1) Young Generation

`Young Generation`은 젊은 객체가 주둔하고 있는 공간이다.
여기서 일어나는 `GC`는 `Minor GC`라고 하고 자원 소모가 적다.

>   보통 `Minor GC`는 `Stop the World` 현상이 일어나지 않는다고 생각하는 경우가 많은데
>   이는 사실이 아니고 살아있는 객체로 인해 `Eden` 영역이 가득차서 `Survivor` 영역으로 옮기는 경우
>   `Stop the World` 현상이 발생하여 생각보다 많은 시간 동안 모든 스레드가 멈춘다.
>   https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc

`Young Generation` 영역은 `Eden` 영역 하나와 `Survivor` 영역 두 개로 나누어진다.

1.  `Eden` 영역

    -   생성된 객체는 `Eden` 영역에 저장된다.
        만약 `Eden` 영역이 가득차게 되면 `Minor GC`가 발생한다.

2.  `Survivor` 영역

    -   `Survivor` 영역은 `From Survivor`과 `To Survivor`로 구분짓는데
        `Minor GC` 이후 살아남은 객체들이 `Eden` 영역에서 `From Survivor`로 이동한다.
        한 `Survivor` 영역이 가득차면 살아있는 객체들은 반대 `Survivor` 영역으로 이동하게 된다.
        이 과정을 일정 횟수만큼 반복하면 `Old Generation`으로 이동한다.

        >   사실 `From Survivor`에서 `To Survivor`로 객체가 이동하는 것이 아니라
        >   `From Survivor`와 `To Survivor`의 논리적인 위치를 바꾼다.

##### 2) Old Generation

`Old Generation`은 늙은 객체가 주둔하고 있는 공간이다.
여기서 일어나는 `GC`는 `Major GC`라고 하고 자원 소모가 크다.

>   `Major GC`와 `Full GC`를 혼용하는 경우가 많다.
>   `Major GC`는 대상이 `Old Generation`에 국한되지만
>   `Full GC`는 대상이 `Heap Area` 전체로 넓다.

`Old Generation`은 보통 `Young Generation` 보다 넓고 객체의 생명주기가 보통 짧은 특성 때문에
`Major GC`가 발생하는 빈도는 적다.
하지만 `Major GC`로 인해 발생하는 `Stop The World`는 애플리케이션의 성능에 엄청난 영향을 끼칠 수 있다.

##### 3) Metaspace

`Metaspace` 영역은 `Java 8` 이후에 생긴 영역으로 기존에 존재하던
`Permanent Generation` 영역을 대체하기 위해서 등장하였다.

기존 `Permanent Generation` 영역에서는 `JVM`에서 어떠한 객체를 설명하는데
필요한 정보들을 저장하는 영역이다.

>   `Permanent Generation`과 `Method Area`는 본질적으로 가지고 있는 정보가 다른 것 같다.
>   (`Static`한 정보들을 가지고 있다고 알고 있었는데 정확하게는 모르겠다.)
>
>   >   A third generation closely related to the tenured generation is the permanent generation. The permanent generation is special because it holds data needed by the virtual machine to describe objects that do not have an equivalence at the Java language level. For example objects describing classes and methods are stored in the permanent generation.
>   >   <참조 https://stackoverflow.com/questions/9095748/method-area-and-permgen>

이런 `Permanent Generation` 영역은 제한된 크기를 가지고 있어
`OOME (Out of Memory Error)`를 발생시켜 개발자들의 골칫거리를 만들었다.
그래서 `Java 8` 이후 부터는 `Metaspace`라는 영역이 `Permanent Generation` 영역을 대체하였다.

`Metaspace`는 `JVM`이 관리하는 `Permanent Generation` 영역과는 다르게
`OS`단에서 관리하여 제한된 크기를 가지는 경우가 없다.

#### 3-3. Stack Area

`Stack Area`는 `Stack Frame`을 저장하는 공간이며 `Method Call`당 하나씩 생성된다.
스레드 당 하나의 `Stack Frame`을 가진다.

#### 3-4. PC Register

`PC Register`도 `Stack Frame`과 마찬가지로 스레드 당 하나의 `PC Register`를 가지게 된다.
현재 실행 중인 명령의 주소를 가지고 있다.

멀티 스레딩시 `PC Register`의 값을 토대로 저장했다가 다른 일 하고 다시 돌아올 수 있게 한다.

#### 3-5. Native Method Stack

바이트 코드가 아닌 다른 언어 (C, C++ 등)를 통하여 `Stack Frame`이 생길 시
`Stack Area`가 아닌 `Native Method Stack`에 `Stack Frame`이 쌓이게 된다.
이런 다른 언어를 `JVM` 위에서 돌아가게 하기 위해서는
`JNI (Java Native Interface)` 규격에 맞게 적절하게 변경해야 한다.
이렇게 다른 언어로 작성된 코드들을 `Native Method Library`라고 한다.

### 4. Execution Engine

`Execution Engine`은 바이트 코드를 실행하는 주체이다.
`Runtime Data Area`에 존재하는 바이트 코드를 가져와서 실행하게 된다.

`Execution Engine`이 바이트 코드를 해석하는 방법은 두 가지가 있다.

#### 4-1. Interpreter

`Interpreter` 방식은 바이트 코드를 한 줄 한 줄 해석하며 실행한다.
`Execution Engine`은 일반적으로 이 방식을 채택하지만
한 줄식 해석하는 것이 성능에 좋지 못하고 느리기 때문에
다음과 같은 `JIT Compile` 방식을 함께 사용한다.

#### 4-2. JIT Compile

`JIT Compile` 방식은 일정 바이트 코드를 컴파일하여 `Native Code`로 변경한 뒤
더 이상 `Interpreter` 방식으로 해석하지 않고 `Native Code`를 그대로 가져와서
실행하는 방식으로 해석하는 방식이다.

일정 바이트 코드가 여러 번 반복되면 극적인 성능 향상을 볼 수 있지만
한두 번 반복될 코드라면 차라리 `Interpreter` 방식으로 해석하는 것이 효율적이다.
그래서 `Execution Engine`은 두 방식을 모두 사용한다.

### 5. Garbage Collector

`Heap Area` 영역에서 소개했듯이 `GC`는 동작한다.
`System.gc()` 메소드를 이용해서  `GC`를 호출하게 할 순 있지만 무조건적인 실행을 보장하지는 않는다.

### 6. 참조

>   [JVM Structure]
>   https://dzone.com/articles/jvm-architecture-explained
>
>   [Minor GC vs Major GC vs Full GC]
>   https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc