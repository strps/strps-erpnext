# Phase 3: Product Store Implementation

This phase establishes the product store for the tattoo shop, enabling inventory management and sales of tattoo supplies (e.g., inks, needles) and merchandise (e.g., aftercare creams, t-shirts). It leverages ERPNext’s `Item` and `Stock` modules, integrates with the `Appointment` DocType for tracking materials used, and sets up point-of-sale (POS) functionality. An optional e-commerce setup is included for online sales. Estimated timeline: 3-5 days.

## Task Breakdown

### Task 1: Set Up Inventory with ERPNext Item and Warehouse
- **Description**: Create and categorize products in ERPNext’s `Item` DocType and set up a warehouse for inventory tracking.
- **Subtasks**:
  - **Create Item Groups**: In Desk, go to **Stock > Item Group > New**. Create groups like “Tattoo Supplies” (for inks, needles) and “Merchandise” (for t-shirts, creams). Set parent as “All Item Groups”. (30 minutes)
  - **Create Items**: In **Stock > Item > New**, create sample items (e.g., “Black Ink”, “Aftercare Cream 50ml”, “Studio T-Shirt”). Fields: `item_code`, `item_name`, `item_group`, `stock_uom` (e.g., Unit, Liter), `standard_rate`, `is_stock_item` (Check for stockable items). Add custom field `shelf_life` (Date) for supplies via Customize Form. (1 hour)
  - **Set Up Warehouse**: In **Stock > Warehouse > New**, create a warehouse (e.g., “Shop Storage”). Set type as “Warehouse” and link to company. (20 minutes)
  - **Add Initial Stock**: Use **Stock > Stock Entry > New** (Material Receipt) to add initial quantities for test items (e.g., 10 units of Black Ink). (30 minutes)
  - **Test Inventory**: Verify items appear in Item list and stock levels update in Stock Summary report. (20 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 55 minutes**
- **Notes**: Use meaningful `item_code` formats (e.g., “INK-BLK-001” for Black Ink). Ensure `is_stock_item` is unchecked for non-stock items like services.

### Task 2: Configure POS for In-Shop Sales
- **Description**: Set up ERPNext’s POS system for quick in-shop sales of products (e.g., aftercare creams sold post-appointment).
- **Subtasks**:
  - **Create POS Profile**: In Desk, go to **Selling > POS Profile > New**. Set:
    - Name: “Tattoo Shop POS”
    - Company: Your tattoo shop company
    - Warehouse: “Shop Storage”
    - Customer Group: “All Customer Groups”
    - Item Groups: “Tattoo Supplies”, “Merchandise”
    - Payment Methods: Add Cash, Card, etc.
    (45 minutes)
  - **Configure POS Settings**: In **Setup > POS Settings**, enable features like “Update Stock” and set default customer (e.g., “Walk-in Customer”). (20 minutes)
  - **Test POS**: Open POS interface (**Selling > Point of Sale**), simulate a sale (e.g., Aftercare Cream), verify stock deduction and invoice creation. (30 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **1 hour 50 minutes**
- **Notes**: Ensure a “Walk-in Customer” exists in ERPNext `Customer` DocType for POS sales without specific clients. Train staff on POS UI if needed.

### Task 3: Link Products to Appointments
- **Description**: Add a child table to the `Appointment` DocType to track products used during a session (e.g., ink, needles), enabling inventory updates and invoicing.
- **Subtasks**:
  - **Add Child Table**: In Desk, go to **Customize Form > Appointment**. Add a Table field `products_used` (Child DocType: create new `Appointment Product` with fields `item_code` (Link to Item), `qty` (Float), `rate` (Currency, fetched from Item)). (30 minutes)
  - **Update Server Logic**: In `tattoo_shop/tattoo_shop/doctype/appointment/appointment.py`, add logic to deduct stock when `status` changes to “Completed”:
    ```python
    import frappe
    from frappe.model.document import Document

    class Appointment(Document):
        def on_submit(self):
            if self.status == "Completed" and self.products_used:
                for product in self.products_used:
                    frappe.get_doc({
                        "doctype": "Stock Entry",
                        "stock_entry_type": "Material Issue",
                        "items": [{
                            "item_code": product.item_code,
                            "qty": product.qty,
                            "s_warehouse": "Shop Storage"
                        }]
                    }).insert().submit()
    ```
    (1 hour)
  - **Test Integration**: Create an appointment, add products used (e.g., 0.1L Black Ink), complete it, and verify stock deduction in Stock Summary. (30 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 15 minutes**
- **Notes**: Ensure stock entries respect warehouse settings. Handle negative stock scenarios by enabling “Allow Negative Stock” in **Stock Settings** if needed during testing.

### Task 4: Enable E-commerce (Optional)
- **Description**: Set up an online store for products using ERPNext’s Website module, allowing clients to browse and purchase items like merchandise or aftercare products.
- **Subtasks**:
  - **Enable Website Module**: In **Setup > Company**, ensure “Website” module is enabled. Activate Website settings in **Setup > Website Settings**. (20 minutes)
  - **Create Web Pages**: In **Website > Web Page > New**, create a product catalog page. Use Item Groups to organize products (e.g., “Merchandise”). Enable “Show in Website” for relevant Items. (45 minutes)
  - **Configure Shopping Cart**: In **Website > Shopping Cart Settings**, enable cart, set payment gateway (e.g., Stripe, via **Setup > Integrations**). (30 minutes)
  - **Test E-commerce**: Publish the catalog page (e.g., `/shop`), add items to cart, simulate checkout, and verify Sales Order creation. (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 35 minutes**
- **Notes**: This task is optional and can be skipped if e-commerce isn’t needed immediately. Customize the catalog’s appearance using Jinja templates or CSS in Web Page settings.

## Total Estimated Time
- **Task 1**: 2 hours 55 minutes
- **Task 2**: 1 hour 50 minutes
- **Task 3**: 2 hours 15 minutes
- **Task 4**: 2 hours 35 minutes (optional)
- **Total (without e-commerce)**: **7 hours**
- **Total (with e-commerce)**: **9 hours 35 minutes** (~3-5 days with breaks, testing, or minor issues)

## Notes and Best Practices
- **Environment**: Work in a development bench with `developer_mode: 1`. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`).
- **Dependencies**: Requires Phase 1 (`Customer`, `Artist`, `Appointment`, `Artist Availability`) and Phase 2 (appointment system) to be complete. ERPNext’s Stock and Selling modules must be active.
- **Testing**: Use test items and stock entries to verify inventory updates. Simulate POS and appointment scenarios with different users (Admin, Artist).
- **Potential Issues**:
  - Stock mismatches: Ensure warehouse is correctly linked in all operations.
  - E-commerce permissions: Verify guest access for web catalog if public-facing.
  - Child table errors: Check `Appointment Product` DocType is properly created before linking.
- **Resources**: Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).
