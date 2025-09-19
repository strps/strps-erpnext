# Phase 2: Appointment System Implementation

This phase builds the core appointment scheduling functionality for the tattoo shop app, integrating with ERPNext’s `Customer` DocType and the custom `Artist`, `Appointment`, and `Artist Availability` DocTypes. It includes scheduling logic, artist availability checks, workflows, a client-facing web form, and notifications. Estimated timeline: 3-5 days.

## Task Breakdown

### Task 1: Implement Scheduling Logic

- **Description**: Add server-side and client-side logic to the `Appointment` DocType to handle scheduling, including overbooking prevention and time calculations.
- **Subtasks**:
  - **Add Overbooking Validation**: Create a `validate` method in `tattoo_shop/tattoo_shop/doctype/appointment/appointment.py` to check for artist availability conflicts. Query `Artist Availability` and existing `Appointment` records. (1 hour)
  - **Add Client-Side Duration Logic**: Create `appointment.js` in `tattoo_shop/public/js/` to auto-calculate `end_time` based on `start_time` and `duration`. Use Frappe’s client-side API. (45 minutes)
  - **Test Logic**: Create test appointments in Desk, attempt overlapping bookings, and verify duration calculations. (30 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache` to apply changes. (15 minutes)
- **Estimated Time**: **2 hours 30 minutes**
- **Notes**: Ensure `Artist Availability` records exist for testing. Use Frappe’s `frappe.throw` for validation errors.

### Task 2: Integrate Artist Availability

- **Description**: Modify the `Appointment` validation to restrict bookings to available artist slots (from `Artist Availability` DocType). Add client-side logic to show only available slots in the booking form.
- **Subtasks**:
  - **Update Server-Side Validation**: Extend `appointment.py` to query `Artist Availability` for the selected artist and date/time. (1 hour)
  - **Client-Side Slot Filtering**: Add JavaScript in `appointment.js` to fetch and display available slots for the selected artist and date using Frappe’s `frappe.call`. (1 hour 15 minutes)
  - **Test Integration**: Test bookings with various artist availability scenarios (e.g., recurring slots, specific dates). (45 minutes)
  - **Migrate and Clear Cache**: Run `bench migrate` and `bench clear-cache`. (15 minutes)
- **Estimated Time**: **3 hours 15 minutes**
- **Notes**: Optimize queries to avoid performance issues with large availability datasets. Test edge cases (e.g., no availability).

### Task 3: Set Up Workflow for Appointment Status

- **Description**: Create a workflow for the `Appointment` DocType to manage status transitions (Scheduled → Confirmed → Completed → Cancelled).
- **Subtasks**:
  - **Create Workflow**: In Desk, go to **Workflow &gt; New**. Set DocType to `Appointment`, define states (Scheduled, Confirmed, Completed, Cancelled), and transitions (e.g., Admin confirms, Client cancels). (45 minutes)
  - **Assign Roles**: Link transitions to roles (e.g., “Tattoo Shop Admin” for confirmations, “Client” for cancellations). Create roles if needed via **Setup &gt; Role**. (30 minutes)
  - **Test Workflow**: Create appointments, transition through states, and verify role-based restrictions. (30 minutes)
- **Estimated Time**: **1 hour 45 minutes**
- **Notes**: Ensure roles exist before workflow setup. Test with multiple users to confirm permissions.

### Task 4: Create Web Form for Bookings

- **Description**: Build a client-facing Web Form for booking appointments, linked to ERPNext’s `Customer` and restricted to available artist slots.
- **Subtasks**:
  - **Create Web Form**: In Desk, go to **Web Form &gt; New**. Select `Appointment` DocType, add fields (`customer`, `artist`, `appointment_date`, `start_time`, `tattoo_description`, `deposit_required`). Enable “Allow Guest” for public access. (1 hour)
  - **Customize Form Logic**: Add JavaScript to the Web Form (via Desk UI) to populate `artist` and `start_time` based on `Artist Availability` using `frappe.call`. (1 hour 15 minutes)
  - **Publish Form**: Set URL (e.g., `/book-appointment`), customize styling if needed (via CSS in Web Form settings). (30 minutes)
  - **Test Form**: Book appointments as a guest and logged-in user, verify slot restrictions and customer linking. (45 minutes)
- **Estimated Time**: **3 hours 30 minutes**
- **Notes**: Ensure the form is mobile-responsive. Test customer auto-creation if no ERPNext `Customer` exists.

### Task 5: Configure Notifications

- **Description**: Set up automated email notifications for appointment status changes (e.g., confirmation, cancellation). Explore SMS integration for future use.
- **Subtasks**:
  - **Create Email Template**: In Desk, go to **Email Template &gt; New**. Create templates for confirmation, cancellation (e.g., “Dear {{ doc.customer }}, your appointment with {{ doc.artist }} is confirmed for {{ doc.appointment_date }}”). (45 minutes)
  - **Link to Appointment**: In `Appointment` DocType, enable “Send Email Notifications” and link templates to status changes. (30 minutes)
  - **Test Notifications**: Trigger status changes, verify emails are sent correctly. (30 minutes)
  - **Plan SMS (Optional)**: Research Twilio integration for SMS; document API setup for Phase 6. (30 minutes)
- **Estimated Time**: **2 hours 15 minutes**
- **Notes**: SMTP must be configured in ERPNext (Setup &gt; Email Account). SMS requires a third-party gateway (implement in later phase).

## Total Estimated Time

- **Task 1**: 2 hours 30 minutes
- **Task 2**: 3 hours 15 minutes
- **Task 3**: 1 hour 45 minutes
- **Task 4**: 3 hours 30 minutes
- **Task 5**: 2 hours 15 minutes
- **Total**: **13 hours 15 minutes** (\~3-5 days with breaks, testing, or minor issues)

## Notes and Best Practices

- **Environment**: Work in a development bench with `developer_mode: 1`. Commit changes to Git after each task (`git add . && git commit -m "Task X completed"`).
- **Dependencies**: Phase 1 DocTypes (`Customer`, `Artist`, `Appointment`, `Artist Availability`) must be complete.
- **Testing**: Use test users (Admin, Client, Artist) to verify functionality. Create test availability records for artists.
- **Potential Issues**:
  - Slow queries in availability checks: Optimize with indexes if needed.
  - Web Form permissions: Ensure “Allow Guest” works for public bookings.
  - Workflow conflicts: Verify role assignments match user capabilities.
- **Resources**: Frappe Docs (frappeframework.com/docs), ERPNext Docs (docs.erpnext.com), Forum (discuss.erpnext.com).