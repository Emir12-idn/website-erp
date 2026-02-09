# API Documentation
# Accurate Enterprise 5 - Clone System

## Document Information
- **Project Name**: Accurate Enterprise 5 Clone
- **API Version**: v1
- **Date**: February 2026
- **Base URL**: `https://api.yourdomain.com/api/v1`

---

## 1. AUTHENTICATION

### 1.1 Login

**Endpoint**: `POST /auth/login`

**Request Body**:
```json
{
  "username": "john.doe",
  "password": "SecurePassword123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 900,
    "user": {
      "id": "uuid",
      "username": "john.doe",
      "email": "john@example.com",
      "full_name": "John Doe"
    }
  }
}
```

### 1.2 Refresh Token

**Endpoint**: `POST /auth/refresh`

**Request Body**:
```json
{
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 1.3 Logout

**Endpoint**: `POST /auth/logout`

**Headers**: `Authorization: Bearer {access_token}`

---

## 2. COMPANY MANAGEMENT

### 2.1 Get Companies

**Endpoint**: `GET /companies`

**Query Parameters**:
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20)
- `status` (string): Filter by status (active, suspended)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "company_code": "COMP001",
        "company_name": "PT. Example Indonesia",
        "subscription_plan": "professional",
        "subscription_status": "active",
        "created_at": "2026-01-15T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "total_pages": 5
    }
  }
}
```

### 2.2 Create Company

**Endpoint**: `POST /companies`

**Request Body**:
```json
{
  "company_code": "COMP002",
  "company_name": "PT. New Company",
  "tax_id": "01.234.567.8-901.000",
  "email": "info@newcompany.com",
  "phone": "+62-21-1234567",
  "address": "Jl. Sudirman No. 123",
  "city": "Jakarta",
  "subscription_plan": "professional"
}
```

### 2.3 Switch Company

**Endpoint**: `POST /companies/{company_id}/switch`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "company_id": "uuid",
    "company_name": "PT. Example Indonesia",
    "access_token": "new_token_with_company_context"
  }
}
```

---

## 3. CHART OF ACCOUNTS

### 3.1 List Accounts

**Endpoint**: `GET /accounting/accounts`

**Query Parameters**:
- `account_type` (string): ASSET, LIABILITY, EQUITY, REVENUE, EXPENSE
- `is_active` (boolean): true/false
- `parent_id` (uuid): Filter by parent account
- `level` (integer): Filter by level

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "account_code": "1-10000",
      "account_name": "Cash in Bank",
      "account_type": "ASSET",
      "parent_id": null,
      "level": 1,
      "normal_balance": "DEBIT",
      "current_balance": 50000000.00,
      "is_active": true
    }
  ]
}
```

### 3.2 Create Account

**Endpoint**: `POST /accounting/accounts`

**Request Body**:
```json
{
  "account_code": "1-10100",
  "account_name": "Bank BCA",
  "account_type": "ASSET",
  "parent_id": "uuid_of_parent",
  "normal_balance": "DEBIT",
  "currency_code": "IDR",
  "allow_manual_entry": true
}
```

### 3.3 Get Account Balance

**Endpoint**: `GET /accounting/accounts/{account_id}/balance`

**Query Parameters**:
- `start_date` (date): YYYY-MM-DD
- `end_date` (date): YYYY-MM-DD

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "account_id": "uuid",
    "account_code": "1-10000",
    "account_name": "Cash in Bank",
    "opening_balance": 45000000.00,
    "total_debit": 15000000.00,
    "total_credit": 10000000.00,
    "closing_balance": 50000000.00,
    "currency_code": "IDR"
  }
}
```

---

## 4. GENERAL LEDGER

### 4.1 Create Journal Entry

**Endpoint**: `POST /accounting/journals`

**Request Body**:
```json
{
  "journal_date": "2026-02-09",
  "journal_type": "GENERAL",
  "description": "Monthly depreciation expense",
  "details": [
    {
      "line_number": 1,
      "account_id": "uuid_expense_account",
      "description": "Depreciation - Equipment",
      "debit_amount": 5000000.00,
      "credit_amount": 0
    },
    {
      "line_number": 2,
      "account_id": "uuid_accumulated_depreciation",
      "description": "Accumulated Depreciation",
      "debit_amount": 0,
      "credit_amount": 5000000.00
    }
  ]
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "journal_number": "GJ/2026/02/00001",
    "journal_date": "2026-02-09",
    "total_debit": 5000000.00,
    "total_credit": 5000000.00,
    "status": "draft"
  }
}
```

### 4.2 Post Journal Entry

**Endpoint**: `POST /accounting/journals/{journal_id}/post`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "journal_number": "GJ/2026/02/00001",
    "status": "posted",
    "posted_at": "2026-02-09T15:30:00Z",
    "posted_by": "uuid_of_user"
  }
}
```

