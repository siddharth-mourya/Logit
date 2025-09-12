We need a way to **bootstrap the very first Company Admin**.

### Seed the First User (Recommended for Backend Systems)**

* In your backend startup (or with a script), check if any user with `role: "CompanyAdmin"` exists.
* If not, create one automatically from environment variables.
* This will create a entry in User table

```ts
// seedAdmin.ts (run once at startup)
import bcrypt from "bcrypt";
import { User, UserRoles } from "../User/User.model";

export async function seedAdmin() {
  // check if a CompanyAdmin already exists
  const existing = await User.findOne({ role: UserRoles.CompanyAdmin });
  if (!existing) {
    const hashed = await bcrypt.hash(process.env.DEFAULT_ADMIN_PASSWORD!, 10);
    await User.create({
      email: process.env.DEFAULT_ADMIN_EMAIL,
      password: hashed,
      role: UserRoles.CompanyAdmin,
      name: "Default Admin",
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
