# Web to WAP Software - System Architecture

## 1. Architecture Overview

### 1.1 System Architecture Pattern
The system follows a **Three-Tier Architecture (MVC Pattern)** suitable for 2007 web applications:

```
┌─────────────────────────────────────────────────────────┐
│           PRESENTATION LAYER (JSP/HTML/CSS)             │
│  - User Interface                                        │
│  - Web Forms & Controls                                  │
│  - Client-side JavaScript (AJAX)                         │
└────────────────┬────────────────────────────────────────┘
                 │ HTTP/HTTPS
┌────────────────▼────────────────────────────────────────┐
│        APPLICATION LAYER (Java Servlets/JSP)            │
│  - Business Logic                                        │
│  - Request Handling                                      │
│  - Session Management                                    │
│  - File Processing                                       │
└────────────────┬────────────────────────────────────────┘
                 │ JDBC
┌────────────────▼────────────────────────────────────────┐
│            DATA LAYER (Database Access)                  │
│  - Database Connection Pool                              │
│  - DAO Pattern Implementation                            │
│  - Query Execution                                       │
│  - Data Persistence                                      │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Component Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Web Browser (2007)                    │
│         Internet Explorer 6+, Firefox 2+, Safari 3+      │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼──────────┐    ┌────────▼──────────┐
│   HTML Parser    │    │   WML Generator   │
│  - Tokenizer     │    │  - Element Mapper │
│  - DOM Builder   │    │  - Card Creator   │
│  - Validator     │    │  - Validator      │
└───────┬──────────┘    └────────┬──────────┘
        │                        │
        └────────────┬───────────┘
                     │
            ┌────────▼──────────┐
            │ Conversion Engine │
            │  - Orchestrator   │
            │  - Flow Control   │
            │  - Error Handler  │
            └────────┬──────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
┌──────���▼──────────┐    ┌────────▼──────────┐
│  Optimization    │    │  File Storage &   │
│  - Image Opt.    │    │  Database Layer   │
│  - CSS Reducer   │    │  - File Manager   │
│  - Size Reducer  │    │  - DAO Classes    │
└──────────────────┘    └───────────────────┘
```

## 2. Module Specifications (2007 Technologies)

### 2.1 HTML Parser Module
**Technology:** Java 1.5+, DOM API

```
Input: HTML/XHTML Document
       ↓
   [Tokenizer]
   - Lexical analysis
   - Token generation
       ↓
   [Parser]
   - Syntax analysis
   - DOM tree construction
       ↓
   [Validator]
   - Well-formedness check
   - Standards compliance
       ↓
Output: DOM Tree Object
```

**Key Classes:**
- `HtmlLexer.java` - Tokenization
- `HtmlParser.java` - Parsing logic
- `DomTree.java` - DOM representation

### 2.2 WML Generator Module
**Technology:** Java 1.5+, XML Generation

```
Input: DOM Tree
       ↓
   [Element Mapper]
   - HTML→WML mapping
   - Attribute conversion
       ↓
   [Card Structurer]
   - Deck creation
   - Card organization
       ↓
   [Content Formatter]
   - Text formatting
   - Entity encoding
       ↓
   [WML Validator]
   - Schema validation
   - Spec compliance
       ↓
Output: Valid WML 1.2 Document
```

**Key Classes:**
- `WmlGenerator.java` - Main generator
- `ElementMapper.java` - HTML→WML mapping
- `CardStructurer.java` - Card organization

### 2.3 Conversion Engine
**Technology:** Java 1.5+, Design Patterns (Strategy, Pipeline)

```
Input: HTML File
       ↓
   [Conversion Pipeline]
   Phase 1: Load & Validate
   Phase 2: Parse to DOM
   Phase 3: Optimize Content
   Phase 4: Convert to WML
   Phase 5: Validate Output
   Phase 6: Store Result
       ↓
Output: WML File + Metadata
```

### 2.4 Optimization Module
**Technology:** Java 1.5+, Image Libraries (JAI or ImageMagick)

