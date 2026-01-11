# 🚀 로컬 개발 환경 설정 가이드

## 필수 사항

로컬에서 Jekyll을 실행할 때 GitHub Pages와 동일하게 작동하려면 다음 명령어를 사용해야 합니다.

## 설치

```bash
# 1. Ruby 설치 확인
ruby --version

# 2. Bundler 설치 (없는 경우)
gem install bundler

# 3. 의존성 설치
bundle install

```

## 실행 방법

### 올바른 실행 방법 (GitHub Pages와 동일)

```bash
# 프로젝트 루트에서 실행
bundle exec jekyll serve --source docs --destination _site

```

또는 더 간단하게:

```bash
cd docs
bundle exec jekyll serve

```

### 서버 시작 후

브라우저에서 `http://localhost:4000` 또는 `http://127.0.0.1:4000`으로 접속하세요.

## 문제 해결

### Remote Theme가 적용되지 않는 경우

1. **플러그인 확인**
   - `_config.yml`에 `jekyll-remote-theme`이 `plugins` 목록에 있는지 확인
   - `bundle list | grep remote` 명령어로 플러그인 설치 확인

2. **네트워크 연결 확인**
   - Remote theme는 GitHub에서 테마를 다운로드해야 하므로 인터넷 연결이 필요합니다
   - 방화벽이나 프록시 설정이 GitHub API 접근을 막고 있는지 확인

3. **캐시 삭제**
   ```bash
   rm -rf .jekyll-cache
   rm -rf _site
   bundle exec jekyll clean
   bundle exec jekyll serve --source docs

   ```

4. **플러그인 재설치**
   ```bash
   bundle update jekyll-remote-theme
   bundle exec jekyll serve --source docs

   ```

## 빌드 명령어 (GitHub Actions와 동일)

```bash
bundle exec jekyll build -s docs -d _site

```

## 추가 팁

- `--incremental` 옵션을 사용하면 변경된 파일만 재빌드하여 더 빠릅니다:
  ```bash
  bundle exec jekyll serve --source docs --incremental

  ```

- `--livereload` 옵션을 사용하면 파일 변경 시 자동으로 브라우저가 새로고침됩니다:
  ```bash
  bundle exec jekyll serve --source docs --livereload

  ```
