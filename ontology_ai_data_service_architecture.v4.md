# Ontology 与 Ontology Service：面向企业 AI 的语义控制面架构

> 本文面向企业架构师、数据平台建设者、Ontology 建模者、AI Agent 和数据服务开发者，回答六个问题：
>
> 1. Ontology 到底是什么，为什么在 AI 时代重新成为企业基础设施？
> 2. Ontology 与 Schema、Knowledge Graph、Data Catalog、Mapping、Lineage、Capability 的边界是什么？
> 3. Ontology Service 应该负责什么，又必须拒绝什么？
> 4. 企业如何从一个业务域开始建设、治理和持续演进 Ontology？
> 5. Agent 如何渐进式发现语义、生成 Semantic Plan，并安全地使用企业数据与能力？
> 6. 如何判断一个 Ontology Service 没有滑向“超级数据平台”？

本文的核心立场是：

> **Ontology 是对业务现实世界的抽象、形式化和机器可理解的语义模型；Ontology Service 是把这套模型提供给 Agent 和应用使用的薄的 Semantic Control Plane（语义控制面），而不是数据库、Universal Query Engine 或 Workflow Engine。**

---

## 一、定位与核心价值

### 1.1 Ontology 是什么

简单来说，本体就是一套让机器和人都能听懂的“领域标准普通话”，也可以理解为企业统一的业务世界模型。

更严格地说：

> **Ontology 是对某个业务领域中现实世界概念的抽象、形式化和机器可理解的语义模型。**

它回答：

- 这个业务世界里有哪些重要的实体和概念？
- 每类实体有哪些属性？
- 实体之间存在什么业务关系？
- 每个概念、属性和关系的业务含义是什么？
- 哪些语义约束、基数约束和适用条件必须成立？
- 不同系统中的数据，如何被理解为同一个业务对象？
- 围绕这些对象，可以发现哪些读取能力和业务动作？

Ontology 首先描述的是“现实世界是什么”，而不是某个数据库里的表和字段。

例如，电商领域可能有：

```text
Customer
Product
Order
Payment
Shipment
```

以及：

```text
Customer ── places ──> Order
Order    ── contains ─> Product
Order    ── paid_by ──> Payment
Order    ── fulfilled_by ──> Shipment
```

这里描述的是业务世界中的概念与关系，而不是 `mysql.order`、`es.product_index` 或 `kafka.order_event` 的物理结构。

### 1.2 Ontology 的四个核心要素

一个可落地的 Ontology 至少包含四类基本元素。

#### 1.2.1 类 / 概念（Classes / Concepts）

类是领域内的抽象分类，回答“这个世界里有什么类型的对象”。

例如：

```text
Customer
Product
Order
Account
Transaction
Device
```

类不等于某一张表。一个 `Customer` 可能同时由 CRM 主数据、客户画像、实时状态和行为明细等多个系统共同表示。

#### 1.2.2 属性（Properties / Attributes）

属性是某个类所拥有的特征，回答“这个对象具有什么信息”。

例如：

```text
Customer.customerId
Customer.name
Customer.phone
Customer.riskLevel

Product.productId
Product.price
Product.inventory
```

属性应携带业务含义、语义类型、适用范围、敏感级别、数据新鲜度和来源等信息，而不只是一个字符串字段名。

例如：

```yaml
concept: Customer.phone
semanticType: PhoneNumber
meaning: Customer's primary contact phone
sensitivity: PII
applicability: ActiveCustomer
```

#### 1.2.3 关系（Relations）

关系连接不同类或不同实例，回答“对象之间如何发生业务联系”。

例如：

```text
Customer ── owns ──> Account
Customer ── places ──> Order
Account  ── has ────> Transaction
Device   ── used_by ─> Customer
```

关系不仅是技术上的 join key，还应说明关系语义、方向、基数、有效期和访问限制。例如，`Account belongs_to Customer` 与 `Customer has Account` 是同一业务关系的不同视角，而不是两个互不相关的字段关联。

#### 1.2.4 实例（Instances / Individuals）

实例是具体的数据点，回答“这个世界中当前有哪些具体对象”。

例如：

```text
Customer#12345
Product#IPHONE15
Order#ORD-20260929-001
```

“张三”是 `Customer` 的一个实例，“iPhone 15”是 `Product` 的一个实例。

必须区分：

> **Ontology 规定实例是什么、如何描述以及如何相互关联；Ontology 并不因此必须保存所有实例数据。**

实例可以由关系型数据库、文档数据库、搜索引擎、事件流、业务 API，或者在确有需要时由 Knowledge Graph 提供。

### 1.3 Ontology、Schema 与 Knowledge Graph 的直观区别

为了不混淆三者，可以使用一个直观比喻：

| 概念 | 比喻 | 主要回答 | 典型内容 |
|---|---|---|---|
| **Schema** | 某栋楼的施工图纸 | 某个系统如何存储数据？ | MySQL 表、字段、类型、索引、约束 |
| **Ontology** | 整个城市的规划与建筑规范 | 业务世界是什么？应如何理解？ | Customer、Order、owns、PhoneNumber、业务约束 |
| **Knowledge Graph** | 城市里真实建好的楼、街道和居民 | 当前有哪些事实和实例？ | 张三是 Customer，张三拥有 Account-001 |

在更形式化的表达中：

> **Knowledge Graph 可以看作 Ontology 加上实例和事实，但 Ontology 不等于 Knowledge Graph。**

例如：

```text
Ontology:
Customer ── owns ──> Account

Knowledge Graph:
Customer#12345 ── owns ──> Account#98765
```

`mysql.customer` 的表结构属于 Schema；`Customer`、`owns` 和 `Account` 的业务定义属于 Ontology；`Customer#12345 owns Account#98765` 这样的当前事实属于 Knowledge Graph 或其他事实数据服务。

### 1.4 为什么大数据和 AI 时代重新需要 Ontology

过去的数据建设主要围绕数据仓库、ETL、维度建模和表结构展开。但企业现实世界中的数据越来越分散：

```text
MySQL       customer
ClickHouse  customer_behavior
Elasticsearch customer_profile
Hive        customer_snapshot
Redis       customer_realtime_status
Kafka       customer_event
Business API customer-service
```

同一个业务概念可能在不同部门使用不同名称：

```text
CRM       client_no
Sales     cust_id
Finance   customer_number
Risk      party_id
```

仅依赖表名、字段名和 API 名称，Agent 很难可靠判断：

- 哪些数据表示同一个实体；
- 哪个字段的业务含义与目标任务相关；
- 哪个数据源是权威源；
- 哪些数据是实时状态，哪些只是历史快照；
- 哪些关系是真实业务关系，哪些只是技术 join；
- 哪些属性属于 PII 或受限语义；
- 哪些能力可以读取，哪些动作需要审批或工作流。

Ontology 的价值集中体现在三个方面。

#### 1.4.1 构建统一语义层

Ontology 超越具体数据库表，在最上层建立稳定的业务上下文：

```text
物理名称不同：client_no / cust_id / party_id
业务含义统一：Customer.customerId
```

底层表可以迁移、拆分、合并或替换，只要业务含义不变，Ontology 及其对外语义接口就可以保持稳定。

#### 1.4.2 为 LLM 和 Agent 提供企业世界模型

