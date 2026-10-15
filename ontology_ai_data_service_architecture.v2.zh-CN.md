# Ontology 与 Ontology Service：企业 AI 数据服务中的语义层

本文重点不是具体工程实现，而是明确 Ontology 是什么、Ontology Service
应该处于什么位置，以及它与 Agent、数据服务、物理数据源之间的边界。

------------------------------------------------------------------------

## 1. 核心结论

可以把 Ontology 最简洁地理解为：

> **Ontology 是对某个业务领域中现实世界概念的抽象模型。**

它主要回答：

1.  这个业务世界里 **有哪些实体/概念**？
2.  实体有哪些 **属性**？
3.  实体之间有什么 **关系**？
4.  这些概念和关系有哪些 **语义、约束和业务规则**？

因此：

> **Ontology首先描述"现实世界中的概念是什么"，而不是直接描述数据库里的表和字段。**

企业中的 MySQL、TiDB、HDFS、Kafka、ClickHouse、ES、HBase、Redis、Hive
等数据源，是这些现实世界概念在企业信息系统中的**物理数据表示/投影**。

于是形成：

``` text
                    Reality
                       │
                       │ 抽象
                       ▼
                  Ontology
                       │
           ┌───────────┼───────────┐
           │           │           │
        Entity      Relation    Constraint
           │
           │ 映射
           ▼
       Data Mapping
           │
     ┌─────┼─────┬──────┐
     ▼     ▼     ▼      ▼
   MySQL  ES  ClickHouse Hive
```

这是本次讨论最重要的认识。

------------------------------------------------------------------------

## 2. Ontology 不是"数据库 Schema 的高级包装"

例如现实世界中存在一个概念：

``` text
User
```

Ontology 中首先定义：

``` text
Entity: User

Properties:
    userId
    name
    phone
    registerTime
    status

Relationships:
    User ──owns──> Account
    User ──uses──> Device
    User ──creates──> Order
```

然后企业内部可能存在：

``` text
MySQL.user
ES.user_profile
Hive.user_snapshot
ClickHouse.user_behavior
Redis.user_realtime_status
```

它们都可能只是 `User` 这个现实概念在不同系统中的数据表示。

因此：

``` text
User ≠ MySQL.user
```

更准确的是：

``` text
MySQL.user
    ──represents──>
Ontology.User
```

Ontology 和物理数据之间应该存在一个明确的映射层。

------------------------------------------------------------------------

## 3. 三层理解：现实世界、Ontology、物理数据

可以把整个体系理解成三层。

``` text
                    现实业务世界
                         │
                         │ 抽象
                         ▼
                  ┌──────────────┐
                  │   Ontology   │
                  │              │
                  │ Entity       │
                  │ Property     │
                  │ Relationship │
                  │ Constraint   │
                  └──────┬───────┘
                         │
                         │ Mapping
                         ▼
                  ┌──────────────┐
                  │ Data Assets  │
                  │              │
                  │ Table        │
                  │ Index        │
                  │ Topic        │
                  │ Dataset      │
                  └──────┬───────┘
                         │
                         ▼
              MySQL / TiDB / ES /
              Kafka / HDFS / Hive /
              ClickHouse / HBase /
              Redis / ...
```

其中：

### Reality

企业真正关心的业务世界：

``` text
User
Account
Device
Order
Product
Transaction
LoginEvent
...
```

### Ontology

对这些概念进行抽象和形式化：

``` text
User
 ├── userId
 ├── name
 └── status

User ──owns──> Account
User ──uses──> Device
User ──creates──> Order
```

### Data Assets

现实企业 IT 系统里真正存在的数据：

``` text
mysql.user
es.user_profile
clickhouse.login_event
hive.user_daily
kafka.login_event
...
```

------------------------------------------------------------------------

## 4. Entity Type 与具体 Instance 也需要区分

Ontology 首先描述的是**实体类型（Entity Type）**：

``` text
User
Device
Order
Product
Account
```

而企业数据中的具体记录代表的是**具体实例（Instance）**：

``` text
User#12345
Order#99881
Device#AABBCC
```

关系可以理解成：

``` text
                    Ontology
                       │
                  Entity Type
                       │
                     User
                       │
                 Mapping Rule
                       │
                       ▼
                 Data Asset
                       │
                 mysql.user
                       │
                       ▼
                  Data Record
                       │
                 id = 12345
                       │
                       ▼
                User#12345
```

