## 문제 의식
- 컨트롤러에 들어오고, request 에서 뭔가를 꺼내는 기존 방식은 불편하고 필요없는 코드를 너무 많은곳에 양산하게됨. 
- 그냥 컨트롤러 맨 앞부터 파라미터로 받으면 좋겠음. request에서 뭔가를 꺼내서 맵핑하는게 싫음

## 설정 방법
HandlerMethodArgumentResolver 를 구현한 커스텀 Resolver를 만듬
```java
public class OsoriModelArgumentResolver implements HandlerMethodArgumentResolver {
	@Override
	public boolean supportsParameter(MethodParameter methodParameter) {
		return OsoriModel.class.isAssignableFrom(methodParameter.getParameterType());
	}

	@Override
	public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer,
			NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {

		HttpServletRequest httpServletRequest = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
		OsoriModel userModel = new OsoriModel(httpServletRequest);
		return userModel;
	}
}
```

스프링 설정 ArgumentResolver 에 추가함. (Java config 로 하고 싶다!!!!!!!!!)
```xml
<mvc:annotation-driven>
		<mvc:argument-resolvers>
			<bean class="com.osori.common.resolver.OsoriModelArgumentResolver"/>
		</mvc:argument-resolvers>
		<mvc:message-converters>
			<ref bean="jsonHttpMessageConverter"/>
		</mvc:message-converters>
</mvc:annotation-driven>
```

그러면? 아래와 같이 쓸 수 있다. 이제 우리는 request 객체를 Controller 에서 안볼수 있다.
```java
@RequestMapping(value = "/test/event/{orderId}", method = RequestMethod.POST)
public ModelAndView test(OsoriModel OsoriModel, @PathVariable String orderId) {
   Param param = new Param(OsoriModel.getUserId(), orderId);
...
}
```