LLM 能理解自然语言，但并不天然理解企业内部几百张表、几千个字段和复杂的数据权限。如果让 Agent 直接从物理 Schema 猜测业务含义，容易出现：

- 选错表或字段；
- 把快照当作实时数据；
- 把同名但不同义的概念混为一谈；
- 忽略关系基数和业务约束；
- 生成不满足权限或业务条件的查询。

Ontology 让 Agent 先理解：

```text
Customer → Account → Transaction
```

再通过 Mapping、Capability 和权限上下文进入实际数据与服务。对于 GraphRAG 等场景，Ontology 也可以作为企业知识导航图，帮助检索和推理使用统一语义，而不是把未经治理的表结构直接暴露给模型。

#### 1.4.3 提升数据资产的互操作性

跨系统协作的主要障碍往往不是数据格式，而是语义不一致。Ontology 提供一套机器可理解的共享词汇表，使不同系统、团队和 Agent 能够围绕同一业务概念交换数据。

OWL、RDF 等 W3C 标准可以作为表达形式，但企业不应为了使用 Ontology 而强制所有数据改造成某种图数据库或标准格式。标准是表达和互操作手段，不是对存储架构的强制规定。

### 1.5 Ontology Service 的核心定位

如果 Ontology 是企业业务世界的语义模型，那么 Ontology Service 是提供这套模型的服务层：

> **Ontology Service 是企业 AI 数据架构中的薄的 Semantic Control Plane（语义控制面）。**

它让 Agent 和应用能够：

1. 从自然语言概念发现实体、属性和关系；
2. 理解业务定义、语义约束和适用范围；
3. 找到对应的数据表示、数据源和数据质量信息；
4. 发现可用的读取能力和业务动作；
5. 生成或校验与执行层解耦的 Semantic Plan；
6. 在变更发生时进行语义影响分析和治理审批。

它不意味着 Ontology Service 必须拥有：

- 所有企业实例数据；
- 所有数据资产的副本；
- 一个新的超级数据库；
- 一个 Universal Query Engine；
- 一个全功能 Knowledge Graph；
- Data Catalog 或 Lineage 系统的全部能力；
- 所有查询、计算、写操作和工作流执行能力。

最简洁的边界是：

> **Ontology 描述世界；Data / Action Service 操作世界；Ontology Service 负责把两者以可信、可发现、可治理的语义方式连接起来。**

---

## 二、概念边界与五层模型

### 2.1 一张统一的概念边界表

这些系统都在“描述企业世界”，但描述的角度不同。逻辑边界必须清晰，即使物理上暂时部署在同一个平台中。

| 概念 | 核心问题 | 主要对象 | 典型内容 | 与 Ontology 的关系 |
|---|---|---|---|---|
| **Ontology** | 世界是什么？ | 业务概念、属性、关系、约束 | Customer、Order、owns、PhoneNumber、状态约束 | 定义业务世界的语义模型 |
| **Schema** | 某个系统如何存储？ | 表、列、类型、索引、约束 | `mysql.customer.phone varchar(32)` | 是物理系统的结构，不等于业务语义 |
| **Knowledge Graph** | 当前有哪些事实？ | 实例、事实、实体关系 | 张三拥有账户 001 | 可选的事实与复杂关系层，消费 Ontology 定义 |
| **Data Catalog / Metadata** | 数据资产在哪里？ | 数据库、表、字段、Topic、Dataset | Owner、位置、类型、质量、敏感等级 | Ontology 应引用，而不是复制 |
| **Data Mapping** | 业务概念如何被数据表示？ | 语义对象到物理资产的映射 | `Customer.phone → crm.customer.mobile` | 连接语义世界与数据世界 |
| **Data Lineage** | 数据如何产生和流转？ | 来源、加工、派生和依赖 | Kafka → Flink → ClickHouse | 为可信度、影响分析和新鲜度提供输入 |
| **Query Capability** | 能读取和分析什么？ | 查询、搜索、聚合、推理接口 | 查客户、搜订单、统计交易 | 描述可执行的读能力，不负责语义定义 |
| **Action Capability** | 能改变什么？ | 写入、审批、状态变更和业务动作 | 冻结账户、创建订单、退款 | 需要更强的权限、事务、幂等和工作流约束 |
| **Execution Service** | 如何真正执行？ | SQL、API、搜索、工作流和事务系统 | MySQL、GraphQL、Order Service | 执行 Semantic Plan 或 Capability Contract |
| **Agent** | 如何为用户目标组合语义和能力？ | 任务、计划、决策、反馈 | 发现、规划、调用、解释 | 消费语义控制面，不应直接猜测物理实现 |

概念可以在同一产品中协同，但职责不能因为部署合并而混淆。

### 2.2 Ontology 与 Data Catalog：引用而不是复制

Data Catalog 描述数据资产：

```text
mysql.customer.phone
  type: varchar(32)
  owner: CRM Team
  classification: PII
  updatedAt: ...
```

Ontology 描述业务概念：

```text
Customer.phone
  semanticType: PhoneNumber
  meaning: Customer primary contact phone
  required: true
```

两者回答的是不同问题：

- Ontology：这个属性在业务上是什么意思？
- Catalog：这个字段在哪里、谁负责、质量如何、是否敏感？

推荐关系是：

```text
Ontology.Customer.phone
        │ semantic mapping
        ▼
Data Catalog Asset
        │
        ▼
Physical Data
```

Ontology 可以引用 Catalog 的资产标识、Owner、敏感等级、质量和新鲜度，但不应复制一份完整 Catalog。否则会形成元数据双写、状态不一致和责任不清。

### 2.3 Mapping：业务语义与物理数据之间的独立层

Mapping 不应被简化为一对一字段对应：

```text
logicalField → physicalField
```

真实企业中，一个业务属性可能涉及：

- 字段重命名；
- 类型、单位和编码转换；
- 多表 join；
- 多源合并；
- 派生计算；
- 有效时间过滤；
- 主数据与快照选择；
- 数据质量和新鲜度判断；
- 数据脱敏和语义授权。

因此应把中间的语义数据表示独立出来：

```text
Business Ontology
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

```text
Customer
  ├── CustomerIdentity
  │     └── CRM customer master
  ├── CustomerBehavior
  │     └── ClickHouse behavior events
  └── CustomerRealtimeStatus
        └── Redis realtime status
```

业务实体不应被某一张表定义。表是某个系统对业务的物理表示，Mapping 才是连接两者的显式契约。

### 2.4 Lineage：为语义可信度提供证据，但不替代语义模型

Lineage 关注：

```text
数据从哪里来？
经过了什么处理？
何时更新？
影响了哪些下游资产？
```

例如：

```text
Kafka.login_event
        ↓
Flink enrichment
        ↓
ClickHouse.customer_behavior
        ↓
Ontology.Customer.lastLoginTime
```

Ontology 可以引用或消费 Lineage 信息，用于：

- 判断数据来源和可信度；
- 评估数据新鲜度；
- 进行 Schema / Mapping 变更影响分析；
- 向 Agent 解释结果 provenance；
- 发现下游能力和报告的潜在影响。

但 Ontology Service 不应因此重新实现一个完整 Lineage Engine。

### 2.5 五层模型：从业务世界到物理系统

一个适合工程落地的五层模型如下：

```text
L1  Business Ontology
        ↓
