# Kimai Server

- Kimai is an open-source, web-based time tracking tool that helps track work hours, manage projects/clients, and generate reports or invoices.
- Its purpose is to make time management and billing easier for freelancers, teams, and companies through a simple browser-based interface.
- We cloned the Kimai GitHub project into our EC2 instance and used MariaDB as the backend database to store user and project data.
- Nginx was configured as the web server, working with PHP-FPM to process Kimai’s PHP code and serve the application.
- We enabled access through port 443 (HTTPS) so that Kimai could be opened securely using the EC2 instance’s public IP address.
- We then added users, projects, and activities, creating member accounts so they could log their working hours, while the admin could monitor all activity and reports.

---

### Admin Credentials:
- *Username:* admin
- *Password:* kimaiadmin
### MariaDB credentials:
- *user:* "kimaiuser"
- *password:* "StrongPassword123!"
- *database:* "kimai2”

---

### To Restore kimai db use following command:

```
mysql -u kimaiuser -p kimai2 < kimai_db_backup.sql
```

### To restore files use following command: 
 
```
tar -xzf kimai_files_backup.tar.gz -C /var/www/ 
```

---

### Sending User report to Admin

- Connected to Kimai Database – Used mysql.connector to connect directly to the kimai2 database and fetch weekly and total worked hours of users.
- Processed Timesheet Data – Wrote SQL queries to calculate each user’s weekly work duration and overall total work duration.
- Generated PDF Report – Used the fpdf library to format the data into a tabular report, including user, project, activity, weekly time, and total time.
- Saved the Report – Exported the generated report as a PDF file (kimai_db_weekly_report.pdf).
- Configured Email Setup – Used Python’s smtplib with Gmail SMTP server (smtp.gmail.com) and app password authentication.
- Sent Report to Admin – Attached the PDF report to an email and automatically sent it to the admin (sanjanaproject36@gmail.com).
- Configured a cron job to automatically execute the report generation and email script every time we run below code, with output logged for monitoring.
  
  ```
  * ** * * * /usr/bin/python3 /home/ec2-user/kimai_db_report.py >> /home/ec2-user/kimai_cron.log 2>&1 
  ```
  
- Manual Execution Option – Provided the flexibility to run the script manually whenever required (e.g., by executing python3 kimai_db_report.py).

---

### Changes need to be done in kimai_db_report.py:

```
gmail_user = "yelagandulasupraja@gmail.com"    # replace it with sender mail
gmail_app_password = "rvfrdedhonyhjdea"          #create a password with sendermail using this link -- https://myaccount.google.com/apppasswords (sender mail must have two step verification to create password)
admin_email = "sanjanaproject36@gmail.com"     #replace it with receiver mail
```



