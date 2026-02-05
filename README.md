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
SEQ,CMD,TARGET,ARG1[,ARG2...]


| 필드 | 의미 | 예시 |
|---|---|---|
| `SEQ` | 요청 번호(추적용) | `0 ~ 65535` |
| `CMD` | 명령 종류 | `SV`, `DC`, `STOP`, `PING`, `GET` |
| `TARGET` | 제어 대상 | `EAR_L`, `WHEEL_R`, `ALL` |
| `ARGn` | 명령 인자 | 각도, 속도 등 |

### 3.2 Response (Arduino → Jetson)

**성공**
SEQ,OK[,INFO]
**실패**
SEQ,ERR,ERRCODE[,INFO]


| 필드 | 의미 |
|---|---|
| `SEQ` | 요청과 동일한 번호 |
| `ERRCODE` | 에러 종류 |
| `INFO` | 부가 설명(선택) |

---

## 4) 명령 정의

### 4.1 서보 제어: `SV`

| 항목 | 내용 |
|---|---|
| CMD | `SV` |
| TARGET | `EAR_L`, `EAR_R`, `NECK_PITCH` |
| ARG1 | 각도(deg) |
| 각도 범위 | `0 ~ 180` *(임시, 캘리브레이션 예정)* |

**예시**
12,SV,EAR_L,140

---

### 4.2 DC 모터 제어: `DC`

| 항목 | 내용 |
|---|---|
| CMD | `DC` |
| TARGET | `WHEEL_L`, `WHEEL_R` |
| ARG1 | 속도 |
| 속도 범위 | `-255 ~ +255` |
| 부호 의미 | `+` 전진 / `-` 후진 / `0` 정지 |

**예시**
20,DC,WHEEL_L,120

---

### 4.3 전체 정지: `STOP`

| 항목 | 내용 |
|---|---|
| CMD | `STOP` |
| TARGET | `ALL` |
| 동작 | 모든 DC 모터 속도 0 |
| 서보 처리 | 현재 위치 유지 *(권장)* |

**예시**
50,STOP,ALL

---

### 4.4 연결 확인: `PING`

| 항목 | 내용 |
|---|---|
| CMD | `PING` |
| TARGET | `NA` |
| 목적 | 통신 생존 확인 |

**예시**
1,PING,NA

**응답 예시**
1,OK,PONG

---

### 4.5 상태 조회(선택): `GET`

| 항목 | 내용 |
|---|---|
| CMD | `GET` |
| TARGET | `VER`, `STATE` 등 |
| 목적 | 버전/상태 조회 |

**예시**
2,GET,VER

**응답 예시**
2,OK,VER,0.1

---

## 5) 에러 코드

| ERRCODE | 의미 | 예시 상황 |
|---|---|---|
| `ID` | 대상 ID 오류 | 없는 `TARGET` |
| `RANGE` | 값 범위 초과 | 각도/속도 범위 초과 |
| `FMT` | 형식 오류 | 필드 수 부족, 파싱 실패 |
| `BUSY` | 내부 처리 중(선택) | 타이밍/자원 문제 |
| `NA` | 미지원 명령 | 없는 `CMD` |

**에러 예시**
23,ERR,RANGE,ANGLE_OUT

