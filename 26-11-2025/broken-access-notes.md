# 🔐 **Broken Access Control — Deep Explanation**

Broken Access Control happens when the system **fails to reliably enforce the rule: _who is allowed to do what_**.

Everything else (URLs, roles, tokens, sessions, buttons) is just implementation detail.

---

# 1️⃣ **The Core Idea**

Access control = **boundary enforcement**.

A system has:

- **Subjects** (users, admins, services)
- **Objects** (resources: orders, accounts, files)
- **Actions** (read, update, delete)

Broken Access Control happens when the server **trusts the client** to enforce these boundaries.

**Deep principle:**

> If the client can see the resource ID or action, it can attempt it.
> The server must assume nothing and verify everything.

---

# 2️⃣ **Two Fundamental Failure Modes**

Every access control bug belongs to one of these:

---

## **A. Horizontal Access Control Failure**

User A accesses data of User B.

Example:

```
GET /api/orders/123
```

User A changes the URL to:

```
GET /api/orders/124
```

And the server returns data.

**Deep cause:**
The server _assumed_ that if the client requests an order, it must belong to the logged-in user.

This is **Insecure Direct Object Reference (IDOR)** — the most common subtype.

---

## **B. Vertical Access Control Failure**

A normal user performs admin actions.

Example:
Frontend hides “Delete User” button, but backend still accepts:

```
POST /admin/delete-user?id=10
```

**Deep cause:**
The UI enforced the rule, not the backend.

---

# 3️⃣ **How attackers think**

Attackers operate by applying this simple logic:

> “If the client knows it, I can try it.”

They change:

- The HTTP method
- The path
- The ID
- The query parameter
- The role
- The cookie
- The JWT token
- Hidden form fields

The attacker **asks the server a question the UI never asked**.

If the backend forgot to check permissions, attack succeeds.

---

# 4️⃣ **Real examples**

### **Example 1: Changing the ID**

```
GET /api/users/10
```

Authenticated as user 10.

Try:

```
GET /api/users/11
```

If data appears → **broken access control**.

---

### **Example 2: Missing authorization middleware**

Developer protects some routes but forgets one:

```js
app.get('/admin/stats', adminMiddleware);
app.post('/admin/delete-user', (req, res) => { ... });  // missing middleware
```

This single missing line becomes a privilege escalation.

---

### **Example 3: Frontend-only restriction**

HTML hides admin buttons:

```html
<button class="admin-only" style="display:none">Delete User</button>
```

But API accepts:

```
DELETE /api/user/42
```

UI ≠ security.

---

### **Example 4: Parameter tampering**

Normal user submits:

```
role=user
```

Attacker changes to:

```
role=admin
```

Backend accepts → privilege escalation.

---

### **Example 5: Forced browsing**

Accessing URLs directly:

```
/admin
/debug
/config
```

If accessible → broken access control.

---

# 5️⃣ **Root cause (deep explanation)**

The fundamental mistake is:

### **Assuming clients enforce constraints.**

But clients:

- Are under attacker’s control
- Can modify any data
- Can call any endpoint
- Can alter requests beyond what UI allows

A system is secure only when:

> Access control is enforced **after authentication**, **in the backend**, **based on trusted server-side data**.

---

# 6️⃣ **How to fix (conceptually)**

### ✔ Server-side checks for every request

Every action should pass through a permission check.

### ✔ Use indirect references

Instead of:

```
/invoice/123
```

Use:

```
/invoice/me
```

### ✔ Deny by default

Explicitly allow only what’s permitted.

### ✔ Don’t rely on UI

UI is a convenience layer, not security.

### ✔ Centralized authorization

One module/middleware that all routes flow through.

---

# 🧠 **One-sentence summary**

> Broken access control happens when the system assumes the user will behave, instead of forcing the system to check every permission on every request.

---
