# Database Design Document (DDD)
# Accurate Enterprise 5 - Clone System

## Document Information
- **Project Name**: Accurate Enterprise 5 Clone
- **Document Version**: 1.0
- **Date**: February 2026
- **Author**: Database Architect Team

---

## 1. DATABASE OVERVIEW

### 1.1 Database Management System
**Primary DBMS**: PostgreSQL 14+

**Reasons for Selection**:
- ACID compliance
- Advanced JSON support (JSONB)
- Excellent performance with complex queries
- Robust full-text search capabilities
- Open-source with commercial support available
- Strong community and ecosystem
- Support for partitioning and sharding

### 1.2 Database Strategy

#### Multi-Tenancy Architecture
**Database-per-Tenant Approach**

```
Master Database (tenant_registry)
├── User Management
├── Company Registry
├── Subscription Management
└── Global Configuration

Company Databases (per tenant)
├── company_001_production
├── company_002_production
├── company_003_production
└── ...
```

**Benefits**:
- Strong data isolation
- Independent scaling per company
- Easier backup and restore
- Regulatory compliance
- Custom schema per company if needed

**Challenges**:
- More complex connection management
- Higher resource overhead
- Schema migration across databases

---

## 2. MASTER DATABASE SCHEMA

### 2.1 Companies Table

```sql
CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_code VARCHAR(20) UNIQUE NOT NULL,
    company_name VARCHAR(255) NOT NULL,
    database_name VARCHAR(100) UNIQUE NOT NULL,
    database_host VARCHAR(255) DEFAULT 'localhost',
    database_port INTEGER DEFAULT 5432,
    
    -- Company Information
    tax_id VARCHAR(50),
    registration_number VARCHAR(100),
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) DEFAULT 'Indonesia',
    phone VARCHAR(50),
    email VARCHAR(255),
    website VARCHAR(255),
    
    -- Subscription
    subscription_plan VARCHAR(50) NOT NULL, -- starter, professional, enterprise
    subscription_status VARCHAR(20) DEFAULT 'active', -- active, suspended, cancelled
    subscription_start_date DATE NOT NULL,
    subscription_end_date DATE,
    max_users INTEGER DEFAULT 10,
    storage_limit_gb INTEGER DEFAULT 10,
    
    -- Configuration
    default_currency VARCHAR(3) DEFAULT 'IDR',
    fiscal_year_start INTEGER DEFAULT 1, -- Month: 1=January
    timezone VARCHAR(50) DEFAULT 'Asia/Jakarta',
    language VARCHAR(10) DEFAULT 'id',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_demo BOOLEAN DEFAULT false,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_by UUID,
    
    CONSTRAINT chk_subscription_dates CHECK (subscription_end_date IS NULL OR subscription_end_date > subscription_start_date)
);

CREATE INDEX idx_companies_code ON companies(company_code);
CREATE INDEX idx_companies_status ON companies(subscription_status);
CREATE INDEX idx_companies_active ON companies(is_active);

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_companies_updated_at BEFORE UPDATE ON companies
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 2.2 Users Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified BOOLEAN DEFAULT false,
    email_verified_at TIMESTAMP,
    
    -- Authentication
    password_hash VARCHAR(255) NOT NULL,
    password_changed_at TIMESTAMP,
    password_reset_token VARCHAR(255),
    password_reset_expires TIMESTAMP,
    
    -- Two-Factor Authentication
    two_factor_enabled BOOLEAN DEFAULT false,
    two_factor_secret VARCHAR(255),
    
    -- Personal Information
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    full_name VARCHAR(255),
    phone VARCHAR(50),
    avatar_url TEXT,
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_locked BOOLEAN DEFAULT false,
    locked_at TIMESTAMP,
    locked_reason TEXT,
    failed_login_attempts INTEGER DEFAULT 0,
    last_login_at TIMESTAMP,
    last_login_ip INET,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_active ON users(is_active);

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 2.3 Company Users Table

```sql
CREATE TABLE company_users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Role & Permissions
    role VARCHAR(50) NOT NULL, -- admin, manager, user, viewer
    custom_permissions JSONB DEFAULT '[]',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_default_company BOOLEAN DEFAULT false,
    
    -- Audit
    invited_by UUID REFERENCES users(id),
    invited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    joined_at TIMESTAMP,
    last_access_at TIMESTAMP,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT uq_company_user UNIQUE(company_id, user_id)
);

CREATE INDEX idx_company_users_company ON company_users(company_id);
CREATE INDEX idx_company_users_user ON company_users(user_id);
CREATE INDEX idx_company_users_role ON company_users(role);