这意味着：

> **Ontology 不是保存企业所有数据，而是定义"这些数据代表什么"。**

------------------------------------------------------------------------

## 5. Ontology Service 应该是什么

如果把 Ontology 落地为独立服务，可以将它理解为：

> **企业 AI 数据架构中的 Semantic Control Plane（语义控制面）。**

它不是 Data Plane。

它不应该成为一个新的超级数据库，也不应该负责直接执行所有 SQL、ES
Query、Kafka 操作等。

它的核心职责是：

``` text
                  Ontology Service
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Business Model    Data Mapping      Capability
        │                │                │
   “是什么”           “在哪里”          “怎么做”
```

------------------------------------------------------------------------

## 6. Ontology Service 的核心职责

### 6.1 Business Semantic Model

定义业务世界：

``` text
Entity
Property
Relationship
Definition
Constraint
```

例如：

``` text
User
 ├── userId
 ├── name
 ├── phone
 └── registerTime

User ──owns──> Account
User ──uses──> Device
User ──creates──> Order
```

这是 Ontology 最核心的部分。

------------------------------------------------------------------------

### 6.2 Data Mapping

把逻辑世界映射到物理数据世界。

例如：

``` text
User.userId
    ↓
mysql.user.id

User.name
    ↓
mysql.user.user_name

User.phone
    ↓
mysql.user.mobile
```

同一个逻辑实体也可能对应多个数据资产：

``` text
                    User
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    MySQL            ES           Hive
       │             │             │
   主数据          搜索模型       历史快照
```

这也是 Agent 不应该直接面对大量物理表、字段名称的原因。

------------------------------------------------------------------------

### 6.3 Relationship / Semantic Graph

Ontology 不仅描述实体，还描述实体之间的语义关系：

``` text
User ──owns──> Account
User ──uses──> Device
User ──creates──> Order
Order ──contains──> Product
Transaction ──belongs_to──> User
```

这使 Agent 能够进行领域层面的关系发现，而不是仅仅依赖数据库 Foreign
Key。

------------------------------------------------------------------------

### 6.4 Data Lineage

企业数据通常存在：

> 原始数据 → 加工数据 → 聚合数据 → 业务数据

例如：

``` text
Kafka
login_event_raw
      │
      ▼
Flink
      │
      ▼
ClickHouse
login_event_detail
      │
      ▼
Hive
user_login_daily
      │
      ▼
ClickHouse
user_risk_profile
```

Ontology / Semantic Service 可以记录这些数据之间的语义关系和血缘关系：

``` text
login_event_detail
    derived_from
login_event_raw
```

以及：

``` text
user_login_daily
    derived_from
login_event_detail
```

这对于 Agent 判断数据来源、口径、时效性和可信度非常重要。

------------------------------------------------------------------------

### 6.5 Capability / Operation Description

Ontology 不应该只告诉 Agent：

> "这个东西是什么。"

还应该告诉 Agent：

> "围绕这个东西，可以做什么。"

例如：

``` text
User
 ├── getUser()
 ├── getOrders()
 ├── getLoginHistory()
 └── getRiskProfile()
```

更进一步，可以描述 Action 的契约：

``` text
freezeAccount

input:
    accountId
    reason

permission:
    RiskOperator

precondition:
    account.status == ACTIVE

executor:
    AccountService.freeze()
```

这里非常重要：

> **Ontology 可以描述"如何操作"，但不一定自己执行操作。**

真正执行仍然由：

``` text
AccountService
QueryService
SearchService
WorkflowService
...
```

完成。

------------------------------------------------------------------------

### 6.6 Governance / Constraints

Ontology Service 还可以描述：

``` text
数据 Owner
数据质量
数据更新时间
数据敏感级别
权限要求
数据有效期
数据地域
数据成本
```

例如：

``` text
User.phone
    ↓
PII
    ↓
需要特定权限
```

或者：

``` text
user_risk_profile
    ↓
T+1 数据
    ↓
不是实时数据
```

这能避免 Agent 选择错误的数据源或错误地解释数据。

------------------------------------------------------------------------

## 7. Ontology 与 Data Service 的边界

这是本次讨论中形成的一个非常重要的架构原则：

> **Ontology 描述世界；Data Service 操作世界。**

建议边界：

