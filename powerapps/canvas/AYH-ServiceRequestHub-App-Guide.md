# AYH Service Request Hub - Canvas App Guide

**App Name**: AYH Service Request Hub
**Type**: Canvas App (Phone layout)
**Date Created**: 2025-09-25

## App Structure

### Screen Navigation
The app automatically shows different screens based on user role:
- **Staff** → StaffScreen (request submission)
- **Department Agent/Manager** → DepartmentScreen (work queues)
- **Hospital Operations Director** → DirectorScreen (approval dashboard)
- **Admin** → AdminScreen (system overview)

### Role Detection Logic
```javascript
// OnStart of App
Set(varUserEmail, Lower(User().Email));
Set(varUserDepartment, LookUp(Departments, Lower(Manager.Email) = varUserEmail));

// Role flags
Set(varIsManager, !IsBlank(varUserDepartment));
Set(varIsDirector, varUserEmail = "director@yourhospital.com");
Set(varIsAdmin, varUserEmail = "admin@yourhospital.com");
```

## Screen Details

### 1. StaffScreen - Request Submission
**Purpose**: Allow hospital staff to submit service requests

**Key Controls**:
- txtTitle: Request title
- txtDescription: Detailed description
- drpCategory: Request category dropdown
- drpPriority: Priority level dropdown
- drpDepartment: Department dropdown
- txtCost: Estimated cost (triggers director approval if >$5,000)
- txtJustification: Business justification (visible when cost >$5,000)
- btnSubmit: Submit request button

**Director Approval Logic**:
```javascript
// Auto-flag for director approval if:
'Requires Director Approval':
    Value(txtCost.Text) > 5000 ||
    drpPriority.Selected.Value = "Critical" ||
    drpCategory.Selected.Value in ["Clinical Support", "Maintenance"]

// Set initial status
Status: If(
    Value(txtCost.Text) > 5000 || drpPriority.Selected.Value = "Critical",
    {Value: "Pending Director Approval"},
    {Value: "New"}
)
```

**My Requests Gallery**:
- Shows user's submitted requests
- Filtered by: Lower('Requested By'.Email) = varUserEmail

### 2. DepartmentScreen - Work Queue Management
**Purpose**: Department agents and managers handle assigned requests

**Key Controls**:
- togUnassigned: Show unassigned requests
- togMyQueue: Show requests assigned to me
- togAll: Show all department requests
- galDepartmentQueue: Gallery of filtered requests
- btnAssignToMe: Assign request to current user
- btnResolve: Mark request as resolved

**Gallery Filter Logic**:
```javascript
Filter(
    Requests,
    // Department requests only
    'Owning Department'.Id = varUserDepartment.Id &&
    // Exclude director approval pending
    Status.Value <> "Pending Director Approval" &&
    // Toggle filters
    (
        (togUnassigned.Value && IsBlank('Assigned To')) ||
        (togMyQueue.Value && Lower('Assigned To'.Email) = varUserEmail) ||
        togAll.Value
    )
)
```

### 3. DirectorScreen - Approval Dashboard
**Purpose**: Hospital Operations Director approves/rejects high-cost requests

**Key Controls**:
- galPendingApprovals: Gallery of requests needing approval
- txtDirectorComments: Director's approval/rejection comments
- btnApprove: Approve request button
- btnReject: Reject request button
- btnNeedInfo: Request more information button

**Pending Approvals Filter**:
```javascript
Filter(
    Requests,
    'Requires Director Approval' = true &&
    'Director Approval Status'.Value = "Pending"
)
```

**Approval Actions**:
- Updates Director Approval Status
- Sets Director Approved By to current user
- Records Director Approval Date
- Adds Director Decision comment
- Changes main Status accordingly

### 4. AdminScreen - System Overview
**Purpose**: System administrators view overall metrics and activity

**Key KPIs**:
- Total Open Requests: CountRows(Filter(Requests, Status.Value <> "Closed"))
- Pending Director Approvals: CountRows(Filter(Requests, 'Director Approval Status'.Value = "Pending"))
- High Priority Items: CountRows(Filter(Requests, Priority.Value = "Critical"))
- Total Cost Pending Approval: Sum of estimated costs for pending approvals

## Key Formulas

### Director Approval Auto-Detection
```javascript
// Triggers director approval requirement
Value(txtCost.Text) > 5000 ||
drpPriority.Selected.Value = "Critical" ||
drpCategory.Selected.Value in ["Clinical Support", "Maintenance"]
```

### Request Assignment
```javascript
// Assign request to current user
Patch(
    ThisItem,
    {
        'Assigned To': User(),
        Status: If(
            ThisItem.Status.Value in ["New", "Triaged"],
            {Value: "In Progress"},
            ThisItem.Status
        )
    }
)
```

### Director Approval
```javascript
// Approve request
Patch(
    ThisItem,
    {
        'Director Approval Status': {Value: "Approved"},
        'Director Approved By': User(),
        'Director Approval Date': Now(),
        'Director Approval Comments': txtDirectorComments.Text,
        Status: {Value: "Director Approved"}
    }
)
```

## Testing Scenarios

### 1. Staff Request Submission
- Submit regular request (< $5,000) → Status: New
- Submit expensive request (> $5,000) → Status: Pending Director Approval
- Submit critical priority → Status: Pending Director Approval
- Submit Clinical Support category → Status: Pending Director Approval

### 2. Department Queue Management
- View unassigned requests in department
- Assign request to self → Status changes to In Progress
- Mark assigned request as resolved → Status: Resolved

### 3. Director Approval Process
- View pending approvals (cost > $5,000 or critical)
- Approve request → Status: Director Approved
- Reject request → Status: Director Rejected
- Request more info → Stays Pending, adds comment

### 4. Admin Overview
- Verify KPI calculations are correct
- Check recent activity shows latest requests
- Confirm pending approval totals match

## Deployment Notes

### Required Connections
- Microsoft Dataverse (to your solution tables)
- Office 365 Users (for user lookups)

### Security Configuration
- App should be shared with appropriate Azure AD groups
- Role detection logic should be updated for production Azure AD group membership

### Environment Variables Needed
- Director email addresses for role detection
- Department manager assignments
- Teams integration settings (for future workflows)

## Troubleshooting

### Common Issues
1. **"Delegation warning"**: Use Filter() instead of Search() for large datasets
2. **Role detection not working**: Check email addresses in role detection logic
3. **Dropdowns empty**: Verify table connections are working
4. **Director approval not triggering**: Check cost field is numeric, not text

### Performance Optimization
- Use delegation-friendly functions (Filter, Sort, etc.)
- Limit gallery item counts with Top() function if needed
- Cache lookup values in variables where possible