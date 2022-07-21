```
sudo gem install jekyll bundler
gem install --user-install ffi -- --enable-libffi-alloc  # m1 인 경우
bundle install
bundle exec jekyll serve --watch
```
## m1에서 빌드시 아래 명령어 수행해야함. github action 상에서 build 하기 위해서.
```
bundle lock --add-platform x86_64-linux
```
