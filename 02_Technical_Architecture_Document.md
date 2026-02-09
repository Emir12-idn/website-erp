# Technical Architecture Document (TAD)
# Accurate Enterprise 5 - Clone System

## Document Information
- **Project Name**: Accurate Enterprise 5 Clone
- **Document Version**: 1.0
- **Date**: February 2026
- **Author**: Solution Architect Team
- **Status**: Draft

---

## 1. INTRODUCTION

### 1.1 Purpose
Dokumen ini menjelaskan arsitektur teknis sistem ERP, termasuk komponen sistem, teknologi yang digunakan, dan interaksi antar komponen.

### 1.2 Scope
- Arsitektur aplikasi
- Arsitektur data
- Arsitektur infrastruktur
- Arsitektur keamanan
- Arsitektur integrasi

### 1.3 References
- System Requirements Document v1.0
- Infrastructure Requirements
- Security Standards

---

## 2. ARCHITECTURAL OVERVIEW

### 2.1 Architectural Style
**Multi-Tier Architecture with Microservices Elements**

```
┌─────────────────────────────────────────────────────────┐
│                  Presentation Layer                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │ Web App  │  │Mobile App│  │Admin Panel│  │Customer ││
│  │ (React)  │  │(React N.)│  │  (React)  │  │ Portal  ││
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘│
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                    API Gateway Layer                    │
│  ┌──────────────────────────────────────────────────┐  │
│  │  API Gateway (Kong / AWS API Gateway)            │  │
│  │  - Authentication                                 │  │
│  │  - Rate Limiting                                  │  │
│  │  - Request Routing                                │  │
│  │  - Load Balancing                                 │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  Application Layer                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │Accounting│  │Inventory │  │ Sales    │  │Purchase ││
│  │ Service  │  │ Service  │  │ Service  │  │Service  ││
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘│
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │ HR &     │  │Reporting │  │Integration│  │ Auth    ││
│  │ Payroll  │  │ Service  │  │ Service   │  │Service  ││
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘│
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                    Data Layer                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │PostgreSQL│  │  Redis   │  │ MinIO/S3 │  │Elastic  ││
│  │(Primary) │  │ (Cache)  │  │ (Files)  │  │Search   ││
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘│
└─────────────────────────────────────────────────────────┘
```

### 2.2 Key Architectural Decisions

#### 2.2.1 Technology Stack Selection

**Backend Framework: Node.js with Express**
- Pros: High performance, large ecosystem, JavaScript full-stack
- Cons: Callback hell (mitigated with async/await)
- Alternative: Python Django, PHP Laravel

**Frontend Framework: React.js**
- Pros: Component reusability, virtual DOM, large community
- Cons: Learning curve for beginners
- Alternative: Vue.js, Angular

**Primary Database: PostgreSQL**
- Pros: ACID compliant, JSON support, mature, open-source
- Cons: More complex than MySQL
- Alternative: MySQL, MongoDB (for specific use cases)

**Cache: Redis**
- Pros: In-memory speed, data structure support, pub/sub
- Cons: Memory-intensive
- Alternative: Memcached

**File Storage: MinIO**
- Pros: S3-compatible, self-hosted, cost-effective
- Cons: Maintenance overhead
- Alternative: AWS S3, Azure Blob Storage

**Search Engine: Elasticsearch**
- Pros: Full-text search, analytics, scalable
- Cons: Resource-intensive
- Alternative: PostgreSQL full-text search for smaller datasets

---

## 3. APPLICATION ARCHITECTURE

### 3.1 Backend Architecture

#### 3.1.1 Layered Architecture

```
┌─────────────────────────────────────────┐
│         Controllers Layer               │
│  (HTTP Request Handling)                │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│        Middleware Layer                 │
│  (Authentication, Validation, Logging)  │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│         Service Layer                   │
│  (Business Logic)                       │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│      Data Access Layer (DAL)            │
│  (ORM, Queries, Transactions)           │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│          Database                       │
└─────────────────────────────────────────┘
```

#### 3.1.2 Module Structure

