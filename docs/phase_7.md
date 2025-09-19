# Phase 7: Consuming Ministerio de Hacienda API for Invoices and Taxes

This phase integrates the tattoo shop management system with the Ministerio de Hacienda’s ATV API to generate, submit, and manage electronic invoices (comprobantes electrónicos) compliant with Costa Rican tax regulations. It extends ERPNext’s `Sales Invoice` to create Hacienda-compliant XML, authenticates with ATV credentials, and handles responses (e.g., acceptance, rejection). Estimated timeline: 3-5 days for a solo developer with moderate Frappe/ERPNext experience.

## Task Breakdown

### Task 1: Obtain and Configure ATV Credentials
- **Description**: Register the tattoo shop in ATV, obtain the cryptographic key, PIN, and API credentials, and configure them in the system for secure API access.
- **Subtasks**:
  - **Register in ATV**: Access `https://atv.hacienda.go.cr/ATV/Login.aspx`, create an account for the tattoo shop (or use existing). Select “Obligado Tributario” or “Representante Legal” profile as per. (30 minutes)[](https://help.hulipractice.com/es/articles/1258842-obtener-datos-de-la-administracion-tributaria-virtual-atv-de-costa-rica)
  - **Generate Cryptographic Key and PIN**: In ATV, go to **Comprobantes Electrónicos > Llave Criptográfica de Producción**, generate key, set 4-digit PIN, and download `.p12` file. Save securely. (30 minutes)
  - **Generate API Credentials**: In ATV, go to **Comprobantes Electrónicos > Generar nueva contraseña en producción**, obtain username (e.g., `cpf-xxxxxxxxx@prod.comprobanteselectronicos.go.cr`) and password. (20 minutes)[](https://help.fygaro.com/es/article/obtener-credenciales-de-hacienda-n2c62q/)
  - **Store Credentials in Frappe**: Create a custom DocType `Hacienda Settings` in `tattoo_shop`:
    - Fields: `atv_username` (Data), `atv_password` (Password), `crypto_key` (Attach, for `.p12` file), `pin` (Password), `environment` (Select: Test, Production)
    - Module: Tattoo Shop
    - Permissions: Tattoo Shop Admin (all)
    Save and run `bench migrate`. (45 minutes)
  - **Test Credentials**: Manually test API authentication using a tool like Postman with the credentials and endpoint `https://api.comprobanteselectronicos.go.cr/recepcion/v1/` to ensure access. (30 minutes)
- **Estimated Time**: **2 hours 35 minutes**
- **Notes**: Store credentials securely; avoid committing to Git. Use test environment (`idp.hacienda.go.cr`) first, as per Hacienda’s sandbox guidelines.[](https://atv.hacienda.go.cr/ATV/ComprobanteElectronico/docs/esquemas/2024/v4.4/comprobantes-electronicos-api.html)

### Task 2: Extend Sales Invoice for Hacienda XML Generation
- **Description**: Customize ERPNext’s `Sales Invoice` to generate XML compliant with Hacienda’s Version 4.3/4.4 structure for factura electrónica, including required fields like clave, issuer, and taxes.
- **Subtasks**:
  - **Add Custom Fields**: In Desk, go to **Customize Form > Sales Invoice**. Add fields:
    - `hacienda_clave` (Data, Read-Only): Unique invoice key (e.g., 506DDMMYY... format)
    - `hacienda_status` (Select: Draft, Sent, Accepted, Rejected)
    - `hacienda_xml` (Long Text, Read-Only): Store generated XML
    - `hacienda_response` (Long Text, Read-Only): Store API response
    (20 minutes)
  - **Generate XML**: Create a Python script in `tattoo_shop/tattoo_shop/hacienda_utils.py` to generate XML based on Version 4.3/4.4 structure (from `https://atv.hacienda.go.cr/ATV/ComprobanteElectronico/frmAnexosyEstructuras.aspx`):
    ```python
    import frappe
    import xml.etree.ElementTree as ET
    from datetime import datetime
    import base64

    def generate_hacienda_xml(invoice):
        root = ET.Element("FacturaElectronica", xmlns="https://cdn.comprobanteselectronicos.go.cr/xml/v4.3/facturaelectronica")
        clave = f"506{datetime.now().strftime('%d%m%y')}{invoice.name[-10:]}00100001010000000001"  # Simplified clave
        ET.SubElement(root, "Clave").text = clave
        ET.SubElement(root, "FechaEmision").text = datetime.now().strftime("%Y-%m-%dT%H:%M:%S-06:00")
        # Add issuer, receiver, items, taxes (based on Version 4.3 schema)
        issuer = ET.SubElement(root, "Emisor")
        ET.SubElement(issuer, "Nombre").text = frappe.db.get_value("Company", invoice.company, "company_name")
        # Add more fields as per schema (e.g., Identificacion, Ubicacion)
        for item in invoice.items:
            detalle = ET.SubElement(root, "DetalleServicio")
            linea = ET.SubElement(detalle, "LineaDetalle")
            ET.SubElement(linea, "Cantidad").text = str(item.qty)
            ET.SubElement(linea, "Descripcion").text = item.item_name
            ET.SubElement(linea, "PrecioUnitario").text = str(item.rate)
            # Add tax fields (e.g., 13% IVA for services)
        xml_str = ET.tostring(root, encoding="unicode")
        frappe.db.set_value("Sales Invoice", invoice.name, {"hacienda_clave": clave, "hacienda_xml": xml_str})
        return xml_str, clave
    ```
    (1 hour 30 minutes)
  - **Trigger XML Generation**: Update `tattoo_shop/tattoo_shop/doctype/appointment/appointment.py` to call `generate_hacienda_xml` when invoice is created (extend Phase 4 logic):
    ```python
    import frappe
    from frappe.model.document import Document
    from tattoo_shop.tattoo_shop.hacienda_utils import generate_hacienda_xml

    class Appointment(Document):
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
                xml, clave = generate_hacienda_xml(inv)
                inv.hacienda_clave = clave
                inv.hacienda_xml = xml
                inv.save()
                self.invoice_generated = 1
                self.save()
    ```
    (45 minutes)
  - **Test XML Generation**: Create a test appointment, complete it, generate invoice, verify XML structure against Version 4.3/4.4 schema (use Hacienda’s validator if available). (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **3 hours 35 minutes**
- **Notes**: XML structure must match Hacienda’s schema (e.g., Version 4.3). Include tax fields (e.g., 13% IVA) and company details. Validate XML manually before API submission.

### Task 3: Submit Invoices to Hacienda API
- **Description**: Implement API calls to submit invoices to Hacienda’s recepcion endpoint and handle responses (e.g., accepted, rejected).
- **Subtasks**:
  - **Add API Submission Logic**: In `hacienda_utils.py`, add a function to sign and submit XML using cryptographic key and PIN:
    ```python
    import frappe
    import requests
    from OpenSSL import crypto
    import base64

    def submit_hacienda_invoice(invoice):
        settings = frappe.get_doc("Hacienda Settings")
        url = "https://api.comprobanteselectronicos.go.cr/recepcion/v1/recepcion" if settings.environment == "Production" else "https://idp.hacienda.go.cr/recepcion/v1/recepcion"
        # Load .p12 key
        with open(settings.crypto_key, "rb") as f:
            p12 = crypto.load_pkcs12(f.read(), settings.pin.encode())
        # Sign XML (simplified; use proper signing library)
        xml = invoice.hacienda_xml
        headers = {
            "Authorization": f"Bearer {get_access_token(settings.atv_username, settings.atv_password)}",
            "Content-Type": "application/xml"
        }
        response = requests.post(url, data=xml, headers=headers)
        response_data = response.json()
        frappe.db.set_value("Sales Invoice", invoice.name, {
            "hacienda_status": "Accepted" if response.status_code == 200 else "Rejected",
            "hacienda_response": str(response_data)
        })
        return response_data

    def get_access_token(username, password):
        # Simplified; use OpenID Connect for OAuth 2.0 token
        token_url = "https://idp.hacienda.go.cr/auth/realms/<realm>/protocol/openid-connect/token"
        response = requests.post(token_url, data={
            "grant_type": "password",
            "username": username,
            "password": password,
            "client_id": "api-prod"  # Adjust based on Hacienda docs
        })
        return response.json().get("access_token")
    ```
    (1 hour 30 minutes)
  - **Add Submission Button**: In **Customize Form > Sales Invoice**, add a custom button “Submit to Hacienda” to trigger `submit_hacienda_invoice` when `hacienda_status` is “Draft”. (20 minutes)
  - **Test API Submission**: Use Hacienda’s test environment to submit a test invoice. Verify response updates `hacienda_status` and `hacienda_response`. Check for errors (e.g., duplicate clave). (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 50 minutes**
- **Notes**: Requires `requests` and `pyOpenSSL` (`pip install requests pyOpenSSL`). Use Hacienda’s test endpoint first. Handle rate limits (check `remaining_requests` in API response).[](https://atv.hacienda.go.cr/ATV/ComprobanteElectronico/docs/esquemas/2024/v4.4/comprobantes-electronicos-api.html)

### Task 4: Handle Hacienda Responses and Tax Compliance
- **Description**: Process API responses, update invoice statuses, and ensure tax compliance (e.g., IVA calculations, reporting).
- **Subtasks**:
  - **Process Responses**: Extend `submit_hacienda_invoice` to parse API responses (e.g., “accepted”, “rejected”, error codes) and update `Sales Invoice` accordingly. Log errors for admin review. (45 minutes)
  - **Add Tax Calculations**: Ensure `Sales Invoice` items include correct tax rates (e.g., 13% IVA for services, configurable per item). In **Stock > Item**, set `taxes` field for each item (e.g., link to “IVA 13%” Tax Rule). (30 minutes)
  - **Create Tax Report**: In Desk, create a Query Report “Hacienda Tax Summary”:
    ```sql
    SELECT 
        i.hacienda_clave, 
        i.customer, 
        i.posting_date, 
        i.total_taxes_and_charges as tax_amount, 
        i.grand_total, 
        i.hacienda_status
    FROM `tabSales Invoice` i
    WHERE i.hacienda_status IN ('Accepted', 'Rejected')
    AND i.posting_date BETWEEN %(from_date)s AND %(to_date)s
    ```
    Add filters: `from_date`, `to_date`. (45 minutes)
  - **Test Compliance**: Submit test invoices, verify tax calculations and report accuracy. Check Hacienda status updates. (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **3 hours**
- **Notes**: Confirm IVA rates with local regulations. Use Hacienda’s validator to ensure XML compliance. Add error handling for rejected invoices.

## Total Estimated Time
- **Task 1**: 2 hours 35 minutes
- **Task 2**: 3 hours 35 minutes
- **Task 3**: 2 hours 50 minutes
- **Task 4**: 3 hours
- **Total**: **12 hours** (~3-5 days with breaks, testing, or minor issues)

## Notes and Best Practices
- **Environment**: Work in a development bench with `developer_mode: 1`. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`). Avoid committing sensitive credentials.
- **Dependencies**: Requires Phases 1-6 (`Customer`, `Artist`, `Appointment`, `Artist Availability`, `Sales Invoice`, `Item`, appointment system, product store, invoicing, reports, portal). Ensure SMTP and payment gateways are configured.
- **Testing**: Use Hacienda’s test environment (`idp.hacienda.go.cr`) for all API calls initially. Validate XML against Version 4.3/4.4 schemas. Test with multiple invoices to handle rate limits and errors.
- **Potential Issues**:
  - API authentication failures: Verify credentials and OAuth token endpoint.
  - XML errors: Cross-check with Hacienda’s schema documentation.
  - Rate limits: Monitor API response headers (`remaining_requests`, `reset_time`) and implement retry logic if needed.
- **Resources**: Hacienda ATV documentation (`https://atv.hacienda.go.cr`), Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).,,[](https://atv.hacienda.go.cr/ATV/ComprobanteElectronico/docs/esquemas/2024/v4.4/comprobantes-electronicos-api.html)[](https://help.hulipractice.com/es/articles/1258842-obtener-datos-de-la-administracion-tributaria-virtual-atv-de-costa-rica)[](https://help.fygaro.com/es/article/obtener-credenciales-de-hacienda-n2c62q/)

