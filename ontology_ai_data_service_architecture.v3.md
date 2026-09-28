# Ontology 与 Ontology Service：企业 AI 数据服务中的语义层

> 本文面向 AI  Agent、企业数据平台、数据服务和企业架构建设者，目标是完整回答四个问题：
>
> 1.  **Ontology / Ontology Service 是什么？**
> 2.  **企业应该如何建设 Ontology Service？**
> 3.  **Ontology 如何初始化、治理和持续维护？**
> 4.  **Agent 如何发现、理解并使用 Ontology Service？**
>
> 本文同时明确 Ontology 与 Knowledge Graph、Data Catalog / Metadata、Data Mapping、Data Lineage、Capability / Tool 等概念之间的边界，避免最终形成一个职责不清晰的"超级 Ontology Service"。

------------------------------------------------------------------------

# 1. 核心结论

可以把 Ontology 最简洁地理解为：

> **Ontology 是对某个业务领域中现实世界概念的抽象、形式化和机器可理解的语义模型。**

它主要回答：

1.  这个业务世界里有哪些实体和概念？
2.  实体有哪些属性？
3.  实体之间有什么关系？
4.  这些概念、属性和关系具有什么业务含义？
5.  存在哪些语义约束、业务约束和适用条件？

因此：

> **Ontology 首先描述"现实世界是什么"，而不是直接描述数据库里的表和字段。**

企业中的 MySQL、TiDB、HDFS、Kafka、ClickHouse、ES、HBase、Redis、Hive
等数据系统，则是现实业务世界在信息系统中的各种物理表示。

可以形成最基本的关系：

``` text
                     Reality
                        │
                        │ 抽象
                        ▼
                   ┌───────────┐
                   │ Ontology  │
                   │           │
                   │ Entity    │
                   │ Property  │
                   │ Relation  │
                   │ Constraint│
                   └─────┬─────┘
                         │
                      Mapping
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        MySQL           ES         ClickHouse
```

而 Ontology Service，则是把这套语义模型作为企业级服务提供出来：

> **Ontology Service 是企业 AI 数据架构中的 Semantic Control Plane（语义控制面）。**

它让 Agent
和其他上层应用能够先理解企业业务世界，再通过映射和能力发现进入实际的数据访问与业务操作。

------------------------------------------------------------------------

# 2. 为什么企业 AI 需要 Ontology

传统的数据访问方式通常从物理数据出发：

``` text
Agent
  │
  ▼
Table / Column / API / Index
  │
  ▼
猜测这些东西分别代表什么
  │
  ▼
执行查询或调用
```

但大型企业中的数据往往是高度分散的。

同一个业务概念可能同时存在于：

``` text
mysql.customer
es.customer_profile
clickhouse.customer_behavior
hive.customer_snapshot
redis.customer_status
kafka.customer_event
```

仅凭表名、字段名和 API 名称，很难可靠判断：

- 哪些数据代表同一个业务实体；
- 哪个字段代表什么业务概念；
- 哪些数据是主数据，哪些只是搜索模型或历史快照；
- 哪些关系是业务关系，哪些只是技术上的关联；
- 哪个数据源适合当前任务；
- 数据的实时性、权限、敏感级别和适用范围是什么。

因此，Agent 不应该首先认识企业的数据库，而应该首先认识：

``` text
Customer
Account
Order
Transaction
Device
RiskSubject
...
```

以及它们之间的业务关系。

于是形成：

``` text
User
  │
  ▼
Agent
  │
  │ 理解业务概念
  ▼
Ontology
  │
  │ 找到语义对应的数据和能力
  ▼
Mapping / Capability
  │
  ▼
Data / Action Services
  │
  ▼
Physical Data
```

这就是 Ontology 在 AI Data Service 架构中的核心价值。

------------------------------------------------------------------------

# 3. 概念边界

本次边界讨论的核心概念：
- Ontology
- Knowledge Graph
- Data Catalog
- Mapping
- Lineage
- Capability

这是整个架构最重要的概念边界之一。

这些系统都在"描述企业世界"，但描述的角度不同。

|概念|核心问题|主要关注对象|典型内容|与 Ontology 的关系|
|-|-|-|-|-|
|**Ontology**|世界是什么？|实体、属性、关系、约束、语义|Customer、Account、Order、Product；Customer owns Account；Customer 的定义、属性含义、关系约束、业务分类|定义业务世界的语义模型，是其他层理解业务概念的基础|
|**Knowledge Graph**|这个世界里现在有什么事实？|业务概念、实体、属性、关系、约束、语义|张三是 Customer；张三 owns Account-001；Account-001 belongs to Product-A；某订单属于某客户|Ontology 定义“允许如何描述世界”，KG 保存“这个世界中具体发生了什么”|
|**Data Catalog/Metadata**|企业把数据存在哪里？|数据库、表、字段、Topic、Index、Dataset、数据资产元数据|MySQL 表、ClickHouse Table、Kafka Topic、ES Index、字段类型、Owner、数据域、敏感等级、更新时间、数据质量信息|描述数据资产本身，而不是业务世界；Ontology 可以引用 Catalog 中的数据资产|
|**Data Mapping**|业务概念如何被企业数据表示？|业务语义到数据资产之间的映射|Customer.id → customer_info.customer_id；Customer.name → customer_info.name；一个业务属性由多个字段组合计算得到|连接 Ontology 与实际数据，是从“业务概念”到“数据表示”的桥梁|
|**Data Lineage**|数据从哪里来、如何产生和流转？|数据加工链路、来源、派生关系|Kafka → Flink → ClickHouse；A表字段 → B表字段 → C指标；ETL/ELT/DAG 加工链路|关注数据如何产生和流动，而不是业务概念本身；Ontology 可以利用 Lineage 做数据可信度、影响分析等|
|**Capability / Tool**|我们能够对这个世界做什么？|查询、搜索、分析、操作、工作流等能力|查询 Customer；搜索 Order；统计交易金额；创建订单；冻结账户；执行风控分析；调用业务 API|Ontology 描述“对哪些业务对象可以做什么”，Capability/Tool 负责真正执行|

这些概念在物理实现上可以协同甚至部署在同一个平台中，但在逻辑架构上应该保持清晰边界。

------------------------------------------------------------------------

# 4. Ontology：描述业务世界

例如一个企业客户领域：

``` text
Customer
 ├── customerId
 ├── name
 ├── phone
 └── riskLevel

Account
 ├── accountId
 └── status

Transaction
 ├── transactionId
 ├── amount
 └── timestamp

Customer ──owns──> Account
Account ──has──> Transaction
Customer ──places──> Order
```

Ontology 关注：

- Entity / Entity Type
- Property
- Relationship
- Business Meaning
- Semantic Type
- Constraint
- Applicability / Scope
- Definition

例如：

