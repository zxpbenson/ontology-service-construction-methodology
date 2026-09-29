# Ontology and Ontology Service: A Semantic Control Plane Architecture for Enterprise AI

> v4 — English edition
>
> This document is intended for enterprise architects, data platform builders, Ontology modelers, AI Agent developers, and data service engineers. It answers six questions:
>
> 1. What exactly is an Ontology, and why has it become enterprise infrastructure again in the AI era?
> 2. What are the boundaries between Ontology, Schema, Knowledge Graph, Data Catalog, Mapping, Lineage, and Capability?
> 3. What should an Ontology Service own, and what must it explicitly refuse to own?
> 4. How should an enterprise build, govern, and continuously evolve an Ontology, starting from one business domain?
> 5. How can an Agent progressively discover semantics, generate a Semantic Plan, and use enterprise data and capabilities safely?
> 6. How can we tell whether an Ontology Service is drifting toward becoming a “super data platform”?

The central position of this document is:

> **An Ontology is an abstraction, formalization, and machine-understandable semantic model of business reality. An Ontology Service is a thin Semantic Control Plane that makes this model available to Agents and applications; it is not a database, Universal Query Engine, or Workflow Engine.**

---

## I. Positioning and Core Value

### 1.1 What Is an Ontology?

In simple terms, an Ontology is a “standard domain language” that both machines and people can understand. It can also be understood as an enterprise-wide business world model.

More precisely:

> **An Ontology is an abstraction, formalization, and machine-understandable semantic model of the concepts in a particular business domain.**

It answers:

- What important entities and concepts exist in this business world?
- What properties does each type of entity have?
- What business relationships exist between entities?
- What do each concept, property, and relationship mean in business terms?
- What semantic constraints, cardinality constraints, and applicability conditions must hold?
- How should data from different systems be understood as the same business object?
- What read capabilities and business actions can be discovered around these objects?

An Ontology describes “what the real world is” before it describes tables and columns in any particular database.

For example, an e-commerce domain may contain:

```text
Customer
Product
Order
Payment
Shipment
```

and:

```text
Customer ── places ──> Order
Order    ── contains ─> Product
Order    ── paid_by ──> Payment
Order    ── fulfilled_by ──> Shipment
```

This describes concepts and relationships in the business world, not the physical structures of `mysql.order`, `es.product_index`, or `kafka.order_event`.

### 1.2 The Four Core Elements of an Ontology

A practical Ontology contains at least four basic categories of elements.

#### 1.2.1 Classes / Concepts

A class is an abstract category in a domain. It answers: “What kinds of objects exist in this world?”

For example:

```text
Customer
Product
Order
Account
Transaction
Device
```

A class is not the same thing as a table. A `Customer` may be represented jointly by CRM master data, customer profiles, real-time status, and behavioral detail systems.

#### 1.2.2 Properties / Attributes

A property is a characteristic owned by a class. It answers: “What information does this object have?”

For example:

```text
Customer.customerId
Customer.name
Customer.phone
Customer.riskLevel

Product.productId
Product.price
Product.inventory
```

A property should carry business meaning, semantic type, scope, sensitivity, data freshness, and provenance. It should not be treated as merely a string field name.

For example:

```yaml
concept: Customer.phone
semanticType: PhoneNumber
meaning: Customer's primary contact phone
sensitivity: PII
applicability: ActiveCustomer
```

#### 1.2.3 Relations

A relation connects different classes or instances. It answers: “How are these objects connected in the business?”

For example:

```text
Customer ── owns ──> Account
Customer ── places ──> Order
Account  ── has ────> Transaction
Device   ── used_by ─> Customer
```

A relation is more than a technical join key. It should define relationship semantics, direction, cardinality, validity period, and access restrictions. For example, `Account belongs_to Customer` and `Customer has Account` are two views of the same business relationship, not two unrelated field associations.

#### 1.2.4 Instances / Individuals

An instance is a concrete data point. It answers: “What specific objects currently exist in this world?”

For example:

```text
Customer#12345
Product#IPHONE15
Order#ORD-20260929-001
```

“Zhang San” is an instance of `Customer`; “iPhone 15” is an instance of `Product`.

The distinction is important:

> **An Ontology defines what an instance is, how it is described, and how it can be related to other instances. It does not therefore have to store all instance data.**

Instances may be provided by relational databases, document databases, search engines, event streams, business APIs, or, where appropriate, a Knowledge Graph.

### 1.3 Ontology, Schema, and Knowledge Graph: An Intuitive Distinction

To avoid confusing the three, consider this analogy:

| Concept | Analogy | Main question | Typical content |
|---|---|---|---|
| **Schema** | The construction blueprint for one building | How does a particular system store data? | MySQL tables, fields, types, indexes, constraints |
| **Ontology** | The city plan and building standards for an entire city | What is the business world, and how should it be understood? | Customer, Order, owns, PhoneNumber, business constraints |
| **Knowledge Graph** | The actual buildings, streets, and residents in the city | What facts and instances currently exist? | Zhang San is a Customer; Zhang San owns Account-001 |

More formally:

> **A Knowledge Graph can be viewed as an Ontology plus instances and facts, but an Ontology is not a Knowledge Graph.**

For example:

```text
Ontology:
Customer ── owns ──> Account

Knowledge Graph:
Customer#12345 ── owns ──> Account#98765
```

The structure of the `mysql.customer` table belongs to Schema. The business definitions of `Customer`, `owns`, and `Account` belong to the Ontology. A current fact such as `Customer#12345 owns Account#98765` belongs to a Knowledge Graph or another fact-data service.

### 1.4 Why Ontology Matters Again in the Big Data and AI Era

Historically, data work centered on data warehouses, ETL, dimensional modeling, and table structures. Enterprise data is now increasingly distributed across systems:

```text
MySQL          customer
ClickHouse     customer_behavior
Elasticsearch  customer_profile
Hive           customer_snapshot
Redis          customer_realtime_status
Kafka          customer_event
Business API   customer-service
```

The same business concept may have different names in different departments:

```text
CRM       client_no
Sales     cust_id
Finance   customer_number
Risk      party_id
```

Based only on table names, field names, and API names, an Agent cannot reliably determine:

- Which data represents the same entity;
- Which field has the business meaning relevant to the task;
- Which data source is authoritative;
- Which data is real-time state and which is only a historical snapshot;
- Which relationships are genuine business relationships and which are merely technical joins;
- Which properties are PII or restricted semantics;
- Which capabilities can be read automatically and which actions require approval or a workflow.

