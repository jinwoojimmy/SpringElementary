# 2장 스프링 프로젝트 생성

## Maven, Gradle

메이븐과 그레이들 모두 동일한 폴더 구조를 사용하며, 필요한 의존 모듈을 관리한다.

프로젝트 빌드와 라이프 사이클, 사이트 생성 등 프로젝트 전반을 위한 관리 도구로서 많은 자바 프로젝트가 이를 사용해서 프로젝트를 관리한다.

* 의존성 전이(Transitive Dependencies)
의존대상이 다시 의존하는 대상까지도 의존 대상에 포함해서 관리하고 다운 받을 수 있다.

* Maven
    * pom.xml
    메이븐 프로젝트에서 핵심은 pom.xml이다. 모든 메이븐 프로젝트 루트 폴더에는 pom.xml이 존재한다.
    메이븐 프로젝트에 대한 설정 정보를 관리하는 파일로, 필요한 의존 모듈, 플러그인 대한 설정을 담는다.
    'mvn compile' 명령어를 실행하면, 지정한 아티팩트 파일을 다운받는다.

* Gradle
다음 gradle파일을 보자

    ```gradle
    apply plugin: 'java'	// 그레이들 java 플러그인 적용

    sourceCompatibility = 1.8	// 소스와 컴파일 결과를 1.8에 맞춤
    targetCompatibility = 1.8
    compileJava.options.encoding = "UTF-8"	// 소스 코드 인코딩으로 UTF-8적용

    repositories {
        mavenCentral()		// 의존 모듈을 메이븐 중앙 리포지토리에서 다운로드
    }

    dependencies {		// spring-context 모듈에 대한 의존을 설정
        compile 'org.springframework:spring-context:5.0.2.RELEASE'
    }

    // 그레이들 래퍼 설정. 소스를 공유할 때 그레이들 설치 없이 그레이들 명령어를 실행할 수 있는 래퍼 생성
    // 'gradle wrapper' 명령어 실행에 성공하면, 프로젝트 루트 폴더에 gradlew.bat, gradlew, gradle 폴더 생성.
    // 그 다음 'gradlew compileJava' 명렁어로 실행
    task wrapper(type: Wrapper) {
        gradleVersion = '4.4'
    }

    ```

## 스프링 기본 예제

```java
/* AppContext.java*/

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration		// 스프링 설정 클래스로 지정
public class AppContext {

	@Bean		// 해당 메드가 생성한 객체를 스프링이 관리하는 빈 객체로 등록함
	public Greeter greeter() {
		Greeter g = new Greeter();
		g.setFormat("%s, 안녕하세요!");
		return g;
	}

}

```


```java
/* Main.java */
// AnnotationConfigApplicationContext 클래스는 자바 설정에서 정보를 읽어와 빈 객체를 생성하고 관리
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx =
				new AnnotationConfigApplicationContext(AppContext.class); // 객체 생성시 앞서 작성한 AppContext 클래스를 생성자 파라미터로 전달. AppContext에서 정의한 @Bean 설정 정보를 읽어와 Greeter객체를 생성 및 초기화

        // getBean 메서드는 AnnotationConfigApplicationContext가 자바 설정을 읽어와 생성한 빈 객체 검색시 사용
		Greeter g1 = ctx.getBean("greeter", Greeter.class);		// 1st param : @Bean 어노테이션의 메서드 이름인 빈 객체의 이름. 2nd param : 검색할 빈 객체 타입.
		Greeter g2 = ctx.getBean("greeter", Greeter.class);
		System.out.println("(g1 == g2) = " + (g1 == g2));		// 결과 : true
		ctx.close();
	}
}


```



## 스프링은 객체 컨테이너
* 스프링 핵심 기능은 객체 생성 및 초기화

* ApplicationContext를 구현하는 AnnotationConfigApplicationContext는 자바 클래스에서 정보를 읽어와 객체 생성과 초기화를 수행. 이 클래스는 자바 어노테이션을 이용한 클래스로부터 객체 설정 정보 가져옴.

* XML, 그루비 코드를 통해 빈을 가져오는 ApplicationContext 구현체들도 존재.

* 공통적으로, 설정 정보로부터 빈(Bean)이라 불리는 객체를 생성하고 그 객체를 내부에 보관.

* ApplicationContext나 BeanFactory 등을 스프링 컨테이너라고 표현함. 빈 객체의 생성, 초기화, 보관, 제거 등을 관리하고 있음.

* 스프링 컨테이너는 빈 이름 - 빈 객체를 연결한 정보를 관리하며, 실제 객체의 생성, 초기화, 의존 주입 등 객체 관리를 위한 다양한 기능을 제공.

* 스프링은 별도 설정을 하지 않는다면, 한 개의 빈 객체만을 생성하며, 이 때 빈 객체는 싱글톤 범위를 갖는다고 표현.