CREATE TRIGGER update_company_users_updated_at BEFORE UPDATE ON company_users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 2.4 Subscription Plans Table

```sql
CREATE TABLE subscription_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_code VARCHAR(50) UNIQUE NOT NULL,
    plan_name VARCHAR(100) NOT NULL,
    description TEXT,
    
    -- Pricing
    monthly_price DECIMAL(10,2) NOT NULL,
    annual_price DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'IDR',
    
    -- Limits
    max_users INTEGER NOT NULL,
    max_companies INTEGER NOT NULL,
    storage_limit_gb INTEGER NOT NULL,
    max_transactions_per_month INTEGER,
    
    -- Features (stored as JSON)
    features JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_public BOOLEAN DEFAULT true,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscription_plans_code ON subscription_plans(plan_code);
CREATE INDEX idx_subscription_plans_active ON subscription_plans(is_active);
```

### 2.5 Audit Logs Table

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Context
    company_id UUID REFERENCES companies(id),
    user_id UUID REFERENCES users(id),
    
    -- Action
    action VARCHAR(50) NOT NULL, -- CREATE, READ, UPDATE, DELETE, LOGIN, LOGOUT
    entity_type VARCHAR(100) NOT NULL,
    entity_id UUID,
    
    -- Details
    old_values JSONB,
    new_values JSONB,
    changes JSONB, -- Only changed fields
    description TEXT,
    
    -- Request Info
    ip_address INET,
    user_agent TEXT,
    request_id VARCHAR(100),
    
    -- Timestamp
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_company ON audit_logs(company_id);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at DESC);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);

-- Partition by month for better performance
CREATE TABLE audit_logs (
    -- columns as above
) PARTITION BY RANGE (created_at);
```

---

## 3. COMPANY DATABASE SCHEMA

### 3.1 Chart of Accounts Module

#### 3.1.1 COA Accounts Table

```sql
CREATE TABLE coa_accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Account Code & Name
    account_code VARCHAR(50) UNIQUE NOT NULL,
    account_name VARCHAR(255) NOT NULL,
    
    -- Account Classification
    account_type VARCHAR(50) NOT NULL, -- ASSET, LIABILITY, EQUITY, REVENUE, EXPENSE
    account_sub_type VARCHAR(50), -- CURRENT_ASSET, FIXED_ASSET, etc.
    account_category VARCHAR(100),
    
    -- Hierarchy
    parent_id UUID REFERENCES coa_accounts(id),
    level INTEGER NOT NULL DEFAULT 1,
    is_header BOOLEAN DEFAULT false,
    has_children BOOLEAN DEFAULT false,
    full_path VARCHAR(500), -- For hierarchical queries
    
    -- Accounting Properties
    normal_balance VARCHAR(10) NOT NULL CHECK (normal_balance IN ('DEBIT', 'CREDIT')),
    currency_code VARCHAR(3) DEFAULT 'IDR',
    allow_manual_entry BOOLEAN DEFAULT true,
    require_project BOOLEAN DEFAULT false,
    require_cost_center BOOLEAN DEFAULT false,
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_system BOOLEAN DEFAULT false, -- System accounts can't be deleted
    
    -- Balance Tracking
    current_balance DECIMAL(20,2) DEFAULT 0,
    ytd_balance DECIMAL(20,2) DEFAULT 0,
    
    -- Tax Settings
    is_tax_account BOOLEAN DEFAULT false,
    tax_type VARCHAR(50),
    
    -- Description
    description TEXT,
    notes TEXT,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID,
    
    CONSTRAINT chk_account_type CHECK (account_type IN ('ASSET', 'LIABILITY', 'EQUITY', 'REVENUE', 'EXPENSE')),
    CONSTRAINT chk_level CHECK (level BETWEEN 1 AND 5)
);

CREATE INDEX idx_coa_accounts_code ON coa_accounts(account_code);
CREATE INDEX idx_coa_accounts_name ON coa_accounts(account_name);
CREATE INDEX idx_coa_accounts_type ON coa_accounts(account_type);
CREATE INDEX idx_coa_accounts_parent ON coa_accounts(parent_id);
CREATE INDEX idx_coa_accounts_active ON coa_accounts(is_active);

CREATE TRIGGER update_coa_accounts_updated_at BEFORE UPDATE ON coa_accounts
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

#### 3.1.2 Account Opening Balance

```sql
CREATE TABLE account_opening_balances (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES coa_accounts(id),
    fiscal_year INTEGER NOT NULL,
    debit_balance DECIMAL(20,2) DEFAULT 0,
    credit_balance DECIMAL(20,2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    
    CONSTRAINT uq_account_fiscal_year UNIQUE(account_id, fiscal_year)
);

CREATE INDEX idx_opening_balances_account ON account_opening_balances(account_id);
CREATE INDEX idx_opening_balances_year ON account_opening_balances(fiscal_year);
```

