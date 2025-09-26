# 🧮 AYH Service Request Hub - PowerFX Formula Reference

## 🎯 **HOSPITAL OPERATIONS DIRECTOR APPROVAL FORMULAS**

### **🔄 Automatic Director Routing Logic**

#### **OnSubmit - Submit Request Screen**:
```powerappfx
// Main submission logic with director routing
SubmitForm(ServiceRequestForm);

// Check if director approval required
If(
    ServiceRequestForm.LastSubmit.'Estimated Cost' > 5000,
    // High-cost request - route to director
    Patch(
        'Service Requests',
        ServiceRequestForm.LastSubmit,
        {
            'Status Value': 'Pending Director Approval',
            'Assigned To': LookUp(
                Users,
                'Security Role' = "AYH Hospital Operations Director"
            ).Email,
            'Approval Required Date': DateAdd(Today(), 2, Days),
            'Director Approval Required': true
        }
    );

    // Create status history record
    CreateRecord(
        'Status History',
        {
            Request: ServiceRequestForm.LastSubmit,
            Status: "Routed to Director - High Cost",
            Comments: "Request over $5,000 requires Hospital Operations Director approval",
            'Changed By': User().Email,
            'Change Date': Now()
        }
    );

    // Show director approval message
    Notify(
        "Request submitted for Hospital Operations Director approval (>$5,000)",
        NotificationType.Success,
        3000
    ),

    // Standard request - route to department
    Patch(
        'Service Requests',
        ServiceRequestForm.LastSubmit,
        {
            'Status Value': 'In Review',
            'Assigned To': LookUp(
                Users,
                Department = ServiceRequestForm.LastSubmit.Department.'Department Name' &&
                'Security Role' in ["AYH Department Agent", "AYH Department Manager"]
            ).Email
        }
    );

    // Create status history
    CreateRecord(
        'Status History',
        {
            Request: ServiceRequestForm.LastSubmit,
            Status: "Submitted for Department Review",
            Comments: "Standard departmental approval process",
            'Changed By': User().Email,
            'Change Date': Now()
        }
    )
);

// Navigate to confirmation screen
Navigate(RequestConfirmationScreen, ServiceRequestForm.LastSubmit)
```

### **🎯 Director Approval Screen Formulas**

#### **Gallery Filter - Pending Director Approvals**:
```powerappfx
// Filter for director approval queue
Filter(
    'Service Requests',
    'Status Value' = "Pending Director Approval" &&
    'Estimated Cost' > 5000 &&
    'Assigned To' = User().Email
)
```

#### **Quick Approve Action**:
```powerappfx
// Director quick approval
UpdateIf(
    'Service Requests',
    ID = DirectorGallery.Selected.ID,
    {
        'Status Value': 'Approved - Director',
        'Approved By': User().Email,
        'Approval Date': Now(),
        'Director Comments': "Quick approved via mobile app"
    }
);

// Log approval in status history
CreateRecord(
    'Status History',
    {
        Request: DirectorGallery.Selected,
        Status: "Approved by Hospital Operations Director",
        Comments: "Approved by " & User().FullName & " - Amount: " &
                 Text(DirectorGallery.Selected.'Estimated Cost', "$#,##0.00"),
        'Changed By': User().Email,
        'Change Date': Now()
    }
);

// Send notification to requester
Office365Outlook.SendEmailV2(
    DirectorGallery.Selected.'Requested By',
    "AYH Service Request Approved - " & DirectorGallery.Selected.'Request Number',
    "Your service request has been approved by the Hospital Operations Director.<br><br>" &
    "<b>Request:</b> " & DirectorGallery.Selected.Title & "<br>" &
    "<b>Amount:</b> " & Text(DirectorGallery.Selected.'Estimated Cost', "$#,##0.00") & "<br>" &
    "<b>Approved by:</b> " & User().FullName & "<br>" &
    "<b>Next Steps:</b> Your request will now proceed to implementation."
);

Notify("Request approved and notification sent to requester", NotificationType.Success)
```

#### **Request More Information**:
```powerappfx
// Director requests additional information
UpdateIf(
    'Service Requests',
    ID = DirectorGallery.Selected.ID,
    {
        'Status Value': 'More Information Required',
        'Info Requested By': User().Email,
        'Info Request Date': Now()
    }
);

// Create comment with director's request
CreateRecord(
    'Comments',
    {
        Request: DirectorGallery.Selected,
        'Comment Type': 'Director Request',
        Comments: InfoRequestTextInput.Text,
        'Created By': User().Email,
        'Created Date': Now(),
        'Is Director Comment': true
    }
);

// Notify requester
Office365Outlook.SendEmailV2(
    DirectorGallery.Selected.'Requested By',
    "Additional Information Required - " & DirectorGallery.Selected.'Request Number',
    "The Hospital Operations Director has requested additional information for your service request.<br><br>" &
    "<b>Request:</b> " & DirectorGallery.Selected.Title & "<br>" &
    "<b>Information Requested:</b><br>" & InfoRequestTextInput.Text & "<br><br>" &
    "Please log into the AYH Service Request Hub to provide the requested information."
)
```

