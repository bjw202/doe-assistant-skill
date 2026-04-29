# 도메인 리서치 멀티에이전트 — Sub-agent A · B · C 프롬프트

## 왜 옵션인가

매 분석마다 자동 호출하면 비용·시간·과잉이 된다. 더 중요하게는, 일상 분석엔 동료가 도메인 시각을 이미 갖고 있을 때가 많다. 도메인 리서치가 진짜 가치 있는 순간은:

- **새 공정·새 재료**라 팀 안에 도메인 시각이 부족할 때
- 결과 패턴이 *예상과 다르거나* 익숙하지 않을 때
- 다음 DOE 방향을 잡을 때 *외부 사례·메커니즘 후보*가 필요할 때
- 동료가 직접 *"새 시각이 필요해"* 또는 *"`/research`"* 트리거할 때

이 옵션 모듈이 **ChatGPT가 한 채팅창에서 절대 흉내 못 내는 차별화 축**이다. 멀티에이전트로 병렬 실행하고, 결과를 보고서 부록으로 통합한다.

## 호출 시점

5단계 메인 파이프라인 *완료 후*. 보고서 본문은 이미 작성돼 있고, 부록(`{report-name}.appendix.md`)에 추가하는 형태.

```
[메인 파이프라인 완료] → 동료 트리거 ("/research" 또는 "도메인 시각 필요해")
  → Sub-agent A · B · C 병렬 실행
  → 결과 통합 → 부록 작성 → 본문 결론 절 보강 (필요 시)
```

## Sub-agent A — 통계 재검증

### 역할
ANOVA 결과·효과 크기·진단 수치를 *다른 방식*으로 재계산해 메인 파이프라인의 수치를 더블체크.

### 입력 (메인 파이프라인에서 전달)
- 분석에 사용한 csv 경로 또는 데이터프레임
- 메인 파이프라인이 산출한 ANOVA 표 / main effect / interaction effect / η² / 진단 4종 수치

### 작업
1. statsmodels로 분석한 결과면 — pingouin 또는 직접 수동 계산으로 재현
2. F·p·η²·CI 값을 비교
3. lack-of-fit·Levene 같은 진단을 다른 라이브러리/구현으로 재실행
4. 일치/불일치 정리

### 출력 (1-2 단락)
- 일치 수치 (간략히)
- 불일치 수치가 있으면 — *"η² 계산식이 type I vs type III에서 차이"* 같은 원인 가설
- 산업 표준(JMP / Minitab) 대비 *예상되는* 차이가 있다면 짚어주기

### 프롬프트 템플릿

```
You are Sub-agent A: statistical re-verification.

Inputs:
- Data: {csv path}
- Pipeline results: {ANOVA table, effects, diagnostics, all numerics}

Task:
1. Re-run the analysis using a different library or manual calculation
   (statsmodels ↔ pingouin ↔ scipy ↔ manual sum-of-squares).
2. Re-run all four diagnostics independently.
3. Compare every numeric to the pipeline's output.

Output (1-2 paragraphs):
- Numeric agreements (brief).
- Any disagreements with hypothesized causes (sum-of-squares type I vs III,
  variance estimation differences, etc.).
- Industry-standard expectations (JMP/Minitab) where they differ from Python.

Tone: neutral, brief. This is a sanity check, not a takedown.
```

---

## Sub-agent B — 도메인 리서치 (★ 핵심)

### 역할
공정 도메인의 메커니즘·트레이드오프·유사 사례를 웹에서 찾아 결과 패턴을 해석할 외부 시각을 제공한다.

### 입력
- factor 이름·단위·수준 (`.doe-meta.yml`)
- 공정 종류 (식각·증착·합성·믹싱·열처리 등 — 인테이크 단계에서 분류)
- 발견된 effect 패턴 (어느 factor가 dominant, 어떤 interaction이 유의)
- 동료가 의문을 가진 부분 (선택)

### 작업
1. 공정 + factor 조합으로 메커니즘 후보 검색
2. 알려진 트레이드오프(예: 식각률↑ vs uniformity↓)
3. 유사 공정·재료에서의 사례
4. 인용 가능한 출처 3-5개 (학술·산업 논문·교과서·신뢰 가능한 기술 블로그)

