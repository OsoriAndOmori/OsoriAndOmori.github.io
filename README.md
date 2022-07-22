# book-shelf
- 오소리 사냥뀬의 Today I Learned
- 누군가가 이해할 수 있도록 설명 못 하면 아는게 아니다.

```shell
# jekyll 과 bundler 추가
sudo gem install jekyll bundler

# m1 인 경우 특별히 실행
gem install --user-install ffi -- --enable-libffi-alloc

# 빌드
bundle install
bundle lock --add-platform x86_64-linux

# 로컬 launch
# draft도 포함해서 보려면 --drafts 옵션 추가
bundle exec jekyll serve --watch --trace --drafts
```
