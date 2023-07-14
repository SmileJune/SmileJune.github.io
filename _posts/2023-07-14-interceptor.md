---
layout: post
title: Interceptor
date: 2023-07-14 20:57 +0900
---

# Interceptor

## 1. 스프링 MVC 핸들러

먼저 HandlerMapping의 목적은 핸들러 메서드를 URL에 매핑하는 것입니다. DispatcherServlet이 요청을 처리할 때 호출할 수 있습니다.

DispatcherServlet이 HandlerAdapter를 사용하여 메서드를 호출합니다.

인터셉터는 요청을 가로채서 처리합니다. 로깅 및 인증 확인과 같은 반복적인 핸들러 코드를 피하는 데 도움이 됩니다.

## 2. HandlerInterceptor 사용법

### 2.1 메이븐 종속성

인터셉터를 사용하려면 spring-seb 종속성을 포함해야 합니다.

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.13</version>
</dependency>
```

### 2.2. 스프링 핸들러 인터셉터

Spring 인터셉터는 **HandlerInterceptorAdapter 클래스를 확장하거나 HandlerInterceptor 인터페이스를 구현하는 클래스**입니다.

HandlerInterceptor에 존재하는 3가지 주요 메서드

- prehandler() - 핸들러 실행 전에 호출됩니다.
- postHandler() - 핸들러가 실행된 후 호출됩니다.
- afterCompletion() - 완료 요청이 완료되고 뷰가 생성된 후 호출됩니다.

### 2.3. preHandler() 구현 형태

```java
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response,
  Object handler) throws Exception {
    // your code

		// true -> 요청을 추가로 처리해주세요.
		// false -> 처리하지 마세요.
    return true;
}
```

### 2.4. postHandler() 구현 형태

이 방법을 사용하여 로그인한 사용자의 아바타를 모델에 추가할 수 있습니다.

```java
@Override
public void postHandle(
  HttpServletRequest request,
  HttpServletResponse response,
  Object handler,
  ModelAndView modelAndView) throws Exception {
    // your code
}
```

### 2.5. afterCompletion() 구현 형태

이 방법을 사용하면 요청 처리가 완료된 후 사용자 지정 논리를 실행할 수 있습니다.

또한 DefaultAnnotationHandlerMapping을 통해 여러 사용자 지정 인터셉터를 등록할 수 있습니다.

```java
@Override
public void afterCompletion(
  HttpServletRequest request,
  HttpServletResponse response,
  Object handler, Exception ex) {
    // your code
}
```

## 3. HandlerInterceptor 예제 (커스텀 로거 인터셉터)

### 3.1. **HandlerInterceptor 구현 (HandlerInterceptor 인터페이스 구현)**

```java
public class LoggerInterceptor implements HandlerInterceptor {
    ...
}
```

### 3.2. 인터셉터에서 로깅 활성화

```java
private static Logger log = LoggerFactory.getLogger(LoggerInterceptor.class);
```

### 3.3. preHandler() 메서드

요청의 출처와 같은 요청 매개변수에 대한 정보를 기록할 수 있다.

```java
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response,
  Object handler) throws Exception {

    log.info("[preHandle][" + request + "]" + "[" + request.getMethod()
      + "]" + request.getRequestURI() + getParameters(request));

    return true;
}
```

### 3.4. postHandler() 메서드

인터셉터는 핸들러 실행 후 DispatcherServlet이 view를 렌더링하기 전에 이 메서드를 호출합니다.

이를 사용하여 ModelAndView에 속성을 추가할 수 있습니다. 또 요청 처리 시간을 계산할 때도 사용가능하다.

```java
@Override
public void postHandle(
  HttpServletRequest request,
  HttpServletResponse response,
  Object handler,
  ModelAndView modelAndView) throws Exception {

    log.info("[postHandle][" + request + "]");
}
```

### 3.5. afterCompletion() 메서드

뷰가 렌더링된 후 이 메서드를 사용하여 요청 및 응답 데이터를 얻을 수 있습니다.

```java
@Override
public void afterCompletion(
  HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
  throws Exception {
    if (ex != null){
        ex.printStackTrace();
    }
    log.info("[afterCompletion][" + request + "][exception: " + ex + "]");
}
```

## 4. 구성 (사용자 정의 인터셉터 추가)

### **addInterceptors() 메서드 재정의**

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggerInterceptor());
    }

}
```

이 구성이 활성화되면 인터셉터가 활성화되고 어플리케이션의 모든 요청이 제대로 기록됩니다.

여러 개의 Spring 인터셉터가 구성된 경우 preHandle() 메서드는 구성 순서로 실행되는 반면 postHandler() 및 afterCompletion() 메서드는 역순으로 호출됩니다.

스프링 대신 스프링 부트를 사용하는 경우 구성 클래스에 @EnableWebMvc를 달 필요가 없습니다.

**참고 자료**
[Introduction to Spring MVC HandlerInterceptor | Baeldung](https://www.baeldung.com/spring-mvc-handlerinterceptor)
