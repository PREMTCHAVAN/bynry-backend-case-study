# Backend Engineering Intern – Case Study
Candidate: Prem Chavan

---

## Part 1: Code Review & Debugging

### Identify Issues
- Code does not check if required data is provided.
- SKU uniqueness is not validated.
- Product is linked to only one warehouse.
- Product and inventory are saved separately.
- Price is not handled properly for decimal values.

### Explain Impact
- Missing data can crash the application.
- Duplicate SKUs can cause confusion.
- Multiple warehouse support is not possible.
- Data inconsistency can occur.
- Incorrect price calculations may happen.

### Provide Fixes
- Validate input data before saving.
- Enforce SKU uniqueness.
- Use a separate inventory table.
- Use a single database transaction.
- Handle price using decimal-safe types.

---

## Part 2: Database Design

### Schema
Company (comp_id, comp_name)  
Warehouse (w_id, w_name, comp_id)  
Product (product_id, product_name, sku, price)  
Inventory (inventory_id, product_id, w_id, quantity)  
Supplier (supplier_id, supplier_name, email)  
Product_Supplier (product_id, supplier_id)  
Bundle (bundle_product_id, child_product_id, quantity)

### Gaps
- Can bundle products be sold separately?
- Can a warehouse belong to more than one company?
- How long should inventory data be stored?
- What happens when inventory reaches zero?

### Decisions
- Separate Company and Warehouse tables support scalability.
- Inventory table supports multiple warehouses.
- Product_Supplier table manages supplier relationships.
- Bundle table supports composite products.
- SKU should be unique to avoid duplicates.

---

## Part 3: API Implementation

### Assumptions
- Each product has a low-stock threshold.
- Recent sales mean sales in the last 30 days.
- One product has one main supplier.
- Inventory is tracked per warehouse.

### Approach
- Get all warehouses of the company.
- Check inventory for each warehouse.
- If stock is low and product has recent sales, generate alert.
- Add supplier details.
- Return alert list.

### Implementation (Logic)

```python
def low_stock_alerts(request, company_id):
    alerts = []
    warehouses = Warehouse.objects.filter(company_id=company_id)

    for warehouse in warehouses:
        inventories = Inventory.objects.filter(warehouse=warehouse)
        for item in inventories:
            product = item.product
            if item.quantity < product.low_stock_threshold:
                if product.has_recent_sales():
                    alerts.append({
                        "product_id": product.id,
                        "product_name": product.name,
                        "sku": product.sku,
                        "warehouse_id": warehouse.id,
                        "warehouse_name": warehouse.name,
                        "current_stock": item.quantity,
                        "threshold": product.low_stock_threshold,
                        "days_until_stockout": 10,
                        "supplier": {
                            "id": product.supplier.id,
                            "name": product.supplier.name,
                            "contact_email": product.supplier.contact_email
                        }
                    })
    return {
        "alerts": alerts,
        "total_alerts": len(alerts)
    }
```
## Case Study Document
Google Docs Link: [[https://docs.google.com/your-doc-link](https://docs.google.com/document/d/1YIqutJmqu3sD5pU-I-78qY0-Hq8_Ovt5b_oLIPj0Lbk/edit?usp=sharing)](https://docs.google.com/document/d/1YIqutJmqu3sD5pU-I-78qY0-Hq8_Ovt5b_oLIPj0Lbk/edit?usp=sharing)
