# doe-assistant — 빌드 작업장

이 프로젝트는 비전문 동료(공정 엔지니어 등)를 위한 **DOE 어시스턴트 스킬**을 빌드하는 작업장이다. Claude Code 우선, Codex 포팅은 thesis 검증 후 결정.

핸드오프: `calm-crunching-kitten.md` (이 문서가 모든 설계 결정의 reference)

## 정체성

이 프로젝트의 산출물은 *실험 결과 동행 해설 어시스턴트*다. 검사관·게이트키퍼가 아니다. 핵심 thesis:

> "동료가 ChatGPT에 던지면 받지 못하는 네 가지 — 통계 어휘의 물리·공정적 의미 풀이, 이전 분석 컨텍스트 인용, 진단 결과의 의미 해석, 근거 있는 다음 스텝 넛지 — 를 한 자리에서 동행하는 어시스턴트."

## 폴더 구조

```
doe-assistant/
├── CLAUDE.md                       # 이 문서 (빌드 지침)
├── calm-crunching-kitten.md        # 핸드오프 (보존)
└── doe-assistant.skill/            # 산출물
    ├── SKILL.md                    # 메인 스킬 진입점
    └── references/
        ├── diagnostics_guide.md    # 진단 4종 어휘·풀이·약화 톤
        ├── interpretation_sop.md   # 통계 어휘 ↔ 물리 의미 매핑
        ├── bakeoff_selfcheck.md    # 자가검증 메모 작성법
        ├── domain_research.md      # Sub-agent A·B·C 프롬프트 ★
        └── report_template.md      # 보고서 골격
```

## 빌드 단계 (현재 위치 표시)

핸드오프 §7의 단계 게이트를 따른다.

| Step | 내용 | 통과 기준 | 상태 |
|---|---|---|---|
| 1 | 동료 1-2명에게 "최근 6개월 DOE 빈도" 인터뷰 | 월 1회 이상 | ⬜ 미실행 |
| 2 | Bake-off 채점 (공개 데이터셋 3-5개) | ChatGPT 평균 4/10 이하 | ⬜ 미실행 |
| 3 | factorial PoC 단일 스킬 구축 | reference 결과와 수치 일치 | 🟡 v0.1 스케치 완료 |
| 4 | 동료 사용자 테스트 5건 | "ChatGPT 대신 쓸 의향" 70%↑ | ⬜ 미실행 |
| 5 | 확장 (RSM 등) 또는 thesis 재검토 | — | ⬜ |

**현재 위치**: Step 3 v0.1 — SKILL.md + references 5종 골격 작성 완료. Python scripts·데이터셋·동료 인터뷰 미진행.

## 우선순위

핸드오프의 명시적 결정에 따라:

1. **Step 1 동료 인터뷰가 가장 큰 리스크 거름**. 빌드 더 진행하기 전에 *"동료가 실제로 DOE를 월 1회 이상 하는가?"* 확인. 답이 NO면 thesis 재검토.
2. Step 2 bake-off는 PoC 코드 진행과 병행 가능 — bake-off 데이터셋 3-5개 정해서 ChatGPT/Claude.ai 단순 프롬프트 결과를 채점표로 정리.
3. Python scripts 작성은 SKILL.md의 톤 규칙(어시스턴트 모드 + 자유 추론 보장)과 충돌하지 않도록 *얇게*. 강제 SOP 코드보다 LLM이 그때그때 statsmodels를 부르는 형태가 어울림.

## 톤 규칙 (스킬 안에서뿐 아니라 빌드 결정에도 적용)

LLM 도구 설계에서 강한 SOP 게이트·차단형 검증보다 **자유 추론을 보장하는 어시스턴트 톤**이 성능을 더 잘 살린다는 것이 이 프로젝트의 출발점. 빌드 결정 시 다음을 지킨다.

- 검증을 *차단*이 아니라 *해설로 동행*하는 형태로 구현
- 모든 통계 어휘에 한 줄 물리·공정적 의미 동반 — `references/interpretation_sop.md` 매핑 확장
- 진단 4종은 항상 실행하고 *의미*를 풀어 결론 어조를 조절 (게이트 아님)
- 다음 스텝 넛지엔 근거 + 대안 시나리오를 함께
- 이전 분석 컨텍스트는 `.doe-meta.yml`로 영속 — 매 보고서 첫 단락에서 인용

## Codex 포팅 — 추후 결정

Step 4(동료 사용자 테스트) 통과 후 Codex CLI로 옮길지 결정한다. 옮길 때:

- SKILL.md 워크플로우는 도구 무관 명세이므로 그대로 활용
- references/는 그대로 활용
- 멀티에이전트 부분(Sub-agent A·B·C)은 Codex의 sub-agent 호출 방식에 맞춰 어댑팅

지금은 Claude Code 우선. Codex는 thesis 검증된 후.

## 라우팅

이 프로젝트 안에서:

| 요청 | 처리 |
|---|---|
| 동료가 DOE csv를 들고 와서 분석 요청 | `doe-assistant.skill` 호출 |
| 자유 EDA · 일반 trend · 일반 stat | "Claude.ai 채팅이 더 빠르실 거예요" 한 줄 안내 |
| 빌드 자체에 대한 결정 (구조 변경·확장 등) | 이 CLAUDE.md 갱신, 핸드오프 참조 |
| Step 1 동료 인터뷰 스크립트 작성 | `moai-cowork:operations-hr` (인터뷰 가이드) |
| Bake-off 데이터셋 선정·채점표 만들기 | `moai-data:data-explorer` + 직접 작업 |
| Python scripts 작성 시점이 오면 | `statsmodels` / `pingouin` 직접 활용, scripts/ 디렉토리 추가 |

## 향후 확장 후보 (Step 5 통과 후)

핸드오프 §6 "1차에서 명시적 제외" 항목들:
- RSM (Response Surface Methodology)
- Mixture design
- Taguchi
- Plackett-Burman
- Power analysis · 사전 design 생성

각 확장은 동료 워크플로우와 맞는지 Step 4에서 같이 확인 후.

## 참조

- 핸드오프 (모든 설계 결정의 출발점): `calm-crunching-kitten.md`
- 메인 스킬: `doe-assistant.skill/SKILL.md`
- thesis 보호 메커니즘: `doe-assistant.skill/references/bakeoff_selfcheck.md`
- 차별화 핵심 모듈: `doe-assistant.skill/references/domain_research.md` (Sub-agent A·B·C)