``` text
Customer.phone

semanticType:
    PhoneNumber

meaning:
    Customer primary contact phone

required:
    true
```

这与：

``` text
mysql.customer.phone

type:
    varchar(32)

owner:
    CRM Team
```

属于不同层次。

前者描述"这个概念在业务世界中是什么意思"，后者描述"这个数据资产在企业数据体系中是什么"。

------------------------------------------------------------------------

# 5. Entity Type 与 Instance：Ontology 不等于业务数据库

Ontology 首先描述的是**实体类型（Entity Type）**：

``` text
Customer
Account
Transaction
Device
Order
```

企业数据中的具体记录则是实例：

``` text
Customer#12345
Account#98765
Transaction#88881
```

可以用一个不完全严格、但非常直观的类比：

``` text
Ontology Entity Type ≈ Class
Ontology Instance   ≈ Object
```

因此：

``` text
Ontology Model
      │
      ├── Customer
      ├── Account
      └── Transaction

Runtime / Knowledge
      │
      ├── Customer#12345
      ├── Account#98765
      └── Transaction#88881
```

这意味着：

> **Ontology 定义"实例是什么、具有什么语义"，但不意味着 Ontology Model 必须保存所有实例数据。**

Ontology Runtime 可以通过 Knowledge Graph、Object Service 或 Data
Service 访问实例：

``` text
resolveConcept("customer")
        │
        ▼
     Customer
        │
        ▼
getCustomer("12345")
        │
        ▼
 Customer#12345
```

这样既保留了统一语义，又避免 Ontology Service
演化成一个巨大的业务数据库。

------------------------------------------------------------------------

# 6. Ontology 与 Knowledge Graph

Ontology 与 Knowledge Graph 高度相关，但职责不同。

最简单的理解是：

> **Ontology 定义如何描述这个世界；Knowledge Graph 保存这个世界当前存在的实例和事实。**

Ontology：

``` text
Customer
Account

Customer ──owns──> Account
```

Knowledge Graph：

``` text
Customer#12345
      │
      └── owns ──> Account#98765
```

进一步：

``` text
Ontology:
Account ──hasTransaction──> Transaction
```

KG：

``` text
Account#98765
      │
      ├── hasTransaction ──> Transaction#88881
      ├── hasTransaction ──> Transaction#88882
      └── hasTransaction ──> Transaction#88883
```

因此，一个自然的架构是：

``` text
                 Ontology
                    │
          ┌─────────┼─────────┐
          │         │         │
        Entity   Relation  Constraint
          │         │         │
          └─────────┼─────────┘
                    ▼
              Knowledge Graph
                    │
             ┌──────┼──────┐
             ▼      ▼      ▼
          Instance Instance Instance
```

物理实现上，Ontology 和 KG 可以存储在同一个系统中。

但：

> **物理合并不等于概念合并。**

------------------------------------------------------------------------

# 7. Data Catalog / Metadata：描述企业的数据资产

Data Catalog 是**数据资产视角**。

它关注：

``` text
MySQL
 └── customer
      ├── id
      ├── name
      ├── phone
      └── create_time

ClickHouse
 └── customer_behavior
      ├── customer_id
      ├── event_type
      └── event_time

Kafka
 └── customer_event
```

Catalog / Metadata 通常描述：

- Database / Schema
- Table
- Column
- Topic
- Index
- Dataset
- Data Type
- Owner
- Permission
- Data Quality
- Sensitivity
- Update Time
- Location
- Lifecycle

例如：

``` text
mysql.customer.phone

type:
    varchar(32)

owner:
    CRM Team

classification:
    PII
```

而 Ontology 描述：

``` text
Customer.phone

semanticType:
    PhoneNumber

meaning:
    Customer primary contact phone

required:
    true
```

两者不是同一个问题。

------------------------------------------------------------------------

# 8. Ontology 与 Data Catalog：引用，而不是复制

推荐：

``` text
Ontology
    │
    │ semantic mapping
    ▼
Data Catalog
    │
    ▼
Physical Data Assets
```

例如：

``` text
Ontology.Customer.phone
        │
        │ mapped_to
        ▼
DataCatalog:mysql.customer.phone
        │
        ▼
MySQL.customer.phone
```

Ontology 负责回答：

> Customer 的 phone 在业务上是什么意思？

Data Catalog 负责回答：

> 这个字段在哪里、是什么类型、谁负责、质量如何、是否敏感？

因此：

> **Ontology 应该引用 Data Catalog，而不是复制 Data Catalog。**

------------------------------------------------------------------------

# 9. Data Mapping：连接业务语义与企业数据

Data Mapping 是 Ontology 落地过程中非常关键的桥梁。

它回答：

> **Ontology 中的业务概念，在企业现有数据体系中如何被表示？**

例如：

``` text
Ontology.Customer
        │
        ├── mapped_to → MySQL.customer
        ├── mapped_to → ClickHouse.customer_behavior
        ├── mapped_to → ES.customer_profile
        └── mapped_to → Redis.customer_realtime
```

更细粒度：

``` text
Ontology.Customer.customerId
        │
        ├── mysql.customer.id
        ├── clickhouse.customer_behavior.customer_id
        └── redis.customer_status.customer_id
```

但在复杂企业中，不建议简单理解为：

``` text
Ontology Property
        ↓
Physical Column
```

更合理的是增加一个**语义数据表示层（Semantic Data Representation）**：

``` text
Ontology
    ↓
Semantic Data Representation
    ↓
Data Mapping
    ↓
Data Catalog
    ↓
Physical Data
```

例如：

``` text
Customer
   │
   ├── CustomerIdentity
   │       ↓
   │   mysql.customer
   │
   ├── CustomerBehavior
   │       ↓
   │   clickhouse.customer_behavior
   │
   └── CustomerRealtimeStatus
           ↓
       redis.customer_status
```

这一层的意义在于：

> **业务实体不应该被某一张数据库表定义；数据表示应该是业务语义与物理数据之间的独立抽象。**

------------------------------------------------------------------------

# 10. Data Lineage：描述数据如何产生和流转

Lineage 解决的是：

> **数据从哪里来？经过什么处理？最后流向哪里？**

例如：

``` text
Kafka.login_event
       │
       ▼
     Flink
       │
       ▼
ClickHouse.customer_behavior
       │
       ▼
Ontology.Customer.lastLoginTime
```

或者：

``` text
MySQL.customer
       │
      CDC
       ▼
Kafka.customer_change
       │
     Flink
       ▼
ClickHouse.customer_snapshot
```

这是数据流和数据血缘，而不是业务世界本身。

因此不建议：

``` text
Ontology
 ├── Entity
 ├── Relationship
 ├── Constraint
 └── Lineage Engine
```

更推荐：

``` text
Ontology
    │
    └── references
             ▼
       Lineage System
```

例如：