### 3.2 General Ledger Module

#### 3.2.1 Journal Headers

```sql
CREATE TABLE journal_headers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Journal Identification
    journal_number VARCHAR(50) UNIQUE NOT NULL,
    journal_date DATE NOT NULL,
    journal_type VARCHAR(50) NOT NULL, -- GENERAL, SALES, PURCHASE, CASH_IN, CASH_OUT, BANK, OPENING, ADJUSTMENT, CLOSING
    reference_type VARCHAR(50), -- SO, PO, INV, PAYMENT, etc.
    reference_number VARCHAR(100),
    reference_id UUID,
    
    -- Period
    period_year INTEGER NOT NULL,
    period_month INTEGER NOT NULL CHECK (period_month BETWEEN 1 AND 12),
    
    -- Amounts
    total_debit DECIMAL(20,2) NOT NULL DEFAULT 0,
    total_credit DECIMAL(20,2) NOT NULL DEFAULT 0,
    
    -- Status & Workflow
    status VARCHAR(20) DEFAULT 'draft', -- draft, submitted, approved, posted, void
    posted_at TIMESTAMP,
    posted_by UUID,
    void_at TIMESTAMP,
    void_by UUID,
    void_reason TEXT,
    
    -- Reversal
    is_reversal BOOLEAN DEFAULT false,
    reversed_journal_id UUID REFERENCES journal_headers(id),
    reversal_journal_id UUID REFERENCES journal_headers(id),
    
    -- Description
    description TEXT,
    notes TEXT,
    
    -- Attachment
    has_attachment BOOLEAN DEFAULT false,
    attachment_count INTEGER DEFAULT 0,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID,
    approved_at TIMESTAMP,
    approved_by UUID,
    
    CONSTRAINT chk_balanced CHECK (total_debit = total_credit),
    CONSTRAINT chk_status CHECK (status IN ('draft', 'submitted', 'approved', 'posted', 'void'))
);

CREATE INDEX idx_journal_headers_number ON journal_headers(journal_number);
CREATE INDEX idx_journal_headers_date ON journal_headers(journal_date);
CREATE INDEX idx_journal_headers_type ON journal_headers(journal_type);
CREATE INDEX idx_journal_headers_period ON journal_headers(period_year, period_month);
CREATE INDEX idx_journal_headers_status ON journal_headers(status);
CREATE INDEX idx_journal_headers_reference ON journal_headers(reference_type, reference_id);

CREATE TRIGGER update_journal_headers_updated_at BEFORE UPDATE ON journal_headers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Partition by year
CREATE TABLE journal_headers (
    -- columns as above
) PARTITION BY RANGE (period_year);
```

#### 3.2.2 Journal Details

```sql
CREATE TABLE journal_details (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    journal_header_id UUID NOT NULL REFERENCES journal_headers(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    
    -- Account
    account_id UUID NOT NULL REFERENCES coa_accounts(id),
    
    -- Amounts
    debit_amount DECIMAL(20,2) DEFAULT 0,
    credit_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Multi-Currency
    currency_code VARCHAR(3) DEFAULT 'IDR',
    exchange_rate DECIMAL(20,6) DEFAULT 1,
    debit_amount_base DECIMAL(20,2) DEFAULT 0, -- In base currency
    credit_amount_base DECIMAL(20,2) DEFAULT 0,
    
    -- Cost Allocation
    cost_center_id UUID,
    project_id UUID,
    department_id UUID,
    
    -- Description
    description TEXT,
    notes TEXT,
    
    -- Tax
    tax_id UUID,
    tax_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT chk_debit_or_credit CHECK (
        (debit_amount > 0 AND credit_amount = 0) OR 
        (debit_amount = 0 AND credit_amount > 0)
    )
);

CREATE INDEX idx_journal_details_header ON journal_details(journal_header_id);
CREATE INDEX idx_journal_details_account ON journal_details(account_id);
CREATE INDEX idx_journal_details_cost_center ON journal_details(cost_center_id);
CREATE INDEX idx_journal_details_project ON journal_details(project_id);

-- Partition by header's period_year (requires partitioning strategy)
```

#### 3.2.3 Fiscal Periods