```
project-root/
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   └── validators/
│   │   ├── accounting/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   └── validators/
│   │   ├── inventory/
│   │   ├── sales/
│   │   ├── purchase/
│   │   ├── hr-payroll/
│   │   └── reporting/
│   ├── common/
│   │   ├── middleware/
│   │   ├── utils/
│   │   ├── constants/
│   │   └── config/
│   ├── database/
│   │   ├── migrations/
│   │   ├── seeds/
│   │   └── connection.js
│   └── app.js
├── tests/
├── docs/
└── package.json
```

#### 3.1.3 API Design Pattern

**RESTful API Standards**
```
GET    /api/v1/customers          # List all customers
GET    /api/v1/customers/:id      # Get customer by ID
POST   /api/v1/customers          # Create new customer
PUT    /api/v1/customers/:id      # Update customer
DELETE /api/v1/customers/:id      # Delete customer
PATCH  /api/v1/customers/:id      # Partial update

# Nested resources
GET    /api/v1/customers/:id/invoices
POST   /api/v1/customers/:id/invoices
```

**GraphQL Endpoint (Optional)**
```
POST /graphql

query {
  customer(id: "123") {
    name
    email
    invoices {
      number
      amount
      status
    }
  }
}
```

### 3.2 Frontend Architecture

#### 3.2.1 React Application Structure

```
frontend/
├── public/
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   ├── Table/
│   │   │   └── Modal/
│   │   ├── layout/
│   │   │   ├── Header/
│   │   │   ├── Sidebar/
│   │   │   └── Footer/
│   │   └── modules/
│   │       ├── accounting/
│   │       ├── inventory/
│   │       └── sales/
│   ├── pages/
│   │   ├── Dashboard/
│   │   ├── Customers/
│   │   ├── Invoices/
│   │   └── Reports/
│   ├── hooks/
│   │   ├── useAuth.js
│   │   ├── useApi.js
│   │   └── useForm.js
│   ├── services/
│   │   ├── api.js
│   │   ├── auth.js
│   │   └── storage.js
│   ├── store/
│   │   ├── actions/
│   │   ├── reducers/
│   │   └── store.js
│   ├── utils/
│   │   ├── formatters.js
│   │   ├── validators.js
│   │   └── constants.js
│   ├── styles/
│   ├── App.js
│   └── index.js
├── package.json
└── webpack.config.js
```

#### 3.2.2 State Management

**Redux Pattern**
```javascript
// Store structure
{
  auth: {
    user: {},
    token: '',
    isAuthenticated: false
  },
  accounting: {
    accounts: [],
    journals: [],
    loading: false,
    error: null
  },
  inventory: {
    items: [],
    warehouses: [],
    stocks: []
  },
  ui: {
    sidebarOpen: true,
    theme: 'light'
  }
}
```

#### 3.2.3 Component Design Patterns

**Container/Presentational Pattern**
```javascript
// Container Component (Smart)
const CustomerListContainer = () => {
  const [customers, setCustomers] = useState([]);
  
  useEffect(() => {
    fetchCustomers().then(setCustomers);
  }, []);
  
  return <CustomerList customers={customers} />;
};

// Presentational Component (Dumb)
const CustomerList = ({ customers }) => (
  <div>
    {customers.map(customer => (
      <CustomerCard key={customer.id} customer={customer} />
    ))}
  </div>
);
```

---

## 4. DATABASE ARCHITECTURE

### 4.1 Database Strategy

#### 4.1.1 Multi-Tenancy Approach

**Database per Tenant (Chosen Approach)**
```
PostgreSQL Server
├── company_001_db
├── company_002_db
├── company_003_db
└── master_db (tenant registry)
```

Pros:
- Strong data isolation
- Easy backup per company
- Independent scaling
- Regulatory compliance

Cons:
- More complex maintenance
- Resource overhead

**Alternatives Considered:**
- Schema per tenant
- Shared database with tenant_id column

#### 4.1.2 Database Naming Convention

```
Production: prod_company_{company_id}_db
Staging: stag_company_{company_id}_db
Development: dev_company_{company_id}_db
```