``` text
Ontology.Customer.lastLoginTime
        │
        ▼
ClickHouse.customer_behavior.last_login
        │
        ▼
      Lineage
        │
        ▼
Kafka.login_event
```

Ontology 可以利用 Lineage 判断：

- 数据来源；
- 数据新鲜度；
- 数据加工过程；
- 数据依赖；
- 影响范围。

但不应该因此拥有 Lineage Engine 的全部职责。

------------------------------------------------------------------------

# 11. Capability / Tool：描述"能做什么"

Capability 属于执行能力层。

例如：

``` text
getCustomerById
searchCustomer
getCustomerTransactions
getCustomerRisk
queryCustomerBehavior
freezeAccount
```

它回答：

> **我们能够对这个世界做什么？**

Capability Descriptor 可以描述：

``` yaml
name: getCustomerTransactions

input:
  customerId: CustomerId
  startTime: DateTime
  endTime: DateTime

output:
  transactions: Transaction[]

preconditions:
  - customerId must exist

permission:
  - customer.read

freshness:
  - near_realtime

provider:
  - TransactionService
```

推荐：

``` text
Ontology
    │
    └── describes / references
              ▼
       Capability Registry
              │
              ▼
       Execution Service
              │
              ▼
         Actual Data
```

这里需要明确：

> **Ontology 可以描述和发现能力，但不一定负责执行能力。**

真正执行查询、搜索、操作、事务和工作流的，仍然应该是专门的 Data / Action
/ Workflow Service。

------------------------------------------------------------------------

# 12. Ontology Service 的定位

如果把 Ontology 落地为独立服务，可以将其理解为：

> **Ontology Service 是企业 AI 数据架构中的 Semantic Control Plane（语义控制面）。**

它不是 Data Plane。

它不应该成为：

- 新的超级数据库；
- Universal Query Engine；
- 全功能 Knowledge Graph；
- Data Catalog 的替代品；
- Lineage Engine；
- Workflow Engine；
- MCP Gateway 的替代品。

它的核心职责可以归纳为：

``` text
Ontology Service
       │
       ├── Business Semantic Model
       │       “是什么”
       │
       ├── Semantic Data Model / Mapping
       │       “如何表示”
       │
       ├── Relationship / Constraint
       │       “如何理解”
       │
       ├── Capability Description
       │       “能做什么”
       │
       └── Governance
               “是否可信、何时生效、谁负责”
```

------------------------------------------------------------------------

# 13. Ontology Service 与 Data Service 的边界

一个非常重要的架构原则是：

> **Ontology 描述世界；Data / Action Service 操作世界。**

推荐的边界：

``` text
                         Agent
                           │
                           ▼
                  ┌───────────────────┐
                  │ Ontology Service  │
                  │                   │
                  │ What              │
                  │ Where             │
                  │ Relationship      │
                  │ Constraint        │
                  │ Capability        │
                  └─────────┬─────────┘
                            │
                     Semantic Plan
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
      Query API         Search API        Action API
          │                 │                 │
          ▼                 ▼                 ▼
     ClickHouse             ES         Business Service
        MySQL              HBase          Workflow
        TiDB                Redis
```

Ontology Service 不应该逐渐变成：

``` text
“什么都自己查”
“什么都自己算”
“什么都自己执行”
```

否则最终会演化成一个新的超级数据中间件。

更合理的定位是：

> **Ontology Service 是 Agent 认识企业世界、发现企业数据与能力的统一语义层。**

------------------------------------------------------------------------

# 14. 一个完整的逻辑架构

综合前面的概念，可以形成：

``` text
                              Agent
                                │
                          MCP / Skills
                                │
                                ▼
                     ┌─────────────────────┐
                     │  Ontology Platform  │
                     │                     │
                     │  Business Ontology  │
                     │  Constraints        │
                     │  Mapping            │
                     │  Capability Desc.   │
                     │  Semantic Discovery │
                     └──────────┬──────────┘
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
      Knowledge Graph      Data Catalog      Capability Registry
             │                  │                  │
             │                  │                  ▼
             │                  │            Execution Services
             │                  ▼                  │
             │           Physical Data Assets     │
             │                                     │
             ▼                                     ▼
        Entity Facts                         Actual Data
```

这里最重要的定位是：

> **Ontology Platform 是 Semantic Center，而不一定是 Data Center。**

它负责把：

``` text
业务世界
   ↓
业务语义
   ↓
企业数据
   ↓
执行能力
```

串起来。

但每一层的实际数据和执行能力仍然可以由专门系统负责。

------------------------------------------------------------------------

# 15. 逻辑边界不等于微服务边界

概念边界与物理服务边界不是一回事。

早期完全可以：

``` text
Ontology Platform
 ├── Ontology Model Store
 ├── Mapping Store
 ├── KG Store
 └── Catalog Connector
```

甚至暂时部署成一个服务。

只要逻辑上保持：

``` text
Ontology
Knowledge Graph
Mapping
Catalog
Capability
```

职责清晰即可。

随着系统规模扩大，再根据实际压力和组织边界拆分。

因此：

> **先建立正确的逻辑边界，再决定物理部署边界。**

------------------------------------------------------------------------

# 16. 如何建设 Ontology Service

Ontology 建设不是"设计几个 Entity JSON"这么简单。

真正的建设过程是：

``` text
企业现实世界
      │
      ▼
业务知识与数据资产发现
      │
      ▼
业务概念识别
      │
      ▼
Ontology 建模
      │
      ├── Entity
      ├── Property
      ├── Relationship
      └── Constraint
      │
      ▼
Semantic Data Representation
      │
      ▼
Data Mapping
      │
      ▼
Capability Mapping
      │
      ▼
Validation
      │
      ▼
Approved Ontology
      │
      ▼
Runtime
```

------------------------------------------------------------------------

# 17. 第一步：明确 Ontology 的边界和业务范围

不要一开始试图建立"整个企业 Ontology"。

应该先确定：

- Domain；
- Business Scope；
- 使用场景；
- 目标 Agent；
- 需要解决的问题；
- 需要接入的数据范围；
- 需要暴露的能力范围。

例如先建立：

``` text
Customer Domain
```

而不是：

``` text
Entire Enterprise Ontology
```

一个可行的初始范围可能是：

``` text
Customer
Account
Order
Transaction
Device
```

然后围绕明确的 Agent 场景逐步扩展。

------------------------------------------------------------------------

# 18. 第二步：发现和整理企业已有知识

Ontology 建设不能凭空开始。

企业通常已经存在大量知识来源：

``` text
Business Documents
Business Glossary
Data Dictionary
Data Catalog
Metadata
Data Lineage
Database Schema
API Definitions
Existing Knowledge Graph
Business Rules
Service Documentation
SQL / ETL
Domain Experts
```

可以形成：

