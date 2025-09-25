# Step 1: Dataverse Data Model with Hospital Operations Director Approval

**Date**: 2025-09-25
**Status**: ✅ COMPLETED
**Version**: Enhanced with Director Approval Workflow

## Data Model Components

### 1. Tables Created (6 Core Tables)

#### Primary Tables
- **ayh_request** - Service requests with director approval workflow
- **ayh_department** - Hospital departments with budget limits
- **ayh_slapolicy** - SLA policies with business hours
- **ayh_comment** - Request comments and updates
- **ayh_statushistory** - Audit trail of status changes
- **ayh_requestassignment** - Assignment tracking

#### Key Enhancements for Director Approval
- `ayh_requiresdirectorapproval` (Boolean) - Flags high-cost/critical requests
- `ayh_directorapprovalstatus` - Pending/Approved/Rejected/Requires Info
- `ayh_directorapprovedby` - Hospital Operations Director user lookup
- `ayh_estimatedcost` - Money field for approval thresholds
- `ayh_businessjustification` - Required memo for director approval

### 2. Choice Sets (7 Option Sets)

| Choice Set | Options | Purpose |
|------------|---------|----------|
| `ayh_category` | Clinical Support, IT, Maintenance, HR, Supplies, Housekeeping, Transport, Other | Request categorization |
| `ayh_priority` | Low, Normal, High, Critical | Priority levels |
| `ayh_channel` | App, Phone, Email, Walk-in | Submission method |
| `ayh_status` | New, Triaged, **Pending Director Approval**, **Director Approved**, **Director Rejected**, In Progress, On Hold, Resolved, Rejected, Closed | Enhanced status with approval workflow |
| `ayh_businesshours` | 24x7, Weekdays_8_17, Custom | SLA calculation basis |
| `ayh_approvalstatus` | Pending, Approved, Rejected, Requires Additional Information | Director approval states |
| `ayh_commenttype` | General, Status Update, Assignment, Customer Communication, Internal Note, **Director Decision** | Comment categorization |

### 3. Security Roles (5 Roles)

#### Role Hierarchy & Permissions
1. **AYH Staff** - Submit requests, view own submissions
2. **AYH Department Agent** - Manage department requests, cannot approve director-level requests
3. **AYH Department Manager** - Agent privileges + reassignment + department budget approval
4. **AYH Hospital Operations Director** - **NEW ROLE** - Full approval authority for high-cost/critical requests
5. **AYH Admin** - System administration and configuration

#### Director Approval Workflow
```
Request Creation → Auto-flag if:
  - Estimated Cost > $5,000
  - Category: Clinical Support/Maintenance
  - Priority: Critical
  - Manual flag: ayh_requiresdirectorapproval = true

Approval Flow:
New → Pending Director Approval → Director Approved/Rejected → Triaged/Closed
```

### 4. Table Schema Details

#### Requests Table (ayh_request) - 21 Attributes
- Auto-numbered: `REQ-{SEQNUM:0000000}`
- Audit enabled for compliance tracking
- Director approval fields integrated
- SLA tracking with first response and resolution targets

#### Departments Table (ayh_department) - 8 Attributes
- Budget limits for auto-approval thresholds
- Teams channel integration
- Cost center mapping

#### SLA Policies Table (ayh_slapolicy) - 7 Attributes
- Configurable business hours (24x7, weekdays, custom)
- Escalation timers
- First response and resolution targets

### 5. Seed Data Generated
- **8 Departments**: IT, Maintenance, HR, Housekeeping, Transport, Clinical Engineering, Security, Food Services
- **8 SLA Policies**: Standard and critical policies for each department type
- **8 Sample Requests**: Including director approval scenarios ($75K MRI repair, $125K cardiac monitors)

### 6. Relationships Configured
- Request ↔ SLA Policy (N:1)
- Request ↔ Departments (N:1 requesting, N:1 owning)
- Request ↔ Comments (1:N with cascade delete)
- Request ↔ Status History (1:N with cascade delete)
- Request ↔ Assignments (1:N with cascade delete)

## Director Approval Features

### Approval Triggers
- **Cost Threshold**: > $5,000
- **Categories**: Clinical Support, Maintenance (high-impact)
- **Priority**: Critical requests
- **Manual Override**: Custom business rules

### Approval Capabilities
- View all requests across departments
- Approve/reject with comments
- Request additional information
- Override SLA policies
- Access dedicated approval dashboard
- Receive escalation notifications

### Integration Points
- **AAD Group**: `${AAD_GROUP_DIRECTOR_ID}` for role assignment
- **Teams**: Director approval notifications
- **Power BI**: Director approval metrics and dashboards
- **Power Automate**: Automated approval workflow triggers

## Files Created
```
dataverse/schema/
├── tables/
│   ├── Requests.json (21 attributes + director approval)
│   ├── Departments.json (8 attributes + budget limits)
│   ├── SLAPolicies.json (7 attributes)
│   ├── Comments.json (7 attributes)
│   ├── StatusHistory.json (7 attributes)
│   └── RequestAssignment.json (8 attributes)
├── choices/
│   └── ChoiceSets.json (7 choice sets with director approval status)
├── security/
│   └── SecurityRoles.json (5 roles including Hospital Operations Director)
├── seeddata/
│   ├── Departments.csv (8 departments)
│   ├── SLAPolicies.csv (8 policies)
│   └── SampleRequests.csv (8 requests including director approval scenarios)
└── relationships/
    └── TableRelationships.json (7 relationships)
```

## Ready for Step 2
Complete Dataverse schema with Hospital Operations Director approval workflow ready for Canvas App development.