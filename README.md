# KevinPark의 기술 기록

GitHub Pages로 운영하는 한국어 기술 블로그이자 경력 근거 저장소입니다.

## 새 글 쓰기

1. `_drafts/post-template.md`를 복사합니다.
2. 파일명을 `_posts/YYYY-MM-DD-english-slug.md` 형식으로 저장합니다.
3. `title`, `description`, `tags`, `reading_time`을 수정합니다.
4. 공개 전에 고객명, 내부 시스템명, 실제 주소·토픽·식별자, 비공개 수치를 제거합니다.
5. `main` 브랜치에 반영하면 GitHub Pages가 자동으로 다시 빌드합니다.

글은 다음 여섯 항목을 포함하는 것을 권장합니다.

> 문제 → 책임 → 판단 → 실행 → 결과 → 배움

## 로컬 미리보기

Ruby와 Bundler가 설치되어 있다면:

```bash
bundle install
bundle exec jekyll serve
```

브라우저에서 `http://localhost:4000`을 엽니다.

## 주요 파일

- `_config.yml`: 사이트 이름, 주소, 기본 설정
- `_posts/`: 공개된 기술 글
- `_drafts/post-template.md`: 새 글 템플릿
- `projects.html`: 대표 프로젝트와 역할
- `about.md`: 이력 요약과 연락 링크
- `assets/css/style.css`: 전체 디자인