The value of an Ontology is concentrated in three areas.

#### 1.4.1 Building a Unified Semantic Layer

An Ontology goes beyond individual database tables and establishes stable business context at the top:

```text
Different physical names: client_no / cust_id / party_id
Unified business meaning:  Customer.customerId
```

Underlying tables may be migrated, split, merged, or replaced. As long as the business meaning remains stable, the Ontology and its semantic interfaces can remain stable as well.

#### 1.4.2 Giving LLMs and Agents an Enterprise World Model

An LLM can understand natural language, but it does not automatically understand hundreds of internal tables, thousands of fields, or complex data permissions. If an Agent must infer business meaning directly from physical Schema, it may:

- Choose the wrong table or field;
- Treat a snapshot as real-time data;
- Conflate concepts that share a name but have different meanings;
- Ignore relationship cardinality and business constraints;
- Generate a query that violates permissions or business conditions.

An Ontology lets the Agent understand this first:

```text
Customer → Account → Transaction
```

The Agent can then enter actual data and services through Mapping, Capability, and permission context. In GraphRAG and similar scenarios, an Ontology can also act as an enterprise knowledge navigation map, helping retrieval and reasoning use consistent semantics instead of exposing unmanaged table structures directly to the model.

#### 1.4.3 Improving Data Asset Interoperability

The main obstacle to cross-system collaboration is often not data format but semantic inconsistency. An Ontology provides a machine-understandable shared vocabulary so that different systems, teams, and Agents can exchange data around the same business concepts.

W3C standards such as OWL and RDF can be used as expression formats. However, an enterprise should not force all data into a particular graph database or standard format merely in order to use an Ontology. Standards are tools for expression and interoperability, not mandatory storage architectures.

### 1.5 The Core Position of Ontology Service

If an Ontology is the semantic model of the enterprise business world, an Ontology Service is the service layer that provides this model:

> **An Ontology Service is a thin Semantic Control Plane in the enterprise AI data architecture.**

It enables Agents and applications to:

1. Discover entities, properties, and relationships from natural-language concepts;
2. Understand business definitions, semantic constraints, and applicability;
3. Find corresponding data representations, data sources, and data quality information;
4. Discover available read capabilities and business actions;
5. Generate or validate a Semantic Plan decoupled from the execution layer;
6. Perform semantic impact analysis and governance approval when changes occur.

This does not mean that an Ontology Service must own:

- All enterprise instance data;
- Copies of all data assets;
- A new super-database;
- A Universal Query Engine;
- A full-featured Knowledge Graph;
- All capabilities of a Data Catalog or Lineage system;
- All query, computation, write, and workflow execution capabilities.

The simplest boundary is:

> **Ontology describes the world; Data / Action Services operate on the world; Ontology Service connects the two through trusted, discoverable, and governable semantics.**

---

## II. Conceptual Boundaries and the Five-Layer Model

### 2.1 One Unified Boundary Table

All of these systems describe the enterprise world, but from different perspectives. Logical boundaries must remain clear even if the components are temporarily deployed on one platform.

| Concept | Core question | Main objects | Typical content | Relationship to Ontology |
|---|---|---|---|---|
| **Ontology** | What is the world? | Business concepts, properties, relations, constraints | Customer, Order, owns, PhoneNumber, state constraints | Defines the semantic model of the business world |
| **Schema** | How does a particular system store data? | Tables, columns, types, indexes, constraints | `mysql.customer.phone varchar(32)` | Physical system structure; not business semantics |
| **Knowledge Graph** | What facts currently exist? | Instances, facts, entity relationships | Zhang San owns Account 001 | Optional facts and complex-relationship layer that consumes Ontology definitions |
| **Data Catalog / Metadata** | Where are the data assets? | Databases, tables, fields, Topics, Datasets | Owner, location, type, quality, sensitivity | Ontology should reference it rather than copy it |
| **Data Mapping** | How is a business concept represented by data? | Mappings from semantic objects to physical assets | `Customer.phone → crm.customer.mobile` | Connects the semantic world to the data world |
| **Data Lineage** | How is data produced and propagated? | Sources, transformations, derivations, dependencies | Kafka → Flink → ClickHouse | Provides evidence for trust, impact analysis, and freshness |
| **Query Capability** | What can be read and analyzed? | Query, search, aggregation, reasoning interfaces | Find customers, search orders, aggregate transactions | Describes executable read capabilities; does not define semantics |
| **Action Capability** | What can be changed? | Writes, approvals, state changes, business actions | Freeze an account, create an order, issue a refund | Requires stronger permission, transaction, idempotency, and workflow controls |
| **Execution Service** | How is it actually executed? | SQL, APIs, search, workflow, transaction systems | MySQL, GraphQL, Order Service | Executes a Semantic Plan or Capability Contract |
| **Agent** | How can semantics and capabilities be composed for a user goal? | Tasks, plans, decisions, feedback | Discover, plan, call, explain | Consumes the semantic control plane; should not guess physical implementation |

These concepts may cooperate within the same product, but deployment consolidation must not blur their responsibilities.

### 2.2 Ontology and Data Catalog: Reference, Do Not Copy

A Data Catalog describes data assets:

```text
mysql.customer.phone
  type: varchar(32)
  owner: CRM Team
  classification: PII
  updatedAt: ...
```

An Ontology describes business concepts:

```text
Customer.phone
  semanticType: PhoneNumber
  meaning: Customer primary contact phone
  required: true
```

They answer different questions:

- Ontology: What does this property mean in the business?
- Catalog: Where is this field, who owns it, what is its quality, and is it sensitive?

The recommended relationship is:

```text
Ontology.Customer.phone
        │ semantic mapping
        ▼
Data Catalog Asset
        │
        ▼
Physical Data
```

An Ontology may reference the Catalog asset identifier, owner, sensitivity, quality, and freshness. It should not copy the entire Catalog. Otherwise, the result is duplicated metadata, inconsistent state, and unclear ownership.

### 2.3 Mapping: An Independent Layer Between Business Semantics and Physical Data

Mapping should not be reduced to a one-to-one field correspondence:

```text
logicalField → physicalField
```

In a real enterprise, one business property may involve:

