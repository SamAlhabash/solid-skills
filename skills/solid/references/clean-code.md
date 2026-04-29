# Clean Code Practices

## What is Clean Code?

Code that is:
- **Easy to understand** - reveals intent clearly
- **Easy to change** - modifications are localized
- **Easy to test** - dependencies are injectable
- **Simple** - no unnecessary complexity

## The Human-Centered Approach

Code has THREE consumers:
1. **Users** - get their needs met
2. **Customers** - make or save money
3. **Developers** - must maintain it

Design for all three, but remember: **developers read code 10x more than they write it.**

## Naming Principles

### 1. Consistency & Uniqueness (HIGHEST PRIORITY)
Same concept = same name everywhere. One name per concept.

```typescript
// BAD: Inconsistent names for same concept
getUserById(id)
fetchCustomerById(id)
retrieveClientById(id)

// GOOD: Consistent
getUser(id)
getOrder(id)
getProduct(id)
```

### 2. Understandability
Use domain language, not technical jargon.

```typescript
// BAD: Technical
const arr = users.filter(u => u.isActive);

// GOOD: Domain language
const activeCustomers = users.filter(user => user.isActive);
```

### 3. Specificity
Avoid vague names: `data`, `info`, `manager`, `handler`, `processor`, `utils`

```typescript
// BAD: Vague
class DataManager { }
function processInfo(data) { }

// GOOD: Specific
class OrderRepository { }
function validatePayment(payment) { }
```

### 4. Brevity (but not at cost of clarity)
Short names are good only if meaning is preserved.

```typescript
// BAD: Too cryptic
const usrLst = getUsrs();

// BAD: Unnecessarily long
const listOfAllActiveUsersInTheSystem = getActiveUsers();

// GOOD: Brief but clear
const activeUsers = getActiveUsers();
```

### 5. Searchability
Names should be unique enough to grep/search.

```typescript
// BAD: Common word, hard to search
const data = fetch();

// GOOD: Unique, searchable
const orderSummary = fetchOrderSummary();
```

### 6. Pronounceability
You should be able to say it in conversation.

```typescript
// BAD
const genymdhms = generateYearMonthDayHourMinuteSecond();

// GOOD
const timestamp = generateTimestamp();
```

### 7. Austerity
Avoid unnecessary filler words.

```typescript
// BAD: Redundant
const userData = user; // 'Data' adds nothing
class UserClass { }    // 'Class' adds nothing

// GOOD
const user = user;
class User { }
```

---

## Object Calisthenics (9 Rules)

These are **exercises for building design intuition**, not production mandates. Practicing them strictly sharpens your sense of cohesion, encapsulation, and single responsibility. In production, apply judgment: the goal is code that is clear and maintainable, not code that scores 9/9 on calisthenics.

### 1. One Level of Indentation per Method

```typescript
// BAD: Multiple levels
function process(orders: Order[]) {
  for (const order of orders) {
    if (order.isValid()) {
      for (const item of order.items) {
        if (item.inStock) {
          // process...
        }
      }
    }
  }
}

// GOOD: Extract methods
function process(orders: Order[]) {
  orders.filter(o => o.isValid()).forEach(processOrder);
}

function processOrder(order: Order) {
  order.items.filter(i => i.inStock).forEach(processItem);
}
```

### 2. Don't Use the ELSE Keyword

Use early returns, guard clauses, or polymorphism.

```typescript
// BAD: else
function getDiscount(user: User): number {
  if (user.isPremium) {
    return 20;
  } else {
    return 0;
  }
}

// GOOD: Early return
function getDiscount(user: User): number {
  if (user.isPremium) return 20;
  return 0;
}
```

### 3. Wrap All Primitives and Strings

Wrap primitives in domain objects **when they carry validation, behavior, or risk of mix-up**. Don't wrap every string just for ceremony.

```typescript
// GOOD: Wrapping adds value — validation + prevents transposition bugs
class Email {
  constructor(private value: string) {
    if (!this.isValid(value)) throw new InvalidEmail();
  }
  private isValid(email: string): boolean { ... }
}

class Age {
  constructor(private value: number) {
    if (value < 0 || value > 150) throw new InvalidAge();
  }
}

function createUser(email: Email, age: Age) { }

// EXERCISE (not always production-appropriate):
// Wrapping every string ID (e.g., a simple internal reference with no validation
// or risk of mix-up) adds indirection without benefit. Apply judgment.
```

### 4. First-Class Collections

When a collection has its own behavior (filtering, totaling, invariants), extract it into its own class. In complex domains this prevents logic from leaking into callers. For simple CRUD, an `Order` with both `items: OrderItem[]` and `customerId` is perfectly fine — don't extract collections just to follow the rule.