```sql
CREATE TABLE fiscal_periods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    fiscal_year INTEGER NOT NULL,
    period_number INTEGER NOT NULL CHECK (period_number BETWEEN 1 AND 12),
    period_name VARCHAR(50) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR(20) DEFAULT 'open', -- open, closed, locked
    closed_at TIMESTAMP,
    closed_by UUID,
    
    CONSTRAINT uq_fiscal_period UNIQUE(fiscal_year, period_number),
    CONSTRAINT chk_period_dates CHECK (end_date > start_date),
    CONSTRAINT chk_period_status CHECK (status IN ('open', 'closed', 'locked'))
);

CREATE INDEX idx_fiscal_periods_year ON fiscal_periods(fiscal_year);
CREATE INDEX idx_fiscal_periods_status ON fiscal_periods(status);
```

### 3.3 Accounts Receivable Module

#### 3.3.1 Customers Table

```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Customer Code & Name
    customer_code VARCHAR(50) UNIQUE NOT NULL,
    customer_name VARCHAR(255) NOT NULL,
    legal_name VARCHAR(255),
    
    -- Tax & Registration
    tax_id VARCHAR(50), -- NPWP for Indonesia
    tax_id_type VARCHAR(50),
    business_registration_number VARCHAR(100),
    
    -- Contact Information
    email VARCHAR(255),
    phone VARCHAR(50),
    mobile VARCHAR(50),
    fax VARCHAR(50),
    website VARCHAR(255),
    
    -- Address (Primary)
    billing_address TEXT,
    billing_city VARCHAR(100),
    billing_state VARCHAR(100),
    billing_postal_code VARCHAR(20),
    billing_country VARCHAR(100) DEFAULT 'Indonesia',
    
    shipping_address TEXT,
    shipping_city VARCHAR(100),
    shipping_state VARCHAR(100),
    shipping_postal_code VARCHAR(20),
    shipping_country VARCHAR(100) DEFAULT 'Indonesia',
    
    -- Business Information
    customer_type VARCHAR(50) DEFAULT 'INDIVIDUAL', -- INDIVIDUAL, COMPANY
    customer_category VARCHAR(50), -- RETAIL, WHOLESALE, CORPORATE, VIP
    industry VARCHAR(100),
    
    -- Financial Settings
    payment_term_id UUID,
    credit_limit DECIMAL(20,2) DEFAULT 0,
    current_balance DECIMAL(20,2) DEFAULT 0,
    price_level_id UUID,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    
    -- Sales Settings
    salesperson_id UUID,
    sales_territory_id UUID,
    
    -- Tax Settings
    is_taxable BOOLEAN DEFAULT true,
    tax_group_id UUID,
    
    -- Banking
    bank_name VARCHAR(255),
    bank_account_number VARCHAR(100),
    bank_account_name VARCHAR(255),
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    on_hold BOOLEAN DEFAULT false,
    hold_reason TEXT,
    
    -- Rating
    credit_rating VARCHAR(20), -- A, B, C, D
    customer_since DATE,
    
    -- Notes
    notes TEXT,
    internal_notes TEXT,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID
);

CREATE INDEX idx_customers_code ON customers(customer_code);
CREATE INDEX idx_customers_name ON customers(customer_name);
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_customers_tax_id ON customers(tax_id);
CREATE INDEX idx_customers_category ON customers(customer_category);
CREATE INDEX idx_customers_active ON customers(is_active);

CREATE TRIGGER update_customers_updated_at BEFORE UPDATE ON customers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

#### 3.3.2 Customer Contacts

```sql
CREATE TABLE customer_contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
    
    -- Contact Information
    contact_name VARCHAR(255) NOT NULL,
    position VARCHAR(100),
    department VARCHAR(100),
    
    email VARCHAR(255),
    phone VARCHAR(50),
    mobile VARCHAR(50),
    
    -- Status
    is_primary BOOLEAN DEFAULT false,
    is_active BOOLEAN DEFAULT true,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_customer_contacts_customer ON customer_contacts(customer_id);
