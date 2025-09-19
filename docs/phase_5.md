# Phase 5: Reporting, Roles, and Portal

This phase implements reporting, user roles, and a client portal for the tattoo shop management system. It includes custom reports for appointments and artist performance, role-based permissions for secure access, and a client portal for booking, viewing appointments, and browsing products. Future permissions for artist-managed availability are planned. Estimated timeline: 3-5 days.

## Task Breakdown

### Task 1: Create Custom Reports
- **Description**: Develop custom reports to provide insights into upcoming appointments and artist performance, leveraging ERPNext’s reporting tools and supplementing with custom queries.
- **Subtasks**:
  - **Create Upcoming Appointments Report**: In Desk, go to **Report > New Report**. Set:
    - Name: “Upcoming Appointments”
    - Report Type: Query Report
    - Ref Doctype: `Appointment`
    - Query: 
      ```sql
      SELECT name, customer, artist, appointment_date, start_time, status, tattoo_description
      FROM `tabAppointment`
      WHERE status IN ('Scheduled', 'Confirmed')
      AND appointment_date >= CURDATE()
      ORDER BY appointment_date, start_time
      ```
    - Add filters: `artist` (Link to Artist), `appointment_date` (Date Range). (45 minutes)
  - **Create Artist Performance Report**: In Desk, create another Query Report:
    - Name: “Artist Performance”
    - Query:
      ```sql
      SELECT 
          a.artist,
          COUNT(*) as total_appointments,
          SUM(CASE WHEN a.status = 'Completed' THEN 1 ELSE 0 END) as completed_appointments,
          SUM(a.duration) as total_duration
      FROM `tabAppointment` a
      WHERE a.appointment_date BETWEEN %(from_date)s AND %(to_date)s
      GROUP BY a.artist
      ```
    - Add filters: `from_date` (Date), `to_date` (Date). (45 minutes)
  - **Add to Workspace**: Update `Tattoo Shop` workspace to include shortcuts to these reports. (20 minutes)
  - **Test Reports**: Run reports with test data, verify filtering and data accuracy. Check export to Excel/PDF. (30 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 35 minutes**
- **Notes**: Use test appointments to ensure reports reflect accurate data. Explore ERPNext’s Sales Analytics for additional insights.

### Task 2: Define Roles and Initial Permissions
- **Description**: Create and assign roles for “Tattoo Shop Admin”, “Artist”, and “Client” to control access to DocTypes and features. Plan for future artist availability permissions.
- **Subtasks**:
  - **Create Roles**: In Desk, go to **Setup > Role > New**. Create:
    - “Tattoo Shop Admin”: Full access to all DocTypes (`Customer`, `Artist`, `Appointment`, `Artist Availability`, `Sales Invoice`, `Item`).
    - “Artist”: Read access to `Artist`, `Appointment` (own records), `Artist Availability` (own records). Write access planned for future availability management.
    - “Client”: Create/read own `Appointment`, read own `Customer`, read `Sales Invoice` (own). (30 minutes)
  - **Set Permissions**: In **Setup > Role Permissions Manager**, configure:
    - `Tattoo Shop Admin`: All permissions for all relevant DocTypes.
    - `Artist`: Restrict `Appointment` to `artist = user` (via Permission Rules), read-only for own `Artist Availability`.
    - `Client`: Restrict `Appointment` and `Sales Invoice` to `customer = user`, read/write own `Customer`. (1 hour)
  - **Plan Artist Availability Permissions**: Document future permission for `Artist` role to edit own `Artist Availability` (e.g., add `allow_self_manage_availability` Check field to `Artist` later). (15 minutes)
  - **Test Permissions**: Log in as test users (Admin, Artist, Client) to verify access restrictions (e.g., Artist sees only own appointments). (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 45 minutes**
- **Notes**: Use `frappe.db.get_value('User', frappe.session.user, 'email')` in permission queries for user-based restrictions. Future artist availability permissions will be implemented in Phase 6.

### Task 3: Enable Client Portal
- **Description**: Set up a client portal using ERPNext’s Website/Portal module, allowing clients to book appointments, view their appointments and invoices, and browse products.
- **Subtasks**:
  - **Enable Portal**: In Desk, go to **Setup > Portal Settings**, enable portal and add “Client” role to allowed roles. (20 minutes)
  - **Configure Portal Menu**: Add menu items:
    - “My Appointments”: Link to `/appointments` (List view of `Appointment` for logged-in user).
    - “My Invoices”: Link to `/invoices` (List view of `Sales Invoice` for logged-in user).
    - “Shop Products”: Link to `/shop` (from Phase 3 e-commerce, if enabled). (30 minutes)
  - **Customize Web Form**: Update the `Appointment` Web Form (from Phase 2) to integrate with portal, ensuring only logged-in Clients can book and view their data. Add client-side script to prefill `customer` field with logged-in user’s `Customer` ID:
    ```javascript
    frappe.web_form.on('customer', (field, value) => {
        if (!value && frappe.session.user !== 'Guest') {
            frappe.call({
                method: 'frappe.client.get_value',
                args: {
                    doctype: 'Customer',
                    filters: { email_id: frappe.session.user },
                    fieldname: 'name'
                },
                callback: (r) => {
                    if (r.message) {
                        frappe.web_form.set_value('customer', r.message.name);
                    }
                }
            });
        }
    });
    ```
    (1 hour)
  - **Test Portal**: Log in as a Client, verify access to appointments, invoices, and shop. Test booking via Web Form and visibility of own data only. (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **2 hours 50 minutes**
- **Notes**: Ensure Phase 2 Web Form is published. If e-commerce wasn’t set up in Phase 3, skip the “Shop Products” menu item. Test with multiple Client users to confirm data isolation.

## Total Estimated Time
- **Task 1**: 2 hours 35 minutes
- **Task 2**: 2 hours 45 minutes
- **Task 3**: 2 hours 50 minutes
- **Total**: **8 hours 10 minutes** (~3-5 days with breaks, testing, or minor issues)

## Notes and Best Practices
- **Environment**: Work in a development bench with `developer_mode: 1`. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`).
- **Dependencies**: Requires Phases 1-4 (`Customer`, `Artist`, `Appointment`, `Artist Availability`, `Sales Invoice`, `Item`, appointment system, product store, invoicing). ERPNext’s Website and Selling modules must be active.
- **Testing**: Use test users with different roles to verify reports, permissions, and portal functionality. Create sufficient test data (e.g., appointments, invoices) for realistic scenarios.
- **Potential Issues**:
  - Report errors: Validate SQL queries for performance and accuracy.
  - Permission conflicts: Ensure `Client` and `Artist` roles restrict data correctly using `user` conditions.
  - Portal access: Verify login functionality and guest restrictions.
- **Resources**: Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).