L2  Semantic Constraints
        ↓
L3  Semantic Data Representation / Mapping
        ↓
L4  Query Capability / Action Capability
        ↓
L5  Physical Data / Systems
```

#### 2.5.1 L1：Business Ontology

回答：现实业务世界是什么？

```text
Customer
Account
Order
Transaction
Device
```

包含 Classes、Properties、Relations 和定义、同义词、语义类型、适用范围等。

#### 2.5.2 L2：Semantic Constraints

回答：这些概念之间什么关系是合法的？

```text
Customer.customerId is UNIQUE
Account belongs_to exactly_one Customer
Transaction.amount >= 0
Order.createTime <= Order.payTime
Order.status = CANCELLED implies cancelTime != null
```

语义约束应与完整业务执行逻辑区分。Ontology 可以表达类型、基数、关系和基本不变量，但复杂计算、审批和状态机仍属于专门的规则、业务或工作流系统。

#### 2.5.3 L3：Semantic Data Representation / Mapping

回答：现实世界如何被企业数据表示？

```text
Customer
  ├── Identity Representation
  ├── Behavior Representation
  └── Realtime Status Representation
```

该层记录物理资产、转换、过滤、时间条件、权威源和降级策略，而不是把任何一个数据库表误认为完整业务实体。

#### 2.5.4 L4：Capability

回答：如何安全地访问数据，或如何改变业务状态？

L4 必须明确拆分为两类：

**Query Capability（数据读取能力）**：

- 查询实体和关系；
- 搜索和过滤；
- 聚合和分析；
- 读取实时状态；
- 生成只读解释或报告。

**Action Capability（业务动作能力）**：

- 创建、更新和删除业务对象；
- 改变订单、账户或工单状态；
- 发起退款、冻结、审批等动作；
- 触发外部副作用或工作流。

二者的工程要求不同：

| 维度 | Query Capability | Action Capability |
|---|---|---|
| 主要风险 | 越权读取、敏感信息泄露、结果误解 | 错误副作用、状态破坏、重复执行 |
| 权限 | 数据集、属性、关系和行级/列级权限 | 角色、职责分离、审批和动作权限 |
| 一致性 | 可接受明确标注的最终一致性 | 通常需要事务、状态校验和强一致边界 |
| 幂等性 | 通常由查询天然满足 | 必须显式设计幂等键和重复请求处理 |
| 执行方式 | Query Service、Search Service、分析引擎 | Action Service、Workflow Engine、事务服务 |
| Agent 默认策略 | 可按权限自动执行并解释来源 | 默认预览、确认或转人工审批 |

Ontology Service 可以描述和发现两类能力，但不应把 Action Runtime 吞并进来。对于写操作，专有 Workflow Engine 通常是更安全的承载位置。

#### 2.5.5 L5：Physical Data / Systems

包括：

```text
MySQL / PostgreSQL / TiDB
ClickHouse / Hive / Spark
Elasticsearch / HBase / Redis
Kafka / Object Storage
Graph Database（可选）
Business API / Workflow Engine
```

L5 是执行和存储的现实，不应反向决定业务语义的全部结构。

### 2.6 Knowledge Graph 的正确定位：可选的复杂关系和事实层

Knowledge Graph 很有价值，但不应被设为所有企业 Ontology 落地的必选存储。

适合引入 KG 的场景包括：

- 多跳关系探索；
- 实体消歧与实体解析；
- 复杂关系推理；
- 事实版本和 provenance 管理；
- GraphRAG；
- 需要高频关系遍历的业务。

不需要强行引入 KG 的场景包括：

- 主要是结构化属性查询；
- 现有关系型或分析型数据服务已经稳定；
- 关系规模有限且可由 API / SQL 联邦访问；
- 企业尚未具备图数据治理和运维能力。

因此推荐的表述是：

> **Knowledge Graph 是 Ontology 可选的事实与复杂关系执行层，而不是 Ontology Service 的必然形态。**

Ontology 可以通过 Mapping 直接联邦查询 MySQL、ClickHouse、ES、API 等现有系统。是否建立 KG，应由关系复杂度、查询模式、实时性、成本和治理能力共同决定。

---

## 三、架构设计与边界原则

### 3.1 静态分层架构：语义控制面与数据/能力执行面分离

推荐的静态架构只有一个核心原则：上层用稳定语义描述意图，下层用专门系统完成查询、计算和动作。

```text
                                  User / Application
                                          │
                                          ▼
                                  AI Agent / Planner
                                          │
                                  Semantic Discovery
                                          │
                                          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Ontology Service：Semantic Control Plane                  │
│                                                                             │
│  Business Ontology  │  Semantic Constraints  │  Mapping  │  Capability     │
│  Definitions        │  Access Policy         │  Provenance│  Registry        │
│                                                                             │
│  Versioning / Approval / Impact Analysis / Feedback Proposal                │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                         Semantic Plan / Contract
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
       Plan Translation   Query / Search      Action / Workflow
       Engine / Adapters  Execution Services  Execution Services
             │                  │                  │
             ▼                  ▼                  ▼
       SQL / Cypher /      MySQL / OLAP /     Business API /
       GraphQL / API       Search / Stream    Workflow Engine
             │                  │                  │
             └──────────────────┴──────────────────┘
                                ▼
                         Physical Data / State

 Data Catalog、Lineage、IAM、DLP、质量和监控系统作为外部治理系统被引用或协同。
 Knowledge Graph 是可选的事实与复杂关系层，不是强制中心数据库。
