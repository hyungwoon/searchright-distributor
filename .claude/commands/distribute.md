---
description: 서치라이트AI 블로그 포스트 URL → 채널 콘텐츠 생성 및 output 저장
argument-hint: "<blog post URL> [채널: linkedin,newsletter,naver,brunch,facebook,blind | 기본: 전체]"
---

사용자가 입력한 인자: $ARGUMENTS

## 인자 파싱

1. **첫 번째 토큰** = URL (필수)
2. **두 번째 토큰** = 채널 선택 (선택, 쉼표 구분)
   - 미입력 시 → 6개 전 채널 생성
   - 입력 시 → 지정 채널만 생성
   - 유효 채널명: `linkedin`, `newsletter`, `naver`, `brunch`, `facebook`, `blind`
   - 예: `/distribute https://blog.searchright.net/abc linkedin,blind`

## 실행 절차 (순서 엄수, 생략 불가)

### 0단계: 에이전트 + 스킬 로드

다음 3개 파일을 병렬로 Read:
- `agents/content-distribution.md`
- `plugins/content-distribution/skills/channel-adaptation/SKILL.md`
- `plugins/content-distribution/skills/seo-geo-optimization/SKILL.md`

### 1단계: URL Fetch

WebFetch로 URL의 본문을 실제로 읽는다.

- **성공 시** → 제목, 본문, URL slug 추출 후 2단계로
- **실패 시** → 아래 대체 절차 실행:
  1. 사용자에게 안내: "URL을 불러올 수 없습니다. 본문을 직접 붙여넣어 주세요."
  2. 사용자가 본문 붙여넣기 시:
     - **제목**: 본문 첫 줄에서 추출 시도 → 불명확하면 사용자에게 확인
     - **slug**: 원본 URL에서 마지막 경로 세그먼트 추출 시도 → 실패 시 제목을 kebab-case로 변환
     - **본문**: 붙여넣은 내용 전체 사용
  3. 추출 결과(제목, slug)를 사용자에게 보여주고 확인 후 2단계로 진행

**slug 추출 규칙**:
- URL 마지막 경로 세그먼트 사용 (예: `/ai-recruiter-trend` → `ai-recruiter-trend`)
- 쿼리파라미터(`?id=123`) 형태면 포스트 제목을 kebab-case로 변환

### 1.5단계: 퍼널 자동 분류

URL fetch 완료 후, 포스트의 마케팅 퍼널 위치를 판단한다.
에이전트 시스템 프롬프트(content-distribution.md)의 "퍼널 포지셔닝" 섹션 기준을 따른다.

**판단 기준**:
- **TOFU (인지)**: HR 트렌드, 업계 동향, 개념 설명, "~란?", "~이란 무엇인가" — 카테고리가 "HR 인사이트"이면서 개념 소개형
- **MOFU (고려)**: 방법론, 비교 분석, 가이드, 체크리스트, "~하는 방법", "~전략" — 카테고리가 "HR 인사이트"이면서 실전 적용형
- **BOFU (전환)**: 고객 사례, 성과 수치, 서비스 활용법, 인터뷰 — 카테고리가 "케이스 스터디" 또는 "브랜디드"

**출력**:
```
🎯 퍼널 분류: [TOFU/MOFU/BOFU]
판단 근거: [1줄 요약]
```

이 분류 결과는 4단계 채널 콘텐츠 생성 시 각 채널의 톤·CTA를 자동 조정하는 데 사용된다.

### 2단계: SEO/GEO 분석 출력

seo-geo-optimization SKILL.md의 키워드 추출 프로토콜(3단계)을 따라 분석 후, 다음 형식으로 출력 (사용자 확인용):

```
📊 SEO/GEO 분석 — [포스트 제목]
원본 URL: [URL]
퍼널: [TOFU/MOFU/BOFU]

주 키워드: [1개]
부 키워드: [3~5개]
롱테일 키워드: [2~3개]
네이버 태그: [10~15개]
GEO 핵심 엔티티: [3~5개]
FAQ 제안:
  Q: ...
  Q: ...
  Q: ...
```

### 2.5단계: 키워드 확인

SEO/GEO 분석 출력 후 사용자에게 안내:
"키워드 수정이 필요하면 알려주세요. 없으면 '계속' 또는 바로 다음 메시지를 보내주세요."

- **수정 요청 시** → 키워드 조정 후 재출력 → 다시 확인 대기
- **확인/계속 시** → 3단계로 진행

### 3단계: output 폴더 생성

`output/YYYY-MM-DD_[slug]/` 폴더 생성 전 중복 확인:

- **동일 경로 폴더가 이미 존재하면** → 사용자에게 안내:
  "이미 같은 날짜/slug로 생성된 콘텐츠가 있습니다. 덮어쓸까요, 아니면 `-v2` 접미어를 붙일까요?"
  - **덮어쓰기** → 기존 폴더 내용 유지한 채 파일 덮어쓰기로 진행
  - **버전 추가** → `output/YYYY-MM-DD_[slug]-v2/` 생성 (이미 v2 존재 시 v3, v4...)
