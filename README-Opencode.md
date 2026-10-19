# OpenCode 프로젝트 하네스 운영

> 적용: 프로젝트 루트의 OpenCode V2 지침·설정 · 기준일: 2026-10-16 · 확인 CLI: opencode 2.0.30

공통 규칙은 [README.md](README.md), 설치·검토·되돌리기는 [OpenCode 설치 가이드](INSTALL_OPENCODE.md)를 따른다. CLI 바이너리·백그라운드 서비스·계정 인증은 사용자 환경의 사전 조건이다. 이 문서의 설치 대상은 프로젝트 파일이다.

## 1. 범위와 파일별 책임

| 범위 | 경로 | 책임 |
| --- | --- | --- |
| 프로젝트·필수 | `AGENTS.md` | Codex와 공유하는 시작 지침과 공통 규칙의 위치 |
| 프로젝트·필수 | `.opencode/opencode.json` | 이 프로젝트의 권한 규칙 |
| 프로젝트·선택 | 하위 `AGENTS.md` | 해당 디렉터리의 예외 규칙. 그 영역을 읽을 때 로드됨 |
| 프로젝트·선택 | `.opencode/skills/<name>/SKILL.md`, `.opencode/commands/` | 프로젝트의 반복 업무 |
| 사용자 | `~/.config/opencode/opencode.json`, `~/.config/opencode/AGENTS.md`, `cli.json`, 사용자 Skill·Plugin·MCP·인증 | 기존 상태 유지. 프로젝트 설치 대상으로 사용하지 않음 |
| 상위 경로 | 프로젝트 위 디렉터리의 `opencode.json`·`.opencode/` | 함께 병합될 수 있음. 존재와 영향만 확인하고 수정하지 않음 |

필수는 이 하네스의 도입 기준이다. OpenCode V2는 현재 디렉터리부터 파일시스템 루트까지 설정을 찾아 병합하고, 모든 `.opencode` 설정이 직접 둔 `opencode.json`보다 우선한다. 시스템(`/etc/opencode`) 설정과 정책은 프로젝트보다 우선한다. 프로젝트 파일의 존재와 실제 적용을 구분한다. [O2]

## 2. 프로젝트 진입 지침

OpenCode V2는 `AGENTS.md`만 지침 파일로 인식하고 `CLAUDE.md`를 대체 파일로 쓰지 않는다. 이 저장소의 루트 `AGENTS.md`는 Codex와 공유하므로 도구 공용 문장으로 유지한다. 기존 파일 전체를 덮어쓰거나 별도 수정 금지 지시를 무시하지 않는다. [O1]

```markdown
# 프로젝트 실행 지침

- 작업 시작 시 루트 README.md의 공통 운영 규칙과 사용 중인 도구의 운영 문서(Codex는 README-Codex.md, OpenCode는 README-Opencode.md)를 읽고 적용한다.
- 이번 작업의 요구사항·완료 조건과 관련 코드만 확인한다.
- 실제 검증 명령과 실행 위치는 이 프로젝트의 스크립트·CI·개발 가이드에서 확인한다.
- 프로젝트 경로와 적용 설정의 출처를 확인하고 사용자 기본값과 구분한다.
- 최소 변경 후 필요한 검사를 실행하고 결과·미검증 범위를 보고한다.
- 승인 경계와 작업 기록은 README.md를 따른다.
- 기존 사용자 변경을 보존하고 범위 밖 수정은 하지 않는다.
```

첫 줄만 Codex 전용 문장에서 공용 문장으로 바뀌고 나머지는 [Codex 진입 지침](README-Codex.md#2-프로젝트-진입-지침)과 같다. `AGENTS.md`가 없으면 이 공용 문장으로 새로 만들고, Codex 전용 문장으로 이미 있으면 첫 줄만 병합한다. 공유 파일이므로 만들거나 바꾼 뒤에는 설치 가이드에 따라 Codex 쪽 진입 지침 문서와 설치 판정도 함께 확인한다. OpenCode 전용 진입 파일을 `instructions`로 추가해 기존 지시를 우회하지 않는다. 사용자 전역 `AGENTS.md`도 함께 결합되며 OpenCode는 파일 사이의 충돌을 해결하지 않는다. [O1]

## 3. 프로젝트 설정

OpenCode를 선택하면 `.opencode/opencode.json`을 준비하고 [설치 가이드](INSTALL_OPENCODE.md#42-프로젝트-opencodejson-준비)의 최소 권한 설정을 병합한다. OpenCode의 기본 정책은 거의 모든 작업과 프로젝트 밖 경로 접근을 묻지 않고 허용한다. 이 하네스는 셸 명령과 프로젝트 밖 경로 접근을 `ask`로 두어 다른 도구의 승인 수준과 맞춘다. 기존 프로젝트 정책이 다르면 충돌과 변경 영향을 검토한다. [O3]

V2 필드 `permissions`를 사용하고 V1 필드(`permission`, `bash`, `task`)를 새로 쓰지 않는다. 이 파일 대신 `~/.config/opencode/opencode.json`을 수정하지 않는다. 시스템·전역 정책이 같은 작업을 차단하면 원인을 기록하고 우회하지 않는다. 프로젝트 설정만으로 인증·네트워크·전역 MCP·Plugin이 격리되지는 않는다. [O2][O3]

## 4. 프로젝트 검토와 일상 운영

1. 프로젝트 루트에서 `opencode debug config`로 설정 출처에 프로젝트 `.opencode/opencode.json`이 포함됐는지 확인한다. 출력에는 전역 설정의 비밀값이 포함될 수 있으므로 경로와 필요한 키만 선별해 기록한다.
2. 프로젝트 루트에서 새 `opencode` 세션을 시작한다. `--auto`처럼 승인을 건너뛰는 옵션을 쓰지 않는다.
3. 적용 지침, 현재 경로, 브랜치, 기존 변경과 실제 검증 명령을 확인한다. 지침 로딩은 모델의 자기 보고만으로 판정하지 않는다.
4. 작은 가역적 작업으로 셸 명령의 승인 요청 표시 → 프로젝트 지침 준수 → 관련 검사 → 자체 검토 → 작업일지 흐름을 검증한다. 검증 중 **Allow always**를 선택하지 않는다. 이 선택은 프로젝트 범위의 영구 허용 규칙을 저장한다.
5. 지침·설정 변경 후 `opencode reload` 또는 새 세션에서 다시 확인한다. 버전·인증·서비스 상태 확인만으로 완료를 판정하지 않는다.

필요한 Skill은 프로젝트 `.opencode/skills/`에 둔다. 사용자 `~/.config/opencode/skills/`와 상위 경로의 Skill은 범위를 구분한다. MCP·Plugin 등 확장은 별도 범위 검토 후 도입하며, 사용자 전역 설치를 이 프로젝트 설치에 포함시키지 않는다. [O4]

## 공식 근거

- [O1: 지침과 AGENTS.md](https://opencode.ai/v2/docs/instructions/)
- [O2: 설정 위치와 우선순위](https://opencode.ai/v2/docs/config/)
- [O3: 권한](https://opencode.ai/v2/docs/permissions/)
- [O4: Skill 경로](https://opencode.ai/v2/docs/skills/)