- Field renaming;
- Type, unit, and code conversion;
- Joins across multiple tables;
- Merging multiple data sources;
- Derived calculations;
- Valid-time filtering;
- Choosing between master data and snapshots;
- Data quality and freshness evaluation;
- Data masking and semantic authorization.

The intermediate semantic data representation should therefore be explicit:

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

For example:

```text
Customer
  ├── CustomerIdentity
  │     └── CRM customer master
  ├── CustomerBehavior
  │     └── ClickHouse behavior events
  └── CustomerRealtimeStatus
        └── Redis realtime status
```

A business entity should not be defined by one table. A table is one system's physical representation of the business; Mapping is the explicit contract that connects the two.

### 2.4 Lineage: Evidence for Semantic Trust, Not a Replacement for the Semantic Model

Lineage focuses on:

```text
Where did the data come from?
What processing did it undergo?
When was it updated?
Which downstream assets are affected?
```

For example:

```text
Kafka.login_event
        ↓
Flink enrichment
        ↓
ClickHouse.customer_behavior
        ↓
Ontology.Customer.lastLoginTime
```

An Ontology can reference or consume Lineage information to:

- Determine data origin and trustworthiness;
- Assess data freshness;
- Perform Schema / Mapping change impact analysis;
- Explain result provenance to an Agent;
- Discover potential impact on downstream capabilities and reports.

However, the Ontology Service should not therefore reimplement a complete Lineage Engine.

### 2.5 The Five-Layer Model: From Business World to Physical Systems

A practical five-layer model is:

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

#### 2.5.1 L1: Business Ontology

Question: What is the real business world?

```text
Customer
Account
Order
Transaction
Device
```

This includes Classes, Properties, Relations, definitions, synonyms, semantic types, and applicability.

#### 2.5.2 L2: Semantic Constraints

Question: Which relationships between these concepts are valid?

```text
Customer.customerId is UNIQUE
Account belongs_to exactly_one Customer
Transaction.amount >= 0
Order.createTime <= Order.payTime
Order.status = CANCELLED implies cancelTime != null
```

Semantic constraints should be distinguished from complete business execution logic. An Ontology can express types, cardinality, relationships, and basic invariants, while complex computation, approval, and state machines remain the responsibility of dedicated rule, business, or workflow systems.

#### 2.5.3 L3: Semantic Data Representation / Mapping

Question: How is the real world represented by enterprise data?

```text
Customer
  ├── Identity Representation
  ├── Behavior Representation
  └── Realtime Status Representation
```

This layer records physical assets, transformations, filters, time conditions, authoritative sources, and fallback policies. It must not mistake any one database table for the complete business entity.

#### 2.5.4 L4: Capability

Question: How can data be accessed safely, or how can business state be changed?

L4 must explicitly separate two categories:

**Query Capability:**

- Query entities and relationships;
- Search and filter;
- Aggregate and analyze;
- Read real-time state;
- Generate read-only explanations or reports.

**Action Capability:**

- Create, update, and delete business objects;
- Change the state of orders, accounts, or work items;
- Initiate refunds, freezes, approvals, and similar actions;
- Trigger external side effects or workflows.

Their engineering requirements differ:

| Dimension | Query Capability | Action Capability |
|---|---|---|
| Primary risks | Unauthorized reading, sensitive-data leakage, misinterpretation of results | Incorrect side effects, state corruption, duplicate execution |
| Permissions | Dataset, property, relation, row-level, and column-level permissions | Role, separation of duties, approval, and action permissions |
| Consistency | Explicitly labeled eventual consistency may be acceptable | Usually requires transactions, state checks, and a strong-consistency boundary |
| Idempotency | Usually satisfied naturally by reads | Must explicitly define idempotency keys and duplicate-request handling |
| Execution | Query Service, Search Service, analytics engine | Action Service, Workflow Engine, transaction service |
| Default Agent policy | May execute automatically when authorized and explain provenance | Preview, confirmation, or human approval by default |

An Ontology Service may describe and discover both categories, but it should not absorb the Action Runtime. For write operations, a dedicated Workflow Engine is generally the safer execution boundary.

#### 2.5.5 L5: Physical Data / Systems

This includes:

```text
MySQL / PostgreSQL / TiDB
ClickHouse / Hive / Spark
Elasticsearch / HBase / Redis
Kafka / Object Storage
Graph Database (optional)
Business API / Workflow Engine
```

L5 is the reality of execution and storage. It should not determine the entire structure of business semantics in reverse.

### 2.6 The Proper Role of a Knowledge Graph: An Optional Facts and Complex-Relations Layer

A Knowledge Graph can be valuable, but it should not be mandatory storage for every enterprise Ontology implementation.

Scenarios that may justify a KG include:

- Multi-hop relationship exploration;
- Entity disambiguation and entity resolution;
- Complex relationship inference;
- Fact versioning and provenance management;
- GraphRAG;
- Business workloads that require frequent relationship traversal.

A KG should not be forced into scenarios where:

- Workloads are primarily structured attribute queries;
- Existing relational or analytical data services are already stable;
- Relationships are limited and can be federated through APIs or SQL;
- The enterprise does not yet have the governance and operational capability for graph data.

The recommended position is therefore:

> **A Knowledge Graph is an optional facts and complex-relationship execution layer for an Ontology, not an inevitable form of Ontology Service.**

An Ontology can use Mapping to federate queries directly to existing MySQL, ClickHouse, ES, API, and other systems. Whether to build a KG should be determined by relationship complexity, query patterns, real-time requirements, cost, and governance capability.

---

## III. Architecture Design and Boundary Principles

### 3.1 Static Layered Architecture: Separate the Semantic Control Plane from Data and Capability Execution

The recommended static architecture has one core principle: the upper layer expresses intent in stable semantics, while dedicated lower-layer systems perform queries, computation, and actions.

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
│                    Ontology Service: Semantic Control Plane                  │
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

 Data Catalog, Lineage, IAM, DLP, quality, and monitoring systems are referenced
 or coordinated as external governance systems.
 Knowledge Graph is an optional facts and complex-relationship layer,
 not a mandatory central database.
