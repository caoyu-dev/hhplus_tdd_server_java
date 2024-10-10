# 프로젝트 다이어그램
![다이어그램](/Users/caoyu/Downloads/photo_2024-10-10 23.26.57.jpeg)

# 프로젝트 마일스톤

## 마일스톤 1: 프로젝트 설정 및 기본 구조 설계
- 목표: 프로젝트의 초기 구조 설정 및 환경 구성
- 작업 내용:
  - 스프링 부트 프로젝트 생성
  - 의존성 추가
  - 요구 사항 분석, API 설계 및 테스트 코드 작성 
  - ERD 설계
## 마일스톤 2: 유저 대기열 토큰 발급 구현
- 목표: 유저 대기열 관리 시스템 구현
- 작업 내용:
  - 토큰 발급 API 구현
  - 대기열을 관리할 수 있는 정보 (대기 순서, 잔여 시간 등) 저장 및 관리 로직 구현
  - 유저 대기열을 폴링 방식으로 상태 확인 가능하도록 설계
  - 단위 테스트 작성 (토큰 발급 및 대기열 상태 조회)
- 유저 대기열 요청 → [결정 노드: 이미 대기열에 있는가?]
  - Yes: 대기열 상태 반환
  - No: [대기열에 추가] → 대기 순서, 잔여 시간 할당 → [토큰 발급]
  - 종료: 토큰 반환
## 마일스톤 3: 예약 가능 날짜 및 좌석 조회 API 구현
- 목표: 예약 가능한 날짜 및 좌석 정보 제공
- 작업 내용:
  - 예약 가능한 날짜 목록 조회 API 구현
  - 특정 날짜에 예약 가능한 좌석 목록 조회 API 구현 (1번부터 50번까지 좌석 관리)
  - 좌석 상태를 저장하고 관리하는 로직 구현
  - 단위 테스트 작성 (날짜 및 좌석 조회)
- 예약 가능한 날짜 요청 → [결정 노드: 날짜가 있는가?]
  - Yes: 예약 가능한 날짜 목록 반환
  - No: 에러 메시지 반환
- 특정 날짜 좌석 조회 요청 → [결정 노드: 좌석이 있는가?]
  - Yes: 예약 가능한 좌석 목록 반환
  - No: 에러 메시지 반환
## 마일스톤 4: 좌석 예약 요청 및 임시 배정 시스템 구현
- 목표: 좌석 예약 요청 시 임시 배정 및 동시성 제어 처리
- 작업 내용:
  - 날짜와 좌석 정보를 입력받아 좌석 예약 처리 API 구현
  - 좌석 예약 시 5분간 해당 좌석 임시 배정 (결제 미완료 시 임시 배정 해제)
  - 임시 배정된 좌석에 대해 다른 사용자가 예약 불가능하도록 동시성 제어 (ReentrantLock 또는 Redis 기반의 분산 락을 사용해 동시성 문제 해결)
  - 단위 테스트 및 통합 테스트 작성 (예약 요청 및 임시 배정 테스트)
- 좌석 예약 요청 → [결정 노드: 좌석 임시 배정 중인가?]
  - Yes: 예약 불가 메시지 반환 (이선좌)
  - No: [좌석 임시 배정] → 5분 타이머 시작
- 결제 요청 확인 → [결정 노드: 결제 완료?]
  - Yes: 좌석 소유권 배정 → [임시 배정 해제]
  - No: 임시 배정 해제
## 마일스톤 5: 잔액 충전 및 조회 API 구현
- 목표: 사용자 잔액을 충전하고 조회하는 기능 구현
- 작업 내용:
  - 잔액 충전 API 구현 (사용자 식별자 및 충전할 금액 입력받아 처리)
  - 잔액 조회 API 구현 (사용자 식별자 기반으로 조회)
  - 결제 처리 전, 잔액 검증 로직 추가
  - 잔액 정보 저장을 위한 DB 설계
  - 단위 테스트 작성 (잔액 충전 및 조회 기능)
- 잔액 충전 요청 → [결정 노드: 충전 금액이 유효한가?]
  - Yes: 잔액 충전 완료
  - No: 에러 메시지 반환
- 잔액 조회 요청 → [결정 노드: 유효한 사용자 식별자인가?]
  - Yes: 잔액 정보 반환
  - No: 에러 메시지 반환
## 마일스톤 6: 결제 API 구현 및 좌석 소유권 배정 시스템 구현
- 목표: 결제 처리 후 좌석 소유권 배정 및 대기열 토큰 만료 처리
- 작업 내용:
  - 결제 처리 API 구현 (잔액 차감 및 결제 내역 저장)
    - 결제 성공 시 좌석 소유권을 해당 유저에게 배정
    - 결제 완료 후 대기열 토큰 만료 처리
    - 결제 실패 또는 미처리 시 임시 배정 해제
    - 결제 내역 저장을 위한 엔티티 설계 및 트랜잭션 처리
  - 단위 테스트 및 통합 테스트 작성 (결제 및 좌석 배정)
- 결제 요청 → [결정 노드: 잔액이 충분한가?]
  - Yes: [잔액 차감] → 좌석 소유권 배정
  - No: 결제 실패 메시지 반환
- [결제 완료] → [대기열 토큰 만료] → [결제 내역 저장]
- [결제 실패] → 임시 배정 해제

## 마일스톤 7: 전체 통합 테스트 및 최종 배포
- 목표: 전체 서비스 기능 통합 테스트 및 최종 배포 
- 작업 내용:
  - 전체 기능 통합 테스트 (유저 대기열, 좌석 임시 배정, 잔액 충전 및 결제 관련 통합 테스트 작성)
  - 배포 환경 설정 및 최종 배포
## 전체 일정 예상
- 총 예상 기간: 7주