```

图中最重要的边界是：

- Ontology Service 负责“是什么、如何理解、如何发现、是否可信”；
- Plan Translation Engine 负责把语义计划适配为执行计划；
- Query / Search Service 负责读数据和分析；
- Action / Workflow Service 负责改变状态和外部副作用；
- Data Catalog、Lineage、IAM 和 DLP 保持各自的事实来源和治理职责。

### 3.2 Ontology Service 的职责

Ontology Service 至少应提供以下能力。

#### 3.2.1 语义模型管理

- Entity Type / Class；
- Property / Attribute；
- Relation；
- Semantic Type；
- Definition、Synonym、Scope；
- Semantic Constraint；
- Object 和属性的敏感级别。

#### 3.2.2 语义发现

- 根据自然语言或标准标识解析概念；
- 发现属性、关系和相关概念；
- 返回定义、约束、适用范围和版本；
- 在当前 Agent 身份下过滤不可见语义。

#### 3.2.3 Mapping 和数据表示管理

- 维护语义数据表示；
- 引用 Data Catalog 资产；
- 记录字段转换、过滤和派生规则；
- 记录 SSOT、优先级、有效期、新鲜度和降级策略；
- 关联 Lineage 和质量信息。

#### 3.2.4 Capability 描述与发现

- 描述 Query Capability 和 Action Capability；
- 记录输入、输出、前置条件和副作用；
- 记录权限、Provider、SLA、幂等和事务要求；
- 返回适合当前用户、Agent 和任务的能力。

#### 3.2.5 治理和演进

- Draft、Review、Validate、Approve、Publish；
- Version、Effective Time、Deprecated Time；
- Owner、Source、Provenance、Confidence；
- Schema Diff、API Diff、业务变化和 Agent 反馈的影响分析；
- 回滚、审计和运行时版本控制。

### 3.3 Ontology Service 明确不负责什么

以下能力可以和 Ontology 协同，但不应默认归入 Ontology Service：

| 不应吞并的能力 | 应由谁负责 |
|---|---|
| 物理数据存储和全量实例副本 | 现有数据库、数据湖、缓存或事实服务 |
| 任意数据源的通用 SQL / DSL 执行 | Query Service、数据引擎、Adapter |
| 复杂搜索和分析计算 | Search / OLAP / Analytics Service |
| 完整数据目录和资产生命周期 | Data Catalog |
| 完整血缘采集和传播 | Lineage System |
| PII 脱敏、身份认证和基础授权 | IAM、DLP、Policy Engine |
| 业务状态机和事务编排 | Business Service、Workflow Engine |
| MCP 协议网关本身 | MCP Gateway 或 Agent Integration Layer |
| 任意领域的知识图谱存储 | 可选 KG / Graph Service |

原则不是禁止组合，而是禁止把所有职责都变成 Ontology Service 的内部实现，从而形成新的平台耦合中心。

### 3.4 Agent 级语义访问控制（Semantic Access Control）

传统 IAM 通常回答“这个主体能不能访问某个 API 或数据表”。企业 Agent 还需要语义级控制：

> **这个 Agent 在当前任务、用户、部门和目的下，能否看到某个业务概念、属性、关系或实例？**

例如同一个 `Customer`：

- 客服 Agent 可以看到姓名、订单和物流状态；
- 风控 Agent 可以看到风险标签和交易关系，但不能看到完整身份证号；
- 营销 Agent 可以看到经过脱敏的联系方式，但不能看到敏感风险关系；
- 外部合作方 Agent 只能看到被授权的客户子集和聚合结果。

语义访问控制至少应覆盖四个层次：

```text
1. Concept Level       是否能发现 Customer / RiskSubject？
2. Property Level      是否能看到 phone / nationalId / riskLevel？
3. Relation Level      是否能看到 owns / related_to / flagged_by？
4. Instance / Row Level 是否能看到某个客户或某个区域的数据？
```

策略可以使用以下条件：

```yaml
subject:
  agentRole: customer_service
  department: north_region
purpose: order_support
allowed:
  concepts: [Customer, Order, Shipment]
  properties: [Customer.name, Customer.phone_masked, Order.status]
  relations: [Customer.places, Order.fulfilled_by]
constraints:
  region: subject.region
  pii: masked
```

关键原则：

1. **权限过滤应发生在语义发现之前或同时发生。** 不应先把完整 Ontology 暴露给 Agent，再依赖 Agent 自律。
2. **拒绝发现与拒绝读取都必须支持。** 某些敏感关系即使存在，也不应向无权 Agent 暴露其名称和结构。
3. **语义授权不能替代底层数据授权。** Ontology Service 的结果必须继续受到执行服务、数据库行列权限和 DLP 检查。
4. **Action Capability 默认采用更强策略。** 写操作应结合职责分离、审批、前置条件、幂等键和工作流状态校验。
5. **授权结果应可解释、可审计。** Agent 或审计人员应知道某个属性被遮蔽、某个关系不可见的策略原因。

### 3.5 Semantic Plan 到物理查询的转换层

Ontology Service 只输出业务语义和计划约束是不够的。必须明确回答：谁把 Semantic Plan 翻译成 SQL、Cypher、GraphQL、搜索 DSL 或业务 API 调用？

这项职责由 **Plan Translation Engine / Adapter Layer** 承担。

#### 3.5.1 责任边界

```text
Agent / Planner
  生成业务意图
        ↓
Ontology Service
  校验概念、关系、约束、权限和可用能力
        ↓
Semantic Plan
  保持与物理技术无关
        ↓
Plan Translation Engine
  选择 Mapping、数据源、能力和适配器
        ↓
Physical Execution Plan
  SQL / Cypher / GraphQL / Search DSL / API request
        ↓
Execution Service
```

Ontology Service 负责提供和校验：

- 概念和关系是否合法；
- Semantic Plan 是否符合语义约束；
- 该 Agent 是否有权访问；
- 哪些 Mapping、数据源和 Capability 可用；
- 结果应携带哪些 provenance、freshness 和脱敏要求。

Translation Engine 负责：

- 解析 Semantic Plan；
- 选择合适的 Query Capability 或 Action Capability；
- 将语义属性映射为物理字段、表达式和 join；
- 选择 SQL、Cypher、GraphQL、搜索 DSL 或 API Adapter；
- 注入行级过滤、时间条件、脱敏和限流；
- 生成可执行计划并返回解释信息；
- 处理多源查询、结果合并和错误分类。

#### 3.5.2 为什么不能让 Ontology Service 直接执行所有查询

如果 Ontology Service 同时维护每种数据库的驱动、SQL 优化器、搜索 DSL 和 API 编排，它会迅速成为新的 Universal Query Engine：

- 语义模型与数据库技术强耦合；
- 每种物理系统的故障和升级都会影响语义服务；
- 查询性能、连接池和资源隔离进入控制面；
- 读写事务和工作流边界变得模糊。

更稳定的做法是：Ontology Service 发布语义契约和计划，Adapter Layer 负责面向具体执行技术的翻译。

### 3.6 多数据源冲突与权威源策略

同一个实体属性可能同时映射到多个物理源：

```text
Customer.phone
  ├── CRM MySQL：主数据，更新后 5 分钟内可见
  ├── ClickHouse：历史快照，延迟 1 小时
  └── Redis：实时缓存，可能短暂过期
```

这些值可能在格式、时效和业务状态上不一致。Mapping 层必须显式声明多源共存规则，而不能让 Adapter 随机选择“最快返回的源”。

#### 3.6.1 属性级 SSOT 与用途级权威

权威源应定义在属性或数据表示层，并允许按用途区分：

```yaml
concept: Customer.phone
sources:
  - id: crm_customer_master
    role: ssot
    priority: 100
    validFor: [customer_contact, compliance]
    freshnessSla: 5m
  - id: customer_realtime_cache
    role: cache
    priority: 80
    validFor: [low_latency_display]
    freshnessSla: 30s
  - id: customer_snapshot
    role: historical_snapshot
    priority: 40
    validFor: [analytics]
    freshnessSla: 1h
conflictPolicy: prefer_ssot_then_freshest
```

需要区分：

- **SSOT（Single Source of Truth）**：业务责任上最终权威的数据源；
- **Serving Source**：为了性能提供服务的缓存、索引或派生源；
- **Historical Source**：用于时间点分析的历史快照；
- **Fallback Source**：主源不可用时的降级来源。

#### 3.6.2 冲突解决与降级策略

策略至少需要定义：

1. 优先级：哪个源在何种用途下优先；
2. 新鲜度：结果是否满足请求要求的时间窗口；
3. 有效期：值在什么时间范围内成立；
4. 格式规范：手机号、货币、时区和编码如何标准化；
5. 冲突处理：返回 SSOT、并列结果、冲突标记还是转人工；
6. 可见性：Agent 是否能看到源差异和冲突原因；
7. 降级：主源不可用时是否允许使用缓存，是否必须降低置信度；
8. 审计：最终采用哪个值、来自哪里、何时读取。

当数据源不一致时，系统不应静默“拍脑袋”合并。应向 Agent 返回类似信息：

```text
Customer.phone = +86-138****8888
selectedSource = crm_customer_master
freshness = 3m
conflictDetected = true
conflictSources = [customer_realtime_cache]
confidence = degraded
```

### 3.7 与 Palantir Operational Ontology 的关系

Palantir 的 Ontology 思路与本文有明显相似之处：都强调通过业务对象、属性、关系和操作能力，把底层数据提升为接近业务世界的抽象。

更重的 Operational Ontology 往往把以下内容整合得更紧：

```text
Objects
Properties
Links
Actions
Functions
Security
Workflow
```

本文推荐的起点更薄：

```text
Semantic Model
  + Constraints
  + Mapping
  + Capability Contract
  + Governance
  + External Execution Services