### 4.3 Get Journal Entry

**Endpoint**: `GET /accounting/journals/{journal_id}`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "journal_number": "GJ/2026/02/00001",
    "journal_date": "2026-02-09",
    "journal_type": "GENERAL",
    "description": "Monthly depreciation expense",
    "total_debit": 5000000.00,
    "total_credit": 5000000.00,
    "status": "posted",
    "details": [
      {
        "line_number": 1,
        "account_code": "6-10100",
        "account_name": "Depreciation Expense",
        "debit_amount": 5000000.00,
        "credit_amount": 0
      },
      {
        "line_number": 2,
        "account_code": "1-30200",
        "account_name": "Accumulated Depreciation",
        "debit_amount": 0,
        "credit_amount": 5000000.00
      }
    ]
  }
}
```

### 4.4 List Journal Entries

**Endpoint**: `GET /accounting/journals`

**Query Parameters**:
- `start_date` (date): Filter from date
- `end_date` (date): Filter to date
- `journal_type` (string): GENERAL, SALES, PURCHASE, etc.
- `status` (string): draft, posted, void
- `page` (integer)
- `limit` (integer)

---

## 5. CUSTOMERS

### 5.1 List Customers

**Endpoint**: `GET /customers`

**Query Parameters**:
- `search` (string): Search by name, code, email
- `customer_type` (string): INDIVIDUAL, COMPANY
- `is_active` (boolean)
- `page` (integer)
- `limit` (integer)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "customer_code": "CUST001",
        "customer_name": "PT. ABC Indonesia",
        "email": "info@abc.co.id",
        "phone": "+62-21-9876543",
        "customer_type": "COMPANY",
        "credit_limit": 100000000.00,
        "current_balance": 25000000.00,
        "is_active": true
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150
    }
  }
}
```

### 5.2 Create Customer

**Endpoint**: `POST /customers`

**Request Body**:
```json
{
  "customer_code": "CUST002",
  "customer_name": "PT. XYZ Corporation",
  "legal_name": "PT. XYZ Corporation Tbk",
  "tax_id": "01.234.567.8-901.000",
  "email": "finance@xyz.co.id",
  "phone": "+62-21-5551234",
  "customer_type": "COMPANY",
  "customer_category": "CORPORATE",
  "billing_address": "Jl. Gatot Subroto No. 45",
  "billing_city": "Jakarta",
  "billing_country": "Indonesia",
  "credit_limit": 50000000.00,
  "payment_term_id": "uuid"
}
```

### 5.3 Get Customer Details

**Endpoint**: `GET /customers/{customer_id}`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "customer_code": "CUST001",
    "customer_name": "PT. ABC Indonesia",
    "tax_id": "01.234.567.8-901.000",
    "email": "info@abc.co.id",
    "phone": "+62-21-9876543",
    "billing_address": "Jl. Sudirman No. 123",
    "billing_city": "Jakarta",
    "credit_limit": 100000000.00,
    "current_balance": 25000000.00,
    "outstanding_invoices": 3,
    "overdue_amount": 5000000.00,
    "is_active": true
  }
}
```

### 5.4 Update Customer

**Endpoint**: `PUT /customers/{customer_id}`

### 5.5 Get Customer Statement

**Endpoint**: `GET /customers/{customer_id}/statement`

**Query Parameters**:
- `start_date` (date)
- `end_date` (date)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "customer": {
      "id": "uuid",
      "customer_name": "PT. ABC Indonesia"
    },
    "opening_balance": 15000000.00,
    "transactions": [
      {
        "date": "2026-02-01",
        "transaction_type": "INVOICE",
        "reference": "INV/2026/02/001",
        "debit": 10000000.00,
        "credit": 0,
        "balance": 25000000.00
      },
      {
        "date": "2026-02-05",
        "transaction_type": "PAYMENT",
        "reference": "RCP/2026/02/001",
        "debit": 0,
        "credit": 5000000.00,
        "balance": 20000000.00
      }
    ],
    "closing_balance": 20000000.00
  }
}
```

---

## 6. SALES INVOICES

### 6.1 Create Sales Invoice

