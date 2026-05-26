# 개발자 가이드라인: 데이터베이스 작명 컨벤션 (Database Naming Conventions)

본 문서는 **PostgreSQL** 및 **MS-SQL** 환경에서 적용되는 엄격한 데이터베이스 설계 및 작명 컨벤션을 정의합니다. 본 규칙은 시스템의 일관성을 유지하고, 혼합 데이터베이스 환경에서의 호환성을 극대화하며, AI 개발 도구(Claude Code, Codex 등)가 쿼리를 자동 생성할 때 코드 가독성과 명확성을 최적화할 수 있도록 설계되었습니다.

---

## 1. 기본 원칙 및 대소문자 규칙

* **전체 소문자 및 스네이크 케이스 (Snake Case):** 테이블, 뷰, 컬럼, 프로시저, 함수, 인덱스 등 모든 데이터베이스 객체의 이름은 반드시 전체 소문자형태의 `snake_case`로만 작성합니다.
* **SQL 예약어 사용 금지:** SQL 예약어(예: `select`, `table`, `user`, `order`, `group`, `date` 등)는 객체나 컬럼의 이름으로 절대 사용할 수 없습니다.

---

## 2. 데이터베이스 객체 접두사 (Prefix)

모든 데이터베이스 객체는 유형에 따라 아래와 같이 엄격하게 정의된 접두사 규칙을 따라야 합니다.

| 객체 유형 (Object Type) | 접두사 패턴 (Prefix) | 설명 및 예시 |
| :--- | :--- | :--- |
| **테이블 (Table)** | `t_` | `t_users`, `t_order_items` |
| **뷰 (View)** | `v_` | `v_active_subscriptions` |
| **저장 프로시저 (Stored Procedure)** | `[데이터베이스명약어]_` | 데이터베이스 이름의 약어(소문자)와 언더바 조합.<br>_예시 (monitoring_survey_verification의 경우):_ `msv_calculate_metrics` |
| **함수 (Function)** | `f_` | `f_get_user_balance` |

---

## 3. 기본키(PK) 및 외래키(FK) 참조 규칙

* **기본키 (Primary Key):**
  * 모든 테이블은 단 하나의 기본키 컬럼을 가져야 하며, 이름은 반드시 `id`로 지정합니다.
  * **데이터 타입:** 각 데이터베이스 플랫폼에서 제공하는 숫자 타입 중 가장 큰 크기를 가지는 자동 증가(Auto-Increment) 형식이어야 합니다. (예: PostgreSQL의 `BIGSERIAL`, MS-SQL의 `BIGINT IDENTITY`)
* **외래키 참조 (Foreign Key References):**
  * 다른 테이블의 기본키를 참조할 때는 컬럼 이름을 `참조테이블명_id` 형식으로 지정합니다. (예: `t_users_id`)
  * **물리 제약 조건 생성 금지:** 데이터베이스 엔진 레벨에서 물리적인 외래키 제약 조건(`FOREIGN KEY` constraints)은 **절대 생성하지 않습니다.** 테이블 간의 관계는 오직 논리적으로만 유지합니다.
* **쿼리 별칭(Alias) 규칙:**
  * 저장 프로시저 내부 또는 복잡한 쿼리에서 여러 테이블을 조인(Join)하여 여러 `id` 컬럼이 동시에 호출될 때도, 반드시 `as 테이블명_id`와 같이 명시적인 별칭을 사용해야 합니다.

---

## 4. 컬럼 작명 및 필수 필드 규칙

* **날짜/시간(Timestamp/Date) 컬럼:** 모든 날짜 및 시간 관련 필드의 이름은 반드시 `_at`으로 끝나야 합니다.
* **테이블 필수 공통 컬럼:** 시스템에 생성되는 모든 테이블에는 데이터 추적 및 관리를 위해 아래의 3개 컬럼을 반드시 포함해야 합니다.

```sql
delete_yn  -- 삭제 여부 상태 관리 (예: 'Y'/'N' 또는 이와 동등한 데이터 구조)
create_at  -- 행(Row)이 최초로 생성된 일시 기록
update_at  -- 행(Row)이 마지막으로 수정된 일시 기록
```

---

## 5. 제약 조건 작명 규칙 (Constraints - 선택 사항)

시스템 자동 생성 이름을 사용하지 않고 제약 조건 및 인덱스의 이름을 명시적으로 지정할 경우, 다음의 구조를 엄격하게 준수합니다.

* **기본키 (PK):** `pk_` + `테이블명`
  * *예시:* `pk_t_users`
* **외래 참조 (FK - 논리적 관계 명시용):** `fk_` + `본테이블명` + `_` + `참조테이블명`
  * *예시:* `fk_t_orders_t_users`
* **고유키 (Unique / UK):** `uk_` + `테이블명` + `_` + `컬럼명`
  * *예시:* `uk_t_users_email`
* **인덱스 (Index / IDX):** `idx_` + `테이블명` + `_` + `컬럼명`
  * *예시:* `idx_t_users_last_name`

---

## 6. 실제 참조 예시 (DDL 템플릿)

아래는 AI 지침에 따라 엄격하게 설계된 추상 DDL 디자인 템플릿 예시입니다.

```sql
-- PostgreSQL 기준 예시 템플릿
CREATE TABLE t_customer_orders (
    id BIGSERIAL,
    t_users_id BIGINT NOT NULL, -- t_users 테이블을 논리적으로 참조
    order_amount NUMERIC(15,2) NOT NULL,
    delete_yn CHAR(1) DEFAULT 'N' NOT NULL,
    create_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    update_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    CONSTRAINT pk_t_customer_orders PRIMARY KEY (id),
    CONSTRAINT uk_t_customer_orders_order_tracking UNIQUE (t_users_id, create_at)
);

CREATE INDEX idx_t_customer_orders_create_at ON t_customer_orders(create_at);
```
