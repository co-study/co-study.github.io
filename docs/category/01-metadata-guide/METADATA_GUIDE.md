# 📝 포스트 메타데이터 가이드

## 📂 포스트 저장 위치

포스트는 **`_posts`** 폴더에 저장합니다.

- 파일명 형식: `YYYY-MM-DD-title.md` (예: `2025-11-03-nestjs-di-study.md`)
- Jekyll이 자동으로 날짜를 인식하고 포스트를 처리합니다.

## 포스트별 메타데이터 추가하기

각 포스트의 front matter에 다음 메타데이터를 추가할 수 있습니다:

```yaml
---
layout: post              # 레이아웃 (post, default 등)
title: "포스트 제목"       # 포스트 제목
date: 2025-11-03          # 작성일 (YYYY-MM-DD 형식)
author: "작성자 이름"      # 작성자
category: Backend          # 카테고리 (Backend, Frontend, DevOps, Algorithm, 기타)
tags:                     # 태그 목록
  - NestJS
  - TypeScript
  - Backend
related_posts:            # 관련 포스트 (선택사항)
  - title: "관련 포스트 제목"
    url: "/2025/11/03/related-post"
---
```

## 사용 가능한 카테고리

- `Backend` - 백엔드 개발 관련
- `Frontend` - 프론트엔드 개발 관련
- `DevOps` - DevOps 및 인프라 관련
- `Algorithm` - 알고리즘 및 자료구조
- `기타` - 기타 스터디

## 포스트별 레이아웃 커스터마이징

각 포스트에서 `layout`을 지정하여 다른 레이아웃을 사용할 수 있습니다:

```yaml
---
layout: post  # 기본 포스트 레이아웃
# 또는
layout: default  # 기본 레이아웃
---
```

## 카테고리별 메뉴

카테고리 페이지는 `/category/{카테고리명}/` 경로로 접근할 수 있습니다:
- `/category/backend/`
- `/category/frontend/`
- `/category/devops/`
- `/category/algorithm/`
- `/category/etc/`

## 예시

```yaml
---
layout: post
title: "NestJS 의존성 주입 원리 이해하기"
date: 2025-11-03
author: "김철수"
category: Backend
tags:
  - NestJS
  - TypeScript
  - Dependency Injection
related_posts:
  - title: "NestJS 모듈 시스템"
    url: "/2025/11/03/nestjs-modules"
---

# 포스트 내용
```

