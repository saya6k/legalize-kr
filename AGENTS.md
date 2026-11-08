# AGENTS.md

이 저장소는 **대한민국 법령 데이터**다. 각 법령은 Markdown 파일이고, 각 개정은 공포일자를 author/committer date로 가진 Git commit이다. 자세한 사용법은 스킬 [`legalize-kr`](.agents/skills/legalize-kr/SKILL.md)에 있다 — 법령 조회·이력·비교·검색·시점 조회 작업 전에 읽어라.

## 구조

```
kr/{법령명(띄어쓰기 제거)}/{법률|시행령|시행규칙|대통령령}.md
```

각 파일 상단에 YAML frontmatter 메타데이터(`제목`, `법령ID`, `공포일자`, `상태`, `출처` 등)가 있다.

## 반드시 지킬 것

- **commit hash는 안정 식별자가 아니다.** 정규화 규칙 개선 시 저장소 전체가 재구성되고 **force-push**된다. 장기 참조는 hash가 아니라 **`법령ID` + `공포일자` + law.go.kr URL**로 하라. 동기화: `git fetch --all && git reset --hard origin/main`.
- **1970-01-01 이전 공포 법령**은 Git 한계로 commit 날짜가 1970-01-01로 고정된다. 실제 공포일은 frontmatter `공포일자`를 신뢰하라.
- **Unicode 정규화**: 가운뎃점 `·`(U+00B7) → `ㆍ`(U+318D). 검색·매칭은 정규화된 형태를 쓴다.

## 이 포크에 대하여

`saya6k/legalize-kr`는 `legalize-kr/legalize-kr`의 포크이며, GitHub Actions(`.github/workflows/sync-upstream.yml`)가 매일 업스트림으로 **미러(hard-reset + force-push)**한다. 업스트림 콘텐츠를 통째로 덮어쓰므로 **`kr/` 등 데이터에 직접 커밋하지 마라 — 다음 동기화에 사라진다.** fork-local 파일(`AGENTS.md`, `.agents/`, sync 워크플로우)만 동기화 시 보존된다. 업스트림에는 어떤 쓰기도 하지 않는다(fetch 전용).
