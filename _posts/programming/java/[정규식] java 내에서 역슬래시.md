## Java -
java는 그냥 string 이 그렇게 되어있어서 역슬래시 하나로는 우리가 의도하는 역슬래시가 아님. 2개써야함
- once in regex `\\`
- and once in String `\\\\`

```java
       @Test
	public void testName() {
		//실제 텍스트는 coffee \ tea 이다.
		String str = "coffee \\ tea".replaceAll("\\\\", "or");

		//여기서 \를 or로 변경 하고 싶기 때문에 \ 4개를 붙여서 or로 변경해줌
		assertEquals("coffee or tea", str);
	}

	@Test
	public void testName2() {
		//실제 텍스트는 coffee \\ tea 이다. 하나씩 정규식으로 바꿔주면 or이 두개 들어가게됨.
		String str = "coffee \\\\ tea".replaceAll("\\\\", "or");

		assertEquals("coffee oror tea", str);
	}

	// BEFORE : <a href=\"google.com\"> click to search </a>
	// AFTER : <a href="google.com"> click to search </a>
	@Test
	public void testName3() {
		// 앞에 \\ 는 \을 나타냄, 뒤에 \" 은 " 을 나타냄
		String s = "<a href=\\\"google.com\\\"> click to search </a>";

                //  \\\\ 이거 4개는 글자 그대로 \ 을 나타냄, \" 은 "을 나타냄.
		String str2 = s.replaceAll("\\\\\"", "\"");
		assertEquals("<a href=\"google.com\"> click to search </a>", str2);

                // 역슬래시를 공백으로 바꾸세요.
		String str3 = s.replaceAll("\\\\", "");
		assertEquals("<a href=\"google.com\"> click to search </a>", str3);
	}
```