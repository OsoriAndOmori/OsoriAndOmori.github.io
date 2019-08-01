# * Revealjs
- 공홈이자 샘플 페이지 : https://revealjs.com/
- Github : https://github.com/hakimel/reveal.js/

# * 원리
- 그냥 html 페이지입니다. 
- README.md 에 마크다운으로 입력해두면 그 내용을 라이브러리가 읽어서 알아서 만들어 줍니다.

```html
<!doctype html>
<html>
	<head>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
		<title>POSTECH-2018</title>
		
		<link rel="stylesheet" href="../../css/reveal.css">
                 <!-- 아래 css 아무거나 바꿔줄 때마다, 전체 인터페이스가 변함..-->
		<link rel="stylesheet" href="../../css/theme/sky.css"> 

		<!-- 코드 테마-->
		<link rel="stylesheet" href="../../lib/css/darkula.css">

		<!-- Printing and PDF exports -->
		<script>
			var link = document.createElement( 'link' );
			link.rel = 'stylesheet';
			link.type = 'text/css';
			link.href = window.location.search.match( /print-pdf/gi ) ? '../../css/print/pdf.css' : '../../css/print/paper.css';
			document.getElementsByTagName( 'head' )[0].appendChild( link );
		</script>
	</head>
	<body>
		<div class="reveal">
			<div class="slides">
                <!-- 핵심 -->
                <section data-markdown="README.md"
                         data-separator="^---"
                         data-separator-vertical="^==="
                         data-separator-notes="^Note:"
                         data-charset="utf-8">
                </section>
			</div>
		</div>

		<script src="../../lib/js/head.min.js"></script>
		<script src="../../js/reveal.js"></script>

		<script>
			// More info about config & dependencies:
			// - https://github.com/hakimel/reveal.js#configuration
			// - https://github.com/hakimel/reveal.js#dependencies
			Reveal.initialize({
                history: true,
				dependencies: [
					{ src: '../../plugin/markdown/marked.js' },
					{ src: '../../plugin/markdown/markdown.js' },
					{ src: '../../plugin/notes/notes.js', async: true },
					{ src: '../../plugin/highlight/highlight.js', async: true, callback: function() { hljs.initHighlightingOnLoad(); } }
				]
			});
		</script>
	</body>
</html>
```

html 도중 요 부분이 핵심입니다.
```
<section data-markdown="README.md"
        data-separator="^---"
        data-separator-vertical="^==="
        data-separator-notes="^Note:"
        data-charset="utf-8">
</section>
```
README.md 에 써있는 markdown 을 읽어서 알아서 만들어 줍니다.
글 도중 `---` 를 입력하면 슬라이드가 나눠지고 
글 도중 `===` 를 입력하면 세로 슬라이드가 추가됩니다.

무튼 더 많은 옵션은 공식 github 에서 읽으시는게 빠를 것 같습니다.

# * 그럼 이걸 어디다가 올려서 좀 편하게 쓸까?
=> Github Pages 활용.

Github 에서는 버튼 하나로 예쁜 개인 페이지를 만들어 주는데요.

<img width="796" alt="2018-11-15 2 40 23" src="https://user-images.githubusercontent.com/22016317/48532748-fd05bf80-e8e4-11e8-96dc-0cb9da605528.png">

만들고나면 
Github Repo : https://github.com/OsoriAndOmori/presentation/  기준
Github Pages : https://osoriandomori.github.io/presentation/ 가 만들어집니다.

Pages Url 뒤쪽을 따로 뭔가 입력안하면 기본적으로 index.html 을 찾아 읽는 것 같습니다.

그러면 저는 

1. `/template/xxx/index.html`에 html file 갖다놓고
2. `/template/xxx/README.md` 에 발표내용 작성
3. https://osoriandomori.github.io/presentation/template/xxx 을 호출했을 때

제 마크다운이 revealjs 를 입고 예뻐질 것 같습니다.

<img width="1023" alt="2018-11-15 2 50 27" src="https://user-images.githubusercontent.com/22016317/48534702-31c94500-e8ec-11e8-9b03-d0bf953b5efc.png">

# * 꾸미는법
위 html 에 주석 적어두기도 했지만,
최상위 디렉토리 에 있는
`css/theme/*.css` 아무거나 로 교체해 주시면 테마가 훅훅 변합니다. 
Custom.css 에 원하는 css 를 추가할 수 있습니다. 물론 시중에 용자들이 다 예쁜걸 많이 만들어놨습니다.ㅎㅎ
퍼와서 html 안에 넣으면 됩니다.

![collage](https://user-images.githubusercontent.com/22016317/48533095-705c0100-e8e6-11e8-8560-54a02e04bdbd.png)


# * 심플하게 써보고 싶으신분들 <요약>
1. `git clone "https://github.com/OsoriAndOmori/presentation"` <별표 구걸>
2. /template/xxx/README.md(실제 발표자료) , README.md(대문) 자신에게 맞게 수정 및 추가.
3. 자기 Repository 로 밀어넣음
4. Repo - Option - Github Pages 버튼 클릭

# * 단점 
- 마크다운이다보니 이미지 편집이 너무 힘듭니다. ㅠㅠ 
- 이미지가 많이 들어가는 발표자료는  결국 ppt로 회귀한다는 슬픈이야기

두서없지만 긴 글 읽어주셔서 감사합니다.