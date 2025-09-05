Alright 👍 — I’ll start fresh and consolidate everything into a **complete MongoDB Data Model** for your IoT monitoring platform.
This version reflects:

* **Company Admin** control of organizations, devices, and port base setup.
* **Org Admin** control of calibration, alerts, and limited port management.
* Support for **historical data**, **multi-tenancy**, and **auth**.

---

# **Complete Data Model (MongoDB)**

---

### 1. **CompanyAdmin**

(Your company’s super-admin users)

```json
CompanyAdmin {
  _id: ObjectId,
  name: String,
  email: String (unique),
  passwordHash: String,
  role: String ("company-admin"),
  createdAt: Date,
  updatedAt: Date
}
```

---

### 2. **Organization**

(Customer organizations)

```json
Organization {
  _id: ObjectId,
  name: String,
  description: String,
  status: String ("active" | "suspended"),
  createdAt: Date,
  updatedAt: Date
}
```

---

### 3. **User**

(Org-level users: admins, operators, viewers)

```json
User {
  _id: ObjectId,
  organizationId: ObjectId (ref → Organization),
  name: String,
  email: String (unique per org),
  passwordHash: String,
  role: String ("org-admin" | "operator" | "viewer"),
  createdAt: Date,
  updatedAt: Date
}
```

---

### 4. **AuthSession** (optional, if refresh tokens needed)

```json
AuthSession {
  _id: ObjectId,
  userId: ObjectId (ref → User),
  refreshToken: String,
  expiresAt: Date,
  createdAt: Date
}
```

---

### 5. **Device**

(Physical IoT devices)

```json
Device {
  _id: ObjectId,
  organizationId: ObjectId (ref → Organization),
  registeredBy: ObjectId (ref → CompanyAdmin),
  name: String,
  serialNumber: String (unique, from manufacturing),
  status: String ("online" | "offline" | "fault"),
  lastSeen: Date,
  portsConfigured: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

---

### 6. **Port**

(Each device has multiple ports)

```json
Port {
  _id: ObjectId,
  deviceId: ObjectId (ref → Device),
  name: String,                  // e.g. "Port 1"
  type: String ("digital" | "analog"),   // set by Company Admin
  unit: String,                  // e.g. "V", "A", "ON/OFF"
  status: String ("active" | "inactive"),

  calibration: {                 // editable by Org Admin
    scaling: Number,             // default 1.0
    offset: Number               // default 0.0
  },

  thresholdRules: [              // editable by Org Admin
    {
      ruleType: String,          // "greaterThan" | "lessThan" | "range"
      value: Number | [Number],  // depends on ruleType
      severity: String           // "warning" | "critical"
    }
  ],

  createdAt: Date,
  updatedAt: Date
}
```

---

### 7. **Values**

(Time-series data collected per port)

```json
Values {
  _id: ObjectId,
  portId: ObjectId (ref → Port),
  deviceId: ObjectId (ref → Device),         // redundant for fast queries
  organizationId: ObjectId (ref → Org),      // redundant for multi-tenant queries
  timestamp: Date,                           // indexed, sorted desc
  value: Number | Boolean,                   // raw reading
  quality: String ("ok" | "faulty" | "missing"), // optional
}
```

**Indexes:**

* `{ portId: 1, timestamp: -1 }` → per-port queries
* `{ deviceId: 1, timestamp: -1 }` → per-device queries
* `{ organizationId: 1, timestamp: -1 }` → org-wide queries

---

### 8. **Alert / Notification Rule**

(Explicit collection for advanced alerting & tracking)

```json
AlertRule {
  _id: ObjectId,
  organizationId: ObjectId,
  portId: ObjectId,
  ruleType: String,             // "greaterThan", "lessThan", "range"
  value: Number | [Number],
  severity: String,             // "warning" | "critical"
  notificationChannels: [String], // "email" | "sms" | "webhook"
  createdBy: ObjectId (ref → OrgAdmin),
  createdAt: Date
}
```

---

### 9. **Alert Events**

(Generated when a rule is triggered)

```json
AlertEvent {
  _id: ObjectId,
  alertRuleId: ObjectId (ref → AlertRule),
  portId: ObjectId,
  deviceId: ObjectId,
  organizationId: ObjectId,
  triggeredAt: Date,
  value: Number,
  status: String ("new" | "acknowledged" | "resolved"),
  acknowledgedBy: ObjectId (ref → User, optional),
  acknowledgedAt: Date (optional)
}
```

---

✅ With this model:

* **Company Admin**: creates orgs, registers devices, defines ports (type/unit).
* **Org Admin**: configures calibration, thresholds, alerts, users.
* **Operators/Viewers**: monitor data, acknowledge alerts.
* **Values** collection handles high-volume time-series data efficiently.

---

Would you like me to **normalize this into a diagram (ERD style)** before I jump to the **UI features in detail**?