**Endpoint**: `POST /sales/invoices`

**Request Body**:
```json
{
  "invoice_date": "2026-02-09",
  "due_date": "2026-03-09",
  "customer_id": "uuid",
  "customer_po_number": "PO-2026-001",
  "sales_order_id": "uuid",
  "currency_code": "IDR",
  "payment_term_id": "uuid",
  "tax_type": "PPN",
  "tax_rate": 11.00,
  "details": [
    {
      "line_number": 1,
      "item_id": "uuid",
      "description": "Product ABC",
      "quantity": 10,
      "unit_of_measure": "PCS",
      "unit_price": 500000.00,
      "discount_percent": 5.00,
      "tax_rate": 11.00,
      "warehouse_id": "uuid"
    },
    {
      "line_number": 2,
      "item_id": "uuid",
      "description": "Product XYZ",
      "quantity": 5,
      "unit_of_measure": "PCS",
      "unit_price": 750000.00,
      "tax_rate": 11.00,
      "warehouse_id": "uuid"
    }
  ],
  "notes": "Thank you for your business",
  "terms_and_conditions": "Payment within 30 days"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "invoice_number": "INV/2026/02/00001",
    "invoice_date": "2026-02-09",
    "due_date": "2026-03-09",
    "customer_name": "PT. ABC Indonesia",
    "subtotal": 8500000.00,
    "discount_amount": 250000.00,
    "tax_amount": 907500.00,
    "total_amount": 9157500.00,
    "status": "draft"
  }
}
```

### 6.2 Get Sales Invoice

**Endpoint**: `GET /sales/invoices/{invoice_id}`

### 6.3 List Sales Invoices

**Endpoint**: `GET /sales/invoices`

**Query Parameters**:
- `customer_id` (uuid)
- `status` (string): draft, approved, sent, partial_paid, paid, overdue
- `start_date` (date)
- `end_date` (date)
- `page`, `limit`

### 6.4 Approve Sales Invoice

**Endpoint**: `POST /sales/invoices/{invoice_id}/approve`

### 6.5 Send Sales Invoice

**Endpoint**: `POST /sales/invoices/{invoice_id}/send`

**Request Body**:
```json
{
  "recipient_email": "finance@customer.com",
  "cc_emails": ["accounting@customer.com"],
  "subject": "Invoice INV/2026/02/00001",
  "message": "Please find attached your invoice"
}
```

### 6.6 Void Sales Invoice

**Endpoint**: `POST /sales/invoices/{invoice_id}/void`

**Request Body**:
```json
{
  "void_reason": "Customer requested cancellation"
}
```

---

## 7. PAYMENT RECEIPTS

### 7.1 Create Payment Receipt

**Endpoint**: `POST /sales/payments`

**Request Body**:
```json
{
  "receipt_date": "2026-02-09",
  "customer_id": "uuid",
  "payment_method": "BANK_TRANSFER",
  "payment_amount": 10000000.00,
  "bank_account_id": "uuid",
  "bank_reference": "TRF/2026/02/12345",
  "allocations": [
    {
      "invoice_id": "uuid",
      "allocated_amount": 9157500.00,
      "discount_amount": 0
    },
    {
      "invoice_id": "uuid",
      "allocated_amount": 842500.00,
      "discount_amount": 0
    }
  ],
  "notes": "Payment received via bank transfer"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "receipt_number": "RCP/2026/02/00001",
    "receipt_date": "2026-02-09",
    "customer_name": "PT. ABC Indonesia",
    "payment_amount": 10000000.00,
    "allocated_amount": 10000000.00,
    "unallocated_amount": 0,
    "status": "draft"
  }
}
```

### 7.2 Post Payment Receipt

**Endpoint**: `POST /sales/payments/{payment_id}/post`

---

## 8. INVENTORY

### 8.1 List Items

**Endpoint**: `GET /inventory/items`