---

## 🔐 **ROLE-BASED SECURITY FORMULAS**

### **Screen Visibility Controls**:

#### **Director-Only Screens**:
```powerappfx
// DirectorApprovalScreen.Visible
User().'Security Role' = "AYH Hospital Operations Director"
```

#### **Manager and Above Access**:
```powerappfx
// DepartmentReportsScreen.Visible
User().'Security Role' in [
    "AYH Department Manager",
    "AYH Hospital Operations Director"
]
```

### **Dynamic Navigation Based on Role**:
```powerappfx
// HomeScreen navigation buttons
Switch(
    User().'Security Role',
    "AYH Staff",
    [
        {Button: "Submit New Request", Screen: SubmitRequestScreen, Color: RGBA(0, 120, 212, 1)},
        {Button: "My Requests", Screen: MyRequestsScreen, Color: RGBA(0, 120, 212, 1)},
        {Button: "Request Status", Screen: RequestStatusScreen, Color: RGBA(0, 120, 212, 1)}
    ],
    "AYH Department Agent",
    [
        {Button: "Submit New Request", Screen: SubmitRequestScreen, Color: RGBA(16, 124, 16, 1)},
        {Button: "My Requests", Screen: MyRequestsScreen, Color: RGBA(16, 124, 16, 1)},
        {Button: "Process Requests", Screen: ProcessRequestsScreen, Color: RGBA(16, 124, 16, 1)},
        {Button: "Department Queue", Screen: DepartmentQueueScreen, Color: RGBA(16, 124, 16, 1)}
    ],
    "AYH Department Manager",
    [
        {Button: "Submit New Request", Screen: SubmitRequestScreen, Color: RGBA(255, 140, 0, 1)},
        {Button: "My Requests", Screen: MyRequestsScreen, Color: RGBA(255, 140, 0, 1)},
        {Button: "Process Requests", Screen: ProcessRequestsScreen, Color: RGBA(255, 140, 0, 1)},
        {Button: "Department Queue", Screen: DepartmentQueueScreen, Color: RGBA(255, 140, 0, 1)},
        {Button: "Approve Requests", Screen: ApproveRequestsScreen, Color: RGBA(255, 140, 0, 1)},
        {Button: "Department Reports", Screen: DepartmentReportsScreen, Color: RGBA(255, 140, 0, 1)}
    ],
    "AYH Hospital Operations Director",
    [
        {Button: "Submit New Request", Screen: SubmitRequestScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "My Requests", Screen: MyRequestsScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Process Requests", Screen: ProcessRequestsScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Department Queue", Screen: DepartmentQueueScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Approve Requests", Screen: ApproveRequestsScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Director Approvals ($5K+)", Screen: DirectorApprovalScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Executive Dashboard", Screen: ExecutiveDashboardScreen, Color: RGBA(209, 52, 56, 1)},
        {Button: "Hospital Reports", Screen: HospitalReportsScreen, Color: RGBA(209, 52, 56, 1)}
    ]
)
```

---

## 📊 **EXECUTIVE DASHBOARD FORMULAS**

### **KPI Calculations**:

#### **Pending Director Approvals Count**:
```powerappfx
CountRows(
    Filter(
        'Service Requests',
        'Status Value' = "Pending Director Approval" &&
        'Estimated Cost' > 5000
    )
)
```

#### **Total Value Pending Director Approval**:
```powerappfx
Sum(
    Filter(
        'Service Requests',
        'Status Value' = "Pending Director Approval" &&
        'Estimated Cost' > 5000
    ),
    'Estimated Cost'
)
```

#### **Average Director Approval Time**:
```powerappfx
// Average days from submission to director approval
Average(
    Filter(
        'Service Requests',
        'Approved By' <> Blank() &&
        'Approval Date' <> Blank() &&
        'Estimated Cost' > 5000
    ),
    DateDiff('Created Date', 'Approval Date', Days)
)
```

### **Chart Data Sources**:

#### **Requests by Department (Director View)**:
```powerappfx
// Group requests by department for donut chart
GroupBy(
    Filter(
        'Service Requests',
        'Created Date' >= DateAdd(Today(), -30, Days) // Last 30 days
    ),
    "Department",
    "RequestCount"
)
```

#### **High-Cost Request Trends**:
```powerappfx
// Monthly trend of requests >$5K
GroupBy(
    Filter(
        'Service Requests',
        'Estimated Cost' > 5000 &&
        'Created Date' >= DateAdd(Today(), -365, Days) // Last year
    ),
    "MonthYear",
    "HighCostCount"
)
```

---

## 🔔 **NOTIFICATION FORMULAS**