### 4.2 Core Database Schema

#### 4.2.1 Master Database (Tenant Registry)

```sql
-- Master database structure
CREATE TABLE companies (
    id UUID PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    database_name VARCHAR(100) UNIQUE NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    subscription_plan VARCHAR(50),
    subscription_expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE company_users (
    id UUID PRIMARY KEY,
    company_id UUID REFERENCES companies(id),
    user_id UUID REFERENCES users(id),
    role VARCHAR(50) NOT NULL,
    permissions JSONB,
    is_active BOOLEAN DEFAULT true,
    UNIQUE(company_id, user_id)
);
```

#### 4.2.2 Company Database Schema (per tenant)

**Chart of Accounts**
```sql
CREATE TABLE coa_accounts (
    id UUID PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    account_type VARCHAR(50) NOT NULL, -- ASSET, LIABILITY, EQUITY, INCOME, EXPENSE
    parent_id UUID REFERENCES coa_accounts(id),
    level INTEGER NOT NULL,
    normal_balance VARCHAR(10) NOT NULL, -- DEBIT, CREDIT
    is_active BOOLEAN DEFAULT true,
    is_header BOOLEAN DEFAULT false,
    currency_code VARCHAR(3) DEFAULT 'IDR',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_by UUID
);

CREATE INDEX idx_coa_accounts_parent ON coa_accounts(parent_id);
CREATE INDEX idx_coa_accounts_type ON coa_accounts(account_type);
```

**Journal Headers**
```sql
CREATE TABLE journal_headers (
    id UUID PRIMARY KEY,
    journal_number VARCHAR(50) UNIQUE NOT NULL,
    journal_date DATE NOT NULL,
    journal_type VARCHAR(50) NOT NULL, -- GENERAL, SALES, PURCHASE, CASH, etc.
    reference_type VARCHAR(50), -- SO, PO, INV, etc.
    reference_id UUID,
    description TEXT,
    total_debit DECIMAL(20,2) NOT NULL,
    total_credit DECIMAL(20,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'draft', -- draft, posted, void
    posted_at TIMESTAMP,
    posted_by UUID,
    period_year INTEGER NOT NULL,
    period_month INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    CONSTRAINT chk_balanced CHECK (total_debit = total_credit)
);

CREATE INDEX idx_journal_headers_date ON journal_headers(journal_date);
CREATE INDEX idx_journal_headers_status ON journal_headers(status);
CREATE INDEX idx_journal_headers_period ON journal_headers(period_year, period_month);
```

**Journal Details**
```sql
CREATE TABLE journal_details (
    id UUID PRIMARY KEY,
    journal_header_id UUID REFERENCES journal_headers(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    account_id UUID REFERENCES coa_accounts(id),
    description TEXT,
    debit_amount DECIMAL(20,2) DEFAULT 0,
    credit_amount DECIMAL(20,2) DEFAULT 0,
    currency_code VARCHAR(3) DEFAULT 'IDR',
    exchange_rate DECIMAL(20,6) DEFAULT 1,
    cost_center_id UUID,
    project_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_journal_details_header ON journal_details(journal_header_id);
CREATE INDEX idx_journal_details_account ON journal_details(account_id);
```

**Customers**
```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    tax_id VARCHAR(50),
    email VARCHAR(255),
    phone VARCHAR(50),
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) DEFAULT 'Indonesia',
    payment_term_id UUID,
    credit_limit DECIMAL(20,2) DEFAULT 0,
    customer_type VARCHAR(50), -- RETAIL, WHOLESALE, CORPORATE
    price_level_id UUID,
    salesperson_id UUID,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_customers_code ON customers(code);
CREATE INDEX idx_customers_name ON customers(name);
```