``` text
                         Agent
                           │
                           ▼
                 ┌───────────────────┐
                 │   Ontology        │
                 │                   │
                 │  What             │
                 │  Where            │
                 │  Relationship     │
                 │  How              │
                 │  Constraint       │
                 └─────────┬─────────┘
                           │
                    Semantic Plan
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Query API        Search API       Action API
          │                │                │
          ▼                ▼                ▼
      ClickHouse          ES          Business Service
```

Ontology Service 不应该逐渐变成：

``` text
“什么都自己查”
“什么都自己算”
“什么都自己执行”
```

否则最终容易演化成一个新的超级数据中间件。

更合理的定位是：

> **作为 Agent 认识企业世界的统一语义层。**

------------------------------------------------------------------------

## 8. Agent 为什么需要 Ontology

传统方式：

``` text
User
 ↓
Agent
 ↓
“找一个叫 user 的表”
 ↓
猜测
```

企业环境中可能存在：

``` text
mysql.user
tiDB.account
es.user_profile
hive.user_snapshot
clickhouse.user_behavior
redis.user_status
```

Agent 很难仅凭表名判断它们与业务概念的关系。

有 Ontology 后：

``` text
User
 │
 ├── masterProfile
 │      ↓
 │   MySQL.user
 │
 ├── searchProfile
 │      ↓
 │   ES.user_profile
 │
 ├── behaviorHistory
 │      ↓
 │   ClickHouse.user_behavior
 │
 └── realtimeStatus
        ↓
     Redis.user_status
```

Agent 首先认识：

``` text
User
```

然后根据具体任务选择：

``` text
masterProfile
searchProfile
behaviorHistory
realtimeStatus
```

最终才进入具体的数据访问层。

------------------------------------------------------------------------

## 9. 一个完整的 AI Data Agent 架构

综合本次讨论，可以形成：

``` text
                              User
                                │
                                ▼
                              Agent
                                │
                                │ 理解目标
                                ▼
                     ┌─────────────────────┐
                     │   Ontology Service  │
                     │                     │
                     │  Business Model     │
                     │  Entity             │
                     │  Property           │
                     │  Relationship       │
                     │  Constraint         │
                     │                     │
                     │  Data Mapping       │
                     │  Lineage            │
                     │  Capability         │
                     │  Governance         │
                     └──────────┬──────────┘
                                │
                         Semantic Plan
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                 ▼
        Query Service      Search Service    Action Service
              │                 │                 │
              ▼                 ▼                 ▼
          ClickHouse            ES          Business API
          MySQL                 HBase       Workflow
          TiDB                  Redis       ...
          Hive
              │
              ▼
       Physical Data Sources
```

这里每层承担不同职责：

  层                    核心问题
  --------------------- --------------------------------
  Agent                 用户到底想完成什么？
  Ontology              这个业务世界是什么样？
  Data Mapping          这些概念在企业数据中如何表示？
  Lineage               数据从哪里来、如何加工？
  Capability            围绕这些对象可以做什么？
  Data/Action Service   如何真正执行？
  Physical Data         数据实际存在哪里？

------------------------------------------------------------------------

## 10. Ontology 可以理解成 Agent 的"企业世界模型"

这是本次讨论中最值得保留的一句话：

> **Agent 不应该直接学习企业所有数据库的
> Schema；它应该首先认识企业的业务世界，而 Ontology
> 就是这个世界的机器可理解模型。**

因此：

``` text
LLM / Agent
    │
    │ 认识
    ▼
Ontology
    │
    │ 映射
    ▼
Data Access Layer
    │
    │ 操作
    ▼
Physical Data
```

可以进一步压缩成：

``` text
Ontology      = 世界是什么
Data Mapping  = 世界在数据里如何表示
Data Service  = 如何读取世界
Action Service= 如何改变世界
Agent         = 为了目标如何利用这些能力
```

------------------------------------------------------------------------

## 11. 与 Palantir Ontology 的关系

Palantir 的 Ontology 思路与上述架构有明显相似性。

Palantir 将 Ontology 作为企业业务世界的核心语义/操作抽象，并进一步把：

``` text
Objects
Properties
Links
Actions
Functions
Security
```

等能力纳入 Ontology 体系。

因此 Palantir 的 Ontology 最终比本文讨论的"轻量 Ontology
Service"更重，更接近：

``` text
Data
+
Semantic Model
+
Logic
+
Action
+
Security
+
Workflow
```

而本次讨论形成的方案更倾向于：

