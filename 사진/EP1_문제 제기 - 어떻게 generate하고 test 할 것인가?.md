

어떻게 해당 프로세스의 신뢰성을 담보할 수 있을 것인가?

### 공통된 문제 의식

1. 어떻게 더 잘 generate 할 수 있을까?
	1. 어떻게 요구사항을 더 명료하게 정의할 수 있을까?
	2. 어떻게 이 요구사항을 AI에게 더 명료하게 전달할 수 있을까?
	3. 어떻게 이 요구사항 구현을 강제할 수 있을까?
	4. 어떻게 요구사항(복제자, 유전자) - 코드(운반자, 몸)가 계속해서 일치하도록 관리할 수 있을까?
	
2. 어떻게 test 할 수 있을까?
	1. 어떻게 요구사항을 더 잘 test 하도록 지시할 수 있을까?
	2. 어떻게 

### 문제상황과 행동 정리
 
- **문제 상황1:** 민감 개인 정보 자동 메일 발송 서비스
	- 수십개의 조합 별로 수십명에게 각각 개별 확인서 PDF를 첨부하여 메일을 발송해야 했음. 현재 확인서는 하나의 통합된 파일로 존재하는 상황임. 메일 발송시 제목, 본문, 파일, 참조, 숨은 참조 등을 통제해야 하는 상황.

- 행동 1 : 발송계획보고서 작성까지 위임.

- **문제상황2 :** 웹 데이터 수집 서비스
	- 다수의 웹사이트에서 데이터를 수집할 수 있는 자동화 시스템을 구축해야 하는 상황. 어떻게 데이터 수집이 적절하게 이루어지고 있다고 진실성을 담보할 수 있을까? 어떻게 테스트를 설계해야 할까? 어떻게 비개발자 이용자들이 이 시스템을 믿게 할 수 있을까?

- **행동2 :** 데이터 수집 단 -> csv 결과를 보고 cc에게 검수하라고 지시. 자동화 검증 실행 단, 헬스 체크

- **문제상황3 :** 사내 DX
	- 사내 자체 내부 관리 웹 및 관리 시스템 구축 과정에서, 어떻게 해당 기능을 구현하게 해야할까. 그리고 어떻게 해당 기능이 잘 작동함을 확인할 수 있을까? 데이터가 CRUD 작동이 정상적으로 이루어지고 있는가? 프론트, 백, DB의 각 층위에서 보안 리스크가 잘 통제되고 있는가?

- **행동3 :** CC에게 지시하고 있지만 통제 확인하지 못하고 있음. 



### 답의 가능성 - SDD

## 작업 설명

작업에 대한 개요와 관련 상세정보를 제공하세요.



<https://claude.ai/public/artifacts/b4a762f6-fdd3-4495-9613-48243802d299>

<https://docs.google.com/document/d/1wwos8D3HwVWRpVqRIZfl4o7Y73oecxLxaSVOPdxhj1s/edit?usp=sharing>

<https://blog.davidlapsley.io/engineering/process/best%20practices/ai-assisted%20development/2026/01/11/spec-driven-development-with-llms.html>

<https://blog.davidlapsley.io/engineering/process/best%20practices/ai-assisted%20development/2026/01/12/sdld-specification-format.html#:~:text=Every%20feature%20in%20an%20SDLD,specific%20directory>