``` text
Enterprise Knowledge
        │
        ├── Data Catalog
        ├── Data Dictionary
        ├── Metadata
        ├── Lineage
        ├── Knowledge Graph
        ├── Schema
        ├── API
        ├── Documents
        └── Business Knowledge
                 │
                 ▼
          Candidate Concepts
```

这里需要特别强调：

> **已有数据治理和知识资产是 Ontology 建设的重要输入，但它们本身并不等于 Ontology。**

------------------------------------------------------------------------

# 19. 第三步：识别业务概念，而不是照抄数据库

不要从：

``` text
mysql.customer
es.customer_profile
hive.customer_daily
```

直接构造 Ontology。

应该先问：

``` text
这个企业业务世界里到底有什么概念？
```

例如：

``` text
Customer
Account
Order
Transaction
Device
```

然后再寻找这些概念在数据体系中的表示。

因此：

``` text
Physical Data
      │
      │ 不是直接复制
      ▼
Business Understanding
      │
      ▼
Ontology
```

这是避免 Ontology 退化成"高级数据库 Schema"的关键。

------------------------------------------------------------------------

# 20. 第四步：定义 Entity / Property / Relationship

以 Customer Domain 为例：

``` text
Customer
 ├── customerId
 ├── name
 ├── phone
 └── status

Account
 ├── accountId
 └── status

Order
 ├── orderId
 ├── amount
 └── status

Transaction
 ├── transactionId
 ├── amount
 └── timestamp
```

关系：

``` text
Customer ──owns──> Account
Customer ──places──> Order
Account ──has──> Transaction
Customer ──uses──> Device
```

每个概念不仅需要名字，还需要明确：

- Definition；
- Semantic Type；
- Owner；
- Scope；
- Applicable Domain；
- Synonyms；
- Constraints；
- Source / Provenance。

------------------------------------------------------------------------

# 21. 第五步：定义语义约束

仅有实体和关系还不够。

例如：

``` text
Customer.customerId UNIQUE

Account belongs_to exactly_one Customer

Order belongs_to one Customer

Transaction.amount >= 0

Order.status = CANCELLED
    ⇒ Order.cancelTime != null

Order.createTime <= Order.payTime

Transaction.currency ∈ CurrencyDictionary
```

这些约束的价值是让 Agent 不仅知道"有哪些对象"，还知道：

> **这些对象之间什么关系是合法的，哪些数据解释和操作是成立的。**

需要区分：

``` text
Semantic Constraint
        │
        ├── 类型约束
        ├── 基数约束
        ├── 关系约束
        └── 基本业务不变量
```

与：

``` text
Executable Business Logic
```

后者可能属于业务服务、规则系统或工作流系统。

Ontology 可以描述其语义前提和适用条件，但不需要吞并完整的业务执行逻辑。

------------------------------------------------------------------------

# 22. 第六步：建立 Semantic Data Representation 与 Mapping

这是从"业务模型"走向"可使用模型"的关键步骤。

例如：

``` text
Customer
   │
   ├── CustomerIdentity
   │       ↓
   │   mysql.customer
   │
   ├── CustomerBehavior
   │       ↓
   │   clickhouse.customer_behavior
   │
   └── CustomerRealtimeStatus
           ↓
       redis.customer_status
```

然后进一步映射到 Data Catalog：

``` text
CustomerIdentity
      │
      ├── mysql.customer.id
      ├── mysql.customer.name
      └── mysql.customer.phone
```

Mapping 需要考虑：

- 字段映射；
- 类型转换；
- 单位转换；
- 编码转换；
- 多表组合；
- 多数据源组合；
- 派生字段；
- 数据过滤；
- 时间有效性；
- 主数据与快照；
- 数据新鲜度。

因此，真正成熟的 Mapping 往往不是简单的：

``` text
logicalField → physicalField
```

而是一个可解释的**语义数据映射定义**。

------------------------------------------------------------------------

# 23. 第七步：建立 Capability Mapping

当 Agent 已经知道：

``` text
Customer
Account
Transaction
```

下一步必须知道：

> "如果我要读取、搜索或改变这些对象，我应该调用什么能力？"

例如：

``` text
Customer
 ├── getCustomerById
 ├── searchCustomer
 ├── getCustomerAccounts
 ├── getCustomerOrders
 └── getCustomerTransactions
```

Capability 应至少描述：

``` yaml
name: getCustomerTransactions

input:
  customerId: CustomerId
  startTime: Timestamp
  endTime: Timestamp

output:
  type: Transaction[]

preconditions:
  - Customer exists

permission:
  - transaction.read

freshness:
  - near_realtime

provider:
  - TransactionService
```

这样 Agent 可以从：

``` text
Business Concept
```

走到：

``` text
Capability
```

再进入真正的执行服务。

------------------------------------------------------------------------

# 24. 第八步：建立可信度与治理信息

Ontology 不能只描述"是什么"，还需要描述：

> **这个定义有多可信、谁负责、从哪里来、什么时候生效。**

至少建议记录：

``` text
Owner
Source
Provenance
Confidence
Version
Effective Time
Deprecated Time
Approval Status
Domain
```

例如：

``` text
Customer.phone

Version:
    3

Status:
    Approved

Owner:
    CRM Domain

Source:
    CRM Data Dictionary
    mysql.customer.phone
    Business Manual v12

Changed:
    2026-09-20

Reason:
    CRM customer contact model changed
```

这样 Ontology 才具备企业级基础设施需要的可追溯性。

------------------------------------------------------------------------

# 25. Ontology 初始化：从 T0 建立第一个可信版本

初始化不是把所有企业数据一次性导入。

建议过程：

``` text
                    Enterprise Knowledge
                           │
                           ▼
                  Candidate Discovery
                           │
                           ▼
                   AI-assisted Mining
                           │
                           ▼
                    Ontology Draft
                           │
                           ▼
                 Domain Expert Review
                           │
                           ▼
                  Mapping Validation
                           │
                           ▼
                  Capability Validation
                           │
                           ▼
                   Approved Version
                           │
                           ▼
                    Runtime Publish
```

其中 AI 可以参与：

- 概念候选发现；
- 同义词识别；
- 实体候选聚类；
- 属性语义推断；
- Schema 与业务概念关联；
- Mapping 候选生成；
- 关系候选发现；
- 文档与数据资产关联。

但：

> **AI 生成的是候选语义，业务专家负责确认最终语义。**

尤其是以下问题不能仅靠 Schema 自动决定：

- 两个概念是否真的属于同一个业务实体；
- 两个字段是否具有相同业务含义；
- 某个关系是否是真正的业务关系；
- 某个概念的定义是否跨部门成立；
- 某个数据源是否能够作为权威数据源。

------------------------------------------------------------------------

# 26. Ontology Studio：为人提供建设和治理入口

如果 Ontology 是长期基础设施，就不应该只有 API。

