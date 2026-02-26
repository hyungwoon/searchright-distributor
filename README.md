# searchright-distributor

서치라이트AI 블로그 포스트 URL 하나로 6개 채널에 최적화된 콘텐츠를 자동 생성하는 Claude Code 에이전트.

## 지원 채널

| # | 채널 | 특징 |
|---|------|------|
| 1 | LinkedIn | 1,000~1,300자, 전문가 톤, 외부링크 본문 마지막 |
| 2 | 뉴스레터 | 제목 30자 이내, 본문 400~600자, CTA 포함 |
| 3 | 네이버 블로그 | 1,500자 이상, SEO 태그 10개+, 이미지 위치 표시 |
| 4 | 브런치 | 에세이 톤, 긴 호흡 |
| 5 | 페이스북 | 짧고 임팩트 있는 포맷 |
| 6 | 블라인드 | 동료 직장인 관점, 브랜드 노출 최소화 |

## 사용법

```
/distribute <blog post URL>                          # 6개 전 채널 생성
/distribute <blog post URL> linkedin,blind           # 지정 채널만 생성
/distribute <blog post URL> naver,brunch,facebook    # 3개 채널만 생성
```

## 실행 흐름

```
/distribute [URL] [채널 선택]
  → 0. 에이전트 + 스킬 로드
  → 1. URL fetch → 제목/본문/slug 추출
  → 2. SEO/GEO 키워드 분석 출력
  → 2.5. 키워드 확인 (수정 가능)
  → 3. output 폴더 생성 (중복 방지)
  → 4. 채널 콘텐츠 병렬 생성
  → 5. 파일 저장
  → 6. 완료 요약 + 히스토리 기록
```

## 폴더 구조

```
searchright-distributor/
├── agents/
│   └── content-distribution.md       # 에이전트 시스템 프롬프트
├── plugins/
│   └── content-distribution/
│       └── skills/
│           ├── channel-adaptation/
│           │   └── SKILL.md           # 6개 채널별 작성 규칙
│           └── seo-geo-optimization/
│               └── SKILL.md           # SEO/GEO 분석 프로토콜
├── .claude/
│   └── commands/
│       └── distribute.md              # /distribute 슬래시 커맨드
└── output/                            # 생성 결과물 (git 제외)
    ├── _history.md                    # 배포 히스토리
    └── YYYY-MM-DD_[slug]/
        ├── 01_linkedin.md
        ├── 02_newsletter.md
        ├── 03_naver-blog.md
        ├── 04_brunch.md
        ├── 05_facebook.md
        └── 06_blind.md
```

## 요구 사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- 이 프로젝트 디렉토리에서 Claude Code 실행

## 설계 원칙

- **추정 금지**: 원본 URL을 반드시 fetch한 후 작성 (창작/기억 기반 작성 불가)
- **브랜드 통일**: 서치라이트AI (공백 없음, AI 대문자)
- **백링크 필수**: 전 채널에 blog.searchright.net 포함
- **SEO + GEO 이원화**: 전통 검색엔진 최적화 + AI 검색(ChatGPT, Perplexity 등) 동시 대응
