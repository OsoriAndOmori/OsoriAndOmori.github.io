# CLAUDE.md

**Chirpy** 테마(v5.2.1) 기반 Jekyll 블로그. GitHub Pages 배포. 글은 한국어.

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
- `assets/lib`는 git 서브모듈(chirpy-static-assets). 직접 수정 금지. `_config.yml`의 `assets.self_host`가
  꺼져 있어 mermaid 등은 CDN(jsdelivr)에서 로드됨.

## 발표 모드 (글별 슬라이드쇼)

front matter에 `presentation: true` 가 있으면 글 상단에 **"▶ 발표 모드"** 버튼이 생긴다.
구현: `_layouts/post.html`(로컬 override)이 버튼 + `_includes/presentation.html` 삽입.
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

Ruby 2.6이 한글 파일명 인코딩을 잘못 잡으므로 **항상** UTF-8 로케일을 export:

```bash
export LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 RUBYOPT="-E utf-8"
bundle exec jekyll build
# 미리보기: bundle exec jekyll serve --watch --drafts   (draft 포함)
```

- `.claude/settings.json`에 이 env를 넣어두면 매번 export 안 해도 됨(로컬 전용, git 추적 안 함).
- 빌드하면 `Gemfile.lock`에 platform 줄이 추가된다 — 커밋 전 `git checkout Gemfile.lock`.
- `_site/`(gitignore됨), `output/`(빌드/스크래치 산출물)은 커밋하지 않는다.
- 새 배포 플랫폼 추가 시: `bundle lock --add-platform x86_64-linux`.

## 커밋

- 산출물(`_site`, `output`, `vendor`)·`Gemfile.lock` 잡변경 제외하고 스테이징.
- 기본 브랜치 `main`에 바로 커밋·푸시하는 워크플로우(개인 블로그).

## 확인해두면 좋을 것 (TODO)

- `_config.yml`의 `lang: ko-Ko` → 표준 Chirpy 로케일은 `ko-KR`. `_data/locales/`에 어느 파일이
  있는지 보고 맞추면 날짜/UI 번역이 정확해짐.
