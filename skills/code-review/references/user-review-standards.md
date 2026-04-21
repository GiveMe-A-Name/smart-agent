# User-Defined Review Standards

These are the owner's judgment dimensions for code review. They are not a mandatory checklist — use them as a reference when evaluating whether a design or implementation concern is worth raising. An issue that clearly violates one of these dimensions is worth surfacing; borderline cases require judgment.

---

## 1. Deep Module Principle

**Principle**: A good abstraction has a simple interface and hides substantial complexity. Interface complexity should be much smaller than the functionality provided.

**Violation signals**:
- Callers must understand the module's internals to use it correctly
- The interface has many parameters or requires a specific calling sequence to work
- Adding a method to the module requires changes in many call sites

**Example**:
```python
# Bad: caller must manage state the module should own
result = cache.get(key)
if result is None:
    result = compute(key)
    cache.set(key, result)
    cache.expire(key, ttl=300)

# Good: module owns the complexity
result = cache.get_or_compute(key, compute, ttl=300)
```

---

## 2. Self-Explanatory Interface

**Principle**: A caller should be able to use a function or module correctly without reading its implementation. The name and signature should fully communicate behavior and constraints.

**Violation signals**:
- Function name does not reflect its side effects
- Parameters require knowledge of internal implementation to provide correctly
- Return values have undocumented special cases or constraints

**Example**:
```python
# Bad: caller must read implementation to know save() sends email
user.save()  # also sends a welcome email if user is new — not obvious from the name

# Good: behavior is explicit at the call site
user.save()
if user.is_new:
    user.send_welcome_email()
```

---

## 3. Rule of Three

**Principle**: An abstraction is justified after the same pattern appears in 3 or more places. Extracting earlier is premature — the pattern hasn't proven itself yet.

**Violation signals**:
- A new helper or utility is introduced but used in only 1–2 places
- An abstraction was created "for future use" with no second concrete caller today
- A class or function could be replaced by a few inline lines at each use site without meaningful loss

**Example**:
```python
# Premature: only used once
def format_user_display_name(user):
    return f"{user.first_name} {user.last_name}"

# Justified: the same formatting appears in views, emails, reports, and export
# — extraction is now clearly warranted
```

---

## 4. DRY Violation

**Principle**: The same logic appearing in multiple places means a bug fix must be applied in multiple places. This is a design problem, not just a style concern.

**Violation signals**:
- The same computation or conditional appears in 2+ places and would need to be updated together
- A bug in one copy likely means the same bug exists elsewhere
- Copy-pasted code with minor surface variations but identical core logic

**Example**:
```python
# Bad: discount logic duplicated across checkout and invoice
# checkout.py
price = price * 0.9 if user.is_premium else price

# invoice.py
amount = amount * 0.9 if user.is_premium else amount

# Good: single source of truth
price = apply_user_discount(price, user)
```

---

## 5. Open/Closed Principle

**Principle**: A change in one place should not require changes in many other places. Modules should be open for extension but closed for modification. Tight coupling is the most common sign this principle is violated.

**Violation signals**:
- Adding a new case requires modifying multiple unrelated files
- Changing a module's internals forces changes to its callers
- Switch/if-else chains that must be extended every time a new type is added

**Example**:
```python
# Bad: adding a new payment method requires modifying this function
def process_payment(type, amount):
    if type == "card":
        ...
    elif type == "paypal":
        ...
    # every new type requires another elif here

# Good: new types extend without modifying core logic
handler = payment_registry.get(type)
handler.process(amount)
```

---

## 6. Complexity Has Two Sources

**Principle**: Code complexity comes from two sources — **dependencies** (how much surrounding context must be understood to read this code) and **obscurity** (information that is hidden or unclear). Reducing these two matters more than any other optimization.

**Violation signals**:
- A function cannot be understood without first reading several other files
- Names that don't reveal intent: single letters, overly generic names like `data`, `handle`, `process`
- Magic numbers or implicit state that requires external context to decode

**Example**:
```python
# Bad: high dependency + high obscurity
def calc(x, d):
    return x * (1 - CONFIG["r"]) if d > 90 else x

# Good: self-contained, intent is clear without context
def apply_late_payment_penalty(amount, days_overdue):
    PENALTY_RATE = 0.05
    PENALTY_THRESHOLD_DAYS = 90
    if days_overdue > PENALTY_THRESHOLD_DAYS:
        return amount * (1 - PENALTY_RATE)
    return amount
```
