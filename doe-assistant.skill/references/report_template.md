# 보고서 골격 — `reports/{YYYY-MM-DD}-{exp-name}.md`

이 템플릿은 *어시스턴트 톤*으로 작성된다. 검사관 톤·지시 톤 회피. 동료가 어깨 너머에서 같이 보는 분석가의 글이다.

---

## 골격

```markdown
# {실험명} 분석 보고서

> 일시: {YYYY-MM-DD}
> 동료: {이름}
> Design: {2^3 full factorial + 3 cp / fractional / RSM 등}
> Project: {.doe-meta.yml의 project 필드}

## 1. 이전 분석 컨텍스트
[.doe-meta.yml의 history에서 직전 1-2건 인용]
"지난번 {YYYY-MM-DD} 라운드에선 {요약 1줄}. 그래서 이번엔 {이번 결정의 동기 1줄}."

(이전 분석이 없으면) "이 프로젝트의 첫 라운드입니다."

## 2. 데이터·설계
- factor: {각 factor 이름·단위·수준 — .doe-meta.yml에서 인용}
- response: {이름·단위·방향(maximize/minimize)}
- run 수: {N}
- center points: {0 또는 N}
- randomization 흔적: {있음/없음/모호}

## 3. 결과

### 3-1. ANOVA
[표 — Source · DF · SS · MS · F · p]

[표 직후 한 단락 — 신호 강도 풀이]
"가장 강한 신호는 온도 (F=45.2 → 잡음 대비 45배 강함, p=0.0003).
다음은 압력 (F=12.1 → 12배), 그 다음이 온도×압력 상호작용 (F=5.3)."

### 3-2. Main effect
[표 — factor · effect · 95% CI · η²]

[표 직후 한 단락 — 물리적 의미]
"온도를 150°C에서 200°C로 올리면 식각률이 평균 +120 Å/min 증가.
효과 크기 η²=0.65로 response 변동의 65%가 온도에서 옴 — 가장 dominant factor."

### 3-3. Interaction effect
[표 + interaction plot 인용]

[한 단락 — 어떤 조합에서 효과가 어떻게 달라지는지 풀이]
"온도×압력 상호작용은 +25 Å/min로 유의(p=0.03).
저압(10 mTorr) 조건에서 온도 효과는 +135, 고압(30 mTorr)에선 +105.
저압일 때 온도 영향력이 더 크다는 뜻 — chemical etching이 dominant인
영역에서 이런 패턴이 자주 나옵니다."

### 3-4. 플롯
- Pareto chart → `figs/pareto.png`
- Main effect plot → `figs/main_effect.png`
- Interaction plot → `figs/interaction.png`

## 4. 진단 + 의미 해석

| 진단 | 결과 | 의미 |
|---|---|---|
| Shapiro-Wilk on residuals | W=0.94, p=0.31 ✓ | 잔차 정규성 통과 — p-value 신뢰 가능 |
| Residual vs fitted | 패턴 없음 ✓ | 선형 모델로 충분 |
| Levene's test | p=0.42 ✓ | 조건별 분산 안정 |
| Lack of fit (cp) | F=1.2, p=0.31 ✓ | 곡률 없음, 이 영역에선 선형 모델로 충분 |

[종합 한 줄]
"진단 4종 모두 통과 — 결론을 단정 톤으로 진술 가능."

(또는 약한 위반이 있다면)
"3/4 통과. Levene이 p=0.04로 약한 위반 — 고온·고압 조건에서 잡음이 컸다는
신호이고, 효과 방향성은 살아있지만 절대 수치는 보수적으로 받아들이는 게 안전."

## 5. 결론

[진단 강도에 따라 어조 조절]

(4/4 통과)
"이번 라운드에서 다음을 확인했습니다:
- 온도가 식각률의 dominant factor (effect 변동의 65%)
- 압력은 약한 음의 효과 (영역 좁힘 효과 인정 가능)
- 온도×압력 상호작용 유의, 저압에서 온도 효과 더 큼"

(2-3/4 통과)
"이번 라운드의 잠정 결론:
- ...
- ...
진단 일부가 약해 절대 수치보다 효과 방향성과 dominance 순서가
더 신뢰할 수 있습니다. 양산 결정 전 반복 실험 권장."

## 6. 다음 스텝 — 근거 + 대안

[단정형 회피, 시나리오 분기]

"이대로 가면:
- **자연스러운 다음 단계** — 압력 영역(10-15 mTorr)으로 좁히고 RSM(Box-Behnken)으로 곡률 잡기. 17 run.
- **만약 RF 파워의 비선형이 의심된다면** — RF에 center points 늘린 추가 2³ + 5 cp factorial 권장. 13 run.
- **빠르게 양산 검증 필요하면** — 현 best 조건(200°C, 10 mTorr, 1000W) replicate 5회로 robustness 확인."

## 7. Bake-off 자가검증 메모

[references/bakeoff_selfcheck.md 형식대로 작성]

이 분석에서 ChatGPT 단순 프롬프트가 빠뜨릴 가능성이 높은 항목:
- ...
- ...
- ...

---

(옵션, 도메인 리서치 멀티에이전트가 호출됐다면)
> 자세한 도메인 메커니즘·다음 설계 비교는 `{report-name}.appendix.md` 참고.
```

---

## 톤 체크리스트 (작성자가 마지막에 점검)

보고서를 닫기 전에 이 항목들을 빠르게 훑는다:

- [ ] 모든 통계 어휘에 한 줄 풀이가 붙었나
- [ ] 첫 단락에 이전 분석 인용이 있나 (있는 경우)
- [ ] 진단 결과가 의미 해석과 함께 보고됐나
- [ ] 결론 어조가 진단 강도에 맞춰 조절됐나
- [ ] 다음 스텝이 근거 + 대안 시나리오로 제시됐나
- [ ] Bake-off 자가검증이 3개 이상 항목을 채웠나

체크리스트는 *습관 reminder*이지 강제 게이트가 아니다. 빠진 항목이 있으면 동료가 정보를 못 받는다는 뜻이라 — 채우고 닫는다.