``` text
              Ontology Service
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Semantic       Mapping      Capability
      Model         Model         Model
       │             │             │
       └─────────────┼─────────────┘
                     │
                     ▼
              External Services
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Query        Search       Action
      Service      Service      Service
```

即：

> **先把 Ontology做成一个相对"薄"的语义控制面，把真正的数据访问、搜索、计算、业务操作放在周边专门服务中。**

这有利于保持 Ontology 的核心关注点：

> **准确描述企业业务世界，并建立业务概念与实际数据/能力之间的映射。**

如果未来业务需要，再逐渐向更完整的 Operational Ontology 演进。

------------------------------------------------------------------------

## 12. 与用户现有规则引擎思维的对应

本次讨论还有一个很有价值的连接点。

用户当前熟悉的规则引擎架构：

``` text
QSPL / GSPL
      ↓
Parser
      ↓
AST
      ↓
IR
      ↓
Optimizer
      ↓
Execution Plan
      ↓
Runtime
```

AI Data Agent 可以形成类似的抽象：

``` text
Natural Language
      ↓
LLM
      ↓
Ontology
      ↓
Semantic IR
      ↓
Query / Action Plan
      ↓
Execution Services
      ↓
Data Sources
```

两者的共同思想是：

> **不要让自然语言直接落到具体执行；中间需要一个明确的、可理解、可验证的语义层/中间表示。**

从这个角度看，Ontology 可以被理解成 AI Data Agent 的一种 **Domain Type
System / Semantic Model**。

------------------------------------------------------------------------

## 13. 最终形成的整体认知

本次讨论最终可以浓缩成下面这张图：

``` text
                         现实业务世界
                              │
                              │ 抽象
                              ▼
                       ┌─────────────┐
                       │  Ontology   │
                       │             │
                       │  Entity     │
                       │  Property   │
                       │  Relation   │
                       │  Constraint │
                       └──────┬──────┘
                              │
                         Data Mapping
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
             MySQL           ES         ClickHouse
             TiDB           HBase          Hive
             Kafka          Redis          HDFS
                │             │             │
                └─────────────┼─────────────┘
                              │
                       Data / Action
                         Services
                              │
                              ▼
                            Agent
                              ▲
                              │
                            User
```

核心思想只有一句话：

> **Ontology 不负责保存企业世界的全部数据，也不负责亲自完成所有操作；它负责定义"这个世界是什么"，并把这个抽象世界与企业真实存在的数据和能力连接起来，让 Agent 可以基于统一的语义模型理解、规划和调用企业能力。**

------------------------------------------------------------------------

## 14. 当前阶段最值得保留的架构原则

### 原则一：先抽象现实世界，再映射数据

不要从：

``` text
MySQL.user
ES.user_profile
Hive.user_daily
```

出发构造 Ontology。

应该从：

``` text
User
Account
Device
Order
Transaction
```

这样的业务概念出发。

------------------------------------------------------------------------

### 原则二：Ontology 与 Physical Data 解耦

``` text
Ontology.User
```

不应该等价于：

``` text
mysql.user
```

而应该存在明确的 Mapping。

这样底层数据技术和存储实现可以变化，而 Agent 的业务语义保持稳定。

------------------------------------------------------------------------

### 原则三：Ontology 描述"如何操作"，但不必亲自执行

``` text
Ontology
   ↓
Capability / Action Contract
   ↓
External Service
   ↓
Actual Execution
```

保持语义层与执行层的职责清晰。

------------------------------------------------------------------------

### 原则四：Ontology 是 Agent 的企业世界模型

Agent 面对的第一层不是：

``` text
Table
Column
Topic
Index
```

而是：

``` text
Entity
Property
Relationship
Business Meaning
Capability
Constraint
```

之后再通过 Ontology Mapping 找到具体的数据和能力。

------------------------------------------------------------------------

### 原则五：先做"薄 Ontology"，再决定是否扩展

第一阶段重点可以放在：

``` text
Business Semantic Model
+
Data Mapping
+
Relationship
+
Lineage
+
Capability Description
+
Governance / Constraint
```

而不要一开始就把：

``` text
Query Engine
Storage
Workflow Runtime
Action Runtime
```

全部塞进 Ontology Service。

------------------------------------------------------------------------

## 15. 一句话总结什么是 Ontology

