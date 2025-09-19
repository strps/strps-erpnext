# Phase 6: Testing, Deployment, and Enhancements

This phase finalizes the tattoo shop management system by conducting comprehensive end-to-end testing, deploying to a production environment, enabling scheduler and backups, and adding enhancements such as artist self-managed availability permissions, external calendar sync, and mobile optimization. Estimated timeline: 3-5 days.

## Task Breakdown

### Task 1: End-to-End Testing
- **Description**: Test the entire system workflow, including appointment booking, product usage, invoicing, payments, reports, and portal functionality, across all user roles (Tattoo Shop Admin, Artist, Client).
- **Subtasks**:
  - **Prepare Test Data**: Create test records in Desk:
    - Customers (5-10, with varied details)
    - Artists (2-3, with availability slots)
    - Appointments (10+, covering Scheduled, Confirmed, Completed, Cancelled statuses)
    - Products (5-10 items, with stock in “Shop Storage”)
    - Invoices and payments (via test Stripe mode). (1 hour)
  - **Test Workflows**:
    - Book appointment via Web Form (Client, Guest), verify artist availability restrictions.
    - Confirm/cancel appointment (Admin, Client).
    - Complete appointment, add products used, generate invoice, process payment.
    - Verify stock updates and accounting entries. (1 hour 30 minutes)
  - **Test Reports and Portal**:
    - Run “Upcoming Appointments” and “Artist Performance” reports, verify data accuracy.
    - Log in as Client, check portal for “My Appointments”, “My Invoices”, “Shop Products”.
    - Test Artist role access (own appointments/availability only). (1 hour)
  - **Test Edge Cases**:
    - Overbooking attempts (should fail).
    - Negative stock scenarios.
    - Guest vs. logged-in booking.
    - Multi-user concurrent access. (1 hour)
  - **Document Issues**: Log any bugs (e.g., permission errors, slow queries) for resolution. (30 minutes)
- **Estimated Time**: **5 hours**
- **Notes**: Use test users for each role. Simulate real-world scenarios (e.g., busy day with multiple bookings). Fix issues by revisiting relevant phase scripts/configurations.

### Task 2: Deploy to Production
- **Description**: Deploy the system to a production environment (local server or Frappe Cloud) and verify functionality.
- **Subtasks**:
  - **Prepare Production Environment**:
    - Local: Ensure server meets requirements (e.g., Ubuntu, Python, MariaDB). Run `bench setup production` to configure Supervisor and Nginx.
    - Frappe Cloud: Create a new site in Frappe Cloud dashboard, install `tattoo_shop` and `erpnext` apps via Apps tab. (1 hour)
  - **Migrate Site**: If moving from development, backup dev site (`bench --site dev_site backup`), restore to production (`bench --site prod_site restore`), and install apps (`bench --site prod_site install-app tattoo_shop erpnext`). (45 minutes)
  - **Run Migrations**: Execute `bench --site prod_site migrate` and `bench clear-cache` to apply all changes. (15 minutes)
  - **Test Production**: Repeat key tests from Task 1 (booking, invoicing, portal) in production environment. Verify SSL, domain, and email settings. (1 hour)
  - **Configure DNS (if needed)**: Point domain (e.g., tattooshop.com) to production server or Frappe Cloud IP. (30 minutes, optional)
- **Estimated Time**: **3 hours 30 minutes** (add 30 minutes if DNS setup is required)
- **Notes**: Use Frappe Cloud for simpler deployment if budget allows. Ensure production backups are enabled before going live.

### Task 3: Enable Scheduler and Backups
- **Description**: Configure Frappe’s scheduler for automated tasks (e.g., notifications) and set up regular backups to ensure data safety.
- **Subtasks**:
  - **Enable Scheduler**: Run `bench setup scheduler` and `bench --site prod_site enable-scheduler` to activate background jobs (e.g., email notifications). Verify scheduler status (`bench doctor`). (30 minutes)
  - **Configure Backups**: Run `bench setup auto-backup` to schedule daily backups. Set retention (e.g., keep 7 days) in **Setup > Backup Settings**. (20 minutes)
  - **Test Scheduler and Backups**: Trigger a test notification (e.g., appointment status change) and verify backup files are created in `sites/prod_site/private/backups`. (30 minutes)
