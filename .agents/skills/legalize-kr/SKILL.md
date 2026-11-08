---
name: legalize-kr
description: 대한민국 법령 데이터 저장소(legalize-kr)를 탐색·조회한다. 법령 현재 전문 보기, 개정 이력 추적, 법률·시행령·시행규칙 비교, 전체 법령 검색, 특정 날짜 시점의 법령 상태 확인이 필요할 때 사용한다. Korean law repository where each law is a Markdown file and each amendment is a Git commit dated by its 공포일자(promulgation date).
---

# legalize-kr 법령 저장소 탐색

대한민국 법령을 Git으로 관리하는 저장소다. **각 법령 = Markdown 파일**, **각 개정 = 공포일자를 가진 Git commit**이다. 데이터 출처는 [국가법령정보센터 OpenAPI](https://open.law.go.kr).

## 저장소 구조

```
kr/
  {법령명(띄어쓰기 제거)}/
    법률.md            # 법률 본문
    시행령.md          # 대통령령
    시행규칙.md        # 부령
    대통령령.md        # 독립 대통령령(부모 법률 없음)
    {파일명}({법령구분}).md   # 같은 경로를 다른 법령ID가 쓸 때 충돌 해소
```

- 디렉토리명은 [law.go.kr](https://www.law.go.kr) URL 규칙과 동일하게 **법령명에서 띄어쓰기를 제거**한 형태다. 예: `kr/민법/`, `kr/친일반민족행위자재산의국가귀속에관한특별법/`.
- 한 법률과 그 시행령·시행규칙은 **같은 디렉토리**에 함께 있다.

## 자주 쓰는 작업

저장소 루트(`legalize-kr/`)에서 실행한다.

```bash
# 법령 현재 전문 보기
cat kr/민법/법률.md

# 개정 이력 보기 (각 commit = 한 번의 개정)
git log -- kr/민법/

# 법률 ↔ 시행령 비교
diff kr/민법/법률.md kr/민법/시행령.md

# 전체 법령에서 특정 단어 검색
grep -r "개인정보" kr/

# 특정 날짜 시점의 법령 상태 (그 날짜 이전 마지막 개정)
git log --before="2025-01-01" -1 -- kr/민법/법률.md
# 그 시점 전문을 보려면:
git show "$(git log --before=2025-01-01 -1 --format=%H -- kr/민법/법률.md):kr/민법/법률.md"
```

법령명 디렉토리를 모를 때는 frontmatter `제목`으로 찾는다:

```bash
grep -rl "^제목: 민법$" kr/
```

## 메타데이터 (YAML Frontmatter)

각 파일 상단에 법령 메타데이터가 있다. 핵심 필드:

| 필드 | 설명 |
|------|------|
| `제목` | 법령명 (Unicode 정규화: `·`→`ㆍ`) |
| `법령ID` | 법령 고유 식별자 (**안정적 참조 키**) |
| `법령MST` | 법령 마스터 식별자 |
| `법령구분` | 법률 / 시행령 / 시행규칙 / 대통령령 등 |
| `소관부처` | YAML 리스트 (복수 가능) |
| `공포일자` / `공포번호` | 실제 공포일 (Git 날짜보다 이 값을 신뢰) |
| `시행일자` | 시행일 |
| `상태` | 시행 / 폐지 |
| `출처` | law.go.kr URL |

## 반드시 주의할 점

1. **commit hash는 안정 식별자가 아니다.** 파싱·정규화 규칙이 개선되면 저장소 전체를 재구성하고 **force-push**한다(이때 모든 hash가 바뀐다). 장기 참조에는 hash 대신 **`법령ID` + `공포일자` + law.go.kr URL**을 보존하고, 재현이 중요하면 release나 자체 스냅샷을 고정하라. 동기화는 `git fetch --all && git reset --hard origin/main`.

2. **1970-01-01 이전 공포 법령**: Git은 Unix Epoch 이전 날짜를 못 다루므로 그런 법령의 **commit 날짜는 1970-01-01로 고정**돼 있다. 실제 공포일은 항상 frontmatter `공포일자`를 본다. → `git log`의 날짜만 믿지 말 것.

3. **Unicode 정규화**: 가운뎃점이 통일돼 있다. `·`(U+00B7) → `ㆍ`(U+318D). 예: `10·27` → `10ㆍ27`. 검색·매칭 시 정규화된 형태(`ㆍ`)를 쓴다.

4. **시행령 vs 독립 대통령령**: 둘 다 법적으로 "대통령령"이지만, 시행령은 부모 법률이 있고(`{법률명} 시행령` → `시행령.md`), 독립 대통령령은 단독 존재(`대통령령.md`).

## 관련 저장소

| 저장소 | 내용 |
|--------|------|
| `legalize-kr/legalize-kr` | 법령 (이 저장소) |
| `legalize-kr/precedent-kr` | 판례 |
| `legalize-kr/admrule-kr` | 행정규칙 |
| `legalize-kr/ordinance-kr` | 자치법규 |

원문은 대한민국 정부 공공저작물(자유 이용), 저장소 구조·메타데이터는 MIT.