### **Director Approval Required Email**:
```powerappfx
// Automated email when >$5K request submitted
Office365Outlook.SendEmailV2(
    LookUp(Users, 'Security Role' = "AYH Hospital Operations Director").Email,
    "[DIRECTOR APPROVAL REQUIRED] AYH Service Request - " & ServiceRequestForm.LastSubmit.'Request Number',

    "<html><body style='font-family: Segoe UI, Arial, sans-serif;'>" &
    "<div style='background: #f3f2f1; padding: 20px; border-left: 4px solid #d13438;'>" &
    "<h2 style='color: #d13438; margin-top: 0;'>🏥 AYH Healthcare - Director Approval Required</h2>" &
    "<p><strong>A high-cost service request requires your approval:</strong></p>" &

    "<table style='border-collapse: collapse; width: 100%; margin: 15px 0;'>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold;'>Request Number:</td>" &
        "<td style='padding: 8px;'>" & ServiceRequestForm.LastSubmit.'Request Number' & "</td></tr>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold;'>Title:</td>" &
        "<td style='padding: 8px;'>" & ServiceRequestForm.LastSubmit.Title & "</td></tr>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold;'>Requested By:</td>" &
        "<td style='padding: 8px;'>" & ServiceRequestForm.LastSubmit.'Requested By' & "</td></tr>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold;'>Department:</td>" &
        "<td style='padding: 8px;'>" & ServiceRequestForm.LastSubmit.Department.'Department Name' & "</td></tr>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold; color: #d13438;'>Estimated Cost:</td>" &
        "<td style='padding: 8px; color: #d13438; font-weight: bold; font-size: 16px;'>" &
        Text(ServiceRequestForm.LastSubmit.'Estimated Cost', "$#,##0.00") & "</td></tr>" &
    "<tr><td style='padding: 8px; background: #faf9f8; font-weight: bold;'>Priority:</td>" &
        "<td style='padding: 8px;'>" & ServiceRequestForm.LastSubmit.Priority & "</td></tr>" &
    "</table>" &

    "<div style='background: #fff4ce; padding: 15px; border-radius: 4px; margin: 15px 0;'>" &
    "<p><strong>Description:</strong></p>" &
    "<p>" & ServiceRequestForm.LastSubmit.Description & "</p>" &
    "</div>" &

    "<p><strong>Action Required:</strong> Please review and approve/reject this request in the AYH Service Request Hub app.</p>" &

    "<div style='text-align: center; margin: 20px 0;'>" &
    "<a href='https://make.powerapps.com' style='background: #d13438; color: white; padding: 12px 25px; text-decoration: none; border-radius: 4px; display: inline-block;'>" &
    "Review Request in AYH App</a>" &
    "</div>" &

    "<hr style='border: none; border-top: 1px solid #edebe9; margin: 20px 0;'>" &
    "<p style='color: #605e5c; font-size: 12px;'>AYH Healthcare Service Request Hub | Automated notification</p>" &
    "</div></body></html>",

    {
        Importance: "High",
        IsBodyHtml: true
    }
)
```

---

## 🔍 **SEARCH AND FILTER FORMULAS**

### **Advanced Request Search**:
```powerappfx
// Smart search across multiple fields
Filter(
    'Service Requests',
    SearchTextInput.Text in Title ||
    SearchTextInput.Text in Description ||
    SearchTextInput.Text in 'Request Number' ||
    SearchTextInput.Text in Department.'Department Name' ||
    SearchTextInput.Text in 'Requested By'
)
```

### **Director Dashboard Filters**:
```powerappfx
// Filter options for director dashboard
Switch(
    DirectorFilterDropdown.Selected.Value,
    "All High-Cost", Filter('Service Requests', 'Estimated Cost' > 5000),
    "Pending My Approval", Filter('Service Requests', 'Status Value' = "Pending Director Approval"),
    "Recently Approved", Filter('Service Requests', 'Approved By' = User().Email && 'Approval Date' >= DateAdd(Today(), -7, Days)),
    "This Month", Filter('Service Requests', 'Created Date' >= DateAdd(Today(), -30, Days)),
    'Service Requests' // Default: All requests
)
```

---

## 🎯 **WORKFLOW STATUS TRACKING**

### **Request Progress Calculation**:
```powerappfx
// Calculate workflow progress percentage
Switch(
    ThisItem.'Status Value',
    "Open", 20,
    "In Review", 40,
    "Pending Director Approval", If(ThisItem.'Estimated Cost' > 5000, 60, 40),
    "Approved - Director", 80,
    "Approved - Department", 80,
    "In Progress", 90,
    "Completed", 100,
    "Rejected - Director", 0,
    "Rejected - Department", 0,
    10 // Default
)
```

### **Dynamic Status Colors**:
```powerappfx
// Status-based color coding
Switch(
    ThisItem.'Status Value',
    "Pending Director Approval", RGBA(209, 52, 56, 1),   // Red - Urgent
    "Approved - Director", RGBA(16, 124, 16, 1),         // Green - Good
    "In Progress", RGBA(255, 140, 0, 1),                 // Orange - In Progress
    "Completed", RGBA(16, 124, 16, 1),                   // Green - Done
    "Rejected - Director", RGBA(168, 0, 0, 1),           // Dark Red - Rejected
    RGBA(0, 120, 212, 1)                                 // Blue - Default
)
```

**These PowerFX formulas bring the AYH Service Request Hub to life with intelligent director approval routing, role-based security, and executive-grade analytics!**