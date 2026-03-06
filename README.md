# Real-World Social Deduction Game Project
현실 공간에서의 정밀한 거리 측정(UWB/BLE)과 LLM 기반의 AI 가이드를 결합한 차세대 소셜 디덕션 게임 플랫폼입니다.

1. 시스템 아키텍처 (System Architecture)
Client: React Native
로비 화면: 방 생성 및 대기열 관리

역할 화면: 개인별 역할(크루원/임포스터) 부여 및 개인화 UI

메인 게임 화면: 미션 목록, 구역 지도, 킬/신고 버튼, AI 가이드 실시간 피드

회의 화면: 토론 및 투표 인터페이스

결과 화면: 승리 조건 달성 시 최종 통계 및 보상 표시

Backend: FastAPI
RoomManager: 실시간 방 세션 관리 (생성/입장/퇴장)

GameEngine: 게임 규칙, 상태 머신, 전역 이벤트 처리

ProximityMgr: UWB/BLE 데이터를 활용한 실시간 거리 행렬 관리

MissionSystem: 플레이어별 미션 할당 및 완료 로직 검증

ItemSystem: 인게임 재화 획득 및 아이템 사용 효과 관리

VoteSystem: 회의 소집, 투표 집계 및 추방 로직

AIDirector: LLM(GPT) 연동을 통한 실시간 상황 해설 및 가이드

Data Storage
Redis: 실시간 게임 세션 및 플레이어 상태 정보 (Fast I/O)

PostgreSQL: 유저 프로필, 게임 전적, 보유 재화 등 영구 데이터

2. 개발 로드맵 (Roadmap)
Phase 1: 설계 (Current): 요구사항 정의, 시스템 아키텍처 및 DB 스키마 설계

Phase 2: 백엔드 코어: 프로젝트 기본 환경 구성, GameEngine 및 WebSocket 서버 구축

Phase 3: 근접 감지 시스템: BLE RSSI 기반 거리 추정 우선 구현 후 UWB 정밀 측정 레이어 추가

Phase 4: AI Director: LLM API 연동, 상황별 공개 해설 및 역할별 가이드 로직 구현

Phase 5: 클라이언트: React Native 앱 UI 구현 및 주요 게임 화면 연동

Phase 6: 통합 테스트: 실제 기기를 활용한 필드 테스트, 게임 밸런싱 및 최종 배포

3. 핵심 시스템 상세 로직
3.1 미션 시스템 (Mission System)
플레이어는 지정된 구역에서 다양한 형태의 미션을 수행합니다.

미션 타입

QR_SCAN: 특정 구역의 QR 코드를 직접 스캔하여 인증

MINI_GAME: 앱 내 UI 인터랙션(배선 연결, 슬라이더 조작 등) 수행

STAY: 특정 구역 내에서 지정된 시간(N초) 동안 체류

미션 상태: PENDING → IN_PROGRESS → COMPLETED / FAILED

배정 규칙

크루원에게만 실제 미션 배정 (임포스터는 동선 기만용 가짜 미션 부여)

플레이어당 설정된 개수만큼 구역별로 고르게 분배

[전체 미션 흐름]

게임 시작 시 assignMissions() 호출 (크루원: 실제 3개 / 임포스터: 가짜 2개)

플레이어 구역 이동 시 move 이벤트 발생

handleZoneEnter()를 통해 해당 구역 미션 활성화 (알림, UI 오픈, 타이머 시작 등)

미션 완료 이벤트 처리 (qr_scan, minigame_complete, STAY 타이머 종료)

재화 지급 및 전체 진행도 업데이트 → 모든 미션 완료 시 크루원 승리

3.2 근접 감지 시스템 (Proximity System)
기기별 사양에 최적화된 하이브리드 거리 측정 전략을 사용합니다.

기기별 전략

UWB 지원: Nearby Interaction (iOS) / UWB Jetpack (Android) 활용 (~10cm 오차)

UWB 미지원: BLE RSSI 기반 거리 추정 (~1~3m 오차)

서버 역할: 토큰 교환 중개, 전역 거리 행렬 관리, 킬 가능 여부 판정, 근접 이벤트 감지

[전체 근접/킬 흐름]

기기별 initUWB() 수행 및 보안 토큰 생성

서버에 토큰 등록 후 타 플레이어에게 브로드캐스트하여 n:n 세션 연결 (실패 시 BLE 폴백)

실시간 거리 측정 후 proximity_update 이벤트로 서버 행렬 갱신

임포스터가 크루원 기준 유효 거리(UWB 1.5m / BLE 3m) 진입 시 killable_targets 활성화

임포스터 킬 버튼 클릭 → 서버 최종 검증 후 킬 확정

3.3 투표 및 회의 시스템 (Voting System)
실제 대면 및 원격 토론을 지원하는 실시간 투표 프로세스입니다.

회의 소집: 시체 신고(시체 5m 이내만 가능) 또는 긴급 버튼(게임당 1회) 사용

투표 규칙: 자기 자신 투표 불가, 사망자 투표 불가, SKIP 가능, 동률 시 추방 없음

[전체 투표 흐름]

validateMeeting()을 통한 소집 조건 검증

startMeeting()으로 VoteSession 생성 및 전체 브로드캐스트

[토론 단계 90초] 매초 tick 이벤트 전송

[투표 단계 30초] 투표 진행 (전원 완료 시 조기 마감)

tally() 집계 후 최다 득표자 추방 및 역할 공개

vote_result 표시 후 게임 복귀 또는 승리 조건 체크

3.4 AI Director (LLM 가이드)
LLM을 활용해 실시간 게임 상황을 중계하고 플레이어에게 전략적인 조언을 제공합니다.

원칙: 정보 불균형 유지 (크루원에게 임포스터 정보 누설 금지), 맥락에 맞는 톤 유지

전략: 공개 해설(GPT-4o-mini / 속도 중시), 개인 가이드(GPT-4o / 전략 정밀도 중시)

[AI 메시지 흐름]

게임 이벤트 발생 시 EventBus를 통해 AIDirector 비동기 호출

상황에 맞는 메시지 생성 (공개 해설 또는 개인별 맞춤 가이드)

ai_message (전체 피드) 및 ai_guide (개별 팝업) 소켓 전송

클라이언트 UI 피드 적재 및 팝업 표시

3.5 아이템 및 경제 시스템 (Item & Economy)
게임을 통한 재화 획득과 전략적 아이템 사용을 지원합니다.

재화 획득: 미션 완료, 무고 투표 참여, 임포스터 추방 기여, 게임 승리

아이템 목록: 연막탄(공용), 지도/방탄조끼/탐지기(크루원 전용), 변장/방해전파(임포스터 전용)

[전체 아이템 흐름]

조건 달성 시 player.currency 증가 및 UI 업데이트

상점 소켓을 통한 아이템 구매 (역할 및 잔액 검증)

아이템 사용 시 ActiveEffect 생성 및 즉시 효과/타이머 적용

킬 시도와 같은 특정 상황 발생 시 ItemSystem에서 수동 효과(방탄조끼 등) 우선 검증

4. 테스트 가이드 (Testing)
환경 세팅: Docker를 활용한 PostgreSQL/Redis 설정 및 FastAPI 서버 실행

단위 테스트: pytest를 활용한 시스템 모듈별(Mission, Vote, Item 등) 독립 검증

통합 테스트: WebSocket 연결을 통한 각 시스템 간의 이벤트 체인 연동 확인

E2E 테스트: 실제 모바일 기기 다수를 활용한 실시간 위치 이동 및 소켓 통신 시뮬레이션

