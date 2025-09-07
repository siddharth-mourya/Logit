## So our model hierarchy becomes:

* **Organization**
  * ↳ **Users**
  * ↳ **Devices**
  * ↳ **Ports**
  * ↳ **Values (time-series readings)**


By keeping **Ports in a separate collection**, you gain:

* **Flexibility** → if some future device has 24 or 32 ports, you don’t break the schema.
* **Independent management** → ports can be created/updated/queried without touching the whole device.
* **Better querying** → you can run queries like: *“Find all analog ports across all devices in Org X with thresholds set”* without scanning devices.

The **extra populate** (device → ports) is a small trade-off, and MongoDB handles it well.

---
