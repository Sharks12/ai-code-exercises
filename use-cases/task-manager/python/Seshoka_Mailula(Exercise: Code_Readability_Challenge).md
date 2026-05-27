# Technical Refactoring and Decomposition Report

---

## 1. Code Readability Challenge (E-Commerce Discount Calculator)

The original `discount` function suffered from poor formatting, cryptic variable names (`d`, `tot`, `p`), and a lack of logical structure.

**Refactoring Strategy:**

- **Variable Renaming:** Renamed `d` to `max_discount`, `tot` to `cart_total`, and `promos` to `active_promotions`.
- **Structure:** Extracted the VIP/Member status logic and the promo application logic into distinct, readable blocks.
- **Benefits:** The code now clearly differentiates between percentage, fixed, and shipping discounts.

**Improved Logic Structure:**

```python
def calculate_best_discount(cart, promotions, user):
    cart_total = sum(item['price'] * item['quantity'] for item in cart)
    max_discount = 0

    # 1. Apply Promotional Discounts
    for promo in promotions:
        if cart_total >= promo.get('min_purchase', 0):
            if promo['type'] == 'percent':
                max_discount = max(max_discount, cart_total * (promo['value'] / 100))
            elif promo['type'] == 'fixed':
                max_discount = max(max_discount, min(promo['value'], cart_total))

    # 2. Apply User Status Discounts
    if user['status'] == 'vip':
        max_discount = max(max_discount, cart_total * 0.05)
    elif user['status'] == 'member' and user['months'] > 6:
        max_discount = max(max_discount, cart_total * 0.02)

    return {
        'original': cart_total,
        'discount': max_discount,
        'final': cart_total - max_discount
    }
```

---

## 2. Function Decomposition Challenge (Sales Report)

The original `generate_sales_report` function was a "God Function" handling everything from validation to PDF generation.

**Decomposition Plan:**

| Responsibility | Proposed Helper Function |
| :--- | :--- |
| Validation | `validate_report_params` |
| Filtering | `filter_sales_by_date` |
| Calculations | `calculate_metrics` |
| Grouping | `group_and_calculate_averages` |
| Exporting | `export_report` |

---

## 3. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a 200+ line function into a logical "story" that any developer can understand in seconds.

**What was the most challenging part?**

Managing the shared data state. Ensuring each helper function handled only its piece of the data without side effects required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This utility can be shared across all reporting modules in the system.

**How did decomposing affect testing?**

It allowed for granular unit testing. I can now test `calculate_metrics` in isolation without needing the full report generation pipeline.

**What did you learn about function size?**

If a function name requires the word "and" (e.g., "process and validate"), it is a sign that it is time to decompose.# Technical Refactoring and Decomposition Report

---

## 1. Code Readability Challenge (E-Commerce Discount Calculator)

The original `discount` function suffered from poor formatting, cryptic variable names (`d`, `tot`, `p`), and a lack of logical structure.

**Refactoring Strategy:**

- **Variable Renaming:** Renamed `d` to `max_discount`, `tot` to `cart_total`, and `promos` to `active_promotions`.
- **Structure:** Extracted the VIP/Member status logic and the promo application logic into distinct, readable blocks.
- **Benefits:** The code now clearly differentiates between percentage, fixed, and shipping discounts.

**Improved Logic Structure:**

```python
def calculate_best_discount(cart, promotions, user):
    cart_total = sum(item['price'] * item['quantity'] for item in cart)
    max_discount = 0

    # 1. Apply Promotional Discounts
    for promo in promotions:
        if cart_total >= promo.get('min_purchase', 0):
            if promo['type'] == 'percent':
                max_discount = max(max_discount, cart_total * (promo['value'] / 100))
            elif promo['type'] == 'fixed':
                max_discount = max(max_discount, min(promo['value'], cart_total))

    # 2. Apply User Status Discounts
    if user['status'] == 'vip':
        max_discount = max(max_discount, cart_total * 0.05)
    elif user['status'] == 'member' and user['months'] > 6:
        max_discount = max(max_discount, cart_total * 0.02)

    return {
        'original': cart_total,
        'discount': max_discount,
        'final': cart_total - max_discount
    }
```

---

