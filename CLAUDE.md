# CLAUDE.md

**Chirpy** 테마(v7.6, gem 방식 — chirpy-starter 구조) 기반 Jekyll 블로그. GitHub Actions → GitHub Pages 배포. 글은 한국어.

테마 파일(`_layouts`/`_includes`/`_sass`/`assets/js`)은 gem이 제공한다. 레포에는 오버라이드만 둔다:
`_layouts/post.html`(발표 모드 버튼 삽입), `_includes/presentation.html`. 업데이트는 `bundle update jekyll-theme-chirpy`.
오버라이드한 파일은 테마 업데이트 시 gem의 원본과 diff 해서 다시 맞춰야 한다.

> CLAUDE.md 자체는 매 요청 컨텍스트에 100% 들어간다. 여기엔 "매번 필요한 규칙"만 짧게 두고,
> 길어질 것 같으면 별도 문서로 빼서 링크만 남긴다.

## 글 작성

- 경로: `_posts/<주제>/YYYY-MM-DD-<제목>.md` (주제 폴더: `think`, `devOps`, `spring`, `database`, `books`, …).
- front matter: `title`, `author: OsoriAndOmori`, `date: YYYY-MM-DD HH:MM:SS +0900`,
  `categories: [대분류, 소분류]`, `tags: [소문자, ...]`.
  - `categories` 첫 항목은 대분류(`Blogging`, `Thinking` 등), 둘째가 소분류.
- **미래 시각 글은 빌드 안 됨.** KST 기준 이미 지난 시각으로 (`09:00:00 +0900` 무난).
- 초안은 `_drafts/`에 두고 `--drafts` 옵션으로 미리보기.
- 다이어그램: front matter에 `mermaid: true` 넣고 ```` ```mermaid ```` 블록 사용. 플래그 없으면 렌더 안 됨.
- 이미지: `assets/img/posts/<슬러그>/…` 에 두고 `/assets/img/posts/<슬러그>/파일.svg` 로 참조.
  직접 만든 SVG 도형도 OK (라이트/다크 양쪽에서 또렷하게 — 자체 밝은 패널 배경 + 진한 선).
- 정적 라이브러리(mermaid 등)는 CDN(jsdelivr)에서 로드된다. mermaid는 v11 — 발표 덱의 mermaid는 `presentation.html`이 직접 렌더한다.

## 발표 모드 (글별 슬라이드쇼)

front matter에 `presentation: true` 가 있으면 글 상단에 **"▶ 발표 모드"** 버튼이 생긴다.
구현: `_layouts/post.html`(gem 원본 복사 + 로컬 수정)이 버튼 + `_includes/presentation.html` 삽입.
오버레이는 자체 구현(외부 라이브러리 없음), 라이트/다크 대응, 키보드·클릭 네비게이션.

**축약 덱을 따로 작성한다** — 본문 전체를 슬라이드로 돌리지 않는다. 글 본문 **맨 끝**에 숨김 블록:

```markdown
<div class="slides-src" hidden markdown="1">

#### 슬라이드 제목
- 키워드 불릿
- 키워드 불릿

![설명](/assets/img/posts/<슬러그>/도형.svg)

---

#### 다음 슬라이드
...

</div>
```

- `---`(hr)로 슬라이드 구분, `####`(h4)가 슬라이드 제목.
- 블록 안에서는 `##`/`###` 쓰지 말 것 — h4라야 본문 TOC에 안 섞인다.
- `hidden`이라 일반 화면엔 안 보임. 스크립트가 이 블록을 복제해 오버레이로 띄운다.
- 덱 안에서도 이미지·```` ```mermaid ```` 블록 동작함.
- `.slides-src` 블록이 없으면 스크립트가 본문을 `##` 기준으로 자동 분할(fallback).
- 발표 내용 수정 = `.slides-src` 블록만 손대기. 본문 산문은 그대로 둔다.
- 네비: `→`/`Space`/클릭 다음, `←`/`Backspace` 이전, `Home`/`End`, `F` 전체화면, `Esc` 종료.

## 로컬 빌드 / 미리보기

Chirpy 7.x는 Ruby 3.1+가 필요하다. 로컬은 Homebrew `ruby@3.3`을 쓴다. 한글 파일명 때문에 UTF-8 로케일도 export:

```bash
export PATH=/opt/homebrew/opt/ruby@3.3/bin:$PATH LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 RUBYOPT="-E utf-8"
bundle install   # vendor/bundle (gitignore됨)
bundle exec jekyll build
# 미리보기: bundle exec jekyll serve --watch --drafts   (draft 포함)
```

- `.claude/launch.json`(로컬 전용, git 추적 안 함)에 이 PATH/env가 들어 있다.
- `Gemfile.lock`은 커밋한다(linux 플랫폼 포함). `_site/`(gitignore됨), `output/`(스크래치)은 커밋하지 않는다.
- CI(`.github/workflows/pages-deploy.yml`)는 build → htmlproofer → `actions/deploy-pages`. 저장소 Settings → Pages → Source가 **GitHub Actions**여야 한다.
- `post_url`은 폴더 경로까지 써야 한다 (`{% post_url think/2026-09-10-제목 %}`).

## 커밋

- 산출물(`_site`, `output`, `vendor`) 제외하고 스테이징.
- 기본 브랜치 `main`에 바로 커밋·푸시하는 워크플로우(개인 블로그).