- g
    
    [https://alistairmavin.com/ears/](https://alistairmavin.com/ears/)
    

### 키워드

1. scope, non scope
    
2. EARS
    
3. glossary
    
4. test and exampel
    
5. 문서야 말로, 그 소프트웨어의 복제자 그 자체이다.
    
    1. 언어를 통해서 지시를 내리는 게 아니라 문서를 통해 내려야 한다.
    
    ### **1.3. LLM을 위한 문서 인터페이스의 필수 요건**
    
    분석된 연구 자료를 종합할 때, Claude Code와 같은 고성능 에이전트와 협업하기 위한 문서 인터페이스는 다음과 같은 5가지 핵심 요건을 충족해야 한다.
    
    2. **기계 가독성(Machine-Readability)과 인간 가독성(Human-Readability)의 균형**: AI가 파싱(Parsing)하기 쉬운 구조(Markdown, JSON, YAML)이면서 동시에 인간이 직관적으로 검토하고 수정할 수 있어야 한다.
    3. **진실의 단일 원천(Single Source of Truth, SSOT)**: 요구사항이 채팅 로그에 파편화되지 않고, 버전 관리 시스템(Git) 내의 특정 파일로 관리되어야 한다.
    4. **원자성(Atomicity)과 모듈화**: 복잡한 구현을 독립적인 작업 단위로 분해하여 AI의 주의력 분산(Attention dispersion)을 방지해야 한다.
    5. **검증 가능성(Verifiability)**: 사양 그 자체가 테스트 코드로 변환되거나, 구현 완료 여부를 판단하는 체크리스트 기능을 수행해야 한다.
    6. **동적 진화(Dynamic Evolution)**: 개발 과정에서 발견되는 새로운 제약 조건이나 기획 변경을 즉시 반영할 수 있는 유연한 구조여야 한다.

### **원칙 1. 의도와 구현의 분리 (Decoupling Intent from Implementation)**

문서 인터페이스는 "어떻게(How)"가 아닌 **"무엇을(What)"과 "왜(Why)"**에 집중해야 한다.

- **이유**: AI에게 구현 세부 사항(예: "React의 useState를 써서...")을 미리 지시하면, AI는 더 나은 최적화 방안을 찾지 않고 지시대로만 수행하려 한다. 반면, "사용자 입력에 따라 실시간으로 UI가 반응해야 한다"는 의도를 전달하면, AI는 프로젝트 컨텍스트에 맞는 최적의 상태 관리 라이브러리를 제안할 수 있다.
- **적용**: `proposal.md` 상단에 반드시 `Context & Goal` 섹션을 배치하여 비즈니스 가치를 명시한다.

### **원칙 3. 진실의 단일 원천 (Single Source of Truth, SSOT)**

합의된 모든 사항은 채팅 로그가 아닌 **버전 관리 가능한 파일**에 기록되어야 한다.

- **이유**: 채팅 로그는 휘발적이며 검색이 어렵다. AI가 "아까 말한 대로 해줘"를 이해하지 못하는 이유는 SSOT가 없기 때문이다. 모든 결정 사항은 `specs/` 디렉토리 내의 문서로 병합(Merge)되어야 하며, AI는 항상 이 문서를 참조하여 코드를 작성해야 한다.

### **원칙 4. 명시적 제외 (Explicit Exclusion)**

범위 충돌(Scope Creep)을 막기 위해, 무엇을 할 것인지 뿐만 아니라 **"무엇을 하지 않을 것인지(Non-Goals)"**를 명시해야 한다.

- **이유**: AI는 종종 과도한 친절함으로 요청하지 않은 기능(예: 불필요한 리팩토링, 스타일 변경)을 수행하여 기존 코드를 망가뜨린다. "이번 변경에서는 인증 모듈을 건드리지 않는다"와 같은 명시적 제약(Negative Constraint)은 이러한 부작용을 효과적으로 차단한다.

### **원칙 5. 루프 검증 (Closed-Loop Verification)**

모든 명세는 **검증 가능한 조건(Acceptance Criteria)**을 포함해야 한다.

- **이유**: "빠르게 동작해야 한다"는 모호한 명세는 오해를 낳는다. "200ms 이내에 응답해야 한다" 또는 "A 시나리오 테스트를 통과해야 한다"와 같이 기계적으로 판별 가능한 조건을 제시해야 AI가 자신의 구현 결과를 스스로 평가(Self-Correction)할 수 있다.

## 핵심 아이디어 → 사양서는 게약서이다.

1. 승인 기준 - 정확하게 지정
2. llm은 이에 따라 이행
3. 테스트 - 구현이 사양과 일치하는지 확인

---

1. 사양서는 저장소에 있어야함.

| **Keyword** | **Meaning**       | **Example**                     |
| ----------- | ----------------- | ------------------------------- |
| WHEN        | Trigger condition | WHEN a client sends GET /health |
| THE         | System component  | THE System                      |
| SHALL       | Mandatory         | SHALL return HTTP 200           |
| SHALL NOT   | Forbidden         | SHALL NOT log secrets           |
| IF          | Conditional!      | IF the cache is nil             |


### SDD를 진행하면서 느꼈던 것들. 

8. SDD
	1. 그냥 프롬프트로 요구사항을 정리하면 관리가안된다. 동기화가 안된다.  
	2. 문제의식
		1. 프로젝트의 목표인 그 spec의 기능 자체가 급격하게 추가되거나 확장되었을 때, 그리고 변경되었고, 시도해보고 싶을 떄 그 ssot를 어떻게 관리할 것인가? 즉, 단일 spec의 기준은 무엇인가?
	3. 그리고 그 맥락을 어떻게 다음 spec에 연결시킬 것인가
	4. 그리고 어떻게 추가적인 요구사항들 사소한 지시들을 spec 문서에 잘 통합시킬 수 있는가? 