- **존재하지 않으면** → 폴더 생성 후 진행

### 4단계: 채널 콘텐츠 생성

채널 선택에 따라(전체 또는 지정 채널) 콘텐츠 생성.
channel-adaptation SKILL.md의 채널별 규칙을 적용하여 각 채널 콘텐츠 작성.

**Task 병렬 실행 시 각 Task에 전달할 컨텍스트**:
- 원본 제목, 본문 전체, 원본 URL
- 1.5단계에서 판단한 퍼널 분류 (TOFU/MOFU/BOFU) + 해당 퍼널의 채널별 톤·CTA 조정 지침
- 2단계에서 추출한 SEO/GEO 키워드 패키지 (주/부/롱테일 키워드, 네이버 태그, GEO 엔티티, FAQ)
- 해당 채널의 channel-adaptation 규칙 (해당 채널 섹션만 발췌하여 전달)
- 해당 채널의 타겟 페르소나 (에이전트 시스템 프롬프트에서 발췌)
- 브랜드 가이드: 서치라이트AI, blog.searchright.net

**채널 목록** (전체 실행 시):
1. LinkedIn
2. 뉴스레터
3. 네이버 블로그
4. 브런치
5. 페이스북
6. 블라인드

**실패 처리**:
- 단일 채널 실패 시 → 실패 채널만 재실행 (성공한 채널 결과는 유지)
- 동일 채널 2회 실패 시 → 해당 채널을 건너뛰고 완료 요약에 실패 표시

### 5단계: 파일 저장

각 채널 콘텐츠를 아래 파일명으로 저장 (생성된 채널만):

```
output/YYYY-MM-DD_[slug]/01_linkedin.md
output/YYYY-MM-DD_[slug]/02_newsletter.md
output/YYYY-MM-DD_[slug]/03_naver-blog.md
output/YYYY-MM-DD_[slug]/04_brunch.md
output/YYYY-MM-DD_[slug]/05_facebook.md
output/YYYY-MM-DD_[slug]/06_blind.md
```

### 6단계: 완료 요약 출력 + 히스토리 기록

**완료 요약 출력**:

```
✅ 배포 콘텐츠 생성 완료

원본: [포스트 제목]
저장 위치: output/YYYY-MM-DD_[slug]/

생성된 파일:
  01_linkedin.md     — [자 수]자
  02_newsletter.md   — 제목: [제목]
  03_naver-blog.md   — [자 수]자, 태그 [N]개
  04_brunch.md       — [자 수]자
  05_facebook.md     — [자 수]자
  06_blind.md        — [자 수]자

다음 액션:
  • LinkedIn, 페이스북: 직접 복붙하여 발행
  • 뉴스레터: Stibee/Mailchimp에서 [이름] 변수 교체 후 발송
  • 네이버 블로그: 이미지 삽입 위치 확인 후 발행
  • 브런치: 발행
  • 블라인드: HR/경력 카테고리 선택 후 발행
```

- 선택적 채널 실행 시 → 생성된 채널만 표시
- 실패한 채널이 있으면 → `⚠️ [채널명]: 생성 실패 (수동 작성 필요)` 추가

**히스토리 기록**:

`output/_history.md`에 새 행 자동 추가. 파일이 없으면 헤더 포함하여 생성:

```markdown
# 배포 히스토리

| 날짜 | 원본 제목 | slug | 생성 채널 | 비고 |
|------|-----------|------|-----------|------|
| YYYY-MM-DD | [제목] | [slug] | 전체(6개) 또는 채널명 나열 | |
```

## 검증 체크리스트 (저장 전 자가 확인)

- [ ] 에이전트 + 스킬 파일 3개를 실제로 Read 했는가?
- [ ] 원본 URL 실제 fetch 여부 (추정 작성 절대 금지)
- [ ] 퍼널 분류(TOFU/MOFU/BOFU) 수행 여부
- [ ] 퍼널에 따른 톤·CTA 조정이 채널별로 반영되었는가?
- [ ] 각 채널의 타겟 페르소나 관점에서 콘텐츠가 작성되었는가?
- [ ] 모든 채널에 백링크(blog.searchright.net) 포함
- [ ] 브랜드명 "서치라이트AI" 표기 통일
- [ ] LinkedIn: 1,000~1,300자, 외부링크 본문 마지막
- [ ] 뉴스레터: 제목 30자 이내, 본문 400~600자
- [ ] 네이버 블로그: 1,500자 이상, 태그 10개 이상
- [ ] 블라인드: 첫 문장 회사명 없음, 홍보 어조 없음
- [ ] 각 채널의 Good 예시와 톤이 유사한가? Bad 예시의 패턴에 해당하지 않는가?
