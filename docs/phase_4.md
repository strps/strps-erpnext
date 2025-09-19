# Phase 4: Invoicing and Payments Integration

This phase implements invoicing and payment processing for the tattoo shop, enabling automatic generation of ERPNext `Sales Invoice` records for completed appointments (including products used), configuring payment gateways for deposits and full payments, and setting up accounting categories in the Chart of Accounts. Estimated timeline: 2-4 days.

## Task Breakdown

### Task 1: Add Auto-Invoicing for Appointments
- **Description**: Add a custom button and script to the `Appointment` DocType to generate a `Sales Invoice` when an appointment is marked "Completed," including charges for the tattoo session and any products used (from the `products_used` child table).
- **Subtasks**:
  - **Add Custom Button**: In Desk, go to **Customize Form > Appointment**. Add a custom button labeled "Generate Invoice" visible when `status` is "Completed". (15 minutes)
  - **Create Client-Side Script**: In `tattoo_shop/public/js/appointment.js`, add a script to create a `Sales Invoice` with a service item (e.g., "Tattoo Session") and products from `products_used`:
    ```javascript
    frappe.ui.form.on('Appointment', {
        refresh: function(frm) {
            if (frm.doc.status === 'Completed' && !frm.doc.invoice_generated) {
                frm.add_custom_button(__('Generate Invoice'), function() {
                    frappe.model.with_doctype('Sales Invoice', function() {
                        let inv = frappe.model.get_new_doc('Sales Invoice');
                        inv.customer = frm.doc.customer;
                        inv.items = [
                            { item_code: 'TATTOO-SERVICE', qty: 1, rate: frm.doc.duration * 2 }, // Example: $2/min
                            ...(frm.doc.products_used || []).map(p => ({
                                item_code: p.item_code,
                                qty: p.qty,
                                rate: p.rate
                            }))
                        ];
                        frappe.set_route('Form', 'Sales Invoice', inv.name);
                    });
                });
            }
        }
    });
    ```
    (1 hour)
  - **Add Server-Side Flag**: In `Appointment` DocType, add a custom field `invoice_generated` (Check) to prevent duplicate invoices. Update `appointment.py` to set this flag on invoice creation:
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
                        "items": [{"item_code": product.item_code, "qty": product.qty, "s_warehouse": "Shop Storage"}]
                    }).insert().submit()
        def after_insert(self):
            if self.status == "Completed" and not self.invoice_generated:
                inv = frappe.get_doc({
                    "doctype": "Sales Invoice",
                    "customer": self.customer,
                    "items": [
                        {"item_code": "TATTOO-SERVICE", "qty": 1, "rate": self.duration * 2},
                        *(self.products_used or []).map(lambda p: {
                            "item_code": p.item_code,
                            "qty": p.qty,
                            "rate": p.rate
                        })
                    ]
                }).insert()
                self.invoice_generated = 1
                self.save()
    ```
    (1 hour)
  - **Create Service Item**: In **Stock > Item > New**, create “TATTOO-SERVICE” (non-stock item, `is_stock_item` unchecked, linked to “Services” Item Group). (15 minutes)
  - **Test Invoicing**: Complete an appointment with products, click “Generate Invoice,” verify `Sales Invoice` creation with correct items and amounts. Check stock updates and `invoice_generated` flag. (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **3 hours 15 minutes**
- **Notes**: Adjust the rate calculation (e.g., `duration * 2`) based on your pricing model. Ensure “TATTOO-SERVICE” item exists before testing.

### Task 2: Configure Payment Gateways and Deposits
- **Description**: Set up a payment gateway (e.g., Stripe) for processing payments and handle deposits for appointments via ERPNext’s `Payment Entry`.
- **Subtasks**:
  - **Configure Payment Gateway**: In Desk, go to **Setup > Integrations > Payment Gateway**. Select Stripe, add API keys (obtain from Stripe dashboard), and enable for online payments. (30 minutes)
  - **Add Deposit Field**: In **Customize Form > Appointment**, ensure `deposit_required` (Check) and add `deposit_amount` (Currency) field. (15 minutes)
  - **Create Payment Request Script**: In `appointment.py`, add logic to create a `Payment Request` for deposits when `deposit_required` is checked:
    ```python
    import frappe
    from frappe.model.document import Document

    class Appointment(Document):
        def before_save(self):
            if self.deposit_required and self.deposit_amount and not self.payment_request:
                pr = frappe.get_doc({
                    "doctype": "Payment Request",
                    "payment_gateway_account": "Stripe",  # Adjust based on setup
                    "reference_doctype": "Appointment",
                    "reference_name": self.name,
                    "amount": self.deposit_amount,
                    "currency": "USD",
                    "email_to": self.customer_email
                }).insert()
                self.payment_request = pr.name
    ```
    (1 hour)
  - **Link to Sales Invoice**: In **Selling > Sales Invoice**, ensure invoices can link to `Payment Request` for full payments via the gateway. (20 minutes)
  - **Test Payments**: Create an appointment with `deposit_required`, verify `Payment Request` email/link, simulate payment via Stripe test mode. Complete an invoice and test full payment. (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **3 hours**
- **Notes**: Requires a Stripe account (or similar gateway). Test in Stripe’s sandbox mode first. Ensure ERPNext’s Email Account is configured for `Payment Request` emails.

### Task 3: Set Up Accounting Categories
- **Description**: Configure ERPNext’s Chart of Accounts to track income from tattoo services and product sales, ensuring financial transactions are properly categorized.
- **Subtasks**:
  - **Customize Chart of Accounts**: In **Accounts > Chart of Accounts**, add accounts under “Income”:
    - “Tattoo Services Income” (Type: Income Account)
    - “Product Sales Income” (Type: Income Account)
    Link to company. (30 minutes)
  - **Link Items to Accounts**: In **Stock > Item**, update “TATTOO-SERVICE” to link to “Tattoo Services Income” and product items (e.g., Black Ink) to “Product Sales Income” via `default_income_account`. (20 minutes)
  - **Test Accounting Entries**: Submit a `Sales Invoice` (from Task 1), verify General Ledger entries reflect correct accounts (Accounts > General Ledger). (30 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **1 hour 35 minutes**
- **Notes**: Ensure company is selected in all accounts. Use ERPNext’s default Chart of Accounts template as a base to avoid errors.

## Total Estimated Time
- **Task 1**: 3 hours 15 minutes
- **Task 2**: 3 hours
- **Task 3**: 1 hour 35 minutes
- **Total**: **7 hours 50 minutes** (~2-4 days with breaks, testing, or minor issues)

## Notes and Best Practices
- **Environment**: Work in a development bench with `developer_mode: 1`. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`).
- **Dependencies**: Requires Phase 1 (`Customer`, `Artist`, `Appointment`, `Artist Availability`), Phase 2 (appointment system), and Phase 3 (product store, including `products_used` child table and “TATTOO-SERVICE” item).
- **Testing**: Use test customers, appointments, and payments (in Stripe test mode) to verify end-to-end flow. Check General Ledger for accounting accuracy.
- **Potential Issues**:
  - Invoice duplication: Ensure `invoice_generated` flag prevents multiple invoices.
  - Payment gateway errors: Verify API keys and SMTP setup for emails.
  - Accounting mismatches: Confirm all items link to correct income accounts.
- **Resources**: Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).