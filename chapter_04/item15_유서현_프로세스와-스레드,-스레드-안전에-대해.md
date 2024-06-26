## 프로세스와 스레드, 스레드 안전에 대해

Effective Java 4장 클래스와 인터페이스 - '아이템 15 - 클래스와 멤버의 접근 권한을 최소화하라' 를 읽고

### 정리

**내용 요약**

어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 바로 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐다. 

- 컴포넌트란 직역하면 구성 요소로, 컴퓨터 소프트웨어에 있어서, 다시 사용할 수 있는 범용성을 위해 개발된 소프트웨어 구성 요소를 일컫는다.

잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다. 오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다. 이를 캡슐화와 정보 은닉이라고 한다.

즉 각 컴포넌트(또는 모듈, 클래스 등)가 자신의 내부 구현(데이터 구조, 알고리즘 등)을 숨기고, 외부 세계와의 상호작용은 오직 정의된 인터페이스인 API (Application Programming Interface)를 통해 이루어진다.

- API란 자바 시스템을 제어하기 위해서 자바에서 제공하는 명령어들을 의미한다. Java SE(JDK)를 설치하면 자바 시스템을 제어하기 위한 API를 제공한다. 자바 개발자들은 자바에서 제공한 API를 이용해서 자바 애플리케이션을 만들게 된다. 패키지 java.lang.*의 클래스들도 자바에서 제공하는 API 중의 하나라고 할 수 있다.
    - https://docs.oracle.com/javase/7/docs/api/index.html 이 문서에서 api들의 오라클 문서 목록을 확인 가능하다.

자바의 각 요소의 접근성은 그 요소가 선언된 위치와 접근 제한자로 정해진다. 이 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심이다.

기본 원칙으로는 모든 클래스와 멤버의 접근성을 가능한 한 가장 낮은 접근 수준으로 부여해야 한다는 것이다.

톱레벨(가장 바깥이라는 의미) 클래스 · 인터페이스에 부여 가능한 접근 수준은 `default`와 `public` 두 가지다. 패키지 외부에서 사용할 필요가 없다면 `default`로 선언하여, API가 아닌 내부 구현으로 만들자. 반면, `public`으로 선언한다면 그 패키지의 API가 되므로 (다른 개발자나 패키지에서 접근 가능하다는 의미) 하위 호환을 위해 꾸준한 관리가 필요하다.

클래스의 공개된 부분을 유심히 살펴, 공개가 꼭 필요한 부분 외의 모든 멤버는 `private`으로 설정하는 것이 캡슐화에 가까워지는 길이다. 특히 `public` 클래스의 인스턴스 필드는 되도록 `public`이 아니어야 하는데, `public` 가변 필드를 가지는 클래스는 일반적으로 **스레드 안전**하지 않기 때문이다.

다만 이렇게 멤버 접근성을 좁히지 못하게 방해하는 제약이 있는데, 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정이 불가하다는 것이다. 이 규칙을 어기면 하위 클래스를 컴파일 시 컴파일 오류가 발생하므로, 이 때는 인터페이스가 정의해둔 메서드를 모두 `public`으로 선언할 수 밖에 없다.

자바 9부터는 모듈 시스템이 도입되면서, 두 가지 암묵적 접근 수준이 추가되었다고 할 수 있다. 패키지가 클래스들의 묶음이듯이, 모듈은 패키지들의 묶음이다. 
모듈은 자신의 패키지 중 공개할 것들을 (관례상 `module-info.java` 파일에) 선언한다. 이렇게 공개하지 않는다면 `public` 멤버라고 해도, 모듈 외부에서는 접근 불가하다. 
모듈 안에서는 선언 여부에 영향을 받지 않으므로, 이를 활용해 클래스를 외부에 공개하지 않으면서도 같은 모듈 내의 패키지 사이에서는 자유롭게 공유 가능하다.

그러나 모듈 시스템을 제대로 활용하기 위해서는, 먼저 패키지들을 모듈 단위로 묶고, 모듈 선언에 패키지들의 모든 의존성을 명시하고, 소스트리를 재배치하는 등 취할 조취가 너무 많으므로, 꼭 필요한 경우가 아니라면 접근 제한자들의 꼼꼼한 설계만으로 접근 수준을 통제함이 좋을 듯 하다.


### 심화 탐구

**출발점**

스레드 안전이라는 개념을 접했지만, 스레드가 프로세스와 유사하다고는 생각하지만 스레드가 정확히 무엇인지, 자바에서 스레드는 어떤 식으로 쓰이는지 파악이 어려워 심화 탐구로 프로세스와 스레드의 차이를 알아보기로 했다.

