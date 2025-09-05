Perfect 👍 — now that we have the **data model locked**, let’s design the **UI features in detail**.
I’ll split this into two main portals (because your company and the organizations have different responsibilities).

---

# **UI Features**

---

## 🔹 **Company Admin Portal** (Manufacturer-side)

### 1. **Authentication**

* Login (email/password, 2FA optional).
* Forgot/reset password.
* Company-admin role-based access.

---

### 2. **Dashboard**

* Overview cards:

  * Total organizations.
  * Total active devices.
  * Online/offline/faulty devices.
  * Data ingestion health (values/sec).
* Global alerts/errors overview.
* Quick actions: “Create Organization”, “Assign Device”.

---

### 3. **Organization Management**

* **List Organizations** (table with filters).
* **Create Organization** (basic details: name, description).
* **Assign Org Admin** (create user → role: `org-admin`).
* Suspend/activate org.
* View org usage (devices count, data volume).

---

### 4. **Device Management**

* **Device Inventory** (list of all devices manufactured).
* Register new device (enter serial number, name).
* Assign device to an organization.
* Monitor device health (last seen, status).
* Retire or reassign device.
* Device detail view:

  * Device info (org, serial, status).
  * List of ports (configured by company).
  * Port type/unit management.

---

### 5. **Port Management** (Company-level setup)

* Configure ports for a device:

  * Port name.
  * Port type (digital/analog).
  * Unit (V, A, °C, etc.).
* Enable/disable ports.
* Define defaults (calibration values = scaling/offset).

---

### 6. **System Monitoring**

* Data ingestion metrics.
* Error rates.
* Uptime of devices/orgs.

---

### 7. **Audit & Logs**

* Track:

  * Org creations.
  * Device registrations.
  * Port modifications.
  * Role changes.
* Export logs (CSV/PDF).

---

---

## 🔹 **Organization Admin/User Portal**

### 1. **Authentication**

* Login (email/password).
* Forgot/reset password.
* Role-based access (`org-admin`, `operator`, `viewer`).

---

### 2. **Dashboard**

* Overview cards:

  * Active devices (online/offline count).
  * Total ports configured.
  * Alerts triggered (new/active/resolved).
* Real-time metrics (last 5–10 readings).
* Quick actions:

  * “Add User”.
  * “Configure Port”.
  * “Set Alert Rule”.

---

### 3. **Device Management**

* View all devices assigned to the org.
* Device detail page:

  * Device info (status, last seen).
  * List of ports (basic metadata from company).
* Org Admin **cannot add/remove devices**, only monitor.

---

### 4. **Port Management** (Org-level)

* Configure calibration values (scaling, offset).
* Enable/disable ports.
* View port metadata (type/unit is fixed by Company).
* Assign thresholds (if quick setup is supported).

---

### 5. **Live Monitoring**

* Device selection → real-time values from each port.
* Visualization:

  * Line charts (analog values).
  * State toggles (digital values).
* Refresh auto/interval.

---

### 6. **Historical Data**

* Query builder:

  * Device → Port(s) → Date range.
* Visualization:

  * Graphs with zoom/pan.
  * Tabular view.
* Export (CSV, Excel, JSON).
* Optional: API token for programmatic access.

---

### 7. **Alert Rules Management**

* Add/edit rules:

  * Select port.
  * Rule type (greaterThan, lessThan, range).
  * Value/range.
  * Severity (warning/critical).
  * Notification channels (email, SMS, webhook).
* List of all rules with edit/delete.

---

### 8. **Alerts & Notifications**

* Active alerts view:

  * Port → Value → Severity → Time.
* Acknowledge/resolve alerts.
* History of alerts (with filters).
* Notification preferences (email IDs, webhook URLs).

---

### 9. **User Management** (Org Admin only)

* Add users (operator/viewer).
* Assign roles.
* Suspend/activate users.
* Reset user password.

---

### 10. **Reports**

* Daily/weekly/monthly summaries:

  * Device uptime.
  * Port performance.
  * Alerts triggered.
* Export PDF/CSV.

---

---

# ✅ User Flows Covered

* **Company Admin** → full control of orgs, devices, base port config.
* **Org Admin** → calibration, alerts, limited port ops, user mgmt.
* **Operators/Viewers** → monitoring, historical data, alert handling.

---

👉 Do you want me to now design a **step-by-step onboarding flow** (e.g., how a new org gets created → device assigned → ports configured → org admin calibrates → alerts set → monitoring)? That will show how these UI pieces come together in practice.
