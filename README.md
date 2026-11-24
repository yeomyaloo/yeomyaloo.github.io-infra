# Stack Overflow — 쌓다 보니 넘쳤다

Docker → Kubernetes → OpenStack 인프라 학습 기록을 올리는 Hugo 사이트입니다.
👉 https://yeomyaloo.github.io/yeomyaloo.github.io-infra/

## 새 글 쓰는 법

1. `hugo new content 01-docker/day02-주제.md` 실행(또는 `archetypes/default.md`를 `posts/챕터/`에 복사)하고, 이미지는 같은 폴더 `images/`에 넣어 `![설명](./images/파일.svg)`로 씁니다.
2. 맨 위 메타데이터(`title`·`date`·`tags`·`summary`·`image`)를 채우고, 공개할 때 `draft: false`로 바꿉니다.
3. `main`에 커밋·푸시하면 GitHub Actions가 빌드해 1~2분 뒤 사이트에 반영됩니다(`hugo server`로 로컬 미리보기).

## 구조

| 경로 | 내용 |
|---|---|
| `posts/` | 글. 하위 폴더 = 챕터(`_index.md`에 챕터 제목·`weight`), `posts/_index.md` = 홈 소개 |
| `archetypes/default.md` | 새 글 템플릿 |
| `content/` | 사이트 전용 페이지(전체 글 목록) |
| `layouts/` | 테마 없이 직접 만든 템플릿 |
| `static/` | CSS, 파비콘, 기본 대표 이미지(`og-default.png`) |
| `hugo.toml` | 사이트 설정(제목, Giscus 댓글 ID 등) |
| `.github/workflows/deploy.yml` | main 푸시 → 빌드 → GitHub Pages 배포 |

새 챕터는 `posts/02-kubernetes/_index.md`를 만들면 사이드바에 자동으로 추가됩니다.
