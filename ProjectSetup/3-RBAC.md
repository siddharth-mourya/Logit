
---

## 🎭 Proposed Role Hierarchy

### **1. CompanyAdmin (Manufacturing Company)**

* Belongs to **your company (the manufacturer)**, not any customer org.
* There will always be **at least one CompanyAdmin**.
* **Responsibilities & Privileges**:

  * Create and manage **Organizations**.
  * Create initial **OrgAdmin** users when onboarding a new org.
  * View all organizations, devices, and users (for support & oversight).
  * Manage **DeviceModels** and pre-register devices (assign IMEIs).
  * Configure global settings (e.g., calibration defaults, notification providers).

---

### **2. OrgAdmin (Customer Organization Admin)**

* First OrgAdmin is created by **CompanyAdmin** during org onboarding.
* Can create **more OrgAdmins** (co-admins) or other lower-level roles.
* **Responsibilities & Privileges**:

  * Manage devices assigned to their organization.
  * Configure **ports, thresholds, and alerts** for their org’s devices.
  * Manage **users** in their organization (invite, deactivate).
  * Can access **all telemetry data** for their org’s devices.

---

### **3. OrgOperator (or simply Operator)**

* Assigned by an OrgAdmin.
* **Responsibilities & Privileges**:

  * Monitor devices and telemetry data.
  * Acknowledge and act on **alerts**.
  * Cannot manage org users or reassign devices.
  * Typically used for **operations staff** who respond to device events.

---

### **4. OrgViewer (Read-Only User)**

* Assigned by OrgAdmin.
* **Responsibilities & Privileges**:

  * View devices, ports, and telemetry.
  * View alerts history.
  * Cannot modify anything (strictly read-only).

---

## 🔒 Optional Roles (future-proofing)

* **SupportEngineer**: Belongs to the manufacturing company; can be temporarily assigned access to an org for troubleshooting (auditable access).
* **OrgIntegrator**: For organizations that want a **technical integration user** (API key, webhook-only account).

---

## 🗝️ Role Hierarchy (Tree View)

```
CompanyAdmin (Manufacturer)
   ├─ Can manage ALL orgs
   ├─ Can create OrgAdmin(s) for any org
   │
   └── OrgAdmin (Org-level superuser)
          ├─ Can create OrgAdmin(s) for same org
          ├─ Can create/manage OrgOperator(s)
          ├─ Can create/manage OrgViewer(s)
          │
          ├── OrgOperator (manage devices, handle alerts)
          └── OrgViewer (read-only)
```

---

## ✅ Why this works well

1. **Scalable**: Roles are minimal but cover 95% of enterprise use cases.
2. **Clear separation**:

   * Manufacturer vs. Customer roles.
   * Admin vs. Operator vs. Viewer at the org level.
3. **Least privilege principle**: Users only get the access they need.
4. **Future ready**: Easy to add new specialized roles (like SupportEngineer) without redesigning.