建议提供 Ontology Studio：

``` text
Ontology Studio
 │
 ├── Entity Modeling
 ├── Property Modeling
 ├── Relationship Modeling
 ├── Constraint Modeling
 ├── Mapping Configuration
 ├── Capability Binding
 ├── Provenance
 ├── Version Management
 ├── Impact Analysis
 ├── AI Proposal Review
 └── Approval / Publish
```

典型流程：

``` text
Draft
  ↓
Review
  ↓
Validate
  ↓
Approve
  ↓
Publish
  ↓
Runtime
```

这样可以把：

``` text
“编辑 Ontology”
```

变成真正可治理的工程流程。

------------------------------------------------------------------------

# 27. Agent 如何使用 Ontology Service

Ontology 的最终价值之一，是让 Agent 能够以业务语义而不是物理数据结构理解企业。

一个完整调用链可以是：

``` text
User Task
    │
    ▼
  Agent
    │
    ▼
Semantic Discovery
    │
    ▼
Ontology
    │
    ▼
Constraints
    │
    ▼
Data Representation / Mapping
    │
    ▼
Capability Discovery
    │
    ▼
Execution Service
    │
    ▼
Physical Data
```

如果任务涉及实例和事实，则可能进入：

``` text
Ontology
   │
   ├──────────────→ Knowledge Graph
   │                       │
   │                       ▼
   │                 Entity / Facts
   │
   └──────────────→ Mapping
                           │
                           ▼
                     Data Services
                           │
                           ▼
                       Actual Data
```

------------------------------------------------------------------------

# 28. Semantic Discovery：Agent 不应该一次获得整个 Ontology

大型企业可能存在数千甚至数万 Entity Types，以及海量 Properties 和 Relationships。

因此，不应该把完整 Ontology 一次塞进 Agent Context。

更合理的是：

> **让 Agent 根据当前任务逐步发现所需要的语义。**

例如：

``` text
resolveConcept("客户")
        │
        ▼
     Customer
        │
        ▼
getRelationships(Customer)
        │
        ├── Account
        ├── Order
        └── Device
        │
        ▼
getRelatedConcepts(Customer, "transaction")
        │
        ▼
    Transaction
        │
        ▼
getConstraints(Transaction)
```

因此 Ontology Service 的重要能力不是简单 CRUD，而是：

> **让 Agent 能够从自然语言概念开始，逐步探索企业语义空间。**

------------------------------------------------------------------------

# 29. MCP、Skills 与 Ontology 的关系

三者解决的是不同问题。

``` text
Ontology
    ↓
“世界是什么？”

Skill
    ↓
“某类任务应该如何完成？”

MCP Tool
    ↓
“具体通过什么标准接口调用能力？”
```

例如：

``` text
customer_risk_analysis Skill
        │
        │ 任务流程
        ▼
Ontology MCP
        │
        │ 发现 Customer / Account / Transaction
        ▼
Data / Action MCP
        │
        │ 查询或执行
        ▼
Enterprise Services
```

因此：

> **MCP / Skills 是 Agent 使用能力的机制；Ontology 是 Agent 使用这些能力时依赖的企业语义模型。**

Ontology 不应该被设计成 Skill。

------------------------------------------------------------------------

# 30. Ontology Service 建议提供的运行时 API

可以把运行时 API 分成两大类。

## 30.1 Semantic Discovery API

用于理解业务世界：

``` text
resolveConcept()
searchOntology()

getEntityType()
getProperties()
getRelationships()
getRelatedEntities()

getConstraints()
getSemanticTypes()
getDefinitions()
```

例如：

``` text
resolveConcept("客户")
        ↓
Customer
```

然后：

``` text
getRelationships(Customer)
        ↓
Account
Order
Device
```

------------------------------------------------------------------------

## 30.2 Mapping / Capability Discovery API

用于从语义世界继续走向数据和能力：

``` text
getDataRepresentations(Customer)
getMappings(Customer)
getDataSources(Customer)

getCapabilities(Customer)
getCapabilities(Transaction)
getCapability(name)
```

例如：

``` text
getCapabilities(Customer)

→ getCustomerById
→ searchCustomer
→ getCustomerAccounts
→ getCustomerOrders
→ getCustomerTransactions
```

Ontology Service 可以负责描述和发现这些能力，但真正执行仍由外部服务完成。

------------------------------------------------------------------------

# 31. Agent 的完整示例

假设用户提出：

> "帮我查一下张三最近 7 天有没有异常交易。"

Agent 不应该首先寻找一个叫：

``` text
customer
```

的数据库表。

它应该首先理解：

``` text
张三
  ↓
Customer
  ↓
Account
  ↓
Transaction
```

再根据语义约束理解：

``` text
Transaction
  ├── time = last 7 days
  └── abnormality = risk / business definition
```

然后发现数据表示：

``` text
Transaction
   │
   ├── TransactionHistory
   └── RiskTransaction
```

再发现能力：

``` text
searchTransactionsByCustomerAndTimeRange()
```

最后：

``` text
Agent
   ↓
Ontology Service
   ↓
Capability Discovery
   ↓
TransactionService
   ↓
ClickHouse / Other Data Service
   ↓
Actual Transactions
```

Agent 不需要知道底层具体 SQL。

------------------------------------------------------------------------

# 32. Semantic Plan：连接语义理解与执行

Agent 在理解任务之后，通常需要形成一个明确的**语义计划（Semantic
Plan）**。

例如：

``` json
{
  "entity": "Customer",
  "identifier": "12345",
  "traversal": [
    {
      "relationship": "owns",
      "target": "Account"
    },
    {
      "relationship": "hasTransaction",
      "target": "Transaction",
      "constraint": {
        "time": "last_7_days",
        "amount": "> 10000"
      }
    }
  ]
}
```

这个计划表达的是：

``` text
业务对象
+
关系路径
+
语义约束
+
任务意图
```

而不是：

``` text
SQL
ES DSL
Redis Command
```

之后由 Planner / Data Service 将它转换成实际的数据访问计划。

因此：

``` text
Natural Language
       ↓
Semantic Understanding
       ↓
Ontology
       ↓
Semantic Plan
       ↓
Capability
       ↓
Execution Service
       ↓
Physical Data
```

这一层的价值是保持：

> **业务语义与物理执行之间的解耦。**

------------------------------------------------------------------------

# 33. Ontology 的五层理解

从业务世界一直到物理数据，可以形成一个非常实用的五层模型：

``` text
L1 Business Ontology
        ↓
L2 Semantic Constraints
        ↓
L3 Semantic Data Representation / Mapping
        ↓
L4 Data / Action Capability
        ↓
L5 Physical Data / Systems
```

## L1：Business Ontology

回答：

> 现实世界是什么？

``` text
Customer
Account
Order
Transaction
Device
```

