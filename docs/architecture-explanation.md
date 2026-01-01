# Architecture Explanation

## Overview
StudyForge uses a 4-layer architecture to separate concerns and enable maintainability.

## Layers

### 1. Presentation Layer
**Purpose:** User interface and interaction

**Components:**
- Command Line Interface (CLI) - Phase 1 implementation
- (Optional) JavaFX GUI - Phase 6
- (Optional) Web REST API - Phase 6

**Responsibilities:**
- Display information to user
- Accept user input
- Call service layer methods
- NO business logic here

---

### 2. Service Layer
**Purpose:** Business logic and coordination

**Components:**
- **UserService** - Authentication, account management
- **MaterialService** - Study material management, file uploads
- **QuizService** - Quiz generation and management
- **AdaptiveLearningService** - Spaced repetition, difficulty adjustment
- **AnalyticsService** - Performance tracking, predictions
- **NLPService** - Text processing, question generation

**Responsibilities:**
- Validate input
- Coordinate multiple DAOs
- Apply algorithms
- Enforce business rules

---

### 3. Data Access Layer (DAO)
**Purpose:** Database operations

**Components:**
- **UserDAO** - CRUD operations on users table
- **MaterialDAO** - CRUD operations on materials table
- **QuizItemDAO** - CRUD operations on quiz_items table
- **SessionDAO** - CRUD operations on study_sessions table
- **SpacedRepetitionDAO** - CRUD operations on spaced_repetition_state table

**Responsibilities:**
- Execute SQL queries
- Map database rows to Java objects
- Handle database connections
- NO business logic here

---

### 4. Database Layer
**Purpose:** Persistent data storage

**Technology:** PostgreSQL 15+

**Tables:**
- users
- materials
- quiz_items
- study_sessions
- item_responses
- spaced_repetition_state
- performance_metrics
- weak_topics

---

### 5. Algorithm Components (Supporting)
**Purpose:** Mathematical/computational logic

**Components:**
- **SM2Algorithm** - SuperMemo 2 spaced repetition
- **LeitnerSystem** - Leitner box spaced repetition
- **ForgettingCurveModel** - Ebbinghaus retention estimation
- **DifficultyAdjuster** - Dynamic difficulty based on performance
- **WeakTopicDetector** - Statistical analysis of problem areas

**Used by:** AdaptiveLearningService

---

## Data Flow Example: User Login

1. **[Presentation]** User enters username and password
2. **[Presentation → Service]** Calls `UserService.login("john", "pass123")`
3. **[Service]** UserService validates input (not empty)
4. **[Service → DAO]** Calls `UserDAO.findByUsername("john")`
5. **[DAO → Database]** Executes `SELECT * FROM users WHERE username = 'john'`
6. **[Database → DAO]** Returns user record
7. **[DAO → Service]** Returns User object
8. **[Service]** Compares password hash using BCrypt
9. **[Service]** If match, updates last_login via `UserDAO.updateLastLogin(userId)`
10. **[Service → Presentation]** Returns User object or throws AuthenticationException
11. **[Presentation]** Displays "Welcome, John!" or "Invalid credentials"

---

## Data Flow Example: Study Session with Spaced Repetition

1. **[Presentation]** User selects "Study Now"
2. **[Presentation → Service]** Calls `AdaptiveLearningService.getNextReviewItem(userId)`
3. **[Service → DAO]** Calls `SpacedRepetitionDAO.findDueItems(userId, today)`
4. **[DAO → Database]** Executes:
```sql
   SELECT * FROM spaced_repetition_state 
   WHERE user_id = ? AND next_review_date <= CURRENT_DATE
   ORDER BY next_review_date ASC
```
5. **[Database → DAO]** Returns list of due items
6. **[DAO → Service]** Returns list of SpacedRepetitionState objects
7. **[Service]** Selects most overdue item (first in list)
8. **[Service → DAO]** Calls `QuizItemDAO.findById(itemId)`
9. **[DAO → Database]** Executes `SELECT * FROM quiz_items WHERE item_id = ?`
10. **[Database → DAO]** Returns quiz item record
11. **[DAO → Service]** Returns QuizItem object
12. **[Service → Presentation]** Returns question to display
13. **[Presentation]** Shows question: "When did the French Revolution begin?"
14. **[User]** Types answer: "1789"
15. **[Presentation → Service]** Calls `processResponse(userId, itemId, isCorrect=true, responseTime=3200)`
16. **[Service]** Gets current spaced repetition state
17. **[Service → Algorithm]** Calls `SM2Algorithm.calculateNextReview(quality=5, interval=6, EF=2.5, repCount=2)`
18. **[Algorithm]** Calculates: newInterval=16 days, newEF=2.6, newRepCount=3
19. **[Algorithm → Service]** Returns new state
20. **[Service → DAO]** Calls `SpacedRepetitionDAO.update(userId, itemId, newEF=2.6, interval=16, nextReview=today+16)`
21. **[DAO → Database]** Executes UPDATE statement
22. **[Service → DAO]** Calls `SessionDAO.recordResponse(sessionId, itemId, isCorrect=true, responseTime=3200)`
23. **[DAO → Database]** Inserts into item_responses table
24. **[Service → Presentation]** Returns success message
25. **[Presentation]** Shows "Correct! Next review in 16 days"

---

## Communication Rules

### ❌ What NOT to do:
- Presentation should NOT talk directly to DAOs (skip Service layer)
- DAOs should NOT contain business logic (only SQL)
- Services should NOT execute SQL directly (use DAOs)
- Database should NOT be accessed except through DAOs

### ✅ Correct flow:
```
Presentation → Service → DAO → Database
                  ↓
              Algorithm
```

---

## Why This Architecture?

### Separation of Concerns
Each layer has ONE job:
- Presentation = display
- Service = logic
- DAO = database
- Database = storage

### Testability
- Can test services without database (mock DAOs)
- Can test DAOs without services
- Can test algorithms independently

### Maintainability
- Change database? Only modify DAOs
- Change UI? Only modify Presentation
- Add new algorithm? Just implement interface

### Scalability
- Can add REST API without changing services
- Can add new features without breaking existing code
- Can swap PostgreSQL for MongoDB by only changing DAOs