- **Estimated Time**: **1 hour 20 minutes**
- **Notes**: Ensure SMTP is configured for notifications (Phase 2). Test backup restoration (`bench restore`) in a sandbox environment.

### Task 4: Implement Enhancements
- **Description**: Add key enhancements: artist self-managed availability permissions, external calendar sync (e.g., Google Calendar export), and mobile optimization for Desk and portal.
- **Subtasks**:
  - **Artist Self-Managed Availability**:
    - Add field to `Artist` DocType: `allow_self_manage_availability` (Check) via **Customize Form**. (15 minutes)
    - Update permissions: In **Setup > Role Permissions Manager**, grant “Artist” role write access to `Artist Availability` where `artist = user` and `allow_self_manage_availability = 1`. (30 minutes)
    - Create Web Form for artists: In **Web Form > New**, create “Manage Availability” for `Artist Availability`, restrict to “Artist” role, prefilter `artist` to logged-in user. (45 minutes)
    - Test: Log in as Artist, add/edit availability slots, verify restrictions for non-authorized artists. (30 minutes)
  - **External Calendar Sync**:
    - Add ICS export: In `appointment.py`, add a method to generate an ICS file for appointments:
      ```python
      import frappe
      from frappe.model.document import Document
      from icalendar import Calendar, Event
      from datetime import datetime

      class Appointment(Document):
          def get_ics(self):
              cal = Calendar()
              event = Event()
              event.add('summary', f"Tattoo Appointment: {self.tattoo_description}")
              start = datetime.strptime(f"{self.appointment_date} {self.start_time}", "%Y-%m-%d %H:%M:%S")
              event.add('dtstart', start)
              end = datetime.strptime(f"{self.appointment_date} {self.end_time}", "%Y-%m-%d %H:%M:%S")
              event.add('dtend', end)
              cal.add_component(event)
              return cal.to_ical()
      ```
      Add a custom button in `appointment.js` to download ICS:
      ```javascript
      frappe.ui.form.on('Appointment', {
          refresh: function(frm) {
              frm.add_custom_button(__('Export to Calendar'), function() {
                  frappe.call({
                      method: 'tattoo_shop.tattoo_shop.doctype.appointment.appointment.get_ics',
                      args: { name: frm.doc.name },
                      callback: function(r) {
                          let blob = new Blob([r.message], { type: 'text/calendar' });
                          let link = document.createElement('a');
                          link.href = window.URL.createObjectURL(blob);
                          link.download = `${frm.doc.name}.ics`;
                          link.click();
                      }
                  });
              });
          }
      });
      ```
      (1 hour 15 minutes)
    - Test ICS export: Download ICS file for an appointment, import into Google Calendar, verify details. (30 minutes)
  - **Mobile Optimization**:
    - Enable PWA: In **Setup > Website Settings**, enable Progressive Web App (PWA) for Desk and portal. (15 minutes)
    - Test responsiveness: Access Web Form, portal, and Desk on mobile devices, adjust CSS in Web Form or Website Theme if needed (e.g., via **Website > Website Theme**). (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **4 hours 25 minutes**
- **Notes**: ICS export requires `icalendar` Python library (`pip install icalendar` in bench environment). Mobile optimization may need custom CSS for specific UI tweaks.

## Total Estimated Time
- **Task 1**: 5 hours
- **Task 2**: 3 hours 30 minutes
- **Task 3**: 1 hour 20 minutes
- **Task 4**: 4 hours 25 minutes
- **Total**: **14 hours 15 minutes** (~3-5 days with breaks, testing, or minor issues)

## Notes and Best Practices
- **Environment**: Work in a development bench with `developer_mode: 1` for enhancements, then migrate to production. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`).
- **Dependencies**: Requires Phases 1-5 (`Customer`, `Artist`, `Appointment`, `Artist Availability`, `Sales Invoice`, `Item`, appointment system, product store, invoicing, reports, portal).
- **Testing**: Use multiple devices for mobile tests. Create test artists with/without `allow_self_manage_availability` to verify permissions. Test backups thoroughly.
- **Potential Issues**:
  - Scheduler failures: Check logs (`bench logs`) if notifications fail.
  - ICS errors: Ensure date/time formats match calendar apps.
  - Deployment issues: Verify production site configuration (e.g., Redis, Node.js).
- **Resources**: Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).