### 출력 (보고서 부록 1-2 페이지)
- **메커니즘 가설** 2-3개. 각각 *"X가 Y인 이유는 Z 메커니즘 때문일 가능성"* 형태.
- **알려진 트레이드오프** — 이번 결과 해석에 영향을 줄 수 있는 것
- **유사 사례** 1-2건 — 다른 재료·다른 공정에서의 비슷한 패턴
- **출처** — 각 가설마다 인용 (URL 또는 서지 정보)

### 프롬프트 템플릿

```
You are Sub-agent B: domain research with web search.

Context:
- Process domain: {etching / deposition / synthesis / mixing / etc.}
- Factors and levels: {from .doe-meta.yml}
- Observed effect pattern: {dominant factors, key interactions, sign and magnitude}
- Optional question from colleague: {free text}

Task:
1. Search the web for mechanism candidates that could explain the observed
   effect pattern in this specific process domain.
2. Identify known trade-offs in this domain that could affect interpretation.
3. Find 1-2 analogous cases in similar materials or processes.
4. Cite 3-5 reliable sources (academic, industrial, technical reports;
   avoid forum posts unless authoritative).

Output (1-2 page appendix in Korean):
- 2-3 mechanism hypotheses, each with a one-paragraph explanation linking
  it to the observed effect pattern.
- Known trade-offs relevant to this analysis.
- 1-2 analogous cases (brief).
- Source list with URL or full citation.

Tone: domain expert briefing a colleague. Concrete, mechanism-focused,
not "general chemistry textbook."

Constraints:
- Do not speculate beyond what sources support.
- Mark hypotheses as hypotheses, not facts.
- If web search returns nothing useful, say so honestly.
```

### 출력 예시 (식각 공정 케이스)

```markdown
## 도메인 리서치 부록 (Sub-agent B)

### 메커니즘 가설

**가설 1. Ion-bombardment vs chemical etching balance**
온도가 dominant factor로 잡힌 건 식각이 chemical-dominated 영역에 있다는
신호일 수 있습니다. 저압(10 mTorr)에선 ion mean free path가 길어져
bombardment 비중이 커지지만, 이번 데이터의 압력 effect가 -45 Å/min로
온도 대비 작은 건 온도 영역(150-200°C)이 chemical activation에
더 민감한 구간이기 때문일 가능성. (참고: Coburn-Winters synergy model)

**가설 2. ...**

### 알려진 트레이드오프
- 식각률↑은 보통 sidewall slope·uniformity↓와 짝을 이룸
- ...

### 유사 사례
- TSMC 7nm Si etch optimization (2019) — 동일 영역에서 RSM으로 +15% 수율
- ...

### 출처
1. Coburn, J. W., & Winters, H. F. (1979). Plasma etching... J. Vac. Sci. Technol.
2. ...
```

이게 *진짜* 차별화다. ChatGPT 채팅 한 번으론 절대 이 깊이로 못 간다.

---

## Sub-agent C — 다음 DOE 설계

### 역할
이번 결과 + Sub-agent A의 검증 + Sub-agent B의 도메인 인사이트를 종합해 다음 실험 후보 1-2개를 *근거와 함께* 권장.

### 입력
- 메인 파이프라인의 결과 + 진단
- Sub-agent A의 검증 결과
- Sub-agent B의 메커니즘 가설·트레이드오프
- (선택) 동료의 다음 라운드 제약 (예산·시간·factor 선호)

### 작업
1. 결과 패턴(어느 factor dominant, lack-of-fit 유무, 잔차 패턴)에서 자연스러운 다음 design 도출
2. 도메인 가설이 시사하는 추가 factor 또는 영역 좁힘
3. 1-2개 후보를 비교 형태로 제시 (RSM vs fractional vs Plackett-Burman 등)

### 출력
- 후보 design 1-2개. 각각:
  - design 유형 (RSM·Box-Behnken·CCD·fractional·추가 factorial 등)
  - factor 이름·새 수준·run 수
  - 근거 (왜 이 design이 자연스러운가, 어느 가설을 검증하는가)
  - 한계 (어느 가설은 못 보는가)
- 비교 표 — *"빠르게 답 내려면 A, 곡선 잡으려면 B"*

### 프롬프트 템플릿