**Sales Invoices**
```sql
CREATE TABLE sales_invoices (
    id UUID PRIMARY KEY,
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    invoice_date DATE NOT NULL,
    due_date DATE NOT NULL,
    customer_id UUID REFERENCES customers(id),
    sales_order_id UUID,
    subtotal DECIMAL(20,2) NOT NULL,
    discount_amount DECIMAL(20,2) DEFAULT 0,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    tax_amount DECIMAL(20,2) DEFAULT 0,
    tax_percent DECIMAL(5,2) DEFAULT 0,
    freight_amount DECIMAL(20,2) DEFAULT 0,
    total_amount DECIMAL(20,2) NOT NULL,
    paid_amount DECIMAL(20,2) DEFAULT 0,
    status VARCHAR(20) DEFAULT 'draft', -- draft, approved, sent, partial_paid, paid, void
    notes TEXT,
    terms TEXT,
    currency_code VARCHAR(3) DEFAULT 'IDR',
    exchange_rate DECIMAL(20,6) DEFAULT 1,
    salesperson_id UUID,
    journal_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    approved_at TIMESTAMP,
    approved_by UUID
);

CREATE INDEX idx_sales_invoices_customer ON sales_invoices(customer_id);
CREATE INDEX idx_sales_invoices_date ON sales_invoices(invoice_date);
CREATE INDEX idx_sales_invoices_status ON sales_invoices(status);
```

