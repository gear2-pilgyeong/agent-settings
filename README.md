# agent-settings

AI 코딩 에이전트(Claude Code, Codex 등)에 쓰는 공용 지침과 스킬입니다.

## 구성

```
agent-settings/
├── AGENTS.md           # 프로젝트와 무관하게 적용하는 기본 지침
└── skills/
    ├── code-design/    # 설계 전 기존 구현·과거 시도 조사, 선택지 제시, 영향 보고
    ├── delegation/     # 서브에이전트 위임 여부, 위임 프롬프트, 모델 선택
    ├── implementation/ # 코드 작성·수정 원칙
    ├── testing/        # 테스트 전략과 작성 원칙
    └── commit/         # 커밋 메시지와 pull request 작성 형식
```

`AGENTS.md`는 각 작업 단계에서 위 스킬을 읽도록 지시하므로 함께 설치합니다.

## 설치

스킬은 [`npx skills`](https://github.com/vercel-labs/skills)로 설치합니다.

```bash
npx skills add gear2-pilgyeong/agent-settings -g            # 전체 설치
npx skills add gear2-pilgyeong/agent-settings -g -s testing # 일부만 설치
```

`AGENTS.md`는 사용하는 도구의 전역 지침 파일로 복사하거나 링크합니다.

| 도구 | 위치 |
|---|---|
| Claude Code | `~/.claude/CLAUDE.md` |
| Codex | `~/.codex/AGENTS.md` |