```
You are Sub-agent C: next-DOE designer.

Context:
- This round's results + diagnostics: {from pipeline}
- Statistical re-verification: {from Sub-agent A}
- Domain mechanisms and trade-offs: {from Sub-agent B}
- Colleague's constraints (optional): {budget, time, factor preferences}

Task:
1. Derive 1-2 natural next-design candidates from the observed pattern.
2. Incorporate domain hypotheses — if Sub-agent B suggested a candidate
   mechanism, check whether the next design tests it.
3. Present candidates with rationale + limitations + run count.

Output (1 page in Korean):
- For each candidate: design type, factors and new levels, run count,
  rationale, what it tests, what it does not test.
- Comparison table.
- A clear "if X, then candidate A; if Y, then candidate B" framing
  rather than a single recommendation.

Tone: assistant suggesting paths, not commanding. Always offer alternatives.
```

---

## 통합 — 부록 작성 규칙

세 sub-agent의 결과를 `{report-name}.appendix.md`로 묶는다. 구조:

```markdown
# 부록 — 도메인 리서치 멀티에이전트 결과

> 트리거: {동료가 호출한 이유 한 줄}
> 실행 시각: {timestamp}

## A. 통계 재검증
{Sub-agent A 출력}

## B. 도메인 리서치
{Sub-agent B 출력}

## C. 다음 DOE 설계 후보
{Sub-agent C 출력}
```

**본문 보고서의 결론·다음 스텝 절도 갱신**한다. 부록의 핵심 인사이트(메커니즘 가설, 권장 design)를 본문 결론에 한 줄로 묶고, *"자세한 근거는 부록 참고"* 로 잇는다. 이렇게 해야 다른 사람이 본문만 읽어도 의사결정에 필요한 정보를 받는다.

### 본문 갱신 규칙 — 부록 핵심을 본문에 묶기

부록만 만들고 끝내면 본문만 읽는 동료가 깊이 있는 정보를 놓친다. 다음 4곳을 갱신한다.

**1. 본문 메타데이터 (최상단)**

부록 파일이 생기면 헤더 직후에 한 줄 추가:

```
> 부록: {report-name}.appendix.md (도메인 리서치 멀티에이전트)
```

**2. 본문 §5 결론 절 끝에 한 단락 추가**

```
**부록 메커니즘 노트** — {Sub-agent B의 핵심 가설 1줄}.
이번 결과 패턴({어느 진단·어느 effect})과 일치하는 메커니즘이
부록 §B에서 제시됐고, 이는 결론의 물리적 해석을 보강합니다.
자세한 근거·인용은 부록 §B 참고.
```

**3. 본문 §6 다음 스텝 절 끝에 한 단락 추가**

```
**부록 권장 design** — Sub-agent C가 비교한 후보:
({후보 A 1줄}, {후보 B 1줄}). 위 자연스러운 다음 단계와의
일치/보완 관계는 부록 §C 비교 표 참고.
```

**4. 본문 §7 Bake-off 자가검증 메모에 한 항목 추가**

```
- 도메인 리서치 멀티에이전트 부록 — Sub-agent B의 메커니즘 가설
  ({핵심 1줄})이 {진단 패턴}과 일치하는 식의 *물리적 근거 보강*.
  ChatGPT 단순 프롬프트로는 한 채팅창에서 이 깊이의 도메인 인용·
  메커니즘 가설·다음 design 비교를 동시에 도달하기 어려움.
```

**갱신하지 않는 절**

- §1 이전 분석 컨텍스트 — 부록은 *이번 라운드의 깊이 보강*이지 이전 컨텍스트가 아님
- §2 데이터·설계 — 사실 정보, 부록과 무관
- §3 결과 / §4 진단 — 수치는 그대로, 해석은 §5에서 갱신

이 규칙을 따르면 본문만 읽어도 의사결정 핵심을 받고, 부록은 *깊이*가 필요한 사람이 더 들어가는 자연스러운 layering이 된다.

## 호출 도구

Claude Code 환경에서는 **Task 도구로 sub-agent 병렬 실행**. 각 sub-agent를 별도 Task로 띄우고 결과를 모은다. Sub-agent B는 WebSearch / WebFetch 도구가 필요하므로 권한 확인 후 진행.

Codex 포팅 시: Codex의 sub-agent 호출 방식에 맞춰 같은 프롬프트 템플릿을 어댑팅. 톤·역할은 동일.
