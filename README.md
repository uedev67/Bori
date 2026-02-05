# Bori
반려 로봇 만들기 프로젝트 


# 통신 프로토콜

# Jetson ↔ Arduino Serial Protocol (v0.1)

이 문서는 Jetson(상위 로직)과 Arduino(모터 제어)가 USB 시리얼로 통신할 때 사용하는 **텍스트 기반 프로토콜**을 정의합니다.

---

## 1) 기본 통신 사양

| 항목 | 내용 | 비고 |
|---|---|---|
| 물리 연결 | USB Serial (USB-C) | Jetson–Arduino 직결 |
| Baudrate | 115200 | 권장 |
| 데이터 형식 | ASCII 텍스트 | 디버깅 용이 |
| 메시지 종료 | `\n` (LF) | 한 줄 = 한 명령 |
| 통신 방식 | Request/Response | ACK/ERR 필수 |

---

## 2) 제어 대상 ID

### 2.1 서보(3개)

| 구분 | ID | 위치 | 모터 |
|---|---|---|---|
| 서보 | `EAR_L` | 왼쪽 귀 | SG90 |
| 서보 | `EAR_R` | 오른쪽 귀 | SG90 |
| 서보 | `NECK_PITCH` | 목 상하 끄덕임 | 서보 1개 |

### 2.2 DC 기어박스 모터(2개)

| 구분 | ID | 위치 | 비고 |
|---|---|---|---|
| DC | `WHEEL_L` | 왼쪽 바퀴 | 기어박스 |
| DC | `WHEEL_R` | 오른쪽 바퀴 | 기어박스 |

---

## 3) 메시지 포맷

### 3.1 Request (Jetson → Arduino)

**형식**