> **Ontology 是企业现实业务世界的抽象模型；Ontology Service 是这个模型的语义控制面。它让 Agent 先理解"企业世界里有什么、它们是什么关系、这些概念在真实数据中如何表示、可以通过什么能力操作"，再由外围 Data/Action Services 完成真正的数据访问与业务执行。**

## 16. 更大的挑战：Ontology 不是一次性建设，而是持续演化的语义基础设施

前面的讨论主要强调了 Ontology 建设初期的巨大工作量：需要把企业已有的数据治理、数据字典、元数据、血缘、知识图谱、业务文档等信息，进一步整理成统一的业务语义模型。

但实际上，这还不是最难的部分。

**更大的挑战是：Ontology 一旦进入生产环境，就不再是一个“一次建设、长期不变”的项目，而会随着业务和数据系统持续变化。**

企业的现实世界一直在变化：

- 新业务上线；
- 新实体出现；
- 原有实体增加或删除属性；
- 实体之间出现新的业务关系；
- 原有业务关系发生变化；
- 业务规则发生变化；
- 数据库表拆分、合并、迁移；
- Kafka Topic、ES Index、Hive 表、ClickHouse 表发生变化；
- 字段重命名、类型变化、语义变化；
- 数据源被替换；
- 数据加工链路发生变化；
- 同一个业务概念在不同部门中的定义发生变化。

因此，Ontology 面临的实际上是一个持续的 **Semantic Evolution（语义演化）** 问题。

## 17 建设期问题：把现实世界第一次“描述出来”

建设初期主要是：

```text
企业已有知识
   │
   ├── Data Catalog
   ├── Data Dictionary
   ├── Metadata
   ├── Data Lineage
   ├── Knowledge Graph
   ├── SQL / Schema
   ├── Business Documents
   └── Business Rules
          │
          ▼
     AI Candidate Mining
          │
          ▼
     Ontology Draft
          │
          ▼
     Human Validation
          │
          ▼
   Approved Ontology
```

这是一个巨大的知识整理和语义统一过程。

但这只是 **T0 时刻**。

真正进入生产之后，问题变成：

```text
                    ┌──────────────┐
                    │   Business   │
                    │   Evolution  │
                    └──────┬───────┘
                           │
                           ▼
┌──────────────┐    ┌──────────────┐
│ Data Sources │───►│ Data Changes │
└──────────────┘    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Semantic    │
                    │   Impact     │
                    │   Analysis   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Ontology     │
                    │ Evolution    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Validation / │
                    │   Approval   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ New Runtime  │
                    │   Semantics  │
                    └──────────────┘
```

所以，Ontology 的真正生命周期应该是：

```text
建设
 ↓
上线
 ↓
业务变化
 ↓
数据变化
 ↓
发现语义影响
 ↓
Ontology 更新
 ↓
重新验证
 ↓
Agent / Service 使用新语义
 ↓
继续变化
 ↺
```

---

## 18. Ontology Maintenance 可能比 Ontology Construction 更难

这是整个架构中非常值得强调的一点：

> **Ontology 最大的成本，未必是第一次建立，而可能是长期维护。**

因为企业系统具有非常强的动态性。

例如最简单的一个实体：

```text
Customer
├── customerId
├── name
├── phone
├── status
└── registerTime
```

随着业务发展：

### 第一阶段

增加会员体系：

```text
Customer
    │
    └── Member
```

### 第二阶段

增加企业客户：

```text
Customer
    ├── IndividualCustomer
    └── CorporateCustomer
```

### 第三阶段

CRM 系统和交易系统产生不同 Customer 定义：

```text
CRM.Customer
Trade.Customer
Risk.Customer
Marketing.Customer
```

这时候已经不是简单增加一个字段的问题，而是：

> **Ontology 中的 `Customer` 到底应该是什么？**

可能需要重新定义：

```text
Customer
    │
    ├── CRMCustomer
    ├── TradingCustomer
    ├── RiskSubject
    └── MarketingCustomer
```

甚至可能发现原来的 `Customer` 抽象本身就是错误的。

这就是典型的 **Semantic Refactoring（语义重构）**。

---

## 19. 数据结构变化不等于 Ontology 变化，但必须能够感知

这里还需要区分两个非常重要的概念：

> **Physical Schema Evolution ≠ Ontology Evolution**

例如：

```text
Ontology:

Customer.phone
```

底层可能从：

```text
mysql.customer.phone
```

迁移成：

```text
mysql.customer.mobile
```

Ontology 本身可能完全不需要变化：