------------------------------------------------------------------------

## L2：Semantic Constraints

回答：

> 这些概念之间有什么语义约束？

``` text
Customer.customerId UNIQUE

Account belongs_to exactly_one Customer

Transaction.amount >= 0

Order.createTime <= Order.payTime
```

------------------------------------------------------------------------

## L3：Semantic Data Representation / Mapping

回答：

> 现实世界如何被企业数据表示？

``` text
Customer
   │
   ├── CustomerIdentity
   │       ↓
   │   MySQL.customer
   │
   ├── CustomerBehavior
   │       ↓
   │   ClickHouse.customer_behavior
   │
   └── CustomerRealtimeStatus
           ↓
       Redis.customer_status
```

------------------------------------------------------------------------

## L4：Data / Action Capability

回答：

> 实际数据和业务操作如何被访问？

``` text
CustomerIdentity
   ├── getById()
   ├── searchByPhone()
   └── batchGet()

Account
   ├── getAccounts()
   └── getBalance()
```

------------------------------------------------------------------------

## L5：Physical Data / Systems

最终实际存在：

``` text
MySQL
TiDB
ClickHouse
Elasticsearch
Kafka
Hive
HBase
Redis
Business Services
Workflow Systems
```

这五层不要混成一个模型。

------------------------------------------------------------------------

# 34. Ontology 的持续维护：真正困难的是 Semantic Evolution

Ontology 不是"一次建设、长期不变"的项目。

企业现实世界持续变化：

- 新业务上线；
- 新实体出现；
- 实体增加或删除属性；
- 新的业务关系出现；
- 原有关系变化；
- 业务规则变化；
- 数据库表拆分、合并和迁移；
- Kafka Topic、ES Index、Hive 表、ClickHouse 表变化；
- 字段重命名、类型变化；
- 数据源替换；
- 数据加工链路变化；
- 不同部门对同一个概念形成不同定义。

因此：

> **Ontology 是一个持续演化的企业语义状态，而不是一份静态配置。**

------------------------------------------------------------------------

# 35. Physical Schema Evolution 不等于 Ontology Evolution

这是维护 Ontology 时必须建立的判断能力。

例如：

``` text
Ontology:
Customer.phone
```

底层从：

``` text
mysql.customer.phone
```

迁移为：

``` text
mysql.customer.mobile
```

如果业务含义没有变化，那么：

``` text
Customer.phone
        │
        ▼
mysql.customer.mobile
```

只是：

> **Data Mapping Evolution**

Ontology 本身不一定需要变化。

但如果业务含义从：

``` text
phone
```

变成：

``` text
primaryContactPhone
```

那么就可能已经是：

> **Business Semantic Evolution**

至少应该区分：

``` text
Physical Schema Evolution
        │
        ▼
Data Mapping Evolution
        │
        ▼
Ontology Evolution
        │
        ▼
Business Evolution
```

不同层级的变化，影响范围完全不同。

------------------------------------------------------------------------

# 36. Semantic Impact Analysis：变化发生后影响谁

一个生产级 Ontology Platform 不应该只回答：

``` text
GET /entity/Customer
GET /relationship/Customer-Order
GET /mapping/Customer
```

还应该回答：

> **企业的数据和业务发生变化之后，Ontology 哪些地方受到影响？**

例如：

``` text
ClickHouse.customer_behavior.phone
```

发生：

``` text
phone → mobile
```

系统应该能够发现：

``` text
customer_behavior.phone
        │
        ├── Ontology.Customer.phone
        ├── Capability A
        ├── Query Definition B
        ├── Report C
        └── Other dependent assets
```

然后形成：

``` text
Impact Analysis

Changed:
    customer_behavior.phone

Affected:
    Customer.phone
    Capability A
    Query Definition B
    Report C

Required Action:
    Update Mapping
    Validate Ontology
    Re-test dependent capabilities
```

这使 Ontology Service 从简单的"语义查询服务"进一步成为：

> **Semantic Control Plane + Semantic Change Management System**

------------------------------------------------------------------------

# 37. Ontology 的 Version / Provenance / Approval

既然 Ontology 会持续变化，就不能把它当成普通配置文件。

至少应该具备：

``` text
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

``` text
Customer.phone

Version:
    3

Status:
    Approved

Owner:
    CRM Domain

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
    5 capabilities
    2 dependent business definitions
```

这样才能做到：

- 可追溯；
- 可审计；
- 可回滚；
- 可解释；
- 可进行变更影响分析。

------------------------------------------------------------------------

# 38. Ontology Maintenance 的完整生命周期

可以把整个生命周期概括成：

``` text
建设
  ↓
上线
  ↓
业务变化 / 数据变化
  ↓
变化检测
  ↓
语义影响分析
  ↓
Ontology Change Proposal
  ↓
人工验证
  ↓
新版本 Ontology
  ↓
重新验证 Mapping / Capability
  ↓
发布
  ↓
Agent / Service 使用新语义
  ↓
继续变化
  ↺
```

因此：

> **Ontology 不是一个最终产物，而是一个持续演化的企业语义状态。**

------------------------------------------------------------------------

# 39. AI 在 Ontology 建设与维护中的作用

AI 的价值不仅在于第一次录入。

## 39.1 建设阶段

``` text
Enterprise Knowledge
        ↓
       AI
        ↓
Ontology Candidates
        ↓
Human Validation
        ↓
Approved Ontology
```

AI 可以帮助：

- 发现概念；
- 聚合同义概念；
- 推断属性含义；
- 发现潜在关系；
- 生成 Mapping 候选；
- 分析文档；
- 关联数据资产；
- 生成初始定义。

------------------------------------------------------------------------

## 39.2 运行阶段

更重要的是：

``` text
Business / Data Changes
        ↓
   Change Detection
        ↓
Semantic Impact Analysis
        ↓
AI Change Proposal
        ↓
Human Validation
        ↓
Ontology Update
        ↓
Validation / Test
        ↓
Publish
```

输入可以包括：

``` text
Schema Diff
SQL Diff
Lineage Diff
API Diff
Business Document Diff
Code Diff
Data Catalog Changes
Actual Query Usage
Agent Usage
```

例如：

``` text
Schema Diff
    ↓
customer_behavior.phone → mobile
    ↓
AI Semantic Analysis
    ↓
“Customer.phone 可能已经迁移到 Customer.mobile，
请确认业务语义和 Mapping。”
```

因此：

> **AI 很难消灭 Ontology 的维护成本，但可以改变维护成本的结构。**

最终目标不是：

> "让 Ontology 不需要人维护。"

而是：

> **让人主要处理真正需要业务判断的语义变化，把发现、分析、影响传播、候选修改、验证等工作尽可能自动化。**

------------------------------------------------------------------------

# 40. Ontology Platform：从 Service 到平台

如果 Ontology 真正成为企业 Agent 基础设施，仅有 Runtime API
往往是不够的。

可以进一步形成：

``` text
                    Ontology Platform
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   Ontology Runtime   Ontology Studio   Change Engine
          │                │                │
          │                │                │
          ▼                ▼                ▼
       Agent         Human Modeling    Change Detection
       Service       / Approval        / Impact Analysis
                           │                │
                           └───────┬────────┘
                                   ▼
                             Version / Audit
