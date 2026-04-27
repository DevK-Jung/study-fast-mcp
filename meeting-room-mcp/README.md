# Meeting Room MCP Server

FastMCP 기반의 회의실 예약 MCP(Model Context Protocol) 서버입니다. Claude 등 LLM과 연동하여 자연어로 회의실을 검색하고 예약할 수 있습니다.

## 주요 기능

- **회의실 검색**: 시간, 수용인원, 위치, 장비 조건으로 사용 가능한 회의실 검색
- **예약 관리**: 예약 생성, 상세 조회, 취소
- **이메일 알림**: 예약 확인/취소/리마인더 이메일 발송 (Mock 모드 지원)
- **SQLite 기반**: 개발 환경에서 별도 DB 설정 없이 바로 실행 가능

## 기술 스택

- **Python** 3.11+
- **FastMCP** 2.12+ — MCP 서버 프레임워크
- **FastAPI** + **Uvicorn** — HTTP 레이어
- **SQLAlchemy** 2.0 — ORM
- **SQLite** (개발) / **MySQL** (프로덕션)
- **Pydantic** v2 — 설정 및 스키마 검증
- **uv** — 패키지 관리

## 프로젝트 구조

```
meeting-room-mcp/
├── src/meeting_room_mcp/
│   ├── server/
│   │   ├── main.py                  # FastMCP 앱 및 서버 엔트리포인트
│   │   ├── room/                    # 회의실 도구, 서비스, 레포지토리
│   │   ├── reservation/             # 예약 도구, 서비스, 레포지토리
│   │   ├── notification/            # 이메일 알림 도구
│   │   └── services/                # 공유 서비스 (이메일 등)
│   ├── config/
│   │   ├── settings.py              # 환경 변수 기반 설정
│   │   └── database_config.py       # DB 연결 및 테이블 관리
│   └── shared/
│       ├── database.py              # SQLAlchemy 데이터베이스 매니저
│       └── models.py                # Pydantic 도메인 모델
├── scripts/
│   ├── start_server.py              # 서버 실행 스크립트
│   └── start_client.py              # 클라이언트 테스트 스크립트
├── data/
│   └── meeting_room.db              # SQLite DB (자동 생성)
├── .env.example                     # 환경 변수 템플릿
├── docker-compose.infra.yaml        # MySQL 인프라 (선택)
└── pyproject.toml
```

## 시작하기

### 1. 의존성 설치

```bash
cd meeting-room-mcp
uv sync
```

### 2. 환경 변수 설정

```bash
cp .env.example .env
# .env 파일에서 필요한 항목 수정 (개발 환경에서는 기본값으로 바로 실행 가능)
```

### 3. 서버 실행

```bash
uv run python scripts/start_server.py
```

서버 시작 시 SQLite DB와 테이블이 자동 생성되며, 샘플 회의실 데이터가 초기화됩니다.

## MCP 도구 목록

| 도구 | 설명 |
|------|------|
| `search_available_rooms` | 조건에 맞는 사용 가능한 회의실 검색 |
| `get_room_info` | 특정 회의실 상세 정보 조회 |
| `get_all_rooms` | 전체 회의실 목록 조회 |
| `create_reservation` | 회의실 예약 생성 |
| `get_reservation_details` | 예약 상세 정보 조회 |
| `cancel_reservation` | 예약 취소 |
| `send_notification` | 예약 이메일 알림 발송 |

## Claude Code / MCP 클라이언트 연동

`mcp.json` 예시:

```json
{
  "servers": {
    "meeting-room-mcp": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "/path/to/meeting-room-mcp/scripts/start_server.py"]
    }
  }
}
```

## 개발 도구

```bash
uv run black .          # 코드 포맷팅
uv run ruff check .     # 린팅
uv run ruff check . --fix  # 린팅 자동 수정
uv run mypy src/        # 타입 체크
uv run pytest           # 테스트 실행
```

## MySQL 사용 (선택)

개발 환경에서 MySQL을 사용하려면:

```bash
docker compose -f docker-compose.infra.yaml up -d
```

이후 `.env`에서 DB 설정을 MySQL로 변경합니다.

## 환경 변수

주요 환경 변수는 `.env.example` 참고. 개발 환경에서는 `EMAIL_MOCK_MODE=True`로 설정 시 실제 SMTP 없이 이메일 알림을 테스트할 수 있습니다.