CREATE INDEX idx_customer_contacts_primary ON customer_contacts(is_primary);
```

#### 3.3.3 Sales Invoices

```sql
CREATE TABLE sales_invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Invoice Number & Date
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    invoice_date DATE NOT NULL,
    due_date DATE NOT NULL,
    
    -- Customer
    customer_id UUID NOT NULL REFERENCES customers(id),
    customer_name VARCHAR(255) NOT NULL,
    customer_address TEXT,
    customer_tax_id VARCHAR(50),
    
    -- Reference
    sales_order_id UUID,
    sales_order_number VARCHAR(50),
    quotation_id UUID,
    delivery_order_id UUID,
    customer_po_number VARCHAR(100),
    
    -- Currency
    currency_code VARCHAR(3) DEFAULT 'IDR',
    exchange_rate DECIMAL(20,6) DEFAULT 1,
    
    -- Amounts
    subtotal DECIMAL(20,2) NOT NULL DEFAULT 0,
    discount_amount DECIMAL(20,2) DEFAULT 0,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    
    -- Tax
    tax_type VARCHAR(50), -- PPN, PPh, etc.
    tax_rate DECIMAL(5,2) DEFAULT 0,
    tax_amount DECIMAL(20,2) DEFAULT 0,
    tax_base DECIMAL(20,2) DEFAULT 0,
    
    -- Additional Charges
    freight_amount DECIMAL(20,2) DEFAULT 0,
    insurance_amount DECIMAL(20,2) DEFAULT 0,
    other_charges DECIMAL(20,2) DEFAULT 0,
    
    -- Total
    total_amount DECIMAL(20,2) NOT NULL DEFAULT 0,
    paid_amount DECIMAL(20,2) DEFAULT 0,
    outstanding_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Payment
    payment_term_id UUID,
    payment_method VARCHAR(50),
    
    -- Status
    status VARCHAR(20) DEFAULT 'draft', -- draft, approved, sent, partial_paid, paid, overdue, void, cancelled
    
    -- Sales Info
    salesperson_id UUID,
    commission_rate DECIMAL(5,2) DEFAULT 0,
    commission_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Shipping
    shipping_method VARCHAR(100),
    shipping_date DATE,
    shipping_address TEXT,
    tracking_number VARCHAR(100),
    
    -- Terms & Notes
    terms_and_conditions TEXT,
    notes TEXT,
    internal_notes TEXT,
    
    -- E-Faktur (Indonesian Tax Invoice)
    is_tax_invoice BOOLEAN DEFAULT false,
    tax_invoice_number VARCHAR(100),
    tax_invoice_date DATE,
    tax_invoice_status VARCHAR(50),
    
    -- Accounting
    journal_id UUID,
    gl_posted BOOLEAN DEFAULT false,
    gl_posted_at TIMESTAMP,
    
    -- Workflow
    submitted_at TIMESTAMP,
    submitted_by UUID,
    approved_at TIMESTAMP,
    approved_by UUID,
    sent_at TIMESTAMP,
    sent_by UUID,
    void_at TIMESTAMP,
    void_by UUID,
    void_reason TEXT,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID
);

CREATE INDEX idx_sales_invoices_number ON sales_invoices(invoice_number);
CREATE INDEX idx_sales_invoices_customer ON sales_invoices(customer_id);
CREATE INDEX idx_sales_invoices_date ON sales_invoices(invoice_date);
CREATE INDEX idx_sales_invoices_due_date ON sales_invoices(due_date);
CREATE INDEX idx_sales_invoices_status ON sales_invoices(status);
CREATE INDEX idx_sales_invoices_order ON sales_invoices(sales_order_id);

-- Partition by invoice_date year
```

#### 3.3.4 Sales Invoice Details

```sql
CREATE TABLE sales_invoice_details (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invoice_id UUID NOT NULL REFERENCES sales_invoices(id) ON DELETE CASCADE,
    line_number INTEGER NOT NULL,
    
    -- Item
    item_id UUID,
    item_code VARCHAR(100),
    item_name VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Quantity & Unit
    quantity DECIMAL(20,4) NOT NULL,
    unit_of_measure VARCHAR(20) NOT NULL,
    
    -- Pricing
    unit_price DECIMAL(20,2) NOT NULL,
    discount_amount DECIMAL(20,2) DEFAULT 0,
    discount_percent DECIMAL(5,2) DEFAULT 0,
    
    -- Tax
    tax_type VARCHAR(50),
    tax_rate DECIMAL(5,2) DEFAULT 0,
    tax_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Total
    line_total DECIMAL(20,2) NOT NULL,
    
    -- Inventory
    warehouse_id UUID,
    cost_of_goods_sold DECIMAL(20,2) DEFAULT 0,
    
    -- Additional Info
    serial_numbers TEXT[], -- Array of serial numbers
    batch_numbers TEXT[],
    
    -- Accounting
    revenue_account_id UUID,
    cogs_account_id UUID,
    
    -- Notes
    notes TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoice_details_invoice ON sales_invoice_details(invoice_id);
CREATE INDEX idx_invoice_details_item ON sales_invoice_details(item_id);
```

#### 3.3.5 Payment Receipts

```sql
CREATE TABLE payment_receipts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Receipt Information
    receipt_number VARCHAR(50) UNIQUE NOT NULL,
    receipt_date DATE NOT NULL,
    
    -- Customer
    customer_id UUID NOT NULL REFERENCES customers(id),
    customer_name VARCHAR(255) NOT NULL,
    
    -- Payment Details
    payment_method VARCHAR(50) NOT NULL, -- CASH, BANK_TRANSFER, CHECK, CREDIT_CARD, etc.
    payment_amount DECIMAL(20,2) NOT NULL,
    
    -- Currency
    currency_code VARCHAR(3) DEFAULT 'IDR',
    exchange_rate DECIMAL(20,6) DEFAULT 1,
    payment_amount_base DECIMAL(20,2), -- In base currency
    
    -- Bank Details (for non-cash)
    bank_account_id UUID,
    bank_reference VARCHAR(100),
    check_number VARCHAR(50),
    check_date DATE,
    
    -- Allocation
    allocated_amount DECIMAL(20,2) DEFAULT 0,
    unallocated_amount DECIMAL(20,2) DEFAULT 0,
    
    -- Status
    status VARCHAR(20) DEFAULT 'draft', -- draft, posted, void, bounced (for checks)
    
    -- Accounting
    cash_account_id UUID,
    journal_id UUID,
    gl_posted BOOLEAN DEFAULT false,
    
    -- Notes
    notes TEXT,
    internal_notes TEXT,
    
    -- Workflow
    posted_at TIMESTAMP,
    posted_by UUID,
    void_at TIMESTAMP,
    void_by UUID,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID
);

