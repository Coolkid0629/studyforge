# Architecture Decision Record (ADR)

This document explains key design decisions for StudyForge.

---

## ADR-001: Use Layered Architecture

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need to design a system that is maintainable, testable, and allows for future expansion (web interface, mobile app, etc.).

**Decision:**
Use 4-layer architecture:
1. Presentation Layer
2. Service Layer
3. Data Access Layer (DAO)
4. Database Layer

**Consequences:**
- ✅ **Positive:**
  - Clear separation of concerns
  - Easy to test each layer independently
  - Can change UI without touching business logic
  - Standard enterprise pattern (good for portfolio)
  
- ⚠️ **Negative:**
  - More files and classes than a monolithic design
  - Slightly more complex for simple operations
  - Need to pass data through multiple layers

**Alternatives Considered:**
- Monolithic (all code in one class) - rejected due to poor maintainability
- MVC pattern - rejected because we're not building a traditional web app yet

---

## ADR-002: Use PostgreSQL as Database

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need a relational database to store users, materials, quiz items, and spaced repetition state. Must handle complex queries for analytics.

**Decision:**
Use PostgreSQL 15+

**Consequences:**
- ✅ **Positive:**
  - Open source and free
  - Excellent for complex queries (analytics)
  - JSON support (future feature: store quiz settings)
  - Strong community and documentation
  - Good for research projects (can export data easily)
  
- ⚠️ **Negative:**
  - Requires installation (slightly harder than SQLite)
  - Overkill for very small datasets
  - Need to learn PostgreSQL-specific features

**Alternatives Considered:**
- **MySQL** - similar but PostgreSQL has better JSON support
- **SQLite** - too simple, doesn't handle concurrent users well
- **MongoDB** - NoSQL not needed, relational data fits better

---

## ADR-003: Use DAO Pattern for Database Access

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need a clean way to separate database operations from business logic.

**Decision:**
Create separate DAO classes for each entity (UserDAO, MaterialDAO, etc.)

**Consequences:**
- ✅ **Positive:**
  - All SQL in one place per entity
  - Easy to mock for testing
  - Can swap databases by only changing DAOs
  - Follows standard Java enterprise pattern
  
- ⚠️ **Negative:**
  - More boilerplate code
  - Need to maintain multiple DAO classes

**Alternatives Considered:**
- **Direct SQL in services** - rejected due to code duplication and poor testability
- **ORM (Hibernate)** - rejected to keep project simpler and learn SQL

---

## ADR-004: Separate Algorithm Components from Services

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need to implement multiple spaced repetition algorithms (SM-2, Leitner) for research comparison.

**Decision:**
Create separate algorithm classes (SM2Algorithm, LeitnerSystem) implementing a common interface (SpacedRepetitionAlgorithm).

**Consequences:**
- ✅ **Positive:**
  - Can easily compare algorithms
  - Can add new algorithms without changing services
  - Algorithms are reusable and testable independently
  - Clean separation: algorithms do math, services coordinate
  
- ⚠️ **Negative:**
  - Slightly more complex than putting algorithm logic directly in service

**Alternatives Considered:**
- **Algorithm logic in AdaptiveLearningService** - rejected due to poor testability and can't easily compare algorithms

---

## ADR-005: Use BCrypt for Password Hashing

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need to store passwords securely. Never store plain text passwords.

**Decision:**
Use BCrypt with 12 salt rounds for password hashing.

**Consequences:**
- ✅ **Positive:**
  - Industry standard for password security
  - Built-in salting (prevents rainbow table attacks)
  - Adaptive (can increase rounds as computers get faster)
  - Easy to use library available
  
- ⚠️ **Negative:**
  - Slightly slower than simpler hashing (this is intentional and good)

**Alternatives Considered:**
- **Plain SHA-256** - rejected, not secure enough (no salt, too fast)
- **Argon2** - better than BCrypt but less common in Java, BCrypt is "good enough"

