# Function Decomposition Challenge: Sales Report Generator

---

## 1. Overview

The original `generate_sales_report` function was a "God Function" that handled validation, data filtering, metrics calculation, grouping, forecasting, chart generation, and formatting. This complexity makes it difficult to test, debug, and maintain.

---

## 2. Decomposition Plan

I have identified the following distinct responsibilities to extract into helper functions:

| Responsibility | Proposed Helper Function | Purpose |
| :--- | :--- | :--- |
| Validation | `validate_report_params` | Validates data, types, and date ranges. |
| Filtering | `filter_sales_by_date` | Isolates sales data within a specific timeframe. |
| Metrics | `calculate_metrics` | Handles sum, average, min, and max calculations. |
| Grouping | `group_and_calculate_averages` | Aggregates data by category/product/region. |
| Forecasting | `generate_forecast_data` | Calculates trends and projects next 3 months. |
| Exporting | `export_report` | Dispatches to specific PDF/Excel/HTML generators. |

---

## 3. Refactored Function (Python Concept)

The main function now acts as a high-level orchestrator, making the business logic immediately readable:

```python
def generate_sales_report(sales_data, **params):
    # 1. Validation
    validate_report_params(sales_data, params)

    # 2. Preparation
    filtered_data = filter_sales_by_date(sales_data, params.get('date_range'))
    filtered_data = apply_filters(filtered_data, params.get('filters'))

    # 3. Calculation & Enrichment
    report_data = {
        'summary': calculate_metrics(filtered_data),
        'grouping': group_and_calculate_averages(filtered_data, params.get('grouping')),
        'forecast': generate_forecast_data(filtered_data) if params.get('report_type') == 'forecast' else None
    }

    # 4. Finalization
    return export_report(report_data, params.get('output_format'))
```

---

## 4. Reflection Questions

**How did breaking down the function improve readability?**

It transformed a monolithic 200+ line function into a logical, 15-line "story." Any developer can now see the steps of the process at a glance without being buried in the specific math of forecasting.

**What was the most challenging part of decomposing?**

Managing the shared data state. Ensuring that each helper function received the specific data subset it needed — without modifying the original `sales_data` list unexpectedly — required careful planning.

**Which extracted function would be most reusable?**

`filter_sales_by_date`. This logic is common to almost every financial report in the system, and having it as a standalone function prevents code duplication in other modules.