**Sales Invoice Details**
```sql
CREATE TABLE sales_invoice_details (
    id UUID PRIMARY KEY,
    invoice_id UUID REFERENCES sales_invoices(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    item_id UUID,
    description TEXT NOT NULL,
    quantity DECIMAL(20,4) NOT NULL,
    unit_of_measure VARCHAR(20),
    unit_price DECIMAL(20,2) NOT NULL,
    discount_amount DECIMAL(20,2) DEFAULT 0,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    tax_amount DECIMAL(20,2) DEFAULT 0,
    line_total DECIMAL(20,2) NOT NULL,
    warehouse_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Inventory Items**
```sql
CREATE TABLE inventory_items (
    id UUID PRIMARY KEY,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    item_type VARCHAR(50) NOT NULL, -- STOCK, SERVICE, NON_STOCK
    category_id UUID,
    base_unit VARCHAR(20) NOT NULL,
    barcode VARCHAR(100),
    purchase_price DECIMAL(20,2),
    selling_price DECIMAL(20,2),
    cost_method VARCHAR(20) DEFAULT 'FIFO', -- FIFO, AVERAGE, STANDARD
    reorder_point DECIMAL(20,4),
    minimum_stock DECIMAL(20,4),
    maximum_stock DECIMAL(20,4),
    is_active BOOLEAN DEFAULT true,
    track_serial BOOLEAN DEFAULT false,
    track_batch BOOLEAN DEFAULT false,
    image_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_items_sku ON inventory_items(sku);
CREATE INDEX idx_items_name ON inventory_items(name);
CREATE INDEX idx_items_category ON inventory_items(category_id);
```

**Stock Transactions**
```sql
CREATE TABLE stock_transactions (
    id UUID PRIMARY KEY,
    transaction_number VARCHAR(50) UNIQUE NOT NULL,
    transaction_date DATE NOT NULL,
    transaction_type VARCHAR(50) NOT NULL, -- IN, OUT, ADJUSTMENT, TRANSFER
    reference_type VARCHAR(50),
    reference_id UUID,
    item_id UUID REFERENCES inventory_items(id),
    warehouse_id UUID,
    quantity DECIMAL(20,4) NOT NULL,
    unit_cost DECIMAL(20,2),
    total_cost DECIMAL(20,2),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID
);

CREATE INDEX idx_stock_trans_item ON stock_transactions(item_id);
CREATE INDEX idx_stock_trans_warehouse ON stock_transactions(warehouse_id);
CREATE INDEX idx_stock_trans_date ON stock_transactions(transaction_date);
```

**Stock Balance (Materialized View)**
```sql
CREATE MATERIALIZED VIEW stock_balance AS
SELECT 
    item_id,
    warehouse_id,
    SUM(CASE WHEN transaction_type IN ('IN', 'ADJUSTMENT_IN') THEN quantity
             WHEN transaction_type IN ('OUT', 'ADJUSTMENT_OUT') THEN -quantity
             ELSE 0 END) as quantity_on_hand,
    MAX(transaction_date) as last_transaction_date
FROM stock_transactions
GROUP BY item_id, warehouse_id;

CREATE INDEX idx_stock_balance_item ON stock_balance(item_id);
CREATE INDEX idx_stock_balance_warehouse ON stock_balance(warehouse_id);
```

### 4.3 Database Optimization

#### 4.3.1 Indexing Strategy
- Primary keys: Clustered indexes
- Foreign keys: Non-clustered indexes
- Frequently queried columns: Composite indexes
- Full-text search columns: GIN indexes (PostgreSQL)

#### 4.3.2 Partitioning Strategy
```sql
-- Partition journal_headers by year
CREATE TABLE journal_headers (
    -- columns
) PARTITION BY RANGE (period_year);

CREATE TABLE journal_headers_2024 PARTITION OF journal_headers
    FOR VALUES FROM (2024) TO (2025);

CREATE TABLE journal_headers_2025 PARTITION OF journal_headers
    FOR VALUES FROM (2025) TO (2026);
```

#### 4.3.3 Query Optimization
- Use prepared statements
- Implement connection pooling
- Use database views for complex queries
- Implement caching for frequent queries
- Regular VACUUM and ANALYZE (PostgreSQL)

---

## 5. INFRASTRUCTURE ARCHITECTURE

### 5.1 Deployment Architecture

#### 5.1.1 Production Environment

```
                    ┌─────────────┐
                    │   Users     │
                    └──────┬──────┘
                           │
                    ┌──────▼───────┐
                    │   Cloudflare │
                    │   CDN + WAF  │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ Load Balancer│
                    │  (Nginx/HAP) │
                    └──────┬───────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐       ┌────▼────┐      ┌────▼────┐
    │  App    │       │  App    │      │  App    │
    │ Server 1│       │ Server 2│      │ Server 3│
    └────┬────┘       └────┬────┘      └────┬────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐       ┌────▼────┐      ┌────▼────┐
    │PostgreSQL       │  Redis  │      │  MinIO  │
    │ Primary │       │ Cluster │      │ Cluster │
    └────┬────┘       └─────────┘      └─────────┘
         │
    ┌────▼────┐
    │PostgreSQL
    │ Replica │
    └─────────┘
```

#### 5.1.2 Container Architecture (Docker)

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app

  app:
    build: .
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 2G

  postgres:
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_PASSWORD}
    volumes:
      - minio_data:/data

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

### 5.2 Server Specifications

#### 5.2.1 Production Servers

**Application Servers (3 instances)**
- CPU: 4 cores (minimum) / 8 cores (recommended)
- RAM: 8 GB (minimum) / 16 GB (recommended)
- Storage: 100 GB SSD
- OS: Ubuntu 22.04 LTS

**Database Server (Primary)**
- CPU: 8 cores
- RAM: 32 GB
- Storage: 500 GB SSD (RAID 10)
- OS: Ubuntu 22.04 LTS
- Backup: Daily full backup + WAL archiving

**Database Server (Replica)**
- CPU: 4 cores
- RAM: 16 GB
- Storage: 500 GB SSD
- OS: Ubuntu 22.04 LTS

**Cache Server (Redis Cluster - 3 nodes)**
- CPU: 2 cores each
- RAM: 8 GB each
- Storage: 50 GB SSD each

**File Storage (MinIO Cluster - 4 nodes)**
- CPU: 4 cores each
- RAM: 8 GB each
- Storage: 1 TB SSD each

### 5.3 Network Architecture

#### 5.3.1 Network Topology
```
Internet
    │
    ▼
┌───────────────┐
│  Firewall     │
│  (WAF)        │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ DMZ Zone      │
│ (Load Bal.)   │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ App Zone      │
│ (App Servers) │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ Data Zone     │
│ (DB, Cache)   │
└───────────────┘
```

#### 5.3.2 Security Zones
- **DMZ Zone**: Load balancers, reverse proxy
- **Application Zone**: Application servers
- **Data Zone**: Databases, cache, file storage
- **Management Zone**: Monitoring, logging, backup

---

## 6. SECURITY ARCHITECTURE

### 6.1 Authentication & Authorization

#### 6.1.1 Authentication Flow

```
User Login
    │
    ▼
Validate Credentials
    │
    ▼
Generate JWT Token
    │
    ├─ Access Token (15 min expiry)
    └─ Refresh Token (7 days expiry)
    │
    ▼
Return Tokens to Client
    │
    ▼
Client stores in HttpOnly Cookie
    │
    ▼
Subsequent Requests
    │
    ▼
Validate Access Token
    │
    ├─ Valid ──────────► Process Request
    └─ Expired ────────► Refresh Token Flow
```

#### 6.1.2 JWT Token Structure

```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user-uuid",
    "username": "john.doe",
    "company_id": "company-uuid",
    "roles": ["accountant", "manager"],
    "permissions": ["read:invoices", "write:invoices"],
    "iat": 1609459200,
    "exp": 1609460100
  }
}
```

#### 6.1.3 Role-Based Access Control (RBAC)

```
Roles Hierarchy:
- Super Admin
  └─ Company Admin
      ├─ Finance Manager
      │   ├─ Accountant
      │   └─ Cashier
      ├─ Sales Manager
      │   └─ Sales
      ├─ Purchase Manager
      │   └─ Purchaser
      └─ Warehouse Manager
          └─ Warehouse Staff
