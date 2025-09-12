We need a way to **bootstrap the very first Company Admin**.

### Seed the First User (Recommended for Backend Systems)**

* In your backend startup (or with a script), check if any user with `role: "CompanyAdmin"` exists.
* If not, create one automatically from environment variables.

```ts
// seedAdmin.ts (run once at startup)
import User from "./models/User";
import bcrypt from "bcrypt";

async function seedAdmin() {
  const existing = await User.findOne({ role: "CompanyAdmin" });
  if (!existing) {
    const hashed = await bcrypt.hash(process.env.DEFAULT_ADMIN_PASSWORD!, 10);
    await User.create({
      email: process.env.DEFAULT_ADMIN_EMAIL,
      password: hashed,
      role: "CompanyAdmin"
    });
    console.log("✅ CompanyAdmin seeded!");
  }
}
```

* Add this in your server `index.ts` before starting the app.
* Set in `.env`:

  ```
  DEFAULT_ADMIN_EMAIL=admin@yourcompany.com
  DEFAULT_ADMIN_PASSWORD=SuperSecret123!
  ```

Now you always have **at least one superuser**.

---