**Image Optimization:**
- Input formats: JPEG, PNG, GIF, BMP
- Output formats: GIF, WBMP (2007 standard mobile formats)
- Compression: 75-85% quality for mobile viewing
- Resizing: 240x320 pixels (typical 2007 mobile screens)

**CSS Optimization:**
- Convert linked CSS to inline styles
- Remove unsupported properties
- Collapse margin/padding values

**Content Reduction:**
- Remove comments and unnecessary whitespace
- Eliminate redundant attributes
- Strip unused classes/IDs

### 2.5 Database Layer (MySQL 5.0+)
**Technology:** Java JDBC, MySQL 5.0+, DAO Pattern

```
Database Schema:
├── users
│   ├── user_id (PK)
│   ├── username
│   ├── password (encrypted)
│   ├── email
│   └── created_date
│
├── conversions
│   ├── conversion_id (PK)
│   ├── user_id (FK)
│   ├── source_file
│   ├── output_file
│   ├── conversion_date
│   └── status
│
└── files
    ├── file_id (PK)
    ├── user_id (FK)
    ├── file_name
    ├── file_path
    ├── file_size
    └── upload_date
```

### 2.6 Web Layer (JSP/Servlets)
**Technology:** JSP 2.0, Servlets 2.4, Apache Tomcat 5.5+

**Key Servlets:**
- `LoginServlet.java` - Authentication
- `FileUploadServlet.java` - File handling
- `ConversionServlet.java` - Conversion requests
- `DownloadServlet.java` - File delivery

**JSP Pages:**
- `login.jsp` - User login
- `dashboard.jsp` - Main interface
- `upload.jsp` - File upload form
- `convert.jsp` - Conversion interface
- `preview.jsp` - Results display

## 3. Data Flow Diagram

```
User (Browser)
    │
    ├─→ [Login/Register] ──→ AuthManager ──→ Database
    │
    ├─→ [Upload File] ──→ FileUploadServlet ──→ FileStorage
    │                                              │
    │                                              ↓
    │                                        [Queue for Conversion]
    │
    ├─→ [Convert] ──→ ConversionServlet ──→ ConversionEngine
    │                                            │
    │                                    ┌───────┼───────┐
    │                                    ↓       ↓       ↓
    │                            HtmlParser WmlGenerator Optimizer
    │                                    │       │       │
    │                                    └───────┼───────┘
    │                                            ↓
    │                                    [Store WML File]
    │                                            │
    │                                    [Update Database]
    │
    ├─→ [Preview] ──→ PreviewServlet ──→ [Display HTML + WML]
    │
    └─→ [Download] ──→ DownloadServlet ──→ [Send WML File]
```

## 4. Database Design (MySQL 5.0+)

```sql
-- Users Table
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, -- Encrypted
    email VARCHAR(100) NOT NULL,
    full_name VARCHAR(100),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login DATETIME,
    is_active BOOLEAN DEFAULT TRUE,
    storage_quota INT DEFAULT 1000 -- MB
);

-- Conversions Table
CREATE TABLE conversions (
    conversion_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    source_file VARCHAR(255),
    source_size INT,
    output_file VARCHAR(255),
    output_size INT,
    conversion_date TIMESTAMP,
    conversion_time INT, -- seconds
    status VARCHAR(20), -- pending, processing, completed, failed
    error_message TEXT,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Files Table
CREATE TABLE files (
    file_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    file_name VARCHAR(255),
    file_path VARCHAR(500),
    file_size INT,
    file_type VARCHAR(20), -- HTML, WML
    upload_date TIMESTAMP,
    conversion_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (conversion_id) REFERENCES conversions(conversion_id)
);

-- Indexes for Performance
CREATE INDEX idx_user_conversions ON conversions(user_id);
CREATE INDEX idx_conversion_date ON conversions(conversion_date);
CREATE INDEX idx_file_user ON files(user_id);
```