```

## Ontology Runtime

服务 Agent 和其他系统：

``` text
查询实体
查询关系
查询约束
获取语义定义
获取数据映射
发现能力
```

## Ontology Studio

服务人：

``` text
创建 Entity
定义 Property
定义 Relationship
定义 Constraint
配置 Mapping
绑定 Capability
查看 Provenance
审批 AI Proposal
发布版本
```

## Change Engine

服务变化：

``` text
Schema Diff
Lineage Diff
API Diff
Business Document Diff
Code / SQL Diff
Catalog Change
Usage Analysis
```

然后：

``` text
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

------------------------------------------------------------------------

# 41. 推荐的完整平台架构

``` text
                              Agent
                                │
                           MCP / Skills
                                │
                                ▼
                    ┌────────────────────────┐
                    │    Ontology Platform   │
                    │                        │
                    │  Semantic Model        │
                    │  Constraint Model      │
                    │  Semantic Data Model   │
                    │  Mapping               │
                    │  Capability Model      │
                    │  Semantic Discovery    │
                    │  Semantic Planning     │
                    │  Governance             │
                    └───────────┬────────────┘
                                │
         ┌──────────────────────┼────────────────────────┐
         │                      │                        │
         ▼                      ▼                        ▼
 Knowledge Graph         Data Catalog / Metadata   Capability Registry
         │                      │                        │
         │                      │                        ▼
         │                      │                 Data / Action Services
         │                      │                        │
         │                      ▼                        │
         │                Physical Assets                │
         │                                               │
         ▼                                               ▼
   Entity / Facts                                  Actual Operations
```

而维护侧：

``` text
Business Changes ─────┐
                      │
Data Changes ─────────┤
                      ▼
                Change Engine
                      │
                      ▼
              Impact Analysis
                      │
                      ▼
                AI Proposal
                      │
                      ▼
                Human Review
                      │
                      ▼
             Versioned Ontology
                      │
                      ▼
                    Runtime
```

------------------------------------------------------------------------

# 42. 不要设计成"超级 Ontology Service"

一个常见的架构陷阱是：

``` text
                     Ontology Service
                          │
          ┌───────────────┼────────────────┐
          │               │                │
      Ontology           KG          Data Catalog
          │               │                │
      Lineage        Query Engine      Metadata
          │               │                │
     MCP Gateway      Workflow          Action
          │               │                │
          └───────────────┼────────────────┘
                          │
                       Everything
```

这种设计短期看起来非常完整，但长期容易导致：

- Ontology Model 与物理数据模型耦合；
- KG 与 Ontology 职责混乱；
- Data Catalog 重复建设；
- Lineage 重复建设；
- Capability / MCP 与语义模型强耦合；
- Query Engine 进入 Ontology；
- 权限、执行、工作流全部进入同一个服务。

最终：

> **Ontology Service 从"语义中心"变成"企业超级数据平台"。**

应该避免这种演化。

------------------------------------------------------------------------

# 43. 一个更稳定的职责划分

推荐保持：

``` text
                    Ontology
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Knowledge       Mapping     Capability
       Graph            │            │
          │             ▼            ▼
          │       Data Catalog   Capability Registry
          │             │            │
          ▼             ▼            ▼
       Facts       Physical Data   Execution Services
```

各层核心问题：

|层|核心问题|
|-|-|
|Ontology|世界是什么？|
|Knowledge Graph|当前有哪些实例和事实？|
|Mapping|业务概念如何被数据表示？|
|Data Catalog|数据资产在哪里、是什么状态？|
|Lineage|数据从哪里来、如何流转？|
|Capability|可以做什么？|
|Execution Service|如何真正执行？|
|Agent|为了用户目标，如何组合这些语义和能力？|

------------------------------------------------------------------------

# 44. 与 Palantir Ontology 的关系

Palantir 的 Ontology
思路与这里讨论的架构存在明显相似性：都强调通过业务对象、属性、关系和操作能力，把企业底层数据提升为更接近业务世界的抽象。

Palantir 的体系通常更重，可能把：

``` text
Objects
Properties
Links
Actions
Functions
Security
Workflow
```

纳入更统一的 Operational Ontology 体系。

本文讨论的方案则更倾向于：

``` text
                Ontology Platform
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Semantic       Mapping     Capability
        Model          Model        Model
          │            │            │
          └────────────┼────────────┘
                       │
                       ▼
                External Services
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           Query     Search     Action
           Service   Service    Service
```

即：

> **先把 Ontology 做成相对"薄"的语义控制面，把真正的数据访问、搜索、计算、业务操作放在周边专门服务中。**

未来如果业务确实需要，可以逐渐向更完整的 Operational Ontology 演进。

------------------------------------------------------------------------

# 45. 五条核心架构原则

## 原则一：先抽象现实世界，再映射数据

不要从：

``` text
MySQL.customer
ES.customer_profile
Hive.customer_daily
```

出发构造 Ontology。

应该从：

``` text
Customer
Account
Device
Order
Transaction
```

这样的业务概念出发。

------------------------------------------------------------------------

## 原则二：Ontology 与 Physical Data 解耦

``` text
Ontology.Customer
```

不应该等价于：

``` text
mysql.customer
```

两者之间应该存在明确的 Mapping。

这样底层数据技术、存储系统和数据组织方式发生变化时，业务语义可以保持稳定。

------------------------------------------------------------------------

## 原则三：Ontology 可以描述能力，但不负责所有执行

``` text
Ontology
   ↓
Capability Contract
   ↓
Execution Service
   ↓
Actual Execution
```

保持语义层与执行层的职责清晰。

------------------------------------------------------------------------

## 原则四：Ontology 是 Agent 的企业世界模型

Agent 首先面对：

``` text
Entity
Property
Relationship
Business Meaning
Constraint
Capability
```

而不是：

``` text
Table
Column
Topic
Index
```

之后再通过 Mapping 和 Capability Discovery 进入具体的数据和执行系统。

------------------------------------------------------------------------

## 原则五：先建立薄而清晰的 Ontology，再决定是否扩展

第一阶段重点应该是：

``` text
Business Semantic Model
+
Semantic Constraints
+
Relationship
+
Semantic Data Representation
+
Data Mapping
+
Capability Description
+
Governance
```

不要一开始就把：

``` text
Storage
Query Engine
Workflow Runtime
Action Runtime
Lineage Engine
Data Catalog
MCP Gateway
```

