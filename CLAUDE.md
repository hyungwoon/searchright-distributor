# 서치라이트AI 콘텐츠 배포 에이전트

블로그 포스트 URL을 입력받아 6개 채널에 최적화된 콘텐츠를 생성하는 독립 에이전트.

## 사용법

```
/distribute <blog post URL>                          # 6개 전 채널 생성
/distribute <blog post URL> linkedin,blind           # 지정 채널만 생성
/distribute <blog post URL> naver,brunch,facebook    # 3개 채널만 생성
```

유효 채널명: `linkedin`, `newsletter`, `naver`, `brunch`, `facebook`, `blind`

## 에이전트 매핑 테이블

| 요청 키워드 | 에이전트 | 파일 | 플러그인 스킬 |
|---|---|---|---|
| 콘텐츠 배포, 채널 적응, 블로그 유통, SEO 배포 | ContentDistribution | `agents/content-distribution.md` | `content-distribution/` → channel-adaptation, seo-geo-optimization |

## 폴더 구조

```
searchright-distributor/
├── agents/
│   └── content-distribution.md       # 에이전트 시스템 프롬프트
├── docs/
│   └── company-profile.md            # 서치라이트AI 기업 프로필 (기업 메모리)
├── plugins/
│   └── content-distribution/
│       └── skills/
│           ├── channel-adaptation/
│           │   └── SKILL.md           # 6개 채널별 작성 규칙 (핵심)
│           └── seo-geo-optimization/
│               └── SKILL.md           # SEO/GEO 분석 프로토콜
├── .claude/
│   └── commands/
│       └── distribute.md              # /distribute 슬래시 커맨드
└── output/                            # 생성 결과물 저장
    ├── _history.md                    # 배포 히스토리 (자동 관리)
    └── YYYY-MM-DD_[slug]/
        ├── 01_linkedin.md
        ├── 02_newsletter.md
        ├── 03_naver-blog.md
        ├── 04_brunch.md
        ├── 05_facebook.md
        └── 06_blind.md
```

## 실행 흐름

```
/distribute [URL] [채널 선택] 입력
  → 0. 에이전트 + 스킬 2개 파일 병렬 로드
  → 1. URL WebFetch → 제목/본문/slug 추출 (실패 시 수동 붙여넣기 대체 절차)
  → 2. SEO/GEO 분석 출력 (사용자 확인용)
  → 2.5. 키워드 확인 (수정 요청 시 조정 후 재출력)
  → 3. output 폴더 생성 (중복 시 덮어쓰기/버전 선택)
  → 4. 채널 콘텐츠 병렬 생성 (전체 또는 선택 채널)
  → 5. 채널별 .md 파일 저장
  → 6. 완료 요약 + 다음 액션 출력 + _history.md 기록
```

## 운영 규칙

- **지식 원천**: `agents/` (시스템 프롬프트) + `plugins/` (SKILL.md) — 항상 파일에서 직접 읽기, 기억/추정 금지
- **결과물 저장**: `output/` 폴더
- **브랜드명**: 서치라이트AI (공백 없음, AI 대문자)
- **블로그 URL**: blog.searchright.net
- **배포 히스토리**: `output/_history.md`에 자동 기록 — 매 배포 완료 시 날짜, 제목, slug, 생성 채널 추가