CREATE INDEX idx_payment_receipts_number ON payment_receipts(receipt_number);
CREATE INDEX idx_payment_receipts_customer ON payment_receipts(customer_id);
CREATE INDEX idx_payment_receipts_date ON payment_receipts(receipt_date);
CREATE INDEX idx_payment_receipts_status ON payment_receipts(status);
```

#### 3.3.6 Payment Allocations

```sql
CREATE TABLE payment_allocations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    payment_receipt_id UUID NOT NULL REFERENCES payment_receipts(id) ON DELETE CASCADE,
    invoice_id UUID NOT NULL REFERENCES sales_invoices(id),
    
    allocated_amount DECIMAL(20,2) NOT NULL,
    discount_amount DECIMAL(20,2) DEFAULT 0,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID
);

CREATE INDEX idx_payment_allocations_receipt ON payment_allocations(payment_receipt_id);
CREATE INDEX idx_payment_allocations_invoice ON payment_allocations(invoice_id);
```

### 3.4 Inventory Management Module

#### 3.4.1 Inventory Items

```sql
CREATE TABLE inventory_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Item Identification
    sku VARCHAR(100) UNIQUE NOT NULL,
    item_name VARCHAR(255) NOT NULL,
    alternate_name VARCHAR(255),
    description TEXT,
    
    -- Classification
    item_type VARCHAR(50) NOT NULL, -- STOCK, NON_STOCK, SERVICE, ASSEMBLY
    category_id UUID,
    subcategory_id UUID,
    brand VARCHAR(100),
    manufacturer VARCHAR(255),
    
    -- Units
    base_unit VARCHAR(20) NOT NULL,
    purchase_unit VARCHAR(20),
    sales_unit VARCHAR(20),
    
    -- Barcodes
    barcode VARCHAR(100),
    barcode_type VARCHAR(20), -- EAN13, CODE128, QR, etc.
    alternate_barcodes TEXT[],
    
    -- Pricing
    purchase_price DECIMAL(20,2) DEFAULT 0,
    selling_price DECIMAL(20,2) DEFAULT 0,
    minimum_selling_price DECIMAL(20,2) DEFAULT 0,
    
    -- Costing
    cost_method VARCHAR(20) DEFAULT 'FIFO', -- FIFO, AVERAGE, STANDARD
    standard_cost DECIMAL(20,2) DEFAULT 0,
    last_cost DECIMAL(20,2) DEFAULT 0,
    average_cost DECIMAL(20,2) DEFAULT 0,
    
    -- Tax
    is_taxable BOOLEAN DEFAULT true,
    tax_type VARCHAR(50),
    tax_rate DECIMAL(5,2) DEFAULT 0,
    
    -- Inventory Tracking
    track_inventory BOOLEAN DEFAULT true,
    track_serial_numbers BOOLEAN DEFAULT false,
    track_batch_numbers BOOLEAN DEFAULT false,
    track_expiry_date BOOLEAN DEFAULT false,
    
    -- Stock Levels
    reorder_point DECIMAL(20,4) DEFAULT 0,
    minimum_stock DECIMAL(20,4) DEFAULT 0,
    maximum_stock DECIMAL(20,4) DEFAULT 0,
    safety_stock DECIMAL(20,4) DEFAULT 0,
    
    -- Physical Properties
    weight DECIMAL(20,4),
    weight_unit VARCHAR(20),
    length DECIMAL(20,4),
    width DECIMAL(20,4),
    height DECIMAL(20,4),
    dimension_unit VARCHAR(20),
    volume DECIMAL(20,4),
    volume_unit VARCHAR(20),
    
    -- Images & Documents
    primary_image_url TEXT,
    image_urls TEXT[],
    document_urls TEXT[],
    
    -- Accounting
    inventory_account_id UUID,
    cogs_account_id UUID,
    revenue_account_id UUID,
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_sellable BOOLEAN DEFAULT true,
    is_purchasable BOOLEAN DEFAULT true,
    discontinued_at DATE,
    
    -- Additional Info
    warranty_period INTEGER, -- in days
    warranty_terms TEXT,
    notes TEXT,
    specifications JSONB,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID
);