```

两者不是互斥选择，而是演进路径上的不同位置：

- 早期先建立可治理、可发现、与执行解耦的语义控制面；
- 当业务确实需要统一对象操作、状态管理和工作流编排时，再谨慎扩大 Operational Ontology；
- 每次扩大职责都必须证明它解决了实际的跨系统协调问题，而不是为了“看起来完整”。

### 3.8 五条架构原则

#### 原则一：先抽象现实世界，再映射数据

不要从 `mysql.customer` 直接反推出企业完整 Ontology。先确定 Customer 的业务定义，再寻找它在各系统中的表示。

#### 原则二：Ontology 与 Physical Data 解耦

```text
Ontology.Customer ≠ mysql.customer
```

两者之间必须有显式、可审计、可演进的 Mapping。

#### 原则三：Ontology 描述能力，但不替代执行

```text
Ontology
  ↓
Capability Contract / Semantic Plan
  ↓
Execution Service
  ↓
Actual Query or Action
```

#### 原则四：语义权限必须与运行时身份绑定

Agent 能发现和读取什么，取决于用户、角色、部门、用途、数据敏感性和实例范围，而不是仅由静态 Ontology 决定。

#### 原则五：先建立薄而清晰的控制面，再按压力扩展

第一阶段应优先解决：

```text
Business Meaning
+ Constraints
+ Mapping
+ Capability Discovery
+ Semantic Access Control
+ Governance
```

不要一开始就把数据库、查询引擎、图数据库、工作流、Catalog、Lineage 和 MCP Gateway 全部塞入 Ontology Service。

---

## 四、建设方法与持续演进

### 4.1 建设目标不是“写几个 Entity JSON”

Ontology 建设的真实过程是：

```text
企业现实世界
    ↓
业务知识与数据资产发现
    ↓
业务概念识别
    ↓
Ontology 建模
    ↓
语义约束与访问策略
    ↓
Semantic Data Representation / Mapping
    ↓
Query / Action Capability Binding
    ↓
Translation 与执行验证
    ↓
业务专家审批
    ↓
Versioned Runtime
```

建设范围应围绕一个明确 Domain 和 Agent 场景开始，而不是试图一次建立“Entire Enterprise Ontology”。

### 4.2 八步初始化方法

#### 第一步：确定 Domain、目标和边界

明确：

- 业务域；
- 用户场景；
- 目标 Agent / Application；
- 需要支持的任务；
- 需要接入的数据范围；
- 需要暴露的 Query / Action Capability；
- 安全和合规限制；
- 成功指标。

例如，第一阶段可以选择“客户交易异常查询”，而不是“整个企业客户 Ontology”。

#### 第二步：盘点已有知识和资产

输入来源包括：

```text
Business Glossary
Data Dictionary
Data Catalog / Metadata
Data Lineage
Database Schema
API / GraphQL Definitions
Existing KG（如有）
Business Rules
Service Documentation
SQL / ETL / Pipeline
Domain Expert Knowledge
Historical Agent Queries
```

这些是 Ontology 的输入证据，但它们本身不等于 Ontology。

#### 第三步：识别业务概念而不是照抄数据库

先问：

```text
这个业务世界里到底有哪些对象、属性和关系？
```

再问：

```text
这些概念在企业数据中如何被表示？
```

如果 `client_no`、`cust_id` 和 `party_id` 在业务上都代表同一类客户标识，应统一到 `Customer.customerId`，但要保留来源、转换和置信度。

#### 第四步：定义四要素和语义约束

为每个核心概念定义：

- Class / Concept；
- Property / Attribute；
- Relation；
- Instance 标识策略；
- Definition、Synonym、Semantic Type；
- Owner、Scope、Provenance；
- 类型、基数、关系和不变量约束；
- 敏感级别和默认访问策略。

示例：

```text
Customer
  Properties:
    customerId: CustomerId, unique
    name: PersonName
    phone: PhoneNumber, PII
    status: CustomerStatus
  Relations:
    owns Account, 0..n
    places Order, 0..n
```

#### 第五步：建立 Semantic Data Representation 和 Mapping

Mapping 需要记录：

- 物理资产标识；
- 字段和表达式；
- 类型、单位、编码转换；
- join 和过滤条件；
- 有效时间；
- SSOT、缓存、快照和 fallback 角色；
- 数据新鲜度和质量门槛；
- Lineage 引用；
- 脱敏和语义访问控制要求。

#### 第六步：绑定 Query / Action Capability

为能力定义：

```yaml
name: getCustomerTransactions
kind: query
input:
  customerId: CustomerId
  startTime: DateTime
  endTime: DateTime
output:
  type: Transaction[]
preconditions:
  - Customer exists
permission:
  - transaction.read
freshness:
  maximumAge: 5m
provider: TransactionQueryService
```

Action Capability 还应额外定义：

```yaml
kind: action
sideEffects: [AccountStateChanged]
idempotencyKey: required
transactionBoundary: AccountService
approval: required_for_high_risk
workflow: account-freeze-workflow
```

#### 第七步：验证 Semantic Plan、Translation 和执行结果

不能只验证 Ontology JSON 是否格式正确，还要验证完整链路：

```text
Natural Language
  ↓
Semantic Discovery
  ↓
Semantic Plan
  ↓
Access Check
  ↓
Plan Translation
  ↓
Physical Execution
  ↓
Result Provenance / Freshness / Quality
```

验证内容包括：

- 语义计划是否违反约束；
- 权限过滤是否生效；
- 物理字段和关系映射是否正确；
- 主源和降级策略是否符合预期；
- 查询结果是否有 provenance；
- Action 是否满足事务、幂等和审批要求；
- 失败时是否能返回可诊断的反馈。

#### 第八步：业务专家审批并发布可信版本

AI 可以生成候选概念、关系、Mapping 和定义，但业务专家必须确认：

- 两个字段是否真的同义；
- 两个实体是否真的属于同一业务对象；
- 某个关系是否是业务关系；
- 某个源能否作为 SSOT；
- 某个属性是否敏感；
- 某个 Action 是否允许 Agent 调用。

> **AI 生成候选语义，业务专家确认最终语义。**

### 4.3 Ontology Studio：面向人的治理入口

如果 Ontology 是长期企业基础设施，就不能只有 Runtime API。建议提供 Ontology Studio，覆盖：

```text
Domain / Entity Modeling
Property / Relation Modeling
Constraint Modeling
Semantic Access Policy
Mapping Configuration
SSOT / Freshness Policy
Capability Binding
Translation Test
Provenance / Lineage
Impact Analysis
AI Proposal Review
Version / Diff
Approval / Publish / Rollback
```

标准生命周期为：

```text
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