**Query Parameters**:
- `search` (string): Search by SKU, name
- `category_id` (uuid)
- `item_type` (string): STOCK, NON_STOCK, SERVICE
- `is_active` (boolean)
- `page`, `limit`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "sku": "PROD-001",
        "item_name": "Product ABC",
        "item_type": "STOCK",
        "category_name": "Electronics",
        "base_unit": "PCS",
        "selling_price": 500000.00,
        "quantity_on_hand": 150,
        "is_active": true
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 500
    }
  }
}
```

### 8.2 Create Item

**Endpoint**: `POST /inventory/items`

**Request Body**:
```json
{
  "sku": "PROD-002",
  "item_name": "Product XYZ",
  "description": "High quality product",
  "item_type": "STOCK",
  "category_id": "uuid",
  "base_unit": "PCS",
  "barcode": "1234567890123",
  "purchase_price": 400000.00,
  "selling_price": 750000.00,
  "cost_method": "FIFO",
  "track_inventory": true,
  "track_serial_numbers": false,
  "reorder_point": 20,
  "minimum_stock": 10,
  "is_active": true
}
```

### 8.3 Get Item Stock

**Endpoint**: `GET /inventory/items/{item_id}/stock`

**Query Parameters**:
- `warehouse_id` (uuid): Optional, filter by warehouse

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "item_id": "uuid",
    "sku": "PROD-001",
    "item_name": "Product ABC",
    "warehouses": [
      {
        "warehouse_id": "uuid",
        "warehouse_name": "Main Warehouse",
        "quantity_on_hand": 120,
        "committed_quantity": 20,
        "available_quantity": 100,
        "last_transaction_date": "2026-02-08"
      },
      {
        "warehouse_id": "uuid",
        "warehouse_name": "Branch Warehouse",
        "quantity_on_hand": 30,
        "committed_quantity": 5,
        "available_quantity": 25,
        "last_transaction_date": "2026-02-07"
      }
    ],
    "total_on_hand": 150,
    "total_available": 125
  }
}
```

### 8.4 Stock Adjustment

**Endpoint**: `POST /inventory/adjustments`

**Request Body**:
```json
{
  "adjustment_date": "2026-02-09",
  "warehouse_id": "uuid",
  "reason": "Physical count adjustment",
  "details": [
    {
      "item_id": "uuid",
      "quantity": 5,
      "adjustment_type": "INCREASE",
      "unit_cost": 400000.00,
      "notes": "Found extra units during count"
    },
    {
      "item_id": "uuid",
      "quantity": 2,
      "adjustment_type": "DECREASE",
      "notes": "Damaged units"
    }
  ]
}
```

### 8.5 Stock Transfer

**Endpoint**: `POST /inventory/transfers`

**Request Body**:
```json
{
  "transfer_date": "2026-02-09",
  "from_warehouse_id": "uuid",
  "to_warehouse_id": "uuid",
  "details": [
    {
      "item_id": "uuid",
      "quantity": 10,
      "unit_of_measure": "PCS"
    }
  ],
  "notes": "Transfer to branch warehouse"
}
```

---

## 9. REPORTING

### 9.1 Balance Sheet

**Endpoint**: `GET /reports/balance-sheet`

**Query Parameters**:
- `as_of_date` (date): YYYY-MM-DD (default: today)
- `comparison_date` (date): Optional for comparison

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "report_date": "2026-02-09",
    "company_name": "PT. Example Indonesia",
    "assets": {
      "current_assets": {
        "cash_and_bank": 50000000.00,
        "accounts_receivable": 75000000.00,
        "inventory": 120000000.00,
        "total": 245000000.00
      },
      "fixed_assets": {
        "equipment": 150000000.00,
        "accumulated_depreciation": -30000000.00,
        "total": 120000000.00
      },
      "total_assets": 365000000.00
    },
    "liabilities": {
      "current_liabilities": {
        "accounts_payable": 45000000.00,
        "tax_payable": 15000000.00,
        "total": 60000000.00
      },
      "total_liabilities": 60000000.00
    },
    "equity": {
      "capital": 250000000.00,
      "retained_earnings": 55000000.00,
      "total_equity": 305000000.00
    }
  }
}
```

### 9.2 Profit & Loss

**Endpoint**: `GET /reports/profit-loss`

**Query Parameters**:
- `start_date` (date)
- `end_date` (date)
- `comparison_period` (string): previous_month, previous_year

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "period": "January 2026",
    "revenue": {
      "sales_revenue": 250000000.00,
      "other_revenue": 5000000.00,
      "total_revenue": 255000000.00
    },
    "cost_of_goods_sold": {
      "cogs": 150000000.00,
      "gross_profit": 105000000.00
    },
    "operating_expenses": {
      "salaries": 30000000.00,
      "rent": 10000000.00,
      "utilities": 5000000.00,
      "depreciation": 5000000.00,
      "total": 50000000.00
    },
    "operating_income": 55000000.00,
    "other_income_expense": {
      "interest_income": 1000000.00,
      "interest_expense": -500000.00,
      "total": 500000.00
    },
    "net_income_before_tax": 55500000.00,
    "tax_expense": -5550000.00,
    "net_income": 49950000.00
  }
}
```