---

## ADR-006: Use Maven for Build Management

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need to manage dependencies (PostgreSQL driver, OpenNLP, PDFBox, etc.) and build process.

**Decision:**
Use Maven as build tool.

**Consequences:**
- ✅ **Positive:**
  - Standard Java build tool
  - Easy dependency management via pom.xml
  - Integration with IDEs (IntelliJ, Eclipse)
  - Can easily add new libraries
  
- ⚠️ **Negative:**
  - pom.xml can be verbose
  - Slightly slower than Gradle

**Alternatives Considered:**
- **Gradle** - newer, faster, but less common in educational projects
- **Manual JAR management** - too error-prone

---

## ADR-007: Use HikariCP for Connection Pooling

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need efficient database connection management. Opening a new connection for each query is slow.

**Decision:**
Use HikariCP connection pool with max 10 connections.

**Consequences:**
- ✅ **Positive:**
  - Fastest connection pool available
  - Reuses connections (much faster)
  - Simple configuration
  - Industry standard
  
- ⚠️ **Negative:**
  - Adds one more dependency

**Alternatives Considered:**
- **No pooling** - rejected, too slow
- **Apache DBCP** - older, slower than HikariCP

---

## ADR-008: Start with CLI, Add GUI Later (Optional)

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need user interface. GUI is time-consuming and not core to research goals.

**Decision:**
Implement command-line interface (CLI) first. GUI (JavaFX or web) is optional in Phase 6.

**Consequences:**
- ✅ **Positive:**
  - Faster development (focus on algorithms and logic)
  - Easier to test and debug
  - Can always add GUI later (layered architecture makes this easy)
  - Research doesn't require fancy UI
  
- ⚠️ **Negative:**
  - Less impressive visually for demos (can be mitigated with video)
  - Not as user-friendly

**Alternatives Considered:**
- **JavaFX GUI first** - rejected, too time-consuming
- **Web app (Spring Boot)** - rejected, overkill for Phase 1-5

---

## ADR-009: Use OpenNLP for NLP Tasks

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
Need to generate questions from text and extract topics. Could use cloud APIs (OpenAI) or local libraries.

**Decision:**
Use Apache OpenNLP for NLP processing.

**Consequences:**
- ✅ **Positive:**
  - Runs locally (no API costs, no internet required)
  - Privacy-friendly (user data stays local)
  - Good for educational projects (learn NLP fundamentals)
  - Free and open source
  
- ⚠️ **Negative:**
  - Less powerful than GPT-4 or Claude
  - Need to download models (~50MB)
  - Question quality won't be as high as AI APIs

**Alternatives Considered:**
- **OpenAI API** - rejected due to costs and need for API key
- **Cloud NLP APIs (Google, AWS)** - rejected, want local processing

---

## ADR-010: Track Individual Responses (item_responses table)

**Date:** 2025-01-01

**Status:** Accepted

**Context:**
For spaced repetition and weak topic detection, need detailed history of every answer.

**Decision:**
Store every quiz response in item_responses table with timestamp, correctness, and response time.

**Consequences:**
- ✅ **Positive:**
  - Can analyze patterns over time
  - Spaced repetition algorithms need this data
  - Can detect weak topics accurately
  - Research requires detailed data
  
- ⚠️ **Negative:**
  - Table grows large over time (but this is manageable)
  - More database writes

**Alternatives Considered:**
- **Only store aggregated stats** - rejected, lose valuable data for algorithms