CREATE INDEX idx_items_sku ON inventory_items(sku);
CREATE INDEX idx_items_name ON inventory_items(item_name);
CREATE INDEX idx_items_type ON inventory_items(item_type);
CREATE INDEX idx_items_category ON inventory_items(category_id);
CREATE INDEX idx_items_barcode ON inventory_items(barcode);
CREATE INDEX idx_items_active ON inventory_items(is_active);

-- Full-text search index
CREATE INDEX idx_items_search ON inventory_items USING GIN(
    to_tsvector('simple', coalesce(item_name,'') || ' ' || coalesce(description,'') || ' ' || coalesce(sku,''))
);
```

#### 3.4.2 Warehouses

```sql
CREATE TABLE warehouses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    warehouse_code VARCHAR(50) UNIQUE NOT NULL,
    warehouse_name VARCHAR(255) NOT NULL,
    
    -- Type
    warehouse_type VARCHAR(50) DEFAULT 'MAIN', -- MAIN, BRANCH, CONSIGNMENT, VIRTUAL, TRANSIT
    
    -- Location
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) DEFAULT 'Indonesia',
    
    -- Contact
    phone VARCHAR(50),
    email VARCHAR(255),
    manager_name VARCHAR(255),
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_default BOOLEAN DEFAULT false,
    
    -- Configuration
    allow_negative_stock BOOLEAN DEFAULT false,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID
);

CREATE INDEX idx_warehouses_code ON warehouses(warehouse_code);
CREATE INDEX idx_warehouses_active ON warehouses(is_active);
```

#### 3.4.3 Stock Transactions

```sql
CREATE TABLE stock_transactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Transaction Identification
    transaction_number VARCHAR(50) UNIQUE NOT NULL,
    transaction_date DATE NOT NULL,
    transaction_type VARCHAR(50) NOT NULL, -- IN, OUT, ADJUSTMENT, TRANSFER, ASSEMBLY, DISASSEMBLY
    sub_type VARCHAR(50), -- PURCHASE, SALES, RETURN, etc.
    
    -- Reference
    reference_type VARCHAR(50),
    reference_number VARCHAR(100),
    reference_id UUID,
    
    -- Item & Location
    item_id UUID NOT NULL REFERENCES inventory_items(id),
    warehouse_id UUID NOT NULL REFERENCES warehouses(id),
    bin_location VARCHAR(100),
    
    -- Quantity
    quantity DECIMAL(20,4) NOT NULL,
    unit_of_measure VARCHAR(20) NOT NULL,
    
    -- Costing
    unit_cost DECIMAL(20,2) DEFAULT 0,
    total_cost DECIMAL(20,2) DEFAULT 0,
    
    -- Batch & Serial
    batch_number VARCHAR(100),
    serial_number VARCHAR(100),
    expiry_date DATE,
    
    -- Running Balance
    quantity_before DECIMAL(20,4),
    quantity_after DECIMAL(20,4),
    
    -- Status
    status VARCHAR(20) DEFAULT 'completed', -- draft, completed, void
    
    -- Notes
    notes TEXT,
    
    -- Audit
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID
);

CREATE INDEX idx_stock_trans_number ON stock_transactions(transaction_number);
CREATE INDEX idx_stock_trans_item ON stock_transactions(item_id);
CREATE INDEX idx_stock_trans_warehouse ON stock_transactions(warehouse_id);
CREATE INDEX idx_stock_trans_date ON stock_transactions(transaction_date);
CREATE INDEX idx_stock_trans_type ON stock_transactions(transaction_type);
CREATE INDEX idx_stock_trans_reference ON stock_transactions(reference_type, reference_id);

-- Partition by transaction_date year
CREATE TABLE stock_transactions (
    -- columns as above
) PARTITION BY RANGE (EXTRACT(YEAR FROM transaction_date));
```

#### 3.4.4 Stock Balance (View)

```sql
CREATE OR REPLACE VIEW stock_balance_view AS
SELECT 
    item_id,
    warehouse_id,
    SUM(CASE 
        WHEN transaction_type IN ('IN', 'ADJUSTMENT') THEN quantity
        WHEN transaction_type IN ('OUT') THEN -quantity
        ELSE 0 
    END) as quantity_on_hand,
    MAX(transaction_date) as last_transaction_date,
    COUNT(*) as transaction_count