### 9.3 AR Aging Report

**Endpoint**: `GET /reports/ar-aging`

**Query Parameters**:
- `as_of_date` (date)
- `customer_id` (uuid): Optional

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "as_of_date": "2026-02-09",
    "customers": [
      {
        "customer_id": "uuid",
        "customer_name": "PT. ABC Indonesia",
        "current": 10000000.00,
        "days_1_30": 5000000.00,
        "days_31_60": 3000000.00,
        "days_61_90": 2000000.00,
        "over_90": 5000000.00,
        "total_outstanding": 25000000.00
      }
    ],
    "summary": {
      "current": 50000000.00,
      "days_1_30": 25000000.00,
      "days_31_60": 15000000.00,
      "days_61_90": 10000000.00,
      "over_90": 20000000.00,
      "total": 120000000.00
    }
  }
}
```

### 9.4 Sales Report

**Endpoint**: `GET /reports/sales`

**Query Parameters**:
- `start_date` (date)
- `end_date` (date)
- `group_by` (string): customer, item, salesperson, date
- `customer_id` (uuid): Optional
- `item_id` (uuid): Optional

---

## 10. ERROR HANDLING

### Standard Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "phone",
        "message": "Phone number is required"
      }
    ]
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| AUTHENTICATION_ERROR | 401 | Invalid or missing authentication |
| PERMISSION_DENIED | 403 | User lacks required permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource conflict (duplicate, etc.) |
| INTERNAL_ERROR | 500 | Internal server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |

---

## 11. RATE LIMITING

- **Rate Limit**: 100 requests per 15 minutes per IP
- **Headers**:
  - `X-RateLimit-Limit`: Total allowed requests
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Unix timestamp when limit resets

**Example Response** (429 Too Many Requests):
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retry_after": 900
  }
}
```

---

## 12. PAGINATION

All list endpoints support pagination:

**Query Parameters**:
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20, max: 100)

**Response Structure**:
```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 500,
      "total_pages": 25,
      "has_next": true,
      "has_prev": false
    }
  }
}
```

---

## 13. SORTING & FILTERING

### Sorting

Use `sort` parameter:
- `sort=name` (ascending)
- `sort=-name` (descending)
- `sort=name,-created_at` (multiple fields)

### Filtering

Use field-specific parameters:
- `customer_id=uuid`
- `status=active`
- `min_amount=1000000`
- `max_amount=5000000`

### Date Range

- `start_date=2026-01-01`
- `end_date=2026-01-31`

### Search

- `search=keyword` (searches across relevant fields)

---

## 14. WEBHOOKS

### Supported Events

- `invoice.created`
- `invoice.approved`
- `invoice.paid`
- `payment.received`
- `customer.created`
- `item.updated`

### Webhook Payload Example

```json
{
  "event": "invoice.paid",
  "timestamp": "2026-02-09T15:30:00Z",
  "data": {
    "id": "uuid",
    "invoice_number": "INV/2026/02/00001",
    "customer_id": "uuid",
    "total_amount": 9157500.00,
    "paid_amount": 9157500.00,
    "status": "paid"
  }
}
```

### Webhook Security

- HMAC-SHA256 signature in `X-Signature` header
- Verify signature before processing

---

## 15. API VERSIONING

- Current version: `v1`
- Version specified in URL: `/api/v1/...`
- When breaking changes needed, new version will be introduced
- Old versions supported for minimum 12 months

---

## 16. SDK & LIBRARIES

### JavaScript/Node.js

```javascript
const AccurateAPI = require('@accurate/api-client');

const client = new AccurateAPI({
  apiKey: 'your_api_key',
  baseURL: 'https://api.yourdomain.com/api/v1'
});

// Example usage
const invoices = await client.sales.invoices.list({
  status: 'unpaid',
  customer_id: 'uuid'
});
```

### Python

```python
from accurate_api import AccurateClient

client = AccurateClient(
    api_key='your_api_key',
    base_url='https://api.yourdomain.com/api/v1'
)

# Example usage
invoices = client.sales.invoices.list(
    status='unpaid',
    customer_id='uuid'
)
```

---

## APPENDICES

### Appendix A: Complete Endpoint List
### Appendix B: Postman Collection
### Appendix C: OpenAPI/Swagger Specification
### Appendix D: Code Examples

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Support**: api-support@yourdomain.com