```text
Customer.phone
        │
        ▼
mysql.customer.mobile
```

这只是 **Data Mapping Evolution**。

但是，如果业务含义发生变化：

```text
phone
```

从“客户注册手机号”变成：

```text
primaryContactPhone
```

那么就可能已经不是简单的数据映射变化，而是业务语义发生变化。

因此至少应该区分：

```text
Physical Schema Evolution
        │
        └── Data Mapping Evolution
                    │
                    ▼
             Ontology Evolution
                    │
                    ▼
              Business Evolution
```

不同层级的变化，影响范围完全不同。

---

## 20. Ontology Service 需要具备“变化感知能力”

因此，一个真正进入企业生产环境的 Ontology Service，不能只是：

```text
GET /entity/Customer
GET /relationship/Customer-Order
GET /mapping/Customer
```

它还需要能够回答：

> **“企业的数据和业务发生变化之后，我的语义模型哪些地方受影响？”**

例如：

```text
ClickHouse.customer_behavior
    customer_id
    phone
    behavior_type
```

发生 Schema Change：

```text
phone → mobile
```

Ontology 系统应该能够发现：

```text
customer_behavior.phone
        │
        ├── Ontology.Customer.phone
        ├── Rule A
        ├── Agent Capability B
        ├── Query Template C
        └── Report D
```

于是系统可以产生：

```text
Impact Analysis

Changed:
    customer_behavior.phone

Affected:
    Customer.phone
    Rule A
    Capability B
    Query Template C
    Report D

Required Action:
    Update Mapping
    Validate Ontology
    Re-test dependent capabilities
```

这时候 Ontology Service 就从一个“语义查询服务”，开始变成了一个：

> **Semantic Control Plane + Semantic Change Management System**

---

## 21. AI 在这里的价值可能比“初始录入”更大

建设初期 AI 可以帮助：

```text
Enterprise Knowledge
       ↓
AI
       ↓
Ontology Candidate
```

但真正长期运行以后，AI 更重要的作用可能是：

```text
Business / Data Changes
          ↓
      AI Detection
          ↓
    Semantic Impact
       Analysis
          ↓
 Ontology Change Proposal
          ↓
   Human Validation
          ↓
    Ontology Update
```

例如：

```text
Schema Diff
     +
SQL Diff
     +
Lineage Diff
     +
API Diff
     +
Business Document Diff
     +
Code Diff
     +
Actual Query / Agent Usage
          │
          ▼
         LLM
          │
          ▼
"Customer.phone 可能已经迁移到
 Customer.mobile，请确认 Ontology Mapping"
```

这比单纯让 AI “帮我录入 Ontology”更有价值。

因为企业最大的痛点不是：

> “我今天能不能把 Ontology 建出来？”

而是：

> **“半年以后，Ontology 还是不是现实世界的真实反映？”**

---

## 22. Ontology 应该具备 Version / Provenance / Approval

既然 Ontology 会持续变化，就不能把它当成一份普通配置文件。

至少应该具备：

```text
Entity / Property / Relationship
            │
            ├── Version
            ├── Owner
            ├── Source
            ├── Provenance
            ├── Confidence
            ├── Effective Time
            ├── Deprecated Time
            └── Approval Status
```

例如：

```text
Customer.phone

Version: 3
Status: Approved
Owner: CRM Domain
Source:
    CRM Data Dictionary
    mysql.customer.phone
    Business Manual v12

Previous:
    Customer.mobile

Changed:
    2026-09-20

Reason:
    CRM customer contact model changed

Impact:
    17 mappings
    5 query capabilities
    2 business rules
```

这样 Ontology 才真正具备企业级基础设施应有的可追溯性。

---

## 23. 更完整的 Ontology 生命周期

因此，可以把整个体系重新概括成：

```text
                 ┌──────────────────────┐
                 │   Enterprise World   │
                 │ Business + Data      │
                 └──────────┬───────────┘
                            │
                    Initial Discovery
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Ontology Construction│
                 │ AI + Human           │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Approved Ontology    │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Ontology Runtime     │
                 │ Agent / Services     │
                 └──────────┬───────────┘
                            │
                            │
          ┌─────────────────┴──────────────────┐
          │                                    │
          ▼                                    ▼
   Business Evolution                   Data Evolution
          │                                    │
          └────────────────┬───────────────────┘
                           ▼
                  Semantic Change Detection
                           │
                           ▼
                    Impact Analysis
                           │
                           ▼
                  AI Change Proposal
                           │
                           ▼
                    Human Approval
                           │
                           ▼
                    New Ontology
                           │
                           └───────────────↺
```

