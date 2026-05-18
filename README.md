# my-learning-agent

대화 기반으로 학습한 내용을 문서화하는 **학습 에이전트**, 그리고 그
에이전트를 평가하기 위한 테스트 케이스 하네스.

`generate_prompt.txt`로 합성한 대화(`dialogue.txt`)를 문서화 에이전트에
입력하고, 그 출력을 `spec.json`의 정답지와 대조해 채점한다. 에이전트
프롬프트를 버전별(`v1`/`v2`)로 비교 평가하는 것이 목적이다.

## 파이프라인

```
generate_prompt.txt  →  (별도 LLM 세션)  →  dialogue.txt
                                                 ↓
                                         학습 에이전트
                                                 ↓
                         v1_output / v2_output  ←→  spec.json(정답지)로 채점
```

## 폴더 구조

```
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
```

케이스 번호는 두 자리 0 패딩(`case_01` …). `spec.json`/정답지의 한국어
필드명(`생성_명세`, `정답지`, `예상_결과_문서` 등)은 채점 LLM이 한국어
키로 채점하도록 한 의도된 설계다.

## 케이스 생성 워크플로우

새 케이스의 `spec.json` + `generate_prompt.txt`를 붙여넣으면 Claude Code가
[`.claude/claude.md`](.claude/claude.md)의 규칙에 따라 폴더 구조와 입력
파일을 일괄 생성한다.