```

### 6.2 Data Security

#### 6.2.1 Encryption

**Data in Transit**
- TLS 1.3 for all connections
- Certificate from Let's Encrypt or commercial CA
- HSTS enabled
- SSL certificate monitoring

**Data at Rest**
- Database encryption (transparent data encryption)
- File storage encryption (AES-256)
- Encrypted backups
- Key rotation policy

#### 6.2.2 Sensitive Data Handling

```javascript
// Password hashing
const bcrypt = require('bcrypt');
const saltRounds = 12;
const hashedPassword = await bcrypt.hash(password, saltRounds);

// Encryption for sensitive fields
const crypto = require('crypto');
const algorithm = 'aes-256-gcm';

function encrypt(text, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag();
  return {
    encrypted,
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex')
  };
}
```

### 6.3 API Security

#### 6.3.1 Rate Limiting

```javascript
// Rate limiting configuration
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later.'
});

app.use('/api/', limiter);
```

#### 6.3.2 Input Validation

```javascript
// Using Joi for validation
const Joi = require('joi');

const invoiceSchema = Joi.object({
  customer_id: Joi.string().uuid().required(),
  invoice_date: Joi.date().required(),
  items: Joi.array().items(
    Joi.object({
      item_id: Joi.string().uuid().required(),
      quantity: Joi.number().positive().required(),
      unit_price: Joi.number().positive().required()
    })
  ).min(1).required()
});
```

### 6.4 Audit Trail

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    company_id UUID NOT NULL,
    action VARCHAR(50) NOT NULL, -- CREATE, UPDATE, DELETE, LOGIN, LOGOUT
    entity_type VARCHAR(100) NOT NULL,
    entity_id UUID,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at);
```

---

## 7. INTEGRATION ARCHITECTURE

### 7.1 API Gateway Pattern

```
External Services
    │
    ▼
┌─────────────────┐
│  API Gateway    │
│  - Authentication
│  - Rate Limiting│
│  - Request Log  │
│  - Routing      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌─────┐   ┌─────┐
│ App │   │ App │
│ API │   │ API │
└─────┘   └─────┘
```

### 7.2 Third-Party Integrations

#### 7.2.1 Payment Gateway Integration

```javascript
// Payment Gateway Abstraction
class PaymentGateway {
  async createPayment(params) {
    throw new Error('Not implemented');
  }
  
  async verifyPayment(transactionId) {
    throw new Error('Not implemented');
  }
}

class MidtransGateway extends PaymentGateway {
  async createPayment(params) {
    // Midtrans implementation
  }
}

class XenditGateway extends PaymentGateway {
  async createPayment(params) {
    // Xendit implementation
  }
}

// Factory pattern
class PaymentGatewayFactory {
  static create(provider) {
    switch(provider) {
      case 'midtrans': return new MidtransGateway();
      case 'xendit': return new XenditGateway();
      default: throw new Error('Unknown provider');
    }
  }
}
```

