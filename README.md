Real-World Social Deduction Game
본 프로젝트는 React Native와 FastAPI를 기반으로 하며, UWB 및 BLE 기술을 활용해 현실 공간에서 플레이하는 위치 기반 소셜 디덕션 게임입니다.

1. 기술 스택
Client: React Native (iOS/Android)

Backend: FastAPI (Python)

Real-time: WebSocket, REST API

Database: PostgreSQL (영구 데이터), Redis (세션 상태)

Hardware: UWB (Nearby Interaction / Jetpack), BLE RSSI

AI: OpenAI GPT-4o / GPT-4o-mini

2. 프로젝트 구조 및 로드맵
시스템 구성
Lobby/Role/Game/Meeting/Result: 클라이언트 주요 화면 구성

RoomManager: 방 생성 및 입퇴장 관리

GameEngine: 규칙 및 상태 이벤트 처리

ProximityMgr: 거리 기반 로직 처리 (킬 범위 등)

MissionSystem: QR, 미니게임, 체류 미션 관리

AIDirector: LLM 기반 실시간 해설 및 가이드

개발 단계
Phase 1 (설계): 요구사항 및 DB/시스템 설계 (현재 단계)

Phase 2 (코어): GameEngine 및 WebSocket 서버 구축

Phase 3 (근접 감지): BLE/UWB 하이브리드 거리 측정 구현

Phase 4 (AI Director): LLM 연동 및 이벤트별 개인화 가이드

Phase 6 (통합): 실제 기기 테스트 및 밸런싱

3. 핵심 시스템 상세
3.1 미션 시스템 (Mission System)
플레이어는 구역별로 고르게 분배된 미션을 수행하여 진행도를 높입니다.

QR_SCAN: 특정 구역의 QR 코드 스캔

MINI_GAME: 앱 내 인터랙티브 미니게임

STAY: 특정 구역 내 N초간 체류

배정: 크루원(실제 미션), 임포스터(동선 기만용 가짜 미션)

3.2 근접 감지 및 킬 (Proximity System)
기기 성능에 따라 정밀도를 조절하는 하이브리드 전략을 사용합니다.

UWB (정밀): 10cm 오차, 방향 감지 가능 (iOS/Android 최신 기기)

BLE (폴백): 1~3m 오차, RSSI 기반 거리 추정

킬 로직: 임포스터가 대상에게 접근 시 서버에서 킬 버튼 활성화 신호 전송

3.3 투표 및 회의 (Voting System)
시체 신고나 긴급 버튼을 통해 회의를 진행하며, 실시간 소켓 통신으로 상태를 동기화합니다.

소집: 시체 발견(5m 이내) 또는 긴급 버튼 클릭

단계: 토론(90s) -> 투표(30s) -> 결과 공개 -> 복귀/종료

규칙: 과반수 득표자 추방 (동률 시 무효), 추방 시 역할 공개 여부 설정 가능

3.4 AI Director
LLM을 활용해 게임의 긴장감을 유지하고 플레이어에게 전략을 제시합니다.

공개 해설: 전체 상황 요약 및 분위기 조성 (GPT-4o-mini)

개인 가이드: 역할에 따른 맞춤형 전략(임포스터 조언 등) 제공 (GPT-4o)

3.5 재화 및 아이템 (Economy)
미션 수행과 게임 기여도에 따라 재화를 획득하고 상점에서 아이템을 구매합니다.

재화 획득: 미션 완료, 무고 투표, 임포스터 추방 기여, 승리 등

주요 아이템:

공용: 연막탄 (위치 숨김)

크루원: 지도 (플레이어 위치 표시), 방탄조끼 (킬 1회 무효), 탐지기 (근처 임포스터 감지)

임포스터: 변장 (닉네임 변경), 방해전파 (특정 구역 미션 중단)

4. 실행 및 테스트
백엔드 테스트
환경 세팅 및 종속성 설치

단위 테스트 (모듈 독립 검증)

통합 테스트 (시스템 연동)

E2E 테스트 (소켓 시뮬레이션)