```

The most important boundaries are:

- Ontology Service owns “what it is, how it is understood, how it is discovered, and whether it is trusted”;
- Plan Translation Engine adapts Semantic Plans into execution plans;
- Query / Search Service reads and analyzes data;
- Action / Workflow Service changes state and performs external side effects;
- Data Catalog, Lineage, IAM, and DLP retain their own sources of truth and governance responsibilities.

### 3.2 Responsibilities of Ontology Service

An Ontology Service should provide at least the following capabilities.

#### 3.2.1 Semantic Model Management

- Entity Type / Class;
- Property / Attribute;
- Relation;
- Semantic Type;
- Definition, Synonym, Scope;
- Semantic Constraint;
- Sensitivity classifications for objects and properties.

#### 3.2.2 Semantic Discovery

- Resolve concepts from natural-language terms or standard identifiers;
- Discover properties, relations, and related concepts;
- Return definitions, constraints, applicability, and versions;
- Filter invisible semantics based on the current Agent identity.

#### 3.2.3 Mapping and Data Representation Management

- Maintain semantic data representations;
- Reference Data Catalog assets;
- Record field transformations, filters, and derivation rules;
- Record SSOT, priorities, validity, freshness, and fallback policies;
- Associate Lineage and quality information.

#### 3.2.4 Capability Description and Discovery

- Describe Query Capabilities and Action Capabilities;
- Record inputs, outputs, preconditions, and side effects;
- Record permissions, Provider, SLA, idempotency, and transaction requirements;
- Return capabilities suitable for the current user, Agent, and task.

#### 3.2.5 Governance and Evolution

- Draft, Review, Validate, Approve, Publish;
- Version, Effective Time, Deprecated Time;
- Owner, Source, Provenance, Confidence;
- Impact analysis for Schema Diff, API Diff, business changes, and Agent feedback;
- Rollback, audit, and runtime version control.

### 3.3 What Ontology Service Explicitly Does Not Own

The following capabilities may cooperate with Ontology, but should not be assumed to belong inside Ontology Service:

| Capability that should not be absorbed | Responsible system |
|---|---|
| Physical data storage and full instance copies | Existing databases, data lakes, caches, or fact services |
| Generic SQL / DSL execution against arbitrary sources | Query Service, data engine, Adapter |
| Complex search and analytical computation | Search / OLAP / Analytics Service |
| Complete data catalog and asset lifecycle | Data Catalog |
| Complete lineage collection and propagation | Lineage System |
| PII masking, identity, and basic authorization | IAM, DLP, Policy Engine |
| Business state machines and transaction orchestration | Business Service, Workflow Engine |
| MCP protocol gateway itself | MCP Gateway or Agent Integration Layer |
| Knowledge Graph storage for arbitrary domains | Optional KG / Graph Service |

The principle is not to prohibit integration. It is to prohibit turning every responsibility into an internal implementation of Ontology Service and thereby creating a new platform coupling center.

### 3.4 Agent-Level Semantic Access Control

Traditional IAM usually answers: “Can this subject access a particular API or data table?” Enterprise Agents also need semantic-level control:

> **Given the current task, user, department, and purpose, can this Agent see a particular business concept, property, relation, or instance?**

Consider the same `Customer` concept:

- A customer-service Agent may see names, orders, and shipment status;
- A risk Agent may see risk labels and transaction relationships, but not the complete national ID;
- A marketing Agent may see masked contact information, but not sensitive risk relationships;
- An external partner Agent may see only an authorized customer subset and aggregated results.

Semantic access control should cover at least four levels:

```text
1. Concept Level        Can the Agent discover Customer / RiskSubject?
2. Property Level       Can it see phone / nationalId / riskLevel?
3. Relation Level       Can it see owns / related_to / flagged_by?
4. Instance / Row Level Can it see a particular customer or region's data?
```

A policy may use conditions such as:

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

Key principles:

1. **Permission filtering must happen before or together with semantic discovery.** Do not expose the complete Ontology first and then rely on Agent self-restraint.
2. **Both discover-deny and read-deny must be supported.** Some sensitive relationships should not reveal their names or structure to an unauthorized Agent even when they exist.
3. **Semantic authorization does not replace physical data authorization.** Ontology Service results must still be enforced by execution services, database row/column policies, and DLP.
4. **Action Capability uses a stronger default policy.** Write operations should include separation of duties, approval, preconditions, idempotency keys, and workflow state checks.
5. **Authorization results must be explainable and auditable.** An Agent or auditor should be able to understand why a property was masked or a relationship was hidden.

### 3.5 The Plan Translation Layer from Semantic Plan to Physical Query

It is not enough for Ontology Service to output business semantics and plan constraints. The architecture must explicitly answer: who translates a Semantic Plan into SQL, Cypher, GraphQL, search DSL, or a business API call?

This responsibility belongs to the **Plan Translation Engine / Adapter Layer**.

#### 3.5.1 Responsibility Boundary

```text
Agent / Planner
  Generate business intent
        ↓
Ontology Service
  Validate concepts, relationships, constraints, permissions, and capabilities
        ↓
Semantic Plan
  Remains independent of physical technology
        ↓
Plan Translation Engine
  Select Mapping, data source, capability, and adapter
        ↓
Physical Execution Plan
  SQL / Cypher / GraphQL / Search DSL / API request
        ↓
Execution Service
```

Ontology Service provides and validates:

- Whether concepts and relationships are valid;
- Whether the Semantic Plan satisfies semantic constraints;
- Whether the current Agent is authorized;
- Which Mappings, data sources, and Capabilities are available;
- Which provenance, freshness, and masking requirements the result must carry.

Translation Engine handles:

- Parsing the Semantic Plan;
- Selecting the appropriate Query Capability or Action Capability;
- Mapping semantic properties to physical fields, expressions, and joins;
- Selecting a SQL, Cypher, GraphQL, search DSL, or API Adapter;
- Injecting row filters, time conditions, masking, and rate limits;
- Generating an executable plan and returning explanation information;
- Handling multi-source queries, result merging, and error classification.

#### 3.5.2 Why Ontology Service Should Not Execute Every Query Directly

If Ontology Service maintains drivers for every database, a SQL optimizer, search DSLs, and API orchestration at the same time, it will quickly become a new Universal Query Engine:

- The semantic model becomes tightly coupled to database technology;
- Failures and upgrades in every physical system affect the semantic service;
- Query performance, connection pools, and resource isolation move into the control plane;
- Read/write transactions and workflow boundaries become ambiguous.

A more stable approach is for Ontology Service to publish semantic contracts and plans, while the Adapter Layer translates them for specific execution technologies.

### 3.6 Multi-Source Conflicts and Authority Policies

The same entity property may be mapped to multiple physical sources:

```text
Customer.phone
  ├── CRM MySQL: master data, visible within 5 minutes of update
  ├── ClickHouse: historical snapshot, one-hour delay
  └── Redis: real-time cache, may be briefly stale