FROM stock_transactions
WHERE status = 'completed'
GROUP BY item_id, warehouse_id;

-- Materialized view for better performance
CREATE MATERIALIZED VIEW stock_balance_mv AS
SELECT * FROM stock_balance_view;

CREATE UNIQUE INDEX idx_stock_balance_mv_item_wh 
ON stock_balance_mv(item_id, warehouse_id);

-- Refresh command
REFRESH MATERIALIZED VIEW CONCURRENTLY stock_balance_mv;
```

---

## 4. INDEXES & PERFORMANCE

### 4.1 Indexing Strategy

**Primary Keys**: Automatic B-tree index
**Foreign Keys**: Create non-clustered index
**Frequently Queried Columns**: Composite indexes
**Date Columns**: B-tree indexes
**Text Search**: GIN or GiST indexes

### 4.2 Composite Indexes

```sql
-- Example composite indexes
CREATE INDEX idx_journal_details_header_account 
ON journal_details(journal_header_id, account_id);

CREATE INDEX idx_sales_invoices_customer_date_status 
ON sales_invoices(customer_id, invoice_date DESC, status);

CREATE INDEX idx_stock_trans_item_warehouse_date 
ON stock_transactions(item_id, warehouse_id, transaction_date DESC);
```

### 4.3 Partial Indexes

```sql
-- Only index active records
CREATE INDEX idx_customers_active_code 
ON customers(customer_code) WHERE is_active = true;

CREATE INDEX idx_invoices_unpaid 
ON sales_invoices(customer_id, due_date) 
WHERE status NOT IN ('paid', 'void');
```

---

## 5. DATA INTEGRITY & CONSTRAINTS

### 5.1 Referential Integrity

All foreign keys use ON DELETE CASCADE or ON DELETE RESTRICT appropriately.

### 5.2 Check Constraints

```sql
-- Ensure dates are logical
ALTER TABLE sales_invoices 
ADD CONSTRAINT chk_invoice_dates 
CHECK (due_date >= invoice_date);

-- Ensure amounts are non-negative
ALTER TABLE sales_invoice_details 
ADD CONSTRAINT chk_positive_amounts 
CHECK (quantity > 0 AND unit_price >= 0);
```

### 5.3 Triggers for Data Integrity

```sql
-- Auto-update invoice totals
CREATE OR REPLACE FUNCTION update_invoice_totals()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE sales_invoices
    SET subtotal = (
        SELECT COALESCE(SUM(line_total), 0)
        FROM sales_invoice_details
        WHERE invoice_id = NEW.invoice_id
    )
    WHERE id = NEW.invoice_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_invoice_totals
AFTER INSERT OR UPDATE OR DELETE ON sales_invoice_details
FOR EACH ROW EXECUTE FUNCTION update_invoice_totals();
```

---

## 6. BACKUP & RECOVERY

### 6.1 Backup Strategy

**Full Backup**: Daily at 2:00 AM
**Incremental Backup**: Every 6 hours
**WAL Archiving**: Continuous
**Retention**: 30 days

### 6.2 Backup Commands

```bash
# Full backup
pg_dump -U postgres -Fc company_001_db > company_001_backup.dump

# Restore
pg_restore -U postgres -d company_001_db company_001_backup.dump

# WAL archiving
archive_command = 'cp %p /backup/wal/%f'
```

---

## 7. DATABASE MAINTENANCE

### 7.1 Regular Maintenance Tasks

```sql
-- Vacuum and analyze
VACUUM ANALYZE;

-- Reindex
REINDEX DATABASE company_001_db;

-- Update statistics
ANALYZE;
```

### 7.2 Monitoring Queries

```sql
-- Check database size
SELECT pg_size_pretty(pg_database_size('company_001_db'));

-- Check table sizes
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Check slow queries
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

---

## 8. MIGRATION STRATEGY

### 8.1 Schema Version Control

Use migration tools like:
- Flyway
- Liquibase
- Node-pg-migrate

### 8.2 Migration Template

```sql
-- V1.0.1__Add_customer_rating.sql
BEGIN;

ALTER TABLE customers ADD COLUMN credit_rating VARCHAR(20);
ALTER TABLE customers ADD COLUMN customer_since DATE;

CREATE INDEX idx_customers_rating ON customers(credit_rating);

COMMIT;
```

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Next Review**: May 2026