---
```

## **🚀 Full Project Structure (After Issue #3 - Week 1)**

After Issue #3 (Set Up Project Structure), your project will look like this:

studyforge/
├── .git/
├── .gitignore
├── README.md
├── pom.xml                         ← Maven configuration
├── docs/
│   ├── architecture.png
│   ├── architecture-explanation.md
│   └── architecture-decisions.md
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/
    │   │       └── studyforge/
    │   │           ├── Main.java   ← Entry point (empty for now)
    │   │           ├── model/      ← Created in Issue #4
    │   │           ├── dao/        ← Created in Issue #6+
    │   │           ├── service/    ← Created in Issue #7+
    │   │           ├── algorithm/  ← Created in Issue #13+
    │   │           ├── nlp/        ← Created in Issue #22+
    │   │           ├── analytics/  ← Created in Issue #27+
    │   │           ├── util/       ← Helper classes
    │   │           └── config/     ← Configuration classes
    │   └── resources/
    │       ├── database.properties      ← Database config (in .gitignore)
    │       ├── database.properties.example  ← Template
    │       ├── schema.sql               ← Created in Issue #2
    │       └── nlp-models/              ← OpenNLP models (Issue #21)
    └── test/
        └── java/
            └── com/
                └── studyforge/
                    ├── algorithm/
                    ├── service/
                    └── dao/
```


---

## **📋 Full Project Structure (After Issue #48 - Project Complete)**
```

studyforge/
├── .git/
├── .gitignore
├── README.md
├── pom.xml
├── LICENSE
├── PROGRESS.md                     ← Weekly progress logs
│
├── docs/
│   ├── architecture.png
│   ├── architecture-explanation.md
│   ├── architecture-decisions.md
│   ├── database-schema.png         ← ERD diagram
│   ├── algorithm-explanation.md    ← How SM-2 works
│   ├── research-methodology.md     ← Experiment design
│   ├── research-report.pdf         ← Final research paper
│   └── presentation.pptx           ← Slides
│
├── experiments/
│   ├── data/
│   │   ├── sm2-results.csv
│   │   └── leitner-results.csv
│   ├── results/
│   │   ├── learning-curves.png
│   │   └── comparison-chart.png
│   └── analysis/
│       └── statistical-analysis.py
│
├── src/
│   ├── main/
│   │   ├── java/com/studyforge/
│   │   │   ├── Main.java
│   │   │   │
│   │   │   ├── model/
│   │   │   │   ├── User.java
│   │   │   │   ├── Material.java
│   │   │   │   ├── QuizItem.java
│   │   │   │   ├── StudySession.java
│   │   │   │   ├── SpacedRepetitionState.java
│   │   │   │   ├── ItemResponse.java
│   │   │   │   ├── PerformanceMetric.java
│   │   │   │   └── WeakTopic.java
│   │   │   │
│   │   │   ├── dao/
│   │   │   │   ├── UserDAO.java
│   │   │   │   ├── MaterialDAO.java
│   │   │   │   ├── QuizItemDAO.java
│   │   │   │   ├── SessionDAO.java
│   │   │   │   ├── SpacedRepetitionDAO.java
│   │   │   │   ├── PerformanceDAO.java
│   │   │   │   └── WeakTopicDAO.java
│   │   │   │
│   │   │   ├── service/
│   │   │   │   ├── UserService.java
│   │   │   │   ├── MaterialService.java
│   │   │   │   ├── QuizService.java
│   │   │   │   ├── AdaptiveLearningService.java
│   │   │   │   ├── AnalyticsService.java
│   │   │   │   └── NLPService.java
│   │   │   │
│   │   │   ├── algorithm/
│   │   │   │   ├── SpacedRepetitionAlgorithm.java (interface)
│   │   │   │   ├── SM2Algorithm.java
│   │   │   │   ├── LeitnerSystem.java
│   │   │   │   ├── ForgettingCurveModel.java
│   │   │   │   ├── DifficultyAdjuster.java
│   │   │   │   └── WeakTopicDetector.java
│   │   │   │
│   │   │   ├── nlp/
│   │   │   │   ├── TextSummarizer.java
│   │   │   │   ├── QuestionGenerator.java
│   │   │   │   ├── TopicExtractor.java
│   │   │   │   └── AnswerAnalyzer.java
│   │   │   │
│   │   │   ├── analytics/
│   │   │   │   ├── PerformanceAnalyzer.java
│   │   │   │   ├── LearningCurveGenerator.java
│   │   │   │   ├── PredictiveModel.java
│   │   │   │   └── DataExporter.java
│   │   │   │
│   │   │   ├── util/
│   │   │   │   ├── PasswordHasher.java
│   │   │   │   ├── PDFExtractor.java
│   │   │   │   ├── DateUtils.java
│   │   │   │   └── ValidationUtils.java
│   │   │   │
│   │   │   └── config/
│   │   │       ├── DatabaseConfig.java
│   │   │       ├── NLPConfig.java
│   │   │       └── AppConfig.java
│   │   │
│   │   └── resources/
│   │       ├── database.properties (gitignored)
│   │       ├── database.properties.example
│   │       ├── schema.sql
│   │       ├── log4j2.xml
│   │       └── nlp-models/
│   │           ├── en-sent.bin
│   │           ├── en-token.bin
│   │           ├── en-pos-maxent.bin
│   │           └── en-ner-person.bin
│   │
│   └── test/
│       └── java/com/studyforge/
│           ├── algorithm/
│           │   ├── SM2AlgorithmTest.java
│           │   ├── LeitnerSystemTest.java
│           │   └── ForgettingCurveModelTest.java
│           ├── service/
│           │   ├── UserServiceTest.java
│           │   ├── AdaptiveLearningServiceTest.java
│           │   └── QuizServiceTest.java
│           ├── dao/
│           │   ├── UserDAOTest.java
│           │   └── MaterialDAOTest.java
│           └── integration/
│               └── EndToEndTest.java
│
└── target/                         ← Maven build output (gitignored)
    ├── classes/
    ├── test-classes/
    └── studyforge-0.1.0-SNAPSHOT.jar
```


