# 🛡️ Agentic Payroll Validation Workflow Demo

A fully autonomous, self-healing payroll data validation system built with **Make.com** and **JavaScript**. This demo showcases how intelligent automation can validate, map, and route payroll data across multiple vendor systems with minimal human intervention.

## ✨ Features

- **Autonomous Validation** - Automatically detects schema mismatches and data inconsistencies
- **Self-Healing** - Intelligently maps columns to correct vendor specifications
- **Demo-Friendly Scoring** - 95% base confidence for vendor spec data; lenient with extra fields
- **Multi-Vendor Support** - Works with Workday, ADP, Gusto, BambooHR, SAP, and more
- **Smart Routing** - Routes valid data to success path; flags issues for escalation
- **Real-Time Processing** - Instant validation and response via webhooks
- **Learning System** - Knowledge base stores solutions for common issues
- **Zero Authentication** - Demo environment; no login required

## 🚀 Quick Start

### Try the Live Form

👉 **[Open the Payroll Validation Form](https://napville2000.github.io/Agentic-Workflow-Demo/payroll-form.html)**

1. **Upload a CSV file** (sample files included below)
2. **Select a vendor source** (Workday, ADP, Gusto, etc.)
3. **Provide notification email** (optional)
4. **Click "Validate & Import"**

Results appear instantly with incident ID and validation details.

---

## 📋 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     User Form (HTML/JS)                      │
│         https://napville2000.github.io/.../payroll-form.html │
└────────────────────┬────────────────────────────────────────┘
                     │ CSV File Upload
                     ↓
┌─────────────────────────────────────────────────────────────┐
│                    Make.com Workflow                         │
├─────────────────────────────────────────────────────────────┤
│  1. Parse CSV → Array Aggregator                            │
│  2. Fetch Vendor Specs (Google Sheets)                      │
│  3. Fetch KB Articles (Google Sheets)                       │
│  4. JavaScript Validation Engine                           │
│     • Map columns by position                              │
│     • Calculate confidence score (95% base)                │
│     • Flag issues                                          │
│  5. Router: Success vs Escalation                          │
├─────────────────────────────────────────────────────────────┤
│  SUCCESS PATH                │  ESCALATION PATH            │
│  ├─ Webhook Response (200)   │  ├─ Webhook Response (400) │
│  ├─ Log to Google Sheets     │  ├─ Log to Google Sheets   │
│  └─ Email Notification       │  └─ Alert Email            │
└─────────────────────────────────────────────────────────────┘
                     ↓
            Vendor-Ready JSON
```
## 🔧 Make.com Workflow

![Make.com Workflow Architecture](https://drive.google.com/uc?export=view&id=1FILcvq6XtNlX10wkPhlUPYgejKk48fIF)

*Complete end-to-end payroll validation workflow in Make.com*
---

## 📊 Test Data

Sample CSV files are included in the repo. Each tests a different scenario:

### 1. **Valid Data** (Happy Path)
```csv
EMP001,john.smith@company.com,5000,3800,2025-06-15
EMP002,sarah.johnson@company.com,4500,3450,2025-06-15
EMP003,mike.brown@company.com,6000,4500,2025-06-15
```
**Expected:** Status = success, Confidence = 93%

### 2. **Extra Columns** (Lenient Handling)
```csv
EMP001,john.smith@company.com,5000,3800,2025-06-15,Engineering,Full-Time
EMP002,sarah.johnson@company.com,4500,3450,2025-06-15,Sales,Full-Time
```
**Expected:** Status = success, Confidence = 91% (extra columns ignored)

### 3. **Missing Columns** (Escalation)
```csv
EMP001,john.smith@company.com,5000
EMP002,sarah.johnson@company.com,4500
```
**Expected:** Status = error, Escalates to manual review

### 4. **Large Dataset** (Scale Testing)
10+ employee records to test performance with larger payloads.

### 5. **Empty File** (Error Handling)
Completely empty CSV to test error handling and fallback routing.

---

## 🔧 Core Components

### 1. **JavaScript Validation Engine** (Make Code Module)

```javascript
// Key features:
- Unwraps collection outputs
- Extracts vendor specifications dynamically
- Maps CSV columns by position
- Calculates confidence scores
- Returns structured validation result
- Includes csvRecords for downstream processing
```

**Output Format:**
```json
{
  "incident_id": "INC - 2025-12-16 08:04:18",
  "timestamp": "2025-12-16T08:04:18.330Z",
  "status": "success",
  "vendor_source": "Partner_Workday",
  "confidence_score": 93,
  "can_auto_proceed": true,
  "record_count": 5,
  "corrected_headers": {
    "col1": "PayrollEmployeeId",
    "col2": "GrossPay",
    "col3": "EmployeeEmail",
    "col4": "NetPay",
    "col5": "PaymentDate"
  },
  "csvRecords": [ /* transformed data */ ],
  "issues": [ /* any warnings */ ],
  "error": null
}
```

### 2. **Smart Router**

Routes based on validation status:

**SUCCESS ROUTE:**
- Condition: `status = "success" AND can_auto_proceed = true`
- Actions: Webhook response, logging, notifications

**ESCALATION ROUTE:**
- Condition: `status = "error" OR can_auto_proceed = false`
- Actions: Error response, escalation logging, alert email

### 3. **Data Transformation**

Uses Make.com's `map()` function to dynamically transform records:

```javascript
map(csvRecords; {
  "PayrollEmployeeId": .col1,
  "GrossPay": .col3,
  "EmployeeEmail": .col2,
  "NetPay": .col4,
  "PaymentDate": .col5
})
```

Works with ANY number of records, scalable to thousands.

---

## 🎯 Confidence Scoring Logic

### Demo Mode (Current)
```
Base Score: 95% (vendor spec data is trusted)
Extra Columns: -2 each
Escalations: -10 each (capped at 89%)
Auto-Proceed Threshold: ≥90% + status="success"
```

### Example Calculations
- 5 required fields, 5 columns: **95%** ✅
- 5 required fields, 6 columns: **93%** ✅ (1 extra × -2)
- 5 required fields, 3 columns: **ERROR** ❌ (not enough)

---

## 📚 Supported Vendors

| Vendor | Employee ID Field | Gross Pay | Email | Net Pay | Payment Date |
|--------|------------------|-----------|-------|---------|--------------|
| **Workday** | Employee_ID | Gross_Pay | Email_Address | Net_Pay | Payment_Date |
| **ADP** | empno | grosspay | email | netpay | paydate |
| **Gusto** | employee_id | gross_pay | email | net_pay | check_date |
| **BambooHR** | employeeId | grossPay | email | netPay | payDate |
| **SAP** | PERNR | BRUT | EMAIL | NETO | PDATE |

Specifications stored in public Google Sheet for easy updates.

---

## 🔐 Important Disclaimers

⚠️ **This is a DEMO environment only**

- ✋ Does NOT sync to production payroll systems
- 🔓 No authentication required
- 📊 Data stored in public Google Sheets (demonstration purposes only)
- 🧪 Use test/sample data only
- 🚫 Not intended for real payroll processing

---

## 🛠️ Tech Stack

- **Frontend:** HTML5, Vanilla JavaScript, CSS3
- **Backend:** Make.com (formerly Integromat)
- **Validation Engine:** JavaScript (Make Code module)
- **Data Storage:** Google Sheets API
- **Webhooks:** Make.com HTTP modules
- **Notifications:** Email (Make.com)

---

## 📈 Workflow Execution

### Step-by-Step Flow

1. **User uploads CSV via web form**
   - File is base64 encoded
   - Source vendor selected
   - Notification email provided (optional, defaults to napville2000@gmail.com)

2. **Make.com receives webhook**
   - Decodes CSV content
   - Converts to array of objects
   - Passes to parser

3. **CSV Parser**
   - Splits content by newlines
   - Creates col1, col2, col3... structure
   - Passes to Array Aggregator

4. **Fetch Vendor Specs**
   - Queries Google Sheets for selected vendor
   - Returns: expected fields, field names, aliases, documentation

5. **Fetch KB Articles**
   - Retrieves known issues and solutions
   - Stores for future reference

6. **JavaScript Validation**
   - Extracts headers from CSV
   - Maps to vendor spec by position
   - Calculates confidence score
   - Transforms data structure
   - Returns validation result

7. **Router Decision**
   - If success: continues to success path
   - If error: routes to escalation path

8. **Success Path (Valid Data)**
   - Sends HTTP 200 response with formatted JSON
   - Logs to "Success Log" sheet
   - Sends confirmation email
   - Data ready for vendor import

9. **Escalation Path (Issues)**
   - Sends HTTP 400 error response
   - Logs to "Escalation Log" sheet
   - Sends alert email to team
   - Marks for manual review

---

## 📝 Response Examples

### Success Response (HTTP 200)
```json
{
  "incident_id": "INC - 2025-12-16 08:04:18",
  "status": "success",
  "confidence_score": 93,
  "can_auto_proceed": true,
  "vendor_source": "Partner_Workday",
  "record_count": 5,
  "data": [
    {
      "PayrollEmployeeId": "EMP001",
      "GrossPay": "5000",
      "EmployeeEmail": "john.smith@company.com",
      "NetPay": "3800",
      "PaymentDate": "2025-06-15"
    },
    ...
  ],
  "message": "Payroll validation successful"
}
```

### Error Response (HTTP 400)
```json
{
  "incident_id": "INC - 2025-12-16 08:05:22",
  "status": "error",
  "error": "Not enough columns: CSV has 3, vendor spec expects 5",
  "vendor_source": "Partner_Workday",
  "message": "Payroll validation failed - requires manual review"
}
```

---

## 🧪 Testing the Workflow

### Test Sequence

1. **Start with valid data** (test_payroll_1_valid.csv)
   - Verify success path works
   - Check webhook response
   - Confirm email notification

2. **Test with extra columns** (test_payroll_3_extra_columns.csv)
   - Verify lenient handling
   - Confirm confidence = 91%
   - Check that data still processes

3. **Test escalation** (test_payroll_2_missing_column.csv)
   - Verify error detection
   - Confirm escalation routing
   - Check alert email

4. **Scale testing** (test_payroll_4_large_dataset.csv)
   - Verify performance with 10+ records
   - Check `map()` function handles all records
   - Monitor execution time

5. **Error handling** (test_payroll_5_empty.csv)
   - Verify graceful error messages
   - Confirm fallback routing

---

## 🔍 Key Implementation Details

### Variable Scoping in Make.com
```javascript
// Access variables via input.variableName pattern
const processedCsvRecords = input.csvRecords?.array || input.csvRecords;
const processedVendorSpecData = input.vendorSpecData?.array || input.vendorSpecData;
```

### Dynamic Data Transformation
```javascript
// Uses Make.com map() for scalability
data: map(1.csvRecords; {
  "PayrollEmployeeId": .col1,
  "GrossPay": .col3,
  "EmployeeEmail": .col2,
  "NetPay": .col4,
  "PaymentDate": .col5
})
```

### Email Fallback Formula
```javascript
if(empty(1.notification_email); "napville2000@gmail.com"; 1.notification_email)
```

---

## 📊 Performance Metrics

- **Validation Speed:** ~680ms (including all lookups)
- **CSV Parse:** <50ms
- **Spec Lookup:** <100ms
- **Validation:** <200ms
- **Webhook Response:** <300ms
- **Total Workflow:** <1 second

---

## 🎓 Learning Outcomes

This demo teaches:

✅ Make.com workflow architecture
✅ JavaScript validation engines
✅ Data mapping and transformation
✅ Conditional routing logic
✅ Error handling and escalation
✅ Google Sheets integration
✅ REST API webhook patterns
✅ Email notification systems
✅ Scalable data processing with `map()`

---

## 🚀 Production Considerations

To deploy to production:

1. **Authentication:** Add OAuth/API key authentication
2. **Database:** Replace Google Sheets with proper database
3. **Audit Logging:** Enhanced logging and compliance tracking
4. **Error Handling:** More granular error categories
5. **Retry Logic:** Automatic retry with exponential backoff
6. **Validation:** Add email/amount validation rules
7. **Encryption:** Secure data in transit and at rest
8. **Rate Limiting:** Implement throttling for large batches
9. API Endpoints connections to Vendor Specs
10. Logic/Workflow to insert formatted payload to client's Payroll System 

---

## 📞 Support & Questions

For issues, questions, or suggestions:
1. Check the [GitHub Issues](https://github.com/napville2000/Agentic-Workflow-Demo/issues)
2. Review test files for expected behavior
3. Consult Make.com documentation for workflow modifications

---

## 📄 License

This demo is provided as-is for educational and demonstration purposes.

---

## 🙏 Acknowledgments

Built with:
- [Make.com](https://make.com) - Workflow automation
- [Google Sheets API](https://developers.google.com/sheets) - Data storage
- [Vanilla JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) - Validation logic

---

**Last Updated:** December 16, 2025

**Status:** ✅ Fully Functional Demo

---

## Quick Links

- 🌐 **[Live Form](https://napville2000.github.io/Agentic-Workflow-Demo/payroll-form.html)**
- 📁 **[Sample Files]([https://github.com/napville2000/Agentic-Workflow-Demo/tree/main/Sample-Import-Files](https://drive.google.com/drive/folders/1tOSJ1RHZvlDR_KAQLm9-ZxdCPicWmiPO?usp=drive_link))**
- 🛠️ **[Make.com Workflow]([https://make.com](https://us2.make.com/public/shared-scenario/zX1szMLhWKj/payroll-guardrail-complete-final))** (configure your own)
- 📊 **[Vendor Specs Sheet]([https://docs.google.com/spreadsheets](https://docs.google.com/spreadsheets/d/1cjaOGFcqMBvSyfz1mT6r9U9uoT50HBVpnDBwEnVBA8E/edit?gid=1495552387#gid=1495552387))** (shared reference)
