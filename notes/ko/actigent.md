# Actigent — Phase 1

다양한 서비스의 활동을 수집할 Actigent의 B4A 기반 개발 환경입니다. 이번 단계에서는 수집 기능·API·스케줄러·마이그레이션을 구현하지 않습니다.

## 저장소와 기준 버전

- 원격: https://github.com/Blueprint4Agent/actigent — 공개 Fork.
- Upstream: https://github.com/Blueprint4Agent/B4FastAPI.git.
- 시작 커밋: `b354de05ccdd5567ff249be2a13142dc029bc395`.
- B4React 고정 커밋: `3caa104d49f5e09138a53c4b9a0960839e2927e9`.
- 로컬: `/Users/choi-hyk/blueprint4agent/actigent`.
- 원본 B4FastAPI 작업 트리는 수정하지 않으며 Git 이력과 라이선스를 보존합니다.

## 개발 환경

[영문 가이드의 재현 명령](../actigent.md#reproduce-development-setup)을 따릅니다. `make init`, `make install`로 환경을 구성합니다. 이번 단계는 기존 B4A UI와 테스트를 그대로 유지합니다. 브라우저 테스트에 B4A 고정 문구가 있으므로 리브랜딩과 공개 project.json 도입은 후속 작업으로 남깁니다. 필요한 경우 독립 프론트엔드 Fork에서 테스트와 브랜딩을 함께 관리합니다. 소스 패키지 이름과 프론트엔드 코드는 변경하지 않습니다. 로컬 SECRET_KEY는 Python secrets.token_urlsafe(48) 등으로 별도 생성하고 원본 자격 증명을 복사하지 않습니다.

현재 클론의 무시된 backend `.env`에는 `APP_NAME=Actigent API`, `APP_PORT=8100`, `DB_NAME=actigent.db`, `APP_BASE_URL=http://localhost:5180`과 localhost/127.0.0.1:5180 CORS가 설정되어 있습니다. 기존 Tauri origin은 유지합니다. frontend `.env`는 `VITE_API_BASE_URL=http://127.0.0.1:8100`을 사용합니다. 다른 머신에서도 초기화 후 이 값을 설정합니다. SQLite와 메모리 Redis를 사용하며 원본 DB·비밀키는 복사하지 않습니다. 현재 클론에는 독립 서명 키를 생성해 두었습니다.

별도 터미널에서 실행합니다.

```sh
make backend-dev BACKEND_HOST=127.0.0.1 BACKEND_PORT=8100
make frontend-dev FRONTEND_HOST=127.0.0.1 FRONTEND_PORT=5180
```

화면: http://127.0.0.1:5180, API 문서: http://127.0.0.1:8100/docs.
env 변경 후 프론트엔드를 재시작합니다. 전체 기준 검증은 `make verify-plan`, `VERIFY_FULL=1 make verify`입니다. 이메일·OAuth·Stripe·외부 텔레메트리는 비활성화되어 있으며 외부 Redis/Celery, SMTP, 결제, 클라우드 스토리지는 이 로컬 검증의 범위 밖입니다.

## Upstream 동기화

매주 및 보안 수정 발생 시 수동으로 검토합니다. 자동 스케줄은 등록하지 않습니다. [영문 가이드의 동기화 명령](../actigent.md#upstream-synchronization-policy)을 사용하여 최신 origin/main에서 `chore/upstream-sync-YYYYMMDD` 브랜치를 만들고 upstream/main을 `--no-ff` merge합니다. Actigent main을 upstream으로 reset하거나 강제 push하지 않습니다.

로컬 upstream의 push URL은 `DISABLED`, 기본 push 원격은 origin으로 설정합니다. `gh`에는 항상 `--repo Blueprint4Agent/actigent`를 명시합니다. 충돌은 파일별로 검토하여 Actigent 설정과 도메인 기능을 유지하고 프레임워크 수정 사항을 반영합니다. 환경 예제, 의존성, DB 마이그레이션, 계약, 훅, 프론트엔드 pin을 확인합니다. 서브모듈은 기록된 커밋으로만 초기화하며 자동으로 최신 버전을 추적하지 않습니다.

동기화 후 의존성 및 프로젝트 초기화, 계약과 전체 검증을 수행하고 작성한 커밋에는 일치하는 worklog를 포함합니다. Ready PR과 실제 PR 메타데이터 검증 후 merge commit으로 병합합니다. main에는 PR, 대화 해결, 삭제·강제 push 보호를 적용합니다. GitHub Actions는 수동 실행이며 로컬 훅이 서버 CI를 대신 보증하지 않습니다. 향후 upstream 커밋이 상속한 거버넌스 검증에 실패하면 정책 호환성을 명시적으로 해결하고 훅 우회나 이력 재작성은 하지 않습니다.

미해결 merge는 `git merge --abort`, 병합 후 되돌리기는 검토한 revert PR(`git revert -m 1 <sync-merge-sha>`)을 사용합니다. 실제 DB 변경 전 백업과 사본에서의 업그레이드 검증이 필요하며 코드 revert가 DB를 자동 복구하지 않습니다.

B4React는 고정된 upstream 의존성입니다. 향후 Actigent 전용 UI 소스 변경이 필요하면 별도 자식 Fork와 동기화 정책을 먼저 정하고 gitlink를 갱신합니다. 원본 B4React에 서비스 전용 코드를 push하지 않습니다.

## 향후 수집 구조

Adapter는 소스 API·응답을 공통 활동 계약으로 변환하고, Strategy는 polling/webhook·cursor·retry 정책을 분리하며, Factory/registry는 설정에 명시된 소스로 구현을 선택합니다. 구현 전에 식별자, 커서, 멱등성, 자격 증명, rate limit, 취소 정책을 정의합니다. 서비스가 오케스트레이션을, 라우터가 HTTP를, 모델/저장소가 영속화를, 기존 작업 루프가 백그라운드 실행을 담당합니다. 도메인 이벤트는 상태 저장 성공 후 발행합니다. 이번 단계에서는 설계 방향만 기록합니다.
