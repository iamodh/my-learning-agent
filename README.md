# my-learning-agent

대화 기반으로 학습한 내용을 문서화하는 **학습 에이전트**, 그리고 그
에이전트를 평가하기 위한 테스트 케이스 하네스.

## 학습 방식: 탑다운(top-down)

세부부터 쌓아 올리지 않고, 상위 개념에서 출발해 하위 개념으로 내려가며
**지식을 이해하는 탑다운 학습**을 따른다. 그래서 `spec.json`도 상위
개념(`등장_상위_개념`)을 먼저 잡고 그 맥락 안에서만 의미를 갖는 하위
개념(`등장_하위_개념`)을 매다는 구조로 설계돼 있으며, 문서화 결과 역시
상위 개념 문서 아래 하위 섹션이 참조로 연결되는 형태로 채점한다.

`generate_prompt.txt`로 합성한 대화(`dialogue.txt`)를 문서화 에이전트에
입력하고, 그 출력을 `spec.json`의 정답지와 대조해 채점한다. 에이전트
프롬프트를 버전별(`v1`/`v2`/`v3`)로 비교 평가하는 것이 목적이다.

## 파이프라인

```
generate_prompt.txt  →  (별도 LLM 세션)  →  dialogue.txt
                                                 ↓
                                         학습 에이전트
                                                 ↓
                    v1_output / v2_output / v3_output
                                                 ←→  spec.json(정답지)로 채점
```

## 폴더 구조

```
prompts/
  v1.md                  # 학습 에이전트 프롬프트 (v1_output 채점에 사용)
  v2.md                  # 개선 버전 (v2_output 채점에 사용)
  v3.md                  # v2 피드백 반영 버전 (v3_output 채점에 사용)
  v1_feedback.md         # v2 작성용 결함 피드백
  v2_feedback.md         # v3 작성용 결함 피드백
  v3_feedback.md         # v4 작성용 결함 피드백
test_cases/
  case_NN/
    spec.json            # 생성 명세 + 채점용 정답지
    generate_prompt.txt  # 대화 합성용 LLM 프롬프트
    dialogue.txt         # 합성된 대화
    v1_output/
      agent_output/      # 에이전트 출력 (채점 대상)
      score.json         # 채점 결과
    v2_output/
      agent_output/
      score.json
    v3_output/
      agent_output/
      score.json
```

`prompts/vN.md`가 `dialogue.txt`를 입력받아 해당 케이스의
`vN_output/agent_output/`을 생성하는 학습 에이전트 프롬프트다.

케이스 번호는 두 자리 0 패딩(`case_01` …). `spec.json`/정답지의 한국어
필드명(`생성_명세`, `정답지`, `예상_결과_문서` 등)은 채점 LLM이 한국어
키로 채점하도록 한 의도된 설계다.

## 케이스 생성 워크플로우

새 케이스의 `spec.json` + `generate_prompt.txt`를 붙여넣으면 Claude Code가
[`.claude/claude.md`](.claude/claude.md)의 규칙에 따라 폴더 구조와 입력
파일을 일괄 생성한다.

## 작업 도구 이력

- 케이스 생성, v1 실행·채점, v2 프롬프트 작성, v2 결과 생성까지는
  Claude Code를 사용했다.
- v2 결과 채점부터 `v3_feedback.md`까지(v2 채점, `v2_feedback.md`,
  `v3.md`, v3 결과 생성, v3 채점, `v3_feedback.md`)는 1차로 Codex로
  진행했다.
- 이후 평가자를 Claude로 통일하기 위해 v2 채점부터 `v3_feedback.md`까지
  다시 Claude로 수행했다. v2는 10개 케이스 전부 재채점, v3는 1차로 v2
  점수 하위 3개(case_02·05·09)만 재실행·재채점했다.
- v3의 나머지 케이스도 Claude로 통일하기 위해 추가 작업을 진행 중이다.
  case_01·03·04는 Claude로 재실행·재채점을 완료했고(셋 다 FAIL,
  `v3_feedback.md`의 D1 지식 분리 과다 회귀 재현), case_06·07·08·10은
  재실행만 완료된 상태로 채점 대기다.
- 평가 오염을 줄이기 위해 결과 생성과 채점은 각 케이스별 고립 서브에이전트로
  수행했다. 생성 단계에는 `prompts/vN.md`와 `dialogue.txt`만 제공하고,
  채점 단계에는 `spec.json`과 해당 `agent_output/`만 제공했다.
- 권한 시스템상 서브에이전트의 직접 파일 쓰기가 거부되는 경우가 잦아,
  서브에이전트는 펜스 블록으로 내용만 반환하고 메인이 받아 파일에
  기록하는 패턴으로 운영했다(자세한 내용은 [`.claude/CLAUDE.md`](.claude/CLAUDE.md)
  의 *서브에이전트 출력 회수* 절 참고).
