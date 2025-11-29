# Eureka Server

Jun Bank MSA의 **서비스 디스커버리** 서버입니다.

---

## 역할

- 모든 마이크로서비스 등록 및 관리
- 서비스 위치 정보 제공 (Service Registry)
- 헬스 체크 및 장애 감지
- 이중화 구성으로 고가용성 확보

---

## 기술 스택

| 항목 | 버전 |
|------|------|
| Java | 21 |
| Spring Boot | 4.0.0 |
| Spring Cloud | 2025.1.0 |
| Spring Cloud Netflix Eureka | 5.0.x |

---

## 실행 방법

### 로컬 실행

```bash
./gradlew bootRun
```

### Docker 빌드

```bash
./gradlew clean bootJar
docker build -t ghcr.io/jun-bank/eureka-server:latest .
```

---

## 이중화 구성

```
┌─────────────────┐     ┌─────────────────┐
│ eureka-server-1 │◄───►│ eureka-server-2 │
│     :8761       │     │     :8762       │
└─────────────────┘     └─────────────────┘
         ▲                       ▲
         │                       │
         └───────┬───────────────┘
                 │
    ┌────────────┴────────────┐
    │     Microservices       │
    │  (Config, Gateway, ...) │
    └─────────────────────────┘
```

두 Eureka 서버가 서로 등록하여 한 대 장애 시에도 정상 동작합니다.

---

## 포트

| 인스턴스 | 포트 |
|----------|------|
| eureka-server-1 | 8761 |
| eureka-server-2 | 8762 |

---

## 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `SERVER_PORT` | 8761 | 서버 포트 |
| `EUREKA_INSTANCE_HOSTNAME` | localhost | 인스턴스 호스트명 |
| `EUREKA_SELF_PRESERVATION` | false | 자기 보호 모드 |
| `ZIPKIN_ENDPOINT` | http://localhost:9411/api/v2/spans | Zipkin 엔드포인트 |

---

## API 엔드포인트

| 엔드포인트 | 설명 |
|------------|------|
| `/` | Eureka 대시보드 |
| `/eureka/apps` | 등록된 서비스 목록 |
| `/actuator/health` | 헬스 체크 |
| `/actuator/prometheus` | Prometheus 메트릭 |

---

## 디렉토리 구조

```
eureka-server/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/junbank/eurekaserver/
│       │       └── EurekaServerApplication.java
│       └── resources/
│           ├── application.yml
│           └── logback-spring.xml
├── build.gradle
├── settings.gradle
├── Dockerfile
└── README.md
```

---

## 주요 설정 (application.yml)

```yaml
eureka:
  instance:
    hostname: ${EUREKA_INSTANCE_HOSTNAME:localhost}
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

- `register-with-eureka: false` - 자기 자신은 등록하지 않음
- `fetch-registry: false` - 다른 서비스 목록 가져오지 않음
- `enable-self-preservation: false` - 개발 환경에서 빠른 장애 감지