# 🛠️ [re-family] 02. 앱 스펙 및 기술 아키텍처 (Technical Specs)

* **기준 버전**: v2.7 (최신 단일 진실 공급원)
* **작성자**: 변지선
* **업데이트 일자**: 2026년 11월 20일

---

## 1. 3대 핵심 데이터베이스 스키마

### 1.1 Emotion_Logs (마음소파)
* `log_id` (PK, UUID): 로그 고유식별자
* `user_id` (FK, UUID): 작성자 식별자
* `raw_emotion_text` (String): 감정 기록 원문 (**온디바이스 로컬 세션 영구 격리, 서버 전송 불가**)
* `nlp_distortion_tag` (Enum): NLP 엔진 진단 왜곡 태그 (`당위적사고`, `자책감`, `터널시야` 등)
* `control_sphere` (Enum): 통제 원 분류 상태 (`CAN_CONTROL`, `CANNOT_CONTROL`)
* `created_at` (Timestamp): 작성 일시

### 1.2 Care_Schedules (가족달력)
* `schedule_id` (PK, UUID): 일정 고유식별자
* `user_id` (FK, UUID): 일정 소유자 식별자
* `schedule_type` (Enum): 일정 유형 (`CARE` 의무 돌봄, `TIMEOUT` 안심 휴식/숙면)
* `start_time` (Timestamp): 일정 시작 시간
* `end_time` (Timestamp): 일정 종료 시간
* `confirm_status` (Boolean): 단방향 수동 동기화 확인 승인 여부
* `created_at` (Timestamp): 등록 일시

### 1.3 Ondap_Logs (온답 및 얼굴 통화)
* `session_id` (PK, UUID): 소통 세션 고유식별자
* `sender_id` (FK, UUID): 발신인 식별자
* `receiver_id` (FK, UUID): 수신인 식별자
* `recommended_text` (String): HAILEY + SET-UP 대화법 기반 추천 문장
* `step_status` (Enum): 2-Step 가드레일 상태 (`CHIP_TAP` 1단계 주입, `SENT` 2단계 검토 후 최종 전송)
* `call_duration` (Integer): 아바타 통화 시간 (초 단위)
* `created_at` (Timestamp): 세션 기록 일시

---

## 2. Three.js 기반 3D 웹 렌더링 및 사운드 아키텍처
* **3D 그래픽 인프라**: Three.js (r128) 및 Tailwind CSS 기반의 반응형 3D 뷰포트. WebGLRenderer의 톤 매핑(ACESFilmicToneMapping) 적용.
* **절차적 웹 오디오 엔진 (Web Audio API)**: 외부 음원 로딩 없이 실시간 코드로 바람소리, 벽난로, 산새 지저귐, 풀벌레 소리를 동적 합성하는 초경량 사운드스케이프 구축.
* **6단계 공간 상태 머신**: WELCOME -> TRAIL (산책로 이동) -> ENTERING_CABIN -> CABIN_STAND -> SEATED (소파 착석 및 대화) -> EXITING_CABIN (오두막 퇴장)의 부드러운 카메라 LERP 물리 보간 이동.

---

## 3. 온디바이스 보안 및 프라이버시 아키텍처
* **원본 영상 전송량 0 MB**: WebRTC 영상 통화 중 카메라 원본 비디오 데이터는 단말기 외부로 일체 송출되지 않으며, 온디바이스(MediaPipe Face Mesh) 단에서 얼굴·상체 움직임 좌표(JSON)만 추출하여 아바타에 매핑.
* **데이터 영구 소멸 가드레일 (Zero-Persistence)**: 간편인증 정보 및 영상통화 중 음성 인식(STT) 요약문은 다운로드 완료 즉시 서버 데이터 영구 소멸 처리.