```typescript
// GOOD in complex domains: OrderItems has its own behavior
class OrderItems {
  constructor(private items: OrderItem[] = []) {}

  add(item: OrderItem): void { ... }
  total(): Money { ... }
  isEmpty(): boolean { ... }
}

class Order {
  constructor(
    private items: OrderItems,
    private customerId: CustomerId
  ) {}
}

// Fine in simpler contexts:
class Order {
  items: OrderItem[] = [];
  customerId: string;
}
```

### 5. One Dot per Line (Law of Demeter)

Don't chain through object graphs.

```typescript
// BAD: Train wreck
const city = order.customer.address.city;

// GOOD: Tell, don't ask
const city = order.getShippingCity();
```

### 6. Don't Abbreviate

If a name is too long to type, the class is doing too much.

```typescript
// BAD
const custRepo = new CustRepo();
const ord = new Ord();

// GOOD
const customerRepository = new CustomerRepository();
const order = new Order();
```

### 7. Keep All Entities Small

This is an **exercise rule** to develop the habit of splitting responsibilities early. In production, focus on the underlying principle: if a class is growing large, check whether it has multiple responsibilities and split if so. Specific size limits are a heuristic, not a law.

- Classes growing beyond ~100–150 lines are worth reviewing for SRP
- Methods that don't fit on one screen are worth considering for extraction
- Files with many unrelated classes can usually be split by concept

If a class is 60 lines and has one clear responsibility, it's fine. If it's 40 lines and does three things, it needs splitting.

### 8. No Classes with More Than Two Instance Variables

This is an **exercise rule only** — not a production mandate. Practicing it forces you to decompose classes aggressively and builds intuition for cohesion. In production, a cohesive class with 3–5 related fields is perfectly fine. The real signal is whether the fields belong together under one responsibility, not whether you've hit an arbitrary count.

```typescript
// Exercise: Forces decomposition to build intuition
class Order {
  constructor(
    private id: OrderId,
    private details: OrderDetails  // 2 variables max in exercise
  ) {}
}

// Production: This is fine if these fields are cohesive
class Order {
  constructor(
    private id: OrderId,
    private customerId: CustomerId,
    private items: OrderItem[],
    private status: OrderStatus,
    private createdAt: Date
  ) {}
}
```

### 9. No Getters/Setters/Properties

The principle: **prefer behavior-rich objects over data bags**. Objects should tell you what they can do, not just expose their fields. In practice, DTOs, serialization, and framework integration often legitimately need getters — that's fine. The smell to avoid is a class that only has getters/setters and pushes all logic into its callers.

```typescript
// BAD: Data bag — logic lives in the caller
class Account {
  getBalance(): number { return this.balance; }
  setBalance(value: number) { this.balance = value; }
}

// Caller does the work (fragile, scattered logic)
if (account.getBalance() >= amount) {
  account.setBalance(account.getBalance() - amount);
}

// GOOD: Behavior-rich object — logic lives in the object
class Account {
  withdraw(amount: Money): WithdrawResult {
    if (!this.canWithdraw(amount)) {
      return WithdrawResult.insufficientFunds();
    }
    this.balance = this.balance.subtract(amount);
    return WithdrawResult.success();
  }
}

// Caller tells, object decides
const result = account.withdraw(amount);

// Fine: DTOs with getters for serialization or framework integration
class UserDTO {
  constructor(readonly id: string, readonly email: string) {}
}
```

---

## Comments

### When to Write Comments

**Only write comments to explain WHY, not WHAT or HOW.**

Code explains what and how. Comments explain business reasons, non-obvious decisions, or warnings.

```typescript
// BAD: Explains what (redundant)
// Add 1 to counter
counter++;

// GOOD: Explains why
// Compensate for 0-based indexing in legacy API
counter++;
```

### Prefer Self-Documenting Code

Instead of commenting, rename to make intent clear.

```typescript
// BAD: Comment needed
// Check if user can access premium features
if (user.subscriptionLevel >= 2 && !user.isBanned) { }

// GOOD: Self-documenting
if (user.canAccessPremiumFeatures()) { }
```

---

## Formatting

### Vertical Spacing
- Related code together
- Blank lines between concepts
- Most important/public at top

### Horizontal Spacing
- Consistent indentation
- Space around operators
- Max line length ~80-120 characters

### Storytelling
Code should read top-to-bottom like a story. High-level at top, details below.

```typescript
class OrderProcessor {
  // Public API first
  process(order: Order): ProcessResult {
    this.validate(order);
    this.calculateTotals(order);
    return this.save(order);
  }

  // Supporting methods below, in order of appearance
  private validate(order: Order): void { ... }
  private calculateTotals(order: Order): void { ... }
  private save(order: Order): ProcessResult { ... }
}
```