```

These values may differ in format, freshness, and business state. The Mapping layer must explicitly declare multi-source coexistence rules. An Adapter must not simply choose whichever source returns fastest.

#### 3.6.1 Property-Level SSOT and Purpose-Specific Authority

Authority should be defined at the property or data-representation layer and may vary by purpose:

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

The following roles must be distinguished:

- **SSOT (Single Source of Truth):** the ultimately authoritative source from a business-ownership perspective;
- **Serving Source:** a cache, index, or derived source used for performance;
- **Historical Source:** a historical snapshot used for point-in-time analysis;
- **Fallback Source:** a degraded source used when the primary source is unavailable.

#### 3.6.2 Conflict Resolution and Degradation Policies

Policies should define at least:

1. Priority: which source takes precedence for which purpose;
2. Freshness: whether the result satisfies the requested time window;
3. Validity: during which period a value is valid;
4. Normalization: how phone numbers, currencies, time zones, and codes are standardized;
5. Conflict handling: whether to return the SSOT, return values side by side, mark a conflict, or escalate to a person;
6. Visibility: whether the Agent can see source differences and the reason for a conflict;
7. Degradation: whether a cache may be used when the primary source is unavailable and whether confidence must be reduced;
8. Audit: which value was selected, from where, and when it was read.

When sources disagree, the system must not silently merge values arbitrarily. It should return information such as:

```text
Customer.phone = +86-138****8888
selectedSource = crm_customer_master
freshness = 3m
conflictDetected = true
conflictSources = [customer_realtime_cache]
confidence = degraded
```

### 3.7 Relationship to Palantir Operational Ontology

Palantir's Ontology approach has clear similarities to the architecture discussed here: both emphasize raising underlying data into an abstraction closer to the business world through business objects, properties, relationships, and operational capabilities.

A heavier Operational Ontology often integrates the following more tightly:

```text
Objects
Properties
Links
Actions
Functions
Security
Workflow
```

The starting point recommended here is thinner:

```text
Semantic Model
  + Constraints
  + Mapping
  + Capability Contract
  + Governance
  + External Execution Services
```

These are not mutually exclusive choices. They represent different points on an evolution path:

- Start with a governable, discoverable semantic control plane decoupled from execution;
- Expand cautiously toward an Operational Ontology when the business genuinely needs unified object operations, state management, and workflow orchestration;
- Every expansion of scope must prove that it solves a real cross-system coordination problem rather than merely making the platform look complete.

### 3.8 Five Architecture Principles

#### Principle 1: Abstract the Real World Before Mapping Data

Do not infer the entire enterprise Ontology directly from `mysql.customer`. First establish the business definition of Customer, then find its representations across systems.

#### Principle 2: Decouple Ontology from Physical Data

```text
Ontology.Customer ≠ mysql.customer
```

There must be an explicit, auditable, evolvable Mapping between the two.

#### Principle 3: Ontology Describes Capabilities but Does Not Replace Execution

```text
Ontology
  ↓
Capability Contract / Semantic Plan
  ↓
Execution Service
  ↓
Actual Query or Action
```

#### Principle 4: Semantic Permissions Must Be Bound to Runtime Identity

What an Agent can discover and read depends on the user, role, department, purpose, data sensitivity, and instance scope—not only on the static Ontology.

#### Principle 5: Establish a Thin, Clear Control Plane Before Expanding It

The first phase should prioritize:

```text
Business Meaning
+ Constraints
+ Mapping
+ Capability Discovery
+ Semantic Access Control
+ Governance
```

Do not put databases, query engines, graph databases, workflows, Catalog, Lineage, and MCP Gateway into Ontology Service from the beginning.

---

## IV. Construction Method and Continuous Evolution

### 4.1 The Goal Is Not “Write a Few Entity JSON Files”

The real Ontology construction process is:

```text
Enterprise business reality
    ↓
Business knowledge and data asset discovery
    ↓
Business concept identification
    ↓
Ontology modeling
    ↓
Semantic constraints and access policies
    ↓
Semantic Data Representation / Mapping
    ↓
Query / Action Capability binding
    ↓
Translation and execution validation
    ↓
Business expert approval
    ↓