这个循环非常重要。

**Ontology 不是一个最终产物，而是一个持续演化的企业语义状态。**

---

## 24. 因此，Ontology 项目的真正工作量可以分成两部分

### 第一部分：Construction

```text
现实世界
   ↓
业务概念识别
   ↓
实体 / 属性 / 关系
   ↓
数据映射
   ↓
约束 / 规则 / 能力
   ↓
Ontology
```

这是“把世界描述出来”。

### 第二部分：Evolution

```text
世界发生变化
   ↓
发现变化
   ↓
判断语义影响
   ↓
修改 Ontology
   ↓
重新验证 Mapping / Capability / Rule
   ↓
发布新版本
```

这是“让描述一直跟得上世界”。

而第二部分可能是一个企业级 Ontology 系统真正长期、持续、昂贵的部分。

---

## 24. 一个非常重要的架构结论

因此，Ontology Service 如果真的作为企业 Agent 基础设施建设，不能只设计：

```text
Ontology Runtime API
```

还应该考虑：

```text
                 Ontology Platform
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
 Ontology Runtime   Ontology Studio   Change Engine
        │                │                │
        │                │                │
        ▼                ▼                ▼
     Agent          Human Modeling   Change Detection
     Service        / Approval       / Impact Analysis
                         │                │
                         └───────┬────────┘
                                 ▼
                          Version / Audit
```

其中：

### Ontology Runtime

服务 Agent 和其他系统：

```text
查询实体
查询关系
获取数据映射
发现能力
获取约束
```

### Ontology Studio

服务人：

```text
创建 Entity
定义 Property
定义 Relationship
配置 Mapping
定义 Constraint
审批 AI Proposal
查看语义模型
```

### Change Engine

服务“变化”：

```text
Schema Diff
Lineage Diff
API Diff
Business Rule Diff
Document Diff
Code / SQL Diff
Usage Analysis
```

然后：

```text
Change
  ↓
Detection
  ↓
Impact Analysis
  ↓
AI Proposal
  ↓
Human Review
  ↓
Versioned Ontology
  ↓
Runtime
```

这才构成一个完整的 Ontology Platform。

---

## 25. 最终重新理解 Ontology Service

到这里，Ontology Service 的定位可以进一步提升。

它并不只是：

> “给 Agent 提供业务实体定义的一个服务。”

而更接近：

> **企业现实世界的机器可读语义状态（Machine-Readable Semantic State）。**

而 Ontology Platform 要解决的，是：

```text
现实世界
    ↕
业务语义
    ↕
企业数据
    ↕
AI / Agent
```

之间长期保持一致的问题。

所以最终真正需要维护的不是某一张表、某一个 JSON Schema，而是：

```text
World
  ↕
Ontology
  ↕
Data
  ↕
Capability
  ↕
Agent
```

这几个层次之间的 **Semantic Consistency（语义一致性）**。

这也是为什么企业 Ontology 建设很容易从一个看起来“只是建一个 Service”的项目，逐渐演变成一个长期的企业级基础设施建设项目。

---

## 26. 对前面结论的一个重要补充

因此，之前“AI 可以降低 Ontology 建设成本”的说法，需要进一步精确化：

**AI 很难消灭 Ontology 的维护成本，但可以显著改变维护成本的结构。**

过去可能是：

```text
人工发现变化
    ↓
人工分析影响
    ↓
人工修改模型
    ↓
人工检查依赖
    ↓
人工发布
```

未来更可能是：

```text
系统自动发现变化
        ↓
AI 分析语义影响
        ↓
AI 生成修改 Proposal
        ↓
人确认关键语义
        ↓
自动更新依赖
        ↓
自动测试 / 验证
        ↓
发布
```

也就是说：

> **AI 最有价值的地方，不是替人维护 Ontology，而是把“持续维护”从纯人工劳动，变成 AI 驱动的人机协作语义治理。**

最终的目标不是：

> “让 Ontology 不需要人维护。”

而是：

> **“让人只处理真正需要业务判断的语义变化，而把发现、分析、影响传播、候选修改、验证等工作尽可能自动化。”**

这可能才是 Ontology × AI 在企业数据服务中的真正价值所在。