## 2. Function Decomposition Challenge (Sales Report)

The original `generate_sales_report` function was a "God Function" handling everything from validation to PDF generation.

**Decomposition Plan:**

| Responsibility | Proposed Helper Function |
| :--- | :--- |
| Validation | `validate_report_params` |
| Filtering | `filter_sales_by_date` |
| Calculations | `calculate_metrics` |
| Grouping | `group_and_calculate_averages` |
| Exporting | `export_report` |

---

## 3. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a 200+ line function into a logical "story" that any developer can understand in seconds.

**What was the most challenging part?**

Managing the shared data state. Ensuring each helper function handled only its piece of the data without side effects required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This utility can be shared across all reporting modules in the system.

**How did decomposing affect testing?**

It allowed for granular unit testing. I can now test `calculate_metrics` in isolation without needing the full report generation pipeline.

**What did you learn about function size?**

If a function name requires the word "and" (e.g., "process and validate"), it is a sign that it is time to decompose.# Technical Refactoring and Decomposition Report

---

## 1. Code Readability Challenge (E-Commerce Discount Calculator)

The original `discount` function suffered from poor formatting, cryptic variable names (`d`, `tot`, `p`), and a lack of logical structure.

**Refactoring Strategy:**

- **Variable Renaming:** Renamed `d` to `max_discount`, `tot` to `cart_total`, and `promos` to `active_promotions`.
- **Structure:** Extracted the VIP/Member status logic and the promo application logic into distinct, readable blocks.
- **Benefits:** The code now clearly differentiates between percentage, fixed, and shipping discounts.

**Improved Logic Structure:**

```python
def calculate_best_discount(cart, promotions, user):
    cart_total = sum(item['price'] * item['quantity'] for item in cart)
    max_discount = 0

    # 1. Apply Promotional Discounts
    for promo in promotions:
        if cart_total >= promo.get('min_purchase', 0):
            if promo['type'] == 'percent':
                max_discount = max(max_discount, cart_total * (promo['value'] / 100))
            elif promo['type'] == 'fixed':
                max_discount = max(max_discount, min(promo['value'], cart_total))

    # 2. Apply User Status Discounts
    if user['status'] == 'vip':
        max_discount = max(max_discount, cart_total * 0.05)
    elif user['status'] == 'member' and user['months'] > 6:
        max_discount = max(max_discount, cart_total * 0.02)

    return {
        'original': cart_total,
        'discount': max_discount,
        'final': cart_total - max_discount
    }
```

---

## 2. Function Decomposition Challenge (Sales Report)

The original `generate_sales_report` function was a "God Function" handling everything from validation to PDF generation.

**Decomposition Plan:**

| Responsibility | Proposed Helper Function |
| :--- | :--- |
| Validation | `validate_report_params` |
| Filtering | `filter_sales_by_date` |
| Calculations | `calculate_metrics` |
| Grouping | `group_and_calculate_averages` |
| Exporting | `export_report` |

---

## 3. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a 200+ line function into a logical "story" that any developer can understand in seconds.

**What was the most challenging part?**

Managing the shared data state. Ensuring each helper function handled only its piece of the data without side effects required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This utility can be shared across all reporting modules in the system.

**How did decomposing affect testing?**

It allowed for granular unit testing. I can now test `calculate_metrics` in isolation without needing the full report generation pipeline.

**What did you learn about function size?**

If a function name requires the word "and" (e.g., "process and validate"), it is a sign that it is time to decompose.# Technical Refactoring and Decomposition Report

---

## 1. Code Readability Challenge (E-Commerce Discount Calculator)

The original `discount` function suffered from poor formatting, cryptic variable names (`d`, `tot`, `p`), and a lack of logical structure.

**Refactoring Strategy:**

- **Variable Renaming:** Renamed `d` to `max_discount`, `tot` to `cart_total`, and `promos` to `active_promotions`.
- **Structure:** Extracted the VIP/Member status logic and the promo application logic into distinct, readable blocks.
- **Benefits:** The code now clearly differentiates between percentage, fixed, and shipping discounts.

**Improved Logic Structure:**