Versioned Runtime
```

The scope should begin with a clear Domain and Agent scenario rather than attempting to establish an “Entire Enterprise Ontology” in one step.

### 4.2 Eight-Step Initialization Method

#### Step 1: Define the Domain, Goal, and Boundary

Specify:

- Business domain;
- User scenario;
- Target Agent / Application;
- Tasks to support;
- Data scope to connect;
- Query / Action Capabilities to expose;
- Security and compliance constraints;
- Success metrics.

For example, the first phase might focus on “abnormal customer transaction queries” rather than “the entire enterprise customer Ontology.”

#### Step 2: Inventory Existing Knowledge and Assets

Input sources include:

```text
Business Glossary
Data Dictionary
Data Catalog / Metadata
Data Lineage
Database Schema
API / GraphQL Definitions
Existing KG (if any)
Business Rules
Service Documentation
SQL / ETL / Pipeline
Domain Expert Knowledge
Historical Agent Queries
```

These are evidence sources for the Ontology, but they are not the Ontology itself.

#### Step 3: Identify Business Concepts Instead of Copying the Database

First ask:

```text
What objects, properties, and relationships actually exist in this business world?
```

Then ask:

```text
How are these concepts represented in enterprise data?
```

If `client_no`, `cust_id`, and `party_id` all represent the same customer identifier in business terms, they should be unified under `Customer.customerId`, while their sources, transformations, and confidence are retained.

#### Step 4: Define the Four Elements and Semantic Constraints

For each core concept, define:

- Class / Concept;
- Property / Attribute;
- Relation;
- Instance identification strategy;
- Definition, Synonym, Semantic Type;
- Owner, Scope, Provenance;
- Type, cardinality, relationship, and invariant constraints;
- Sensitivity classification and default access policy.

Example:

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

#### Step 5: Establish Semantic Data Representation and Mapping

Mapping should record:

- Physical asset identifiers;
- Fields and expressions;
- Type, unit, and code conversions;
- Join and filtering conditions;
- Validity periods;
- SSOT, cache, snapshot, and fallback roles;
- Data freshness and quality thresholds;
- Lineage references;
- Masking and semantic access-control requirements.

#### Step 6: Bind Query / Action Capabilities

Define each capability as follows:

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

An Action Capability should additionally define:

```yaml
kind: action
sideEffects: [AccountStateChanged]
idempotencyKey: required
transactionBoundary: AccountService
approval: required_for_high_risk
workflow: account-freeze-workflow
```

#### Step 7: Validate the Semantic Plan, Translation, and Execution Result

It is not enough to validate that the Ontology JSON is syntactically correct. The full chain must be tested:

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

Validation should cover:

- Whether the Semantic Plan violates constraints;
- Whether permission filtering is effective;
- Whether physical fields and relationships are mapped correctly;
- Whether authority and fallback policies behave as expected;
- Whether query results include provenance;
- Whether Actions satisfy transaction, idempotency, and approval requirements;
- Whether failures return diagnosable feedback.

#### Step 8: Obtain Business Approval and Publish a Trusted Version

AI can generate candidate concepts, relationships, Mappings, and definitions. Business experts must confirm:

- Whether two fields are genuinely synonymous;
- Whether two entities really represent the same business object;
- Whether a relationship is a business relationship;
- Whether a source can serve as the SSOT;
- Whether a property is sensitive;
- Whether an Action may be called by an Agent.

> **AI generates candidate semantics; business experts confirm the final semantics.**

### 4.3 Ontology Studio: A Governance Entry Point for People

If Ontology is long-lived enterprise infrastructure, it cannot expose only Runtime APIs. An Ontology Studio should cover:

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

The standard lifecycle is:

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

The value of Studio is not merely a more polished editor. It turns “changing semantics” into an engineering process with owners, evidence, impact scope, and approval records.

### 4.4 Version, Provenance, and Approval

An Ontology must not be treated as an ordinary configuration file. Every Entity, Property, Relation, Mapping, and Capability should retain at least:

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

Example:

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

Runtime should support:

- Reading by version;
- Gradual rollout;
- Auditing who approved what and when;
- Returning the semantic version in execution results;
- Rolling back to the previous trusted version when errors occur.

### 4.5 Physical Schema Evolution Is Not Ontology Evolution

Renaming a physical field does not necessarily mean that business semantics changed.

```text
Old Mapping:
Customer.phone → mysql.customer.phone

New Mapping:
Customer.phone → mysql.customer.mobile
```

If the business meaning remains “the customer's primary contact number,” this is only Mapping Evolution.

If the business definition changes from “customer contact phone” to “primary contact person's phone,” however, the applicability, property definition, or constraints may have changed. That may be Ontology Evolution.

At minimum, distinguish:

```text
Physical Schema Evolution
        ↓
Mapping Evolution
        ↓
Ontology Evolution
        ↓
Business Evolution
```

Changes at these levels have different impact scopes, approval levels, and rollback strategies.

### 4.6 Downstream Evolution: Deriving Semantic Impact from Data and System Changes

The following inputs should be continuously monitored:

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

For example:

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

The Change Engine can identify what may be affected, but it cannot determine business semantics based only on a field rename. It must combine Catalog, Lineage, documents, usage records, and business-expert feedback.

### 4.7 Upstream Evolution: Semantic Completion Driven by Agent Runtime Feedback

An evolution loop that relies only on Schema Diff is incomplete. Real Agent usage also reveals semantic gaps:

- The user's concept cannot be resolved;
- Multiple definitions are found but cannot be disambiguated;
- The Semantic Plan violates an unmodeled business constraint;
- No suitable Mapping or Capability exists;
- Query results cannot be explained because sources conflict;
- An Action is rejected by permissions, approval, or preconditions;
- The Agent repeatedly uses synonyms or bypasses standard semantics to access physical tables.

This feedback should enter a Feedback Engine:

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

Feedback must not be written directly into the production Ontology. It should become a candidate proposal containing:

- The original user intent;
- Concepts and plans discovered by the Agent;
- The failure location;
- The specific error category;
- The relevant user, purpose, and permission context;
- The proposed change and affected assets;
- Whether a new business definition or Capability is required.

### 4.8 The Boundary of AI in Construction and Maintenance

AI is well suited to:

- Discover candidate concepts from documents, Schema, SQL, APIs, and query logs;
- Cluster synonyms;
- Infer candidate properties and relationships;
- Generate candidate Mappings;
- Identify potential conflicts and impact scope;
- Generate completion proposals from Agent failures;
- Automatically generate test cases and change descriptions.

AI should not decide without approval:

- Whether concepts from different departments are equivalent;
- Which data source is the SSOT;
- Whether a sensitive relationship is visible to a class of Agents;
- Whether an Action may perform a real-world side effect;
- Whether a business constraint may be removed or relaxed.

The goal is not to eliminate human maintenance of Ontology. The goal is to focus human time on semantic decisions that genuinely require business judgment.

### 4.9 The Bidirectional Dynamic Evolution Loop

This is the second core architecture diagram in the document:

```text
                 ┌──────────────────────────────┐
                 │      Business / Data World    │
                 │                              │
                 │  Business Change              │
                 │  Schema / API / Lineage Diff  │
                 └──────────────┬───────────────┘
                                │ downstream change
                                ▼
                         ┌──────────────┐
                         │ Change Engine │
                         └──────┬───────┘
                                │
                                │
┌───────────────────┐           ▼           ┌────────────────────────┐
│ Agent Runtime      │──upstream feedback──>│ Feedback / Impact Analysis │
│ ambiguity          │                      └──────────┬─────────────┘
│ query failure      │                                  │
│ missing mapping    │                                  ▼
│ policy denial      │                           ┌──────────────┐
│ source conflict    │                           │ AI Proposal   │
└───────────────────┘                           └──────┬───────┘
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

 Downstream loop: physical/business change → detection → impact analysis → proposal → approval → publish
 Upstream loop: Agent feedback → classification → impact analysis → proposal → approval → publish