#### 7.2.2 E-Commerce Integration

```javascript
// E-commerce sync service
class EcommerceSync {
  async syncProducts() {
    const products = await this.getProductsFromERP();
    await this.pushToEcommerce(products);
  }
  
  async syncOrders() {
    const orders = await this.getOrdersFromEcommerce();
    await this.createSalesOrders(orders);
  }
  
  async syncInventory() {
    const stock = await this.getStockFromERP();
    await this.updateEcommerceStock(stock);
  }
}
```

### 7.3 Webhook System

```javascript
// Webhook handler
app.post('/webhooks/:provider', async (req, res) => {
  const { provider } = req.params;
  const signature = req.headers['x-signature'];
  
  // Verify webhook signature
  if (!verifySignature(req.body, signature, provider)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Queue webhook for processing
  await webhookQueue.add({
    provider,
    event: req.body.event,
    data: req.body.data
  });
  
  res.status(200).json({ received: true });
});
```

---

## 8. MONITORING & LOGGING

### 8.1 Application Monitoring

```javascript
// Prometheus metrics
const promClient = require('prom-client');

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status']
});

const databaseQueryDuration = new promClient.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Duration of database queries in seconds',
  labelNames: ['query_type']
});
```

### 8.2 Logging Strategy

```javascript
// Winston logger configuration
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'erp-api' },
  transports: [
    new winston.transports.File({ 
      filename: 'error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'combined.log' 
    })
  ]
});
```

### 8.3 Health Check Endpoints

```javascript
// Health check
app.get('/health', async (req, res) => {
  const health = {
    uptime: process.uptime(),
    status: 'OK',
    timestamp: Date.now(),
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      storage: await checkStorage()
    }
  };
  
  const httpStatus = Object.values(health.checks)
    .every(check => check.status === 'OK') ? 200 : 503;
    
  res.status(httpStatus).json(health);
});
```

---

## 9. DISASTER RECOVERY

### 9.1 Backup Strategy

**Database Backups**
- Full backup: Daily at 2:00 AM
- Incremental backup: Every 6 hours
- WAL archiving: Continuous
- Retention: 30 days
- Offsite backup: Cloud storage

**Application Backups**
- Configuration files: Daily
- Code repository: Git with tags
- File storage: Daily sync to backup location

### 9.2 Recovery Procedures

**RTO (Recovery Time Objective)**: 4 hours
**RPO (Recovery Point Objective)**: 6 hours

**Recovery Steps**:
1. Assess damage
2. Activate backup servers
3. Restore database from latest backup
4. Apply WAL logs if needed
5. Verify data integrity
6. Update DNS/load balancer
7. Monitor system health

---

## 10. SCALABILITY CONSIDERATIONS

### 10.1 Horizontal Scaling

- Stateless application servers
- Session storage in Redis
- Load balancer distribution
- Database read replicas

### 10.2 Vertical Scaling

- Upgrade server resources as needed
- Database server can scale vertically before horizontal

### 10.3 Caching Strategy

```
Request
    │
    ▼
Check Cache (Redis)
    │
    ├─ HIT ───────► Return cached data
    │
    └─ MISS ──────► Query Database
                         │
                         ▼
                    Cache result
                         │
                         ▼
                    Return data
```

---

## APPENDICES

### Appendix A: Technology Versions
- Node.js: 20 LTS
- PostgreSQL: 14+
- Redis: 7+
- React: 18+
- Docker: 24+

### Appendix B: Coding Standards
- ESLint configuration
- Prettier configuration
- Git commit conventions
- Code review checklist

### Appendix C: API Documentation
- Swagger/OpenAPI specification
- API endpoints catalog
- Authentication guide

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Next Review**: May 2026