```python
def calculate_best_discount(cart, promotions, user):
    cart_total = sum(item['price'] * item['quantity'] for item in cart)
    max_discount = 0

    # 1. Apply Promotional Discounts
    for promo in promotions:
        if cart_total >= promo.get('min_purchase', 0):
            if promo['type'] == 'percent':
                max_discount = max(max_discount, cart_total * (promo['value'] / 100))
            elif promo['type'] == 'fixed':
                max_discount = max(max_discount, min(promo['value'], cart_total))

    # 2. Apply User Status Discounts
    if user['status'] == 'vip':
        max_discount = max(max_discount, cart_total * 0.05)
    elif user['status'] == 'member' and user['months'] > 6:
        max_discount = max(max_discount, cart_total * 0.02)

    return {
        'original': cart_total,
        'discount': max_discount,
        'final': cart_total - max_discount
    }
```

---

## 2. Function Decomposition Challenge (Sales Report)

The original `generate_sales_report` function was a "God Function" handling everything from validation to PDF generation.

**Decomposition Plan:**

| Responsibility | Proposed Helper Function |
| :--- | :--- |
| Validation | `validate_report_params` |
| Filtering | `filter_sales_by_date` |
| Calculations | `calculate_metrics` |
| Grouping | `group_and_calculate_averages` |
| Exporting | `export_report` |

---

## 3. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a 200+ line function into a logical "story" that any developer can understand in seconds.

**What was the most challenging part?**

Managing the shared data state. Ensuring each helper function handled only its piece of the data without side effects required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This utility can be shared across all reporting modules in the system.

**How did decomposing affect testing?**

It allowed for granular unit testing. I can now test `calculate_metrics` in isolation without needing the full report generation pipeline.

**What did you learn about function size?**

If a function name requires the word "and" (e.g., "process and validate"), it is a sign that it is time to decompose.# Technical Refactoring and Decomposition Report

---

## 1. Code Readability Challenge (E-Commerce Discount Calculator)

The original `discount` function suffered from poor formatting, cryptic variable names (`d`, `tot`, `p`), and a lack of logical structure.

**Refactoring Strategy:**

- **Variable Renaming:** Renamed `d` to `max_discount`, `tot` to `cart_total`, and `promos` to `active_promotions`.
- **Structure:** Extracted the VIP/Member status logic and the promo application logic into distinct, readable blocks.
- **Benefits:** The code now clearly differentiates between percentage, fixed, and shipping discounts.

**Improved Logic Structure:**

```python
def calculate_best_discount(cart, promotions, user):
    cart_total = sum(item['price'] * item['quantity'] for item in cart)
    max_discount = 0

    # 1. Apply Promotional Discounts
    for promo in promotions:
        if cart_total >= promo.get('min_purchase', 0):
            if promo['type'] == 'percent':
                max_discount = max(max_discount, cart_total * (promo['value'] / 100))
            elif promo['type'] == 'fixed':
                max_discount = max(max_discount, min(promo['value'], cart_total))

    # 2. Apply User Status Discounts
    if user['status'] == 'vip':
        max_discount = max(max_discount, cart_total * 0.05)
    elif user['status'] == 'member' and user['months'] > 6:
        max_discount = max(max_discount, cart_total * 0.02)

    return {
        'original': cart_total,
        'discount': max_discount,
        'final': cart_total - max_discount
    }
```

---

## 2. Function Decomposition Challenge (Sales Report)

The original `generate_sales_report` function was a "God Function" handling everything from validation to PDF generation.

**Decomposition Plan:**

| Responsibility | Proposed Helper Function |
| :--- | :--- |
| Validation | `validate_report_params` |
| Filtering | `filter_sales_by_date` |
| Calculations | `calculate_metrics` |
| Grouping | `group_and_calculate_averages` |
| Exporting | `export_report` |

---

## 3. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a 200+ line function into a logical "story" that any developer can understand in seconds.

**What was the most challenging part?**

Managing the shared data state. Ensuring each helper function handled only its piece of the data without side effects required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This utility can be shared across all reporting modules in the system.

**How did decomposing affect testing?**

It allowed for granular unit testing. I can now test `calculate_metrics` in isolation without needing the full report generation pipeline.

**What did you learn about function size?**

If a function name requires the word "and" (e.g., "process and validate"), it is a sign that it is time to decompose.v