## 5. 2007 Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | HTML 4.01, CSS 2.1 | Standard in 2007 browsers |
| | JavaScript 1.5 | Basic AJAX support (XMLHttpRequest) |
| | JSP 2.0 | Server-side rendering |
| **Backend** | Java 5 (1.5.0) | Enterprise standard |
| | Servlets 2.4 | Web request handling |
| | JDBC | Database connectivity |
| **Application Server** | Apache Tomcat 5.5+ | Industry standard (open-source) |
| **Database** | MySQL 5.0+ | Most popular open-source DB |
| | SQLite 3.x | Alternative (embedded) |
| **Build Tool** | Apache Ant 1.6+ | Standard build automation |
| **Testing** | JUnit 3.8 | Unit testing framework |
| **XML Processing** | Java DOM API | Built-in XML handling |
| **Deployment** | WAR (Web Archive) | Standard Java web app format |

## 6. Security Architecture

```
┌─────────────────────────────────────────────┐
│         Security Layers (2007 Standards)     │
├───────��─────────────────────────────────────┤
│ 1. SSL/TLS (HTTPS)                          │
│    - Encrypted communication                 │
│    - Certificate-based (self-signed OK)     │
├─────────────────────────────────────────────┤
│ 2. Authentication Layer                      │
│    - Username/Password                       │
│    - Password encryption (MD5/SHA-1)         │
│    - Session management                      │
├─────────────────────────────────────────────┤
│ 3. Authorization Layer                       │
│    - Role-based access control (RBAC)        │
│    - User roles: Admin, Content Manager, User|
├─────────────────────────────────────────────┤
│ 4. Input Validation                          │
│    - Server-side validation                  │
│    - Sanitization of all inputs              │
│    - Prevention of SQL injection             │
│    - Prevention of XSS attacks               │
├─────────────────────────────────────────────┤
│ 5. Data Protection                           │
│    - Encrypted password storage              │
│    - Secure file handling                    │
│    - Access control on files                 │
└─────────────────────────────────────────────┘
```

## 7. Deployment Architecture

```
┌────────────────────────────────────────────┐
│       Development Environment               │
│  - IDE: NetBeans/Eclipse                   │
│  - Database: MySQL (localhost)             │
│  - Server: Tomcat (localhost:8080)         │
└────────────────────────────────────────────┘
                    ↓
         [Build & Test]
         - ant clean build
         - ant test
         - Code review
                    ↓
┌────────────────────────────────────────────┐
│       Staging Environment                   │
│  - Same as production (smaller scale)       │
│  - UAT testing                              │
│  - Performance testing                      │
└────────────────────────────────────────────┘
                    ↓
         [Final Testing & Sign-off]
                    ↓
┌────────────────────────────────────────────┐
│       Production Environment                │
│  - Windows Server 2003/Linux                │
│  - MySQL 5.0 database                       │
│  - Apache Tomcat 5.5+                       │
│  - Backup & disaster recovery               │
│  - Monitoring & logging                     │
└────────────────────────────────────────────┘
```

## 8. Performance Optimization Strategies

### 2.1 Database Performance
- Connection pooling (Apache Commons DBCP)
- Query optimization with proper indexes
- Prepared statements for parameterized queries
- Caching of frequently accessed data

### 2.2 File Processing Performance
- Batch conversion for multiple files
- Asynchronous processing with queues
- Streaming for large file handling
- Image compression in background

### 2.3 Web Application Performance
- Page caching (HTTP headers)
- Gzip compression of responses
- Minimize JavaScript files (2007 approach)
- CSS consolidation
- Image optimization

## 9. Integration Points

**External Systems (2007 Context):**
- Mobile devices (via WAP protocol)
- File systems (upload/download)
- Email system (notifications - future feature)
- LDAP/Active Directory (enterprise auth - future feature)

## 10. Scalability Considerations

**For 2007 Era:**
- Single server deployment suitable for 100-500 concurrent users
- Database scaling: MySQL replication for read-heavy workloads
- Load balancing: Consider for high-traffic scenarios
- File storage: Direct filesystem (no need for distributed storage)

---

**Document Version:** 1.0  
**Created:** 2026-07-13  
**Technology Era:** 2007  
**Status:** ACTIVE