Studio 的价值不是提供一个更漂亮的编辑器，而是把“修改语义”变成有责任人、有证据、有影响范围、有审批记录的工程流程。

### 4.4 Version、Provenance 和 Approval

Ontology 不能被当成普通配置文件。每个 Entity、Property、Relation、Mapping 和 Capability 至少应保留：

```text
Version
Owner
Source
Provenance
Confidence
Effective Time
Deprecated Time
Approval Status
Change Reason
Impact Summary
```

示例：

```text
Customer.phone
  version: 3
  status: approved
  owner: CRM Domain
  source:
    - CRM Data Dictionary
    - crm.customer.mobile
    - Business Manual v12
  effectiveAt: 2026-09-20
  reason: CRM customer contact model changed
  impact:
    mappings: 17
    queryCapabilities: 5
    actionCapabilities: 0
```

运行时应支持：

- 按版本读取；
- 灰度发布；
- 审计谁在何时批准了什么；
- 在执行结果中返回语义版本；
- 出现错误时回滚到上一个可信版本。

### 4.5 Physical Schema Evolution 不等于 Ontology Evolution

底层字段改名不一定意味着业务语义变化。

```text
旧 Mapping:
Customer.phone → mysql.customer.phone

新 Mapping:
Customer.phone → mysql.customer.mobile
```

如果业务含义仍是“客户主要联系号码”，这只是 Mapping Evolution。

但如果业务定义从“客户联系电话”变成“客户主要联系人电话”，适用范围、属性定义或约束发生了变化，就可能是 Ontology Evolution。

应至少区分：

```text
Physical Schema Evolution
        ↓
Mapping Evolution
        ↓
Ontology Evolution
        ↓
Business Evolution
```

不同层级的变化，影响范围、审批级别和回滚方式都不同。

### 4.6 下向演进：从数据和系统变化推导语义影响

需要持续监听的输入包括：

```text
Schema Diff
API / GraphQL Diff
SQL / ETL Diff
Lineage Diff
Catalog Change
Data Quality Change
Business Document Diff
Code Diff
```

例如：

```text
ClickHouse.customer_behavior.phone
        ↓ renamed
ClickHouse.customer_behavior.mobile
        ↓
Mapping Impact Analysis
        ↓
Customer.phone
getCustomerTransactions
CustomerProfileReport
        ↓
AI Change Proposal
        ↓
Human Review
```

注意：Change Engine 可以发现“可能受影响”，但不能仅凭字段改名决定业务语义。它需要结合 Catalog、Lineage、文档、使用记录和业务专家反馈。

### 4.7 上向演进：由 Agent 运行反馈驱动语义补全

仅依靠 Schema Diff 的演进闭环是不完整的。Agent 在真实运行中也会暴露语义缺口：

- 无法解析用户说的概念；
- 找到多个定义但无法消歧；
- Semantic Plan 违反未建模的业务约束；
- 没有合适的 Mapping 或 Capability；
- 查询结果因源冲突无法解释；
- Action 被权限、审批或前置条件拒绝；
- Agent 频繁使用同义词或绕过标准语义访问物理表。

这些反馈应进入 Feedback Engine：

```text
Agent Query / Plan / Execution
        ↓
Runtime Feedback
  ambiguity / failure / missing concept /
  missing mapping / policy denial / source conflict
        ↓
Feedback Classification
        ↓
Semantic Impact Analysis
        ↓
AI Proposal
  add concept / refine definition /
  add synonym / fix mapping /
  add constraint / bind capability /
  update access policy
        ↓
Human Review
        ↓
New Ontology Version
        ↓
Runtime Evaluation
```

反馈不应直接自动写入生产 Ontology。它应成为候选提案，并带有：

- 原始用户意图；
- Agent 发现过的概念和计划；
- 失败位置；
- 具体错误分类；
- 相关用户、用途和权限上下文；
- 建议修改以及受影响资产；
- 是否需要新增业务定义或 Capability。

### 4.8 AI 在建设与维护中的边界

AI 适合：

- 从文档、Schema、SQL、API 和查询日志发现候选概念；
- 聚合同义词；
- 推断属性和关系候选；
- 生成 Mapping 候选；
- 识别潜在冲突和影响范围；
- 根据 Agent 失败生成补全提案；
- 自动生成测试用例和变更说明。

AI 不应未经审批自动决定：

- 跨部门概念是否相同；
- 哪个数据源是 SSOT；
- 一个敏感关系是否对某类 Agent 可见；
- 一个 Action 是否可以执行真实副作用；
- 一个业务约束是否可以删除或放宽。

目标不是让 Ontology 不需要人维护，而是让人把时间集中在真正需要业务判断的语义决策上。

### 4.9 双向动态演进闭环

这是全文第二张核心架构图：

```text
                 ┌──────────────────────────────┐
                 │      Business / Data World    │
                 │                              │
                 │  Business Change              │
                 │  Schema / API / Lineage Diff  │
                 └──────────────┬───────────────┘
                                │ 下向变化
                                ▼
                         ┌──────────────┐
                         │ Change Engine │
                         └──────┬───────┘
                                │
                                │
┌───────────────────┐           ▼           ┌────────────────────────┐
│ Agent Runtime      │──上向反馈──>│ Feedback / Impact Analysis │
│ ambiguity          │             └──────────┬─────────────┘
│ query failure      │                        │
│ missing mapping    │                        ▼
│ policy denial      │                 ┌──────────────┐
│ source conflict    │                 │ AI Proposal   │
└───────────────────┘                 └──────┬───────┘
                                             │
                                             ▼
                                      ┌──────────────┐
                                      │ Human Review  │
                                      │ + Validation  │
                                      └──────┬───────┘
                                             │
                                             ▼
                                      ┌──────────────┐
                                      │ Ontology      │
                                      │ Version       │
                                      └──────┬───────┘
                                             │
                                      Approve / Publish
                                             │
                                             ▼
                                      ┌──────────────┐
                                      │ Agent Runtime │
                                      │ + Services    │
                                      └──────────────┘

 下向闭环：物理/业务变化 → 检测 → 影响分析 → 提案 → 审批 → 发布
 上向闭环：Agent 反馈 → 分类 → 影响分析 → 提案 → 审批 → 发布
```

双向闭环的意义是：

> **Ontology 不仅要跟得上底层系统的变化，也要从 Agent 的真实使用中发现业务语义尚未被建模的部分。**

---

## 五、Agent 交互机制与语义执行

### 5.1 Agent 不应一次获得整个 Ontology

大型企业可能拥有数千个 Entity Type、海量属性和复杂关系。把完整 Ontology 一次注入 Context 会造成：

- 上下文浪费；
- 相关性下降；
- 敏感概念过度暴露；
- Agent 更容易错误选择相似概念。

应采用渐进式 Semantic Discovery：

```text
自然语言任务
    ↓
resolveConcept("客户")
    ↓
Customer
    ↓
发现相关属性和关系
    ↓
Account / Order / Device
    ↓
围绕任务继续发现
    ↓
Transaction + 约束 + 可用能力
```

推荐的运行时 API 可以分为三组。