---
```

## **📊 When Each Folder/File is Created**

| Component | Created In | Issue # |
|-----------|------------|---------|
| `docs/` folder | Issue #1 | #1 |
| `architecture.png` | Issue #1 | #1 |
| `architecture-explanation.md` | Issue #1 | #1 |
| `architecture-decisions.md` | Issue #1 | #1 |
| `schema.sql` | Issue #2 | #2 |
| `pom.xml` | Issue #3 | #3 |
| Package folders | Issue #3 | #3 |
| `database.properties` | Issue #3 | #3 |
| `.gitignore` | Issue #3 | #3 |
| `model/` classes | Issue #4 | #4 |
| `DatabaseConfig.java` | Issue #5 | #5 |
| `UserDAO.java` | Issue #6 | #6 |
| `UserService.java` | Issue #7 | #7 |
| `MaterialDAO.java` | Issue #8 | #8 |
| `MaterialService.java` | Issue #9 | #9 |
| ... and so on | Issues #10-48 | Various |
```

---

## **🎯 Summary: Project Structure Timeline**

### **After Issue #1 (Now):**
```
studyforge/
├── README.md
└── docs/
    ├── architecture.png
    ├── architecture-explanation.md
    └── architecture-decisions.md
```

### **After Issue #2 (Day 3-4):**
```
studyforge/
├── README.md
├── docs/ (same as above)
└── src/
    └── main/
        └── resources/
            └── schema.sql
```

### **After Issue #3 (Day 5-7):**
```
studyforge/
├── .gitignore
├── README.md
├── pom.xml
├── docs/ (same)
└── src/
    ├── main/
    │   ├── java/com/studyforge/
    │   │   ├── Main.java (empty)
    │   │   ├── model/
    │   │   ├── dao/
    │   │   ├── service/
    │   │   ├── algorithm/
    │   │   ├── nlp/
    │   │   ├── analytics/
    │   │   ├── util/
    │   │   └── config/
    │   └── resources/
    │       ├── database.properties
    │       ├── database.properties.example
    │       └── schema.sql
    └── test/
        └── java/com/studyforge/
