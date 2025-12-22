# Hansung Gym Management System - Backend

한성대학교 헬스장 통합 관리 시스템의 백엔드 API 서버입니다.

## 목차
- [주요 기능](#주요-기능)
- [기술 스택](#기술-스택)
- [시스템 아키텍처](#시스템-아키텍처)
- [데이터 플로우](#데이터-플로우)
- [프로젝트 구조](#프로젝트-구조)
- [설치 및 실행](#설치-및-실행)
- [API 엔드포인트](#api-엔드포인트)
- [데이터베이스 스키마](#데이터베이스-스키마)

---

## 주요 기능

### 1. 회원 관리
- 회원 가입 및 로그인 (JWT 인증)
- 프로필 관리 (학번, 이름, 연락처, 학과, 학년)
- 역할 기반 권한 관리 (일반회원, 강사, 멘토, 멘티)

### 2. 출석 관리
- 실시간 입/퇴장 기록
- 현재 헬스장 이용 인원 조회 (혼잡도)
- 출석 기반 포인트 자동 적립

### 3. 운동 기록
- 운동 종목별 기록 관리
- 운동 시간 및 칼로리 추적
- 운동 기록 기반 포인트 적립

### 4. 식단 관리
- 식사 기록 (아침/점심/저녁/간식)
- 칼로리 계산 및 추적
- 식단 기록 기반 포인트 적립

### 5. 목표 설정 및 달성
- 개인 목표 설정
- 목표 달성률 추적
- 목표 달성 시 포인트 보상

### 6. 포인트 및 보상 시스템
- 활동 기반 포인트 자동 적립
- 포인트 교환 상품 관리
- 포인트 사용 내역 추적

### 7. 수업 관리
- 수업 등록 및 시간표 관리
- 수강 신청 및 정원 관리
- 강사 배정

### 8. 멘토링 시스템
- 멘토-멘티 매칭
- 멘토링 신청 및 관리

### 9. 신체 기록
- 체중, 근육량, 체지방 기록
- BMI 계산 및 추적
- 신체 변화 그래프

### 10. 뱃지 및 업적
- 활동 기반 뱃지 획득
- 업적 달성 기록

---

## 기술 스택

### Backend Framework
- **Node.js** (v18+) - JavaScript 런타임
- **Express.js** (v4.18) - 웹 프레임워크

### Database
- **MySQL** (v8.0+) - 관계형 데이터베이스
- **mysql2** - MySQL 클라이언트

### Authentication & Security
- **JWT (jsonwebtoken)** - 토큰 기반 인증
- **bcryptjs** - 비밀번호 암호화

### Middleware & Utilities
- **cors** - CORS 설정
- **dotenv** - 환경 변수 관리
- **body-parser** - 요청 본문 파싱
- **cookie-parser** - 쿠키 파싱

### Development Tools
- **nodemon** - 개발 서버 자동 재시작

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│                  (React Frontend - Vercel)                   │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS/REST API
                         │
┌────────────────────────▼────────────────────────────────────┐
│                     API Gateway Layer                        │
│                    (Express.js Server)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  CORS Middleware  │  JWT Auth  │  Body Parser       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    Business Logic Layer                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    Route Handlers                     │  │
│  │  • auth.js        • members.js      • exercises.js   │  │
│  │  • diet.js        • attendance.js   • points.js      │  │
│  │  • goals.js       • classes.js      • mentoring.js   │  │
│  │  • rewards.js     • badges.js       • quests.js      │  │
│  │  • guide.js       • health.js       • admin.js       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    Data Access Layer                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              MySQL Connection Pool                    │  │
│  │              (mysql2/promise)                         │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                      Database Layer                          │
│                    (MySQL Database)                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Tables: Member, Attendance, ExerciseLog, DietLog,   │  │
│  │  Goal, IncentivePolicy, AchievementLog, Reward,      │  │
│  │  Badge, Class, HealthRecord, etc.                    │  │
│  │                                                        │  │
│  │  Triggers: Auto Point Calculation & Reward System    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 데이터 플로우

### 1. 인증 플로우
```
Client → POST /api/auth/login
         ↓
      Validate Credentials (bcrypt)
         ↓
      Generate JWT Token
         ↓
      Return Token + User Info
         ↓
Client stores token in localStorage
         ↓
Subsequent requests include token in Authorization header
```

### 2. 포인트 적립 플로우
```
Client → POST /api/exercises (운동 기록)
         ↓
      INSERT INTO ExerciseLog
         ↓
      [TRIGGER] TRG_Exercise_Batch_Reward
         ↓
      Check unrewarded count >= condition_value
         ↓
      INSERT INTO AchievementLog (points_earned)
         ↓
      [TRIGGER] TRG_Achievement_Point_Earn
         ↓
      UPDATE Member.total_points += points_earned
         ↓
      Return success response
```

### 3. 포인트 사용 플로우
```
Client → POST /api/rewards/exchange
         ↓
      Check Member.total_points >= required_points
         ↓
      INSERT INTO PointExchange (used_points)
         ↓
      [TRIGGER] TRG_Exchange_Point_Use
         ↓
      UPDATE Member.total_points -= used_points
         ↓
      UPDATE Reward.stock_quantity -= 1
         ↓
      Return success response
```

### 4. 출석 체크 플로우
```
Client → POST /api/attendance/checkin
         ↓
      INSERT INTO Attendance (entered_at)
         ↓
      [TRIGGER] TRG_Attendance_Batch_Reward
         ↓
      Auto point calculation if condition met
         ↓
      Return current crowd status
```

---

## 프로젝트 구조

```
BE/
├── src/
│   ├── app.js                    # Express 앱 진입점
│   ├── config/
│   │   └── database.js           # MySQL 연결 설정
│   ├── middleware/
│   │   └── auth.js               # JWT 인증 미들웨어
│   ├── routes/                   # API 라우트
│   │   ├── auth.js               # 인증 (로그인/회원가입)
│   │   ├── members.js            # 회원 관리
│   │   ├── attendance.js         # 출석 관리
│   │   ├── exercises.js          # 운동 기록
│   │   ├── diet.js               # 식단 관리
│   │   ├── goals.js              # 목표 관리
│   │   ├── points.js             # 포인트 조회
│   │   ├── rewards.js            # 보상 상품
│   │   ├── badges.js             # 뱃지 관리
│   │   ├── quests.js             # 퀘스트 시스템
│   │   ├── classes.js            # 수업 관리
│   │   ├── mentoring.js          # 멘토링 시스템
│   │   ├── health.js             # 신체 기록
│   │   ├── guide.js              # 가이드 정보
│   │   └── admin.js              # 관리자 기능
│   ├── controllers/              # 비즈니스 로직 (예정)
│   ├── models/                   # 데이터 모델 (예정)
│   └── utils/                    # 유틸리티 함수 (예정)
├── sql/
│   ├── HS_Health.sql             # 전체 DB 스키마 + 트리거
│   ├── insert_dummy_data.sql     # 테스트 데이터
│   ├── insert_class_data.sql     # 수업 데이터
│   └── add_total_points_column.sql
├── scripts/
│   └── init_rewards.js           # 보상 정책 초기화
├── .env                          # 환경 변수
├── .gitignore
├── package.json
└── README.md
```

---

## 설치 및 실행

### 1. 환경 변수 설정
`.env` 파일을 생성하고 다음 내용을 입력하세요:

```env
# Server
PORT=5001
NODE_ENV=development

# Database
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=hs_health
DB_PORT=3306

# JWT
JWT_SECRET=hansung_gym_secret_key_2025
JWT_EXPIRE=7d

# Frontend URL
FRONTEND_URL=http://localhost:3000
```

### 2. 의존성 설치
```bash
npm install
```

### 3. 데이터베이스 설정
```bash
# MySQL 접속
mysql -u root -p

# 데이터베이스 생성
CREATE DATABASE hs_health;
USE hs_health;

# 스키마 및 트리거 생성
source sql/HS_Health.sql;

# 테스트 데이터 삽입 (선택)
source sql/insert_dummy_data.sql;
source sql/insert_class_data.sql;
```

### 4. 보상 정책 초기화
```bash
node scripts/init_rewards.js
```

### 5. 서버 실행
```bash
# 개발 모드 (nodemon)
npm run dev

# 프로덕션 모드
npm start
```

서버가 `http://localhost:5001`에서 실행됩니다.

---

## API 엔드포인트

### Authentication
- `POST /api/auth/login` - 로그인
- `POST /api/auth/register` - 회원가입
- `GET /api/auth/me` - 현재 사용자 정보
- `POST /api/auth/logout` - 로그아웃

### Members
- `GET /api/members` - 전체 회원 조회
- `GET /api/members/:id` - 특정 회원 조회
- `POST /api/members` - 회원 생성
- `PUT /api/members/:id` - 회원 정보 수정
- `PUT /api/members/:id/profile` - 프로필 업데이트
- `DELETE /api/members/:id` - 회원 삭제

### Attendance
- `POST /api/attendance/checkin` - 입장
- `POST /api/attendance/checkout` - 퇴장
- `GET /api/attendance/current` - 현재 이용 인원
- `GET /api/attendance/history/:memberId` - 출석 기록

### Exercises
- `GET /api/exercises` - 운동 목록
- `POST /api/exercises` - 운동 기록 추가
- `GET /api/exercises/log/:memberId` - 운동 기록 조회
- `POST /api/exercises/list` - 운동 종목 추가

### Diet
- `GET /api/diet` - 식단 기록 조회
- `POST /api/diet` - 식단 기록 추가
- `GET /api/diet/foods` - 음식 목록
- `POST /api/diet/foods` - 음식 추가

### Goals
- `GET /api/goals/:memberId` - 목표 조회
- `POST /api/goals` - 목표 생성
- `PUT /api/goals/:id` - 목표 수정
- `PUT /api/goals/:id/achieve` - 목표 달성
- `DELETE /api/goals/:id` - 목표 삭제

### Points & Rewards
- `GET /api/points/:memberId` - 포인트 조회
- `GET /api/points/history/:memberId` - 포인트 내역
- `GET /api/rewards` - 보상 상품 목록
- `POST /api/rewards/exchange` - 포인트 교환

### Classes
- `GET /api/classes` - 수업 목록
- `GET /api/classes/:id/schedule` - 수업 시간표
- `POST /api/classes/register` - 수강 신청
- `DELETE /api/classes/register/:id` - 수강 취소

### Mentoring
- `POST /api/mentoring/request` - 멘토링 신청
- `GET /api/mentoring/matches` - 매칭 목록
- `PUT /api/mentoring/accept/:id` - 매칭 수락
- `DELETE /api/mentoring/cancel/:id` - 매칭 취소

### Health Records
- `GET /api/health/:memberId` - 신체 기록 조회
- `POST /api/health` - 신체 기록 추가
- `GET /api/health/:memberId/latest` - 최근 기록

### Badges & Quests
- `GET /api/badges/:memberId` - 획득 뱃지 조회
- `GET /api/quests` - 퀘스트 목록
- `POST /api/quests/complete` - 퀘스트 완료

### Admin
- `GET /api/admin/stats` - 통계 조회
- `POST /api/admin/policies` - 보상 정책 관리

---

## 데이터베이스 스키마

### 핵심 테이블

#### Member (회원)
- 회원 기본 정보 (학번, 이름, 연락처, 학과, 학년)
- 역할 타입 (일반/강사/멘토/멘티)
- 포인트 캐싱 (total_points)

#### IncentivePolicy (보상 정책)
- 정책 타입 (운동/식단/목표/출석)
- 달성 조건 (condition_value)
- 지급 포인트 (points_awarded)

#### AchievementLog (업적 로그)
- 포인트 획득 기록
- 정책 연결 (policy_id)
- 포인트 스냅샷

#### ExerciseLog, DietLog, Attendance, Goal
- 각 활동 기록
- achievement_id로 보상 연결

#### Reward (보상 상품)
- 상품명, 필요 포인트, 재고

#### PointExchange (포인트 교환)
- 사용 포인트, 교환 일시

### 자동화 트리거

1. **TRG_Achievement_Point_Earn**
   - AchievementLog 생성 시 Member.total_points 자동 증가

2. **TRG_Exchange_Point_Use**
   - PointExchange 생성 시 Member.total_points 자동 감소

3. **TRG_Exercise_Batch_Reward**
   - 운동 기록 N회 달성 시 자동 포인트 적립

4. **TRG_Diet_Batch_Reward**
   - 식단 기록 N회 달성 시 자동 포인트 적립

5. **TRG_Attendance_Batch_Reward**
   - 출석 N회 달성 시 자동 포인트 적립

6. **TRG_Goal_Batch_Reward**
   - 목표 N개 달성 시 자동 포인트 적립

---

## 라이센스

MIT License

---

## 개발팀

HSU Gym Team

---

## 문의

프로젝트 관련 문의사항은 이슈를 등록해주세요.