**설명**

1. 프로세스와 스레드 (Process vs Thread)

    프로세스(Process)는 일반적으로 cpu에 의해 메모리에 올려져 실행중인 프로그램을 말하며, 자신만의 메모리 공간을 포함한 독립적인 실행 환경을 가지고 있다. 자바 JVM(Java Virtual Machine)은 주로 하나의 프로세스로 실행되며, 동시에 여러 작업을 수행하기 위해서 멀티 스레드를 지원하고 있다.

    스레드(Thread)는 프로세스안에서 실질적으로 작업을 실행하는 단위를 말하며, 자바에서는 JVM(Java Virtual Machine)에 의해 관리된다. 프로세스에는 적어도 한개 이상의 스레드가 있으며, Main 스레드 하나로 시작하여 스레드를 추가 생성하게 되면 멀티 스레드 환경이 된다. 이러한 스레드들은 프로세스의 리소스를 공유하기 때문에 효율적이다.

2. 멀티 스레드(multi thread)와 멀티 프로세스(multi process)
   
    일반적으로 하나의 프로세스는 하나의 스레드를 가지고 작업을 수행한다.

    하지만 멀티 스레드(multi thread)란 하나의 프로세스 내에서 둘 이상의 스레드가 동시에 작업을 수행하는 것이다. 

    번외로, 멀티 프로세스(multi process)는 여러 개의 CPU를 사용하여 여러 프로세스를 동시에 수행하는 것을 의미한다.

3. 스레드 안전(Thread-Safety) 
   
   멀티스레딩 환경에서 여러 스레드가 동시에 같은 객체나 데이터에 접근하더라도 프로그램의 실행 결과가 올바르게 나오고, 예상치 못한 문제가 발생하지 않는 것이다.

    즉, 어떤 코드가 스레드 안전하다고 할 때의 의미는, 그 코드는 여러 스레드로부터 동시에 접근되어 실행되어도 데이터의 일관성과 상태의 안정성을 보장한다는 뜻이다.


<br>


**어려웠던 점**

데몬 스레드에 대한 심화 탐구를 진행하며, 가비지 콜렉터도 데몬 스레드의 일종이라는 TCP 스쿨의 글을 보았다. 

**데몬 스레드(deamon thread)**

- 데몬 스레드(deamon thread)란 다른 일반 스레드의 작업을 돕는 보조적인 역할을 하는 스레드를 의미한다. 데몬 스레드는 일반 스레드가 모두 종료되면 더는 할 일이 없으므로, 데몬 스레드 역시 자동으로 종료된다.

- 데몬 스레드의 생성 방법과 실행 방법은 모두 일반 스레드와 같지만, 실행하기 전에 setDaemon() 메소드를 호출하여 데몬 스레드로 설정하기만 하면 된다.

- 이러한 데몬 스레드는 일정 시간마다 자동으로 수행되는 저장 및 화면 갱신 등에 이용되고 있다.


**가비지 컬렉터(gabage collector)**

- 데몬 스레드를 이용하는 가장 대표적인 예로 가비지 컬렉터(gabage collector)를 들 수 있다.

- 가비지 컬렉터(gabage collector)란 프로그래머가 동적으로 할당한 메모리 중 더 이상 사용하지 않는 영역을 자동으로 찾아내어 해제해 주는 데몬 스레드다.

그러나 https://blogs.oracle.com/javamagazine/post/understanding-garbage-collectors 와 https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html 글 등 어디에서도 데몬 스레드에 대한 언급을 찾을 수 없었다. 

또다른 글 https://www.digitalocean.com/community/tutorials/garbage-collection-in-java 에서는 명확하게 밝히고 있어 더욱 혼란스럽다. 

• Garbage Collector is a **Daemon thread** that keeps running in the background. Basically, it frees up the heap memory by destroying the unreachable objects.

추가적인 탐구가 더 필요할 듯 하다.

참고 자료

[https://ko.wikipedia.org/wiki/구성_요소](https://ko.wikipedia.org/wiki/%EA%B5%AC%EC%84%B1_%EC%9A%94%EC%86%8C)

[http://wiki.hash.kr/index.php/컴포넌트](http://wiki.hash.kr/index.php/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8)

https://opentutorials.org/module/516/6222

https://kadosholy.tistory.com/121

https://www.tcpschool.com/java/java_thread_multi

https://www.baeldung.com/java-thread-safety