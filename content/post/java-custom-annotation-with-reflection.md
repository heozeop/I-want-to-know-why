---
title: "JAVA Reflection API를 이용한 Annotation만들기"
date: 2024-03-16T21:25:46+09:00
draft: false

categories:
- Java
tags:
- Java
keywords:
- Reflection
- Annotation
- Java
---

## TL;DR

1. Reflection이 뭔가 훑어 보기
2. Annotation이 뭔가 바라만 보기
3. Reflection으로 Annotation 다루기

## Reflection이 뭔가

### 정의

Oracle의 문서에 따르면 Reflection의 정의는 아래와 같습니다.

> Reflection is commonly used by programs which require the ability to examine or modify the runtime behavior of applications running in the Java virtual machine.

> Reflection은 JVM에서 동작하고 있는 어플리케이션의 런타임 동작을 검사하거나 바꾸고자 할 때 사용하는 것이다.

https://docs.oracle.com/javase/tutorial/reflect/

### 주의할 점

- **성능 문제 야기**
    - Reflection은 런타임에 동작을 합니다. 그렇기 때문에 동작을 할 때 JVM의 최적화 과정 일부가 동작하지 않을 수 있습니다. 그 결과 Reflection이 없는 코드와 비교했을때 느릴 수 있습니다.
- **보안상 제약**
    - Reflection은 런타임에 동작해야 합니다. 그렇기 때문에 일부 보안이 중요한 맥락에서는 사용하기 어려울 수 있습니다.
- **내부가 드러남**
    - Reflection은 때로 private등 가려진 정보들에 접근하게 됩니다. 이 때문에 예상치 못한 side effect가 발생할 여지가 생기고, 이식성(portability)에 영향을 줍니다.
    

## Annotation은 뭘까

### 정의

위키백과에 따르면 annotation의 정의는 아래와 같습니다.

> **자바 애너테이션**(Java Annotation)은 자바 소스 코드에 추가하여 사용할 수 있는 [메타데이터](https://ko.wikipedia.org/wiki/%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0)의 일종이다. 보통 @ 기호를 앞에 붙여서 사용한다. [JDK](https://ko.wikipedia.org/wiki/JDK) 1.5 버전 이상에서 사용 가능하다. 자바 애너테이션은 [클래스 파일](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9E%98%EC%8A%A4_%ED%8C%8C%EC%9D%BC)에 [임베디드](https://ko.wikipedia.org/wiki/%EC%9E%84%EB%B2%A0%EB%94%94%EB%93%9C)되어 [컴파일러](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC)에 의해 생성된 후 자바 [가상머신](https://ko.wikipedia.org/wiki/%EA%B0%80%EC%83%81%EB%A8%B8%EC%8B%A0)에 포함되어 작동한다.
> 

[https://ko.wikipedia.org/wiki/자바_애너테이션](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98)

### 종류

- **Meta Annotation**
    - Annotation을 정의하는 Annotation
    - @Retention / @Documented / @Traget / @Inherited / @Repeatable
- **기본 Annotation**
    - @Override / @Deprecated / @SuppressWarnings / @SafeVarargs / @FuncationalInterface

### 만드는 방법

직접 만든 Annotation을 Custom Annotation이라고 부릅니다. 할 수 있는 건 별건 없고 적용 범위와 값을 받는 방법을 정의할 수 있습니다.

```java
@Target(ElementType.FIELD) // 적용 범위
@Retention(RetentionPolicy.RUNTIME) // 언제까지 유지할 것인가
public @interface MyAnnotation {
    String value() default "";
    String a() default "";
}

public class MyObject {
	@MyAnnotation("hello") // value()로 값을 호출
	String hello;
	
	@MyAnnotation(a="world") // a()로 값을 호출
	String world;
}
```

### 사용하는 방법

- annotation의 정의를 할 때 하는 방법은 없다.
    - @이 붙었다 한들 결국 interface이기 때문에 body가 들어갈 수 없습니다.
- annotation의 정의가 존재하는 부분을 찾아서 후처리를 해주어야 합니다. 어떻게 할 수 있을까요?

## Annotation 사용하는 방법

### Reflection과 함께 사용하기

- github 코파일럿이나 AWS 코드 위스퍼러를 사용한다면 사용 예에서 바로 예시를 확인할 수도 있습니다.

```java
public class Initializer {
    public static void init(Object obj) {
        Class<?> clazz = obj.getClass();
        
        for(Field field : clazz.getDeclaredFields()) {
            if(field.isAnnotationPresent(MyDefaultValue.class)) {
                final MyAnnotation annotation = field.getAnnotation(MyAnnotation.class);
                field.setAccessible(true);

                if(field.getType() == String.class)) {
                    field.set(obj, annotation.stringValue());
                }
            }
        }
    }
}
```

- 동작을 설명하자면 다음과 같습니다.
    1. getClass로 Class에 접근
    2. getDeclaredFields로 Field 클래스 객체를 가져옴. 
    3. setAccessible로 private 접근을 가능케 함.
    4. getType으로 정의된 타입을 가져옴
    5. set으로 값을 지정해 넣어 줌
- 여기서 Reflection API가 사용된 모든 부분이 나타납니다. 주요하게 볼 것은 결국 Class 클래스를 이용해 데이터에 접근한다는 것입니다.

### Class 클래스

- java.lang.reflect 패키지에 속한 클래스, 인터페이스, 열거형, 어노테이션 등을 포함한 모든 타입의 정보를 추상화
- JVM에서 로드된 클래스에 대한 메타데이터를 담고 있음
- 가능한 것
  - 타입 정보의 조회, 동적 객체 생성, 매서드 호출, 필드 접근 및 수정이 가능합니다.
- 예시 (gpt 생성)
    ```Java
    public class MyClass {
        public void show() {
            System.out.println("Hello, Reflection!");
        }
    }

    public class ReflectionDemo {
        public static void main(String[] args) {
            try {
                // MyClass에 대한 Class 객체 획득
                Class<?> clazz = Class.forName("MyClass");
                
                // MyClass의 인스턴스 생성
                Object myClassInstance = clazz.getDeclaredConstructor().newInstance();
                
                // show 메서드 정보 조회 및 호출
                Method showMethod = clazz.getMethod("show");
                showMethod.invoke(myClassInstance);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```

## 정리
- Annotation은 적용하는 대상에 대해서만 고민해서 만들기
- 처리는 별도의 클래스에서 Reflection을 이용해 하기

### 생각해 볼 것
1. Annotation을 실체화 했을때 JVM에는 Object로 존재하나?
1. 다른 방법은 없을까?