#### 5.1.1 Semantic Discovery API

```text
searchOntology(query, context)
resolveConcept(name, context)
getEntityType(id, context)
getProperties(entity, context)
getRelationships(entity, context)
getRelatedConcepts(entity, relation, context)
getDefinitions(id, context)
getSemanticTypes(id, context)
getConstraints(id, context)
```

`context` 至少应包含用户、Agent 身份、部门、用途、区域、语义版本和安全策略。

#### 5.1.2 Mapping Discovery API

```text
getDataRepresentations(concept, context)
getMappings(concept, purpose, context)
getAuthoritativeSources(property, purpose, context)
getFreshnessPolicy(property, purpose, context)
getProvenance(concept, context)
```

#### 5.1.3 Capability Discovery API

```text
getCapabilities(concept, context)
getQueryCapabilities(concept, context)
getActionCapabilities(concept, context)
getCapability(name, context)
validatePreconditions(capability, input, context)
```

返回结果不应只给出名称，还应包含权限、前置条件、输入输出、数据新鲜度、副作用、事务边界、幂等要求和 Provider。

### 5.2 Ontology、Skills 与 MCP 的关系

三者解决的问题不同：

```text
Ontology
  世界是什么？概念如何关联？

Skill
  某类任务应该采用什么步骤和策略？

MCP Tool
  通过什么标准接口调用某个能力？
```

例如：

```text
customer_risk_analysis Skill
        ↓ 任务流程
Ontology Service
        ↓ 发现 Customer / Account / Transaction
Query Capability / Action Capability
        ↓ MCP 或其他接口
Enterprise Services
```

因此：

> **MCP / Skills 是 Agent 使用能力的机制；Ontology 是 Agent 正确理解企业能力和数据所依赖的语义模型。**

Ontology 可以通过 MCP 暴露，但 Ontology 本身不等于 MCP Gateway，也不应把所有业务工具都收编为 Ontology API。

### 5.3 Semantic Plan：在语义理解和物理执行之间建立契约

Agent 理解用户任务后，应先生成与物理技术无关的 Semantic Plan。

用户问题：

> 帮我查一下张三最近 7 天有没有异常交易。

语义计划可以表达为：

```json
{
  "intent": "find_abnormal_transactions",
  "subject": {
    "concept": "Customer",
    "identifier": "张三"
  },
  "traversal": [
    {
      "relationship": "owns",
      "target": "Account"
    },
    {
      "relationship": "hasTransaction",
      "target": "Transaction"
    }
  ],
  "constraints": {
    "time": "last_7_days",
    "abnormality": {
      "definition": "RiskTransaction"
    }
  },
  "output": {
    "fields": ["transactionId", "amount", "timestamp", "riskReason"]
  }
}
```

它表达：

```text
业务对象
+ 关系路径
+ 语义约束
+ 任务意图
+ 输出要求
```

它不直接表达：

```text
SQL
ES DSL
Cypher
Redis Command
某个内部 API 的私有参数
```

Semantic Plan 的价值是让 Agent 先完成可解释的语义推理，再由 Ontology Service 校验，由 Translation Engine 负责适配物理执行。

### 5.4 Semantic Plan 的校验阶段

Semantic Plan 进入执行前，至少应经过以下检查：

1. **概念校验**：实体、属性和关系是否存在；
2. **语义校验**：关系路径和业务约束是否合法；
3. **权限校验**：当前 Agent 是否能发现和读取这些语义；
4. **数据校验**：是否存在满足用途和新鲜度要求的 Mapping；
5. **能力校验**：是否有匹配的 Query / Action Capability；
6. **风险校验**：是否涉及 PII、敏感关系或高风险动作；
7. **计划校验**：输出、过滤、分页和成本是否符合策略；
8. **版本校验**：计划使用的 Ontology 版本是否仍可执行。

如果校验失败，系统应返回结构化反馈，而不是让 Agent 直接猜一个表或绕过 Ontology。

### 5.5 Plan Translation Engine 的执行路径

完整执行路径是：

```text
User Task
    ↓
Agent Semantic Understanding
    ↓
Progressive Discovery
    ↓
Semantic Plan
    ↓
Ontology Validation + Semantic ACL
    ↓
Capability Selection
    ↓
Plan Translation Engine
    ↓
SQL / Cypher / GraphQL / Search DSL / API Request
    ↓
Execution Service
    ↓
Result Normalization
    ↓
Provenance + Freshness + Quality + Policy Metadata
    ↓
Agent Explanation
```

Translation Engine 可以有多个 Adapter：

```text
Relational Adapter
  Semantic Plan → SQL

Graph Adapter
  Semantic Plan → Cypher / Gremlin

Search Adapter
  Semantic Plan → Search DSL

API Adapter
  Semantic Plan → GraphQL / REST / gRPC request

Workflow Adapter
  Action Capability → Workflow invocation
```

Adapter 输出应保留从语义属性到物理字段、从关系路径到 join / edge、从数据策略到执行过滤条件的可解释映射。

### 5.6 Query Capability 与 Action Capability 的 Agent 交互差异

#### 5.6.1 查询场景

查询可以在权限允许、成本可控和数据新鲜度满足时自动执行，但结果必须带有：

```text
ontologyVersion
source
retrievedAt
freshness
quality
maskingApplied
conflictStatus
```

Agent 不应把“查到一个值”自动解释成“这个值就是业务真相”。

#### 5.6.2 动作场景

Action Capability 涉及外部副作用，建议采用：

```text
发现能力
  ↓
检查权限和前置条件
  ↓
生成动作预览
  ↓
展示影响对象和参数
  ↓
用户确认 / 审批
  ↓
Workflow Engine 执行
  ↓
返回状态、幂等键和审计信息
```

对于高风险动作，即使 Agent 拥有调用权限，也可以要求人工确认或专门审批。Ontology Service 负责描述条件和风险标签，但不应自行绕过 Workflow Engine。

### 5.7 Agent 运行反馈的分类

反馈应被结构化，而不是只记录一段错误文本：

```text
ConceptNotFound
AmbiguousConcept
MissingProperty
MissingRelation
ConstraintViolation
NoMapping
StaleData
SourceConflict
CapabilityNotFound
PermissionDenied
ActionApprovalRequired
TranslationFailure
ExecutionFailure
ResultQualityIssue
```

不同反馈意味着不同的修复方向：

| 反馈 | 可能需要的语义改进 |
|---|---|
| ConceptNotFound | 新增概念、同义词或领域别名 |
| AmbiguousConcept | 补充定义、适用范围、消歧属性 |
| MissingRelation | 建模关系或澄清跨域边界 |
| NoMapping | 建立数据表示和物理映射 |
| SourceConflict | 更新 SSOT、优先级或冲突策略 |
| PermissionDenied | 检查语义策略，不应直接放宽权限 |
| TranslationFailure | 修复 Mapping、Adapter 或能力契约 |
| ConstraintViolation | 补充约束或修正 Agent 计划 |
| ActionApprovalRequired | 进入审批和工作流，而不是重试绕过 |

### 5.8 完整示例：查询客户异常交易

用户提出：

> 帮我查一下张三最近 7 天有没有异常交易。

Agent 的合理流程是：