```

The meaning of the bidirectional loop is:

> **Ontology must not only keep up with changes in underlying systems; it must also discover, from real Agent usage, the parts of business semantics that have not yet been modeled.**

---

## V. Agent Interaction and Semantic Execution

### 5.1 An Agent Should Not Receive the Entire Ontology at Once

A large enterprise may have thousands of Entity Types, vast numbers of properties, and complex relationships. Injecting the entire Ontology into context at once causes:

- Wasted context;
- Lower relevance;
- Excessive exposure of sensitive concepts;
- More frequent selection of the wrong similar concept.

Use progressive Semantic Discovery instead:

```text
Natural-language task
    ↓
resolveConcept("customer")
    ↓
Customer
    ↓
Discover related properties and relationships
    ↓
Account / Order / Device
    ↓
Continue discovery around the task
    ↓
Transaction + constraints + available capabilities
```

Runtime APIs can be divided into three groups.

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

`context` should contain at least the user, Agent identity, department, purpose, region, semantic version, and security policy.

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

Results should not contain only names. They should include permissions, preconditions, inputs, outputs, data freshness, side effects, transaction boundaries, idempotency requirements, and Provider.

### 5.2 The Relationship Between Ontology, Skills, and MCP

The three solve different problems:

```text
Ontology
  What is the world? How are concepts related?

Skill
  What steps and strategy should be used for a class of tasks?

MCP Tool
  Through what standard interface is a capability called?
```

For example:

```text
customer_risk_analysis Skill
        ↓ task flow
Ontology Service
        ↓ discover Customer / Account / Transaction
Query Capability / Action Capability
        ↓ MCP or another interface
