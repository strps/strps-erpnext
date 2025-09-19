Phase 1: Setup and Core Structure Breakdown
Task 1: Verify ERPNext Integration and Configure Company

Description: Ensure ERPNext is installed correctly on the site, verify version compatibility (Frappe and ERPNext v15), and set up the tattoo shop’s company in ERPNext for accounting and operational purposes.
Subtasks:

Check versions: Run bench version to confirm Frappe and ERPNext are on version-15. If not, switch branches (bench switch-to-branch version-15 frappe erpnext --upgrade). (15 minutes)
Verify ERPNext installation: Run bench list-apps to confirm ERPNext is listed. If not, install with bench get-app erpnext --branch version-15 and bench --site your_site_name install-app erpnext. (15 minutes)
Configure Company: In Desk, go to Setup > Company > New. Set name (e.g., “Tattoo Studio”), currency (e.g., USD), and fiscal year. Enable relevant modules (Selling, Stock, Accounting). (30 minutes)
Run migrations: Execute bench migrate, bench clear-cache, and bench restart to apply changes. (10 minutes)


Estimated Time: 1 hour 10 minutes
Notes: Backup the site before changes (bench --site your_site_name backup). If ERPNext is already installed, skip the installation subtask.

Task 2: Customize ERPNext Customer DocType

Description: Extend the ERPNext Customer DocType with tattoo-specific fields (e.g., allergies, tattoo history) to avoid duplicating customer data in the custom app.
Subtasks:

Access Customize Form: In Desk, go to Customize Form, select “Customer”. (5 minutes)
Add fields: Create tattoo_allergies (Text) and tattoo_history (Table, linked to Appointment DocType, to be created later). Set permissions (e.g., visible to “Tattoo Shop Admin”, “Artist”). (20 minutes)
Save and migrate: Save changes, run bench migrate and bench clear-cache. (10 minutes)
Test: Create a test Customer in Desk to verify fields appear correctly. (15 minutes)


Estimated Time: 50 minutes
Notes: The tattoo_history table will be linked after creating the Appointment DocType. Use ERPNext’s standard fields (e.g., customer_name, mobile_no, email_id) for core customer data.

Task 3: Create Custom Artist DocType

Description: Create a custom Artist DocType in your app (tattoo_shop) to manage tattoo artist profiles, including name, specialty, and contact details.
Subtasks:

Create DocType: In Desk, go to DocType > New. Set:

Name: Artist
Module: Tattoo Shop (your app’s module)
Fields: artist_name (Data, Mandatory), specialty (Data), email (Data), phone (Data), bio (Text Editor)
Permissions: Allow “Tattoo Shop Admin” (all), “Artist” (read, restricted later)
(20 minutes)


Save and migrate: Save DocType, run bench migrate, bench clear-cache. (10 minutes)
Test: Create a test Artist record in Desk to verify fields and permissions. (10 minutes)


Estimated Time: 40 minutes
Notes: Ensure the Tattoo Shop module exists in modules.txt and desktop.py (as set up previously). Add an icon (e.g., octicon-person) in desktop.py if desired.

Task 4: Create Custom Appointment DocType

Description: Create an Appointment DocType for managing bookings, linked to ERPNext’s Customer and your Artist DocType, with fields for scheduling and status.
Subtasks:

Create DocType: In Desk, go to DocType > New. Set:

Name: Appointment
Module: Tattoo Shop
Fields: customer (Link to Customer), artist (Link to Artist), appointment_date (Date, Mandatory), start_time (Time), end_time (Time), duration (Int), status (Select: Scheduled, Confirmed, Completed, Cancelled), tattoo_description (Text), deposit_required (Check)
Enable Calendar View: Set appointment_date as the calendar field
Permissions: “Tattoo Shop Admin” (all), “Client” (create, read own), “Artist” (read)
(30 minutes)


Save and migrate: Save, run bench migrate, bench clear-cache. (10 minutes)
Test: Create a test Appointment, verify calendar view in Desk. (15 minutes)


Estimated Time: 55 minutes
Notes: Calendar view requires appointment_date to be set correctly. Client permissions will be refined in Phase 5.

Task 5: Create Artist Availability DocType

Description: Create a custom Artist Availability DocType to define available time slots or days for each artist, enabling slot-based booking in later phases.
Subtasks:

Create DocType: In Desk, go to DocType > New. Set:

Name: Artist Availability
Module: Tattoo Shop
Fields: artist (Link to Artist, Mandatory), day_of_week (Select: Monday, Tuesday, ..., Sunday), start_time (Time), end_time (Time), is_recurring (Check, for weekly availability), specific_date (Date, optional for one-off availability)
Permissions: “Tattoo Shop Admin” (all), “Artist” (read, later extend for write)
(25 minutes)


Save and migrate: Save, run bench migrate, bench clear-cache. (10 minutes)
Test: Create test availability records (e.g., “Artist A available Monday 9 AM–5 PM”) and verify in Desk. (15 minutes)


Estimated Time: 50 minutes
Notes: This DocType will be used in Phase 2 to restrict Appointment bookings to available slots. Permissions for artists to self-manage will be added later.

Task 6: Update Workspace

Description: Update the Tattoo Shop workspace in Desk to include shortcuts to ERPNext Customer, custom Artist, Appointment, Artist Availability, and ERPNext Item (for products).
Subtasks:

Edit Workspace: In Desk, go to Workspace > Tattoo Shop. Add shortcuts:

Customer (ERPNext)
Artist, Appointment, Artist Availability (custom)
Item (ERPNext, for products)
Add calendar shortcut for Appointment (linked to calendar view)
(20 minutes)


Save and test: Save workspace, refresh Desk, verify shortcuts appear in sidebar and work correctly. (10 minutes)


Estimated Time: 30 minutes
Notes: Ensure the workspace is assigned to the Tattoo Shop module and visible to relevant roles (e.g., “Tattoo Shop Admin”).

Total Estimated Time

Task 1: 1 hour 10 minutes
Task 2: 50 minutes
Task 3: 40 minutes
Task 4: 55 minutes
Task 5: 50 minutes
Task 6: 30 minutes
Total: 5 hours 35 minutes (~1-2 days with breaks, testing, or minor issues)

Notes and Best Practices

Environment: Work in a development bench with developer_mode: 1. Commit changes to Git after each task (git add . && git commit -m "Task X completed").
Dependencies: Ensure ERPNext is installed before starting (as per previous guidance). If not, add 30 minutes to Task 1 for installation.
Testing: Test each DocType creation in Desk to catch errors early. Use a test user with “Tattoo Shop Admin” role.
Potential Issues:

Migration failures: Rerun bench update --patch and bench migrate.
Permission conflicts: Ensure roles exist (create via Setup > Role if needed).


Resources: Refer to Frappe Docs (frappeframework.com/docs) and ERPNext Docs (docs.erpnext.com) for DocType and workspace setup.