1. `resolveConcept("张三")`，得到受当前权限约束的 `Customer` 候选；
2. 确认 `Customer` 的身份匹配能力和消歧规则；
3. 发现 `Customer owns Account`、`Account hasTransaction Transaction`；
4. 获取“异常交易”的业务定义、风险标签和适用范围；
5. 获取最近 7 天交易数据的 Mapping、SSOT 和新鲜度；
6. 生成 Semantic Plan；
7. 校验语义约束、权限和能力；
8. 由 Translation Engine 选择 Transaction Query Service；
9. 执行并返回带有来源、时间、质量和脱敏信息的结果；
10. 如果发生歧义、数据冲突或权限拒绝，记录结构化反馈，供后续演进。

Agent 不需要知道底层是否使用 ClickHouse、MySQL、GraphQL 或一个专有风控 API，也不应自行拼接一个未经语义校验的 SQL。

---

## 六、企业落地检查表与总结

### 6.1 Domain 与目标

```text
□ Domain 范围是否明确？
□ 第一阶段场景是否足够聚焦？
□ 目标用户、Agent 和应用是否明确？
□ 成功指标是否可验证？
□ 哪些概念不在当前范围内是否明确？
```

### 6.2 Semantic Model

```text
□ Class / Concept 是否定义？
□ Property / Attribute 是否定义？
□ Relation 是否定义方向和基数？
□ Instance 标识策略是否明确？
□ Definition 是否清晰？
□ Semantic Type 是否明确？
□ 同义词和别名是否处理？
□ 适用范围和领域 Owner 是否明确？
□ 是否区分业务语义与物理 Schema？
```

### 6.3 Semantic Constraints

```text
□ 类型约束是否定义？
□ 唯一性和基数约束是否定义？
□ 关系合法性是否定义？
□ 基本业务不变量是否定义？
□ 状态和时间约束是否定义？
□ 复杂业务逻辑是否仍由规则/服务/工作流负责？
```

### 6.4 Data Representation 与 Mapping

```text
□ 是否有独立的 Semantic Data Representation？
□ Mapping 是否引用 Data Catalog 而非复制？
□ 是否记录字段、类型、单位和编码转换？
□ 是否记录 join、派生和过滤条件？
□ 是否定义 SSOT、缓存、快照和 fallback？
□ 是否定义数据新鲜度和有效期？
□ 是否关联 Lineage 和质量信息？
□ 多源冲突是否可检测、可解释、可审计？
□ 是否有数据脱敏和语义授权要求？
```

### 6.5 Knowledge Graph 的取舍

```text
□ 是否真的需要多跳关系和图推理？
□ 是否已有可复用的图数据能力？
□ KG 是否被定位为可选事实/关系层？
□ 是否避免为了建 Ontology 而强行复制所有实例？
□ 关系型、搜索型和 API 联邦访问是否已经足够？
```

### 6.6 Capability 与执行

```text
□ Query Capability 是否定义？
□ Action Capability 是否定义？
□ 是否明确区分读与写的权限模型？
□ Query 是否有 freshness、quality 和 provenance？
□ Action 是否有前置条件、事务边界和幂等键？
□ 高风险 Action 是否接入 Workflow Engine？
□ Capability Provider 和 SLA 是否记录？
□ Semantic Plan 到 SQL/Cypher/GraphQL/API 的 Translation Engine 是否明确？
□ Ontology Service 是否避免直接吞并所有执行逻辑？
```

### 6.7 Semantic Access Control

```text
□ 是否按 Agent / 用户 / 部门 / 用途过滤概念？
□ 是否支持属性级、关系级和实例级授权？
□ PII 是否支持遮蔽、脱敏和不可发现？
□ 敏感关系是否对无权 Agent 隐藏？
□ 语义权限是否与底层 IAM、DLP 和数据权限联动？
□ Action 是否有更严格的审批和职责分离？
□ 权限拒绝是否可解释、可审计？
```

### 6.8 Runtime 与 Agent

```text
□ 是否支持渐进式 Semantic Discovery？
□ 是否按运行时身份返回语义结果？
□ 是否支持 Semantic Plan？
□ 是否在执行前校验约束和权限？
□ 是否返回来源、新鲜度、质量和语义版本？
□ 是否区分 Query 与 Action 的交互流程？
□ MCP / Skills 与 Ontology 的边界是否清晰？
□ Agent 是否不会被迫直接猜测物理表结构？
```

### 6.9 Governance 与 Evolution

```text
□ 是否有 Owner、Source、Provenance 和 Confidence？
□ 是否有 Version、Effective Time 和 Deprecated Time？
□ 是否有 Draft / Review / Validate / Approve / Publish？
□ 是否支持 Schema / API / Lineage / Catalog 变化检测？
□ 是否支持语义影响分析？
□ 是否支持回滚和审计？
□ 是否有 Agent 失败、歧义和语义缺口反馈闭环？
□ AI 提案是否必须经过业务专家确认？
□ 是否区分 Schema Evolution、Mapping Evolution 和 Ontology Evolution？
```

### 6.10 最终判断：是否滑向了“超级 Ontology Service”

可以用以下问题做反向检查：

```text
□ Ontology Service 是否开始保存所有业务实例副本？
□ 是否正在重新建设 Data Catalog？
□ 是否正在重新建设完整 Lineage Engine？
□ 是否把所有数据库查询都放进控制面？
□ 是否把 MCP Gateway、Workflow 和 Action Runtime 全部内置？
□ 是否要求所有数据必须进入 Knowledge Graph？
□ 是否让语义模型依赖某个具体数据库的表结构？
□ 是否让 Agent 绕过语义层直接访问物理数据？
```

如果多数答案为“是”，Ontology Service 很可能已经从语义控制面滑向了新的超级数据平台。

### 6.11 最终心智模型

可以把完整体系压缩为以下几句话：

> **Ontology 定义企业业务世界是什么。**
>
> **Classes、Properties、Relations 和 Instances 是理解 Ontology 的四个基本入口；其中 Ontology 主要规定结构和语义，实例数据可以由外部系统提供。**
>
> **Schema 描述某个系统如何存储，Knowledge Graph 保存可选的实例和复杂关系事实，Data Catalog 描述数据资产，Lineage 描述数据如何产生和流转。**
>
> **Mapping 把业务概念连接到企业数据，Query Capability 描述可以读取和分析什么，Action Capability 描述可以安全地改变什么。**
>
> **Semantic Plan 表达业务意图、关系路径和语义约束；Plan Translation Engine 再把它翻译成 SQL、Cypher、GraphQL、搜索 DSL 或业务 API 调用。**
>
> **Semantic Access Control 确保不同 Agent 只能发现和使用其身份、用途和范围允许的概念、属性、关系和实例。**
>
> **Ontology Service 是薄的 Semantic Control Plane：它让 Agent 先理解企业世界，再发现数据与能力，最后由专门的执行服务完成查询或动作。**
>
> **Ontology 的演进既由业务、Schema、API 和数据变化驱动，也由 Agent 的歧义、失败、冲突和语义缺口反馈驱动；两条路径都必须经过影响分析、AI 提案、人工审批和版本发布。**

最终可以用一句话概括：

> **Ontology 让企业数据和能力拥有统一的业务语义；Ontology Service 让这种语义以安全、可发现、可治理、可演进的方式服务于 AI Agent，而不把语义控制面变成新的数据面。**