Enterprise Services
```

Therefore:

> **MCP / Skills are mechanisms through which an Agent uses capabilities; Ontology is the semantic model an Agent relies on to understand enterprise capabilities and data correctly.**

Ontology may be exposed through MCP, but Ontology itself is not an MCP Gateway. It should not absorb every business tool into the Ontology API.

### 5.3 Semantic Plan: A Contract Between Semantic Understanding and Physical Execution

After understanding the user's task, an Agent should first generate a Semantic Plan independent of physical technology.

User request:

> Check whether Zhang San has had any abnormal transactions in the last seven days.

The Semantic Plan may be expressed as:

```json
{
  "intent": "find_abnormal_transactions",
  "subject": {
    "concept": "Customer",
    "identifier": "Zhang San"
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

It expresses:

```text
Business objects
+ relationship path
+ semantic constraints
+ task intent
+ output requirements
```

It does not directly express:

```text
SQL
ES DSL
Cypher
Redis Command
Private parameters of an internal API
```

The value of a Semantic Plan is that the Agent performs explainable semantic reasoning first; Ontology Service validates it; and the Translation Engine adapts it to physical execution.

### 5.4 Semantic Plan Validation

Before execution, a Semantic Plan should pass at least these checks:

1. **Concept validation:** Do the entities, properties, and relationships exist?
2. **Semantic validation:** Is the relationship path valid under business constraints?
3. **Permission validation:** Can the current Agent discover and read these semantics?
4. **Data validation:** Does a Mapping exist that satisfies the purpose and freshness requirements?
5. **Capability validation:** Is there a matching Query or Action Capability?
6. **Risk validation:** Does the plan involve PII, sensitive relationships, or high-risk actions?
7. **Plan validation:** Do output, filters, pagination, and cost comply with policy?
8. **Version validation:** Is the Ontology version used by the plan still executable?

If validation fails, the system should return structured feedback rather than allowing the Agent to guess a table or bypass Ontology.

### 5.5 Execution Path Through Plan Translation Engine

The complete execution path is:

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

The Translation Engine may have multiple Adapters:

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

Adapter output should preserve an explainable mapping from semantic properties to physical fields, relationship paths to joins / edges, and data policies to execution filters.

### 5.6 Agent Interaction Differences Between Query and Action Capabilities

#### 5.6.1 Query Scenario

A query may execute automatically when permissions allow it, cost is controlled, and freshness requirements are satisfied. The result must include:

```text
ontologyVersion
source
retrievedAt
freshness
quality
maskingApplied
conflictStatus
```

An Agent must not automatically interpret “a value was returned” as “this value is the business truth.”

#### 5.6.2 Action Scenario

An Action Capability has external side effects. A recommended flow is:

```text
Discover capability
  ↓
Check permissions and preconditions
  ↓
Generate action preview
  ↓
Display affected objects and parameters
  ↓
User confirmation / approval
  ↓
Workflow Engine execution
  ↓
Return status, idempotency key, and audit information
```

For high-risk actions, human confirmation or dedicated approval may be required even when the Agent has invocation permission. Ontology Service describes conditions and risk labels; it must not bypass the Workflow Engine.

### 5.7 Classification of Agent Runtime Feedback

Feedback should be structured rather than recorded as an undifferentiated error string:

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

Different feedback implies different remediation:

| Feedback | Possible semantic improvement |
|---|---|
| ConceptNotFound | Add a concept, synonym, or domain alias |
| AmbiguousConcept | Add definitions, scope, or disambiguating properties |
| MissingRelation | Model the relationship or clarify the cross-domain boundary |
| NoMapping | Establish a data representation and physical Mapping |
| SourceConflict | Update SSOT, priorities, or conflict policy |
| PermissionDenied | Review semantic policy; do not simply loosen permission |
| TranslationFailure | Fix Mapping, Adapter, or Capability Contract |
| ConstraintViolation | Add a constraint or correct the Agent plan |
| ActionApprovalRequired | Enter approval and workflow rather than retrying to bypass it |

### 5.8 Complete Example: Querying Abnormal Customer Transactions

User request:

> Check whether Zhang San has had any abnormal transactions in the last seven days.

A reasonable Agent flow is:

1. `resolveConcept("Zhang San")`, returning `Customer` candidates filtered by current permissions;
2. Confirm the identity-matching capability and disambiguation rules for `Customer`;
3. Discover `Customer owns Account` and `Account hasTransaction Transaction`;
4. Retrieve the business definition, risk labels, and scope of “abnormal transaction”;
5. Retrieve the Mapping, SSOT, and freshness of transaction data for the last seven days;
6. Generate a Semantic Plan;
7. Validate semantic constraints, permissions, and capabilities;
8. Let the Translation Engine select the Transaction Query Service;
9. Execute and return results with source, timestamp, quality, and masking information;
10. If ambiguity, source conflict, or permission denial occurs, record structured feedback for future evolution.

The Agent does not need to know whether the underlying system is ClickHouse, MySQL, GraphQL, or a proprietary risk API. It should not construct SQL that has not passed semantic validation.

---

## VI. Enterprise Adoption Checklist and Summary

### 6.1 Domain and Goal

```text
□ Is the Domain scope explicit?
□ Is the first-phase scenario sufficiently focused?
□ Are the target users, Agents, and applications clear?
□ Are the success metrics verifiable?
□ Is it clear which concepts are outside the current scope?
```

### 6.2 Semantic Model

```text
□ Is each Class / Concept defined?
□ Is each Property / Attribute defined?
□ Are Relation direction and cardinality defined?
□ Is the Instance identification strategy clear?
□ Are Definitions clear?
□ Are Semantic Types explicit?
□ Are synonyms and aliases handled?
□ Are applicability and domain ownership clear?
□ Is business semantics distinguished from physical Schema?
```

### 6.3 Semantic Constraints

```text
□ Are type constraints defined?
□ Are uniqueness and cardinality constraints defined?
□ Is relationship validity defined?
□ Are basic business invariants defined?
□ Are state and temporal constraints defined?
□ Does complex business logic remain in rules, services, or workflows?
```

### 6.4 Data Representation and Mapping

```text
□ Is there an independent Semantic Data Representation?
□ Does Mapping reference rather than copy the Data Catalog?
□ Are field, type, unit, and code conversions recorded?
□ Are joins, derivations, and filters recorded?
□ Are SSOT, cache, snapshot, and fallback roles defined?
□ Are data freshness and validity periods defined?
□ Are Lineage and quality information linked?
□ Can multi-source conflicts be detected, explained, and audited?
□ Are data masking and semantic authorization requirements defined?
```

### 6.5 Knowledge Graph Decisions

```text
□ Are multi-hop relationships and graph reasoning genuinely needed?
□ Is reusable graph-data capability already available?
□ Is KG positioned as an optional facts/relations layer?
□ Are all instances kept out of a forced Ontology copy?
□ Are relational, search, and API federation already sufficient?
```

### 6.6 Capability and Execution

```text
□ Are Query Capabilities defined?
□ Are Action Capabilities defined?
□ Are read and write permission models clearly separated?
□ Do Queries carry freshness, quality, and provenance?
□ Do Actions have preconditions, transaction boundaries, and idempotency keys?
□ Are high-risk Actions connected to a Workflow Engine?
□ Are Capability Providers and SLAs recorded?
□ Is the Translation Engine from Semantic Plan to SQL/Cypher/GraphQL/API explicit?
□ Does Ontology Service avoid absorbing all execution logic directly?
```

### 6.7 Semantic Access Control

```text
□ Are concepts filtered by Agent / user / department / purpose?
□ Are property-level, relation-level, and instance-level permissions supported?
□ Do PII policies support masking, redaction, and undiscoverability?
□ Are sensitive relationships hidden from unauthorized Agents?
□ Are semantic permissions integrated with IAM, DLP, and physical data permissions?
□ Do Actions have stricter approval and separation-of-duties controls?
□ Are permission denials explainable and auditable?
```

### 6.8 Runtime and Agent

```text
□ Is progressive Semantic Discovery supported?
□ Are semantic results returned according to runtime identity?
□ Is Semantic Plan supported?
□ Are constraints and permissions validated before execution?
□ Are source, freshness, quality, and semantic version returned?
□ Are Query and Action interaction flows distinct?
□ Are the boundaries between MCP / Skills and Ontology clear?
□ Is the Agent prevented from having to guess physical table structures directly?
```

### 6.9 Governance and Evolution

```text
□ Are Owner, Source, Provenance, and Confidence recorded?
□ Are Version, Effective Time, and Deprecated Time recorded?
□ Is there a Draft / Review / Validate / Approve / Publish lifecycle?
□ Are Schema / API / Lineage / Catalog changes detected?
□ Is semantic impact analysis supported?
□ Are rollback and audit supported?
□ Is there a feedback loop for Agent failures, ambiguity, and semantic gaps?
□ Must AI proposals be confirmed by business experts?
□ Are Schema Evolution, Mapping Evolution, and Ontology Evolution distinguished?
```

### 6.10 Final Test: Is Ontology Service Becoming a “Super Ontology Service”?

Use these questions as a reverse check:

```text
□ Has Ontology Service started storing copies of all business instances?
□ Is it rebuilding the Data Catalog?
□ Is it rebuilding a complete Lineage Engine?
□ Are all database queries being placed in the control plane?
□ Are MCP Gateway, Workflow, and Action Runtime all being built in?
□ Is every piece of data required to enter a Knowledge Graph?
□ Does the semantic model depend on one particular database's table structure?
□ Can Agents bypass the semantic layer and access physical data directly?
```

If most answers are “yes,” Ontology Service is likely drifting from a semantic control plane toward another super data platform.

### 6.11 Final Mental Model

The entire system can be compressed into the following statements:

> **Ontology defines what the enterprise business world is.**
>
> **Classes, Properties, Relations, and Instances are the four basic entry points for understanding Ontology. Ontology primarily defines structure and semantics; instance data may be provided by external systems.**
>
> **Schema describes how a particular system stores data; Knowledge Graph stores optional instances and complex relationship facts; Data Catalog describes data assets; Lineage describes how data is produced and propagated.**
>
> **Mapping connects business concepts to enterprise data; Query Capability describes what can be read and analyzed; Action Capability describes what can be changed safely.**
>
> **Semantic Plan expresses business intent, relationship paths, and semantic constraints; Plan Translation Engine translates it into SQL, Cypher, GraphQL, search DSL, or business API calls.**
>
> **Semantic Access Control ensures that each Agent can discover and use only the concepts, properties, relationships, and instances allowed by its identity, purpose, and scope.**
>
> **Ontology Service is a thin Semantic Control Plane: it lets Agents understand the enterprise world first, discover data and capabilities next, and finally rely on dedicated execution services to perform queries or actions.**
>
> **Ontology evolves both from business, Schema, API, and data changes and from Agent ambiguity, failure, conflict, and semantic-gap feedback. Both paths must pass through impact analysis, AI proposal, human approval, and version publication.**

The architecture can ultimately be summarized in one sentence:

> **Ontology gives enterprise data and capabilities a unified business semantics; Ontology Service delivers that semantics to AI Agents securely, discoverably, governably, and evolvably—without turning the semantic control plane into another data plane.**