全部塞进 Ontology。

------------------------------------------------------------------------

# 46. 一个完整的 Ontology 建设检查表

建设一个新的 Domain Ontology 时，可以依次检查：

## Domain

``` text
□ Domain 范围是否明确？
□ 使用场景是否明确？
□ 目标 Agent / Application 是否明确？
```

## Semantic Model

``` text
□ Entity 是否定义？
□ Property 是否定义？
□ Relationship 是否定义？
□ Definition 是否清晰？
□ Semantic Type 是否明确？
□ 同义词 / 别名是否处理？
```

## Constraints

``` text
□ 类型约束？
□ 基数约束？
□ 关系约束？
□ 基本业务不变量？
□ 适用范围？
```

## Data

``` text
□ Semantic Data Representation？
□ Data Mapping？
□ Data Catalog 引用？
□ Data Quality？
□ Freshness？
□ Lineage？
```

## Capability

``` text
□ 查询能力？
□ 搜索能力？
□ 聚合 / 分析能力？
□ Action？
□ Preconditions？
□ Permission？
□ Provider？
```

## Governance

``` text
□ Owner？
□ Source？
□ Provenance？
□ Version？
□ Effective Time？
□ Approval Status？
□ Deprecated Time？
□ Impact Analysis？
```

## Runtime

``` text
□ Semantic Discovery？
□ Capability Discovery？
□ MCP 接入？
□ Agent Context 控制？
□ Semantic Plan？
```

------------------------------------------------------------------------

# 47. 最终心智模型

可以把整个体系记成：

``` text
                     ┌──────────────┐
                     │   Ontology   │
                     │  世界是什么   │
                     └──────┬───────┘
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
        Knowledge Graph              Mapping
        世界中的事实               如何映射到数据
               │                         │
               │                         ▼
               │                   Data Catalog
               │                   数据在哪里
               │                         │
               └────────────┬────────────┘
                            │
                            ▼
                       Capability
                       我能做什么
                            │
                            ▼
                    Execution Services
                            │
                            ▼
                      Physical Data
```

而 Agent 位于这个体系的上层：

``` text
                         User
                           │
                           ▼
                         Agent
                           │
                           ▼
                  Semantic Discovery
                           │
                           ▼
                       Ontology
                           │
                 ┌─────────┼─────────┐
                 ▼         ▼         ▼
              Facts     Mapping   Capability
                 │         │         │
                 ▼         ▼         ▼
                KG      Catalog   Services
                                      │
                                      ▼
                                Physical Data
```

------------------------------------------------------------------------

# 48. 最终定义：什么是 Ontology？

> **Ontology 是企业现实业务世界的抽象、形式化和机器可理解的语义模型。**
>
> 它定义企业世界中有哪些重要概念、这些概念具有什么属性、彼此有什么关系，以及这些关系和概念受到哪些语义约束。

# 49. 最终定义：什么是 Ontology Service？

> **Ontology Service 是将企业 Ontology 作为可查询、可发现、可治理的语义基础设施提供给 Agent、数据服务和其他应用的服务层。**

它负责连接：

``` text
业务世界
   ↕
业务语义
   ↕
企业数据
   ↕
企业能力
   ↕
AI / Agent
```

但不意味着它必须拥有这些层的全部数据和执行能力。

------------------------------------------------------------------------

# 50. 最终定义：为什么企业需要 Ontology Service？

因为企业真实世界与企业 IT 数据之间存在巨大的语义鸿沟：

``` text
现实世界
    │
    │ 人类理解
    ▼
Business Concepts
    │
    │ 企业系统实现
    ▼
Tables / Topics / Indexes / APIs
    │
    │ Agent 直接面对
    ▼
LLM
```

Ontology 的作用就是在中间建立一个稳定、显式、机器可理解的语义层：

``` text
现实世界
    │
    ▼
Ontology
    │
    ├── Meaning
    ├── Relationship
    ├── Constraint
    ├── Mapping
    └── Capability
    │
    ▼
Enterprise Data / Services
    │
    ▼
Agent
```

因此：

> **Agent 不应该首先学习企业所有数据库的 Schema；它应该首先认识企业的业务世界，而 Ontology 就是这个世界的机器可理解模型。**

------------------------------------------------------------------------

# 51. 最终定义：Ontology 建设真正要解决什么问题？

Ontology 建设不是简单地：

``` text
建几个 Entity
写几个 JSON
提供几个 API
```

真正需要解决的是：

``` text
现实世界
    ↕
Ontology
    ↕
Data Representation
    ↕
Physical Data
    ↕
Capability
    ↕
Agent
```

这些层之间如何长期保持**语义一致性（Semantic Consistency）**。

因此 Ontology 项目实际上包含两个长期阶段：

``` text
Construction
    │
    ▼
“把世界描述出来”
```

以及：

``` text
Evolution
    │
    ▼
“让描述持续跟得上世界”
```

第二部分往往是企业级 Ontology 真正长期、持续、昂贵的部分。

------------------------------------------------------------------------

# 52. 最终完整架构图

``` text
                         Enterprise World
                    Business + Data + Services
                               │
                               ▼
                    ┌────────────────────┐
                    │  Ontology Platform │
                    │                    │
                    │  Business Model   │
                    │  Constraints       │
                    │  Data Mapping      │
                    │  Capability Model  │
                    │  Discovery         │
                    │  Governance        │
                    └─────────┬──────────┘
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
   Knowledge Graph      Data Catalog /        Capability
   Entity / Facts          Metadata             Registry
          │                   │                    │
          │                   ▼                    ▼
          │             Physical Assets      Execution Services
          │                   │                    │
          └───────────────────┴────────────┬───────┘
                                           ▼
                                      Actual Data
                                           │
                                           ▼
                                         Agent
                                           ▲
                                           │
                                         User
```

而平台自身持续运行：

``` text
Business Evolution ─────┐
                        │
Data Evolution ─────────┤
                        ▼
                 Change Detection
                        │
                        ▼
                 Impact Analysis
                        │
                        ▼
                  AI Proposal
                        │
                        ▼
                  Human Review
                        │
                        ▼
                New Ontology Version
                        │
                        ▼
                     Runtime
                        │
                        └───────────────↺
```

> **最终可以用一句话概括整个架构：**
>
> **Ontology 定义企业世界是什么；**
> **Knowledge Graph 保存这个世界中的实例和事实；**
> **Data Catalog 描述企业的数据资产；**
> **Mapping 把业务语义连接到这些数据资产；**
> **Lineage 描述数据如何产生和流转；**
> **Capability 描述可以对这个世界做什么；**
> **Ontology Service 则把这些语义关系组织成面向 Agent 和应用的统一语义控制面，使 Agent 能够先理解企业世界，再发现数据与能力，最后通过专门的执行服务完成任务。**
