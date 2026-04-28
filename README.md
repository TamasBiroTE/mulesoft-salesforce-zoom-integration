# Salesforce → Zoom Integration using MuleSoft

## Overview

This project implements a real-time integration between Salesforce and Zoom using MuleSoft.

When a Lead is created or updated in Salesforce, the application automatically:

* Creates a Zoom meeting
* Updates the meeting if the date changes
* Deletes the meeting if the status becomes "Cancelled"
* Sends email notifications to the Lead

The integration is event-driven using Salesforce Change Data Capture (CDC).

---

## Architecture

* **Source**: Salesforce CDC (`LeadChangeEvent`)
* **Integration Layer**: MuleSoft (Anypoint Studio / CloudHub)
* **Target Systems**:

  * Zoom API (meeting management)
  * Salesforce (store meeting ID and link)
  * SMTP (email notifications)

---

## Business Logic

### Create Meeting

Triggered when:

* `changeType = CREATE`
* `Demo_Status__c = Scheduled`
* `Demo_Date_Time__c` is not null

Actions:

* Create Zoom meeting
* Store `Meeting ID` and `Join URL` in Salesforce
* Send email notification

---

### Update Meeting

Triggered when:

* `changeType = UPDATE`
* `Demo_Date_Time__c` changed
* Meeting already exists

Actions:

* Update Zoom meeting time
* Send reschedule email

---

### Delete Meeting

Triggered when:

* `changeType = UPDATE`
* `Demo_Status__c = Cancelled`
* Meeting exists

Actions:

* Delete Zoom meeting
* Clear meeting fields in Salesforce
* Send cancellation email

---

### ✔ Token Caching

* Zoom OAuth token is cached using Object Store
* Reduces unnecessary token requests

---

### ✔ Error Handling

* Zoom token retrieval uses retry (`until-successful`)
* Errors are propagated and logged

---

## Setup Instructions

### 1. Clone Repository

```bash
git clone <repo-url>
cd mulesoft-salesforce-zoom-integration
```

---

### 2. Configuration

Create a local config file:

```bash
cp config.yaml config-dev.yaml
```

Fill in real values:

```yaml
zoom:
  clientId: YOUR_CLIENT_ID
  clientSecret: YOUR_CLIENT_SECRET
  accountId: YOUR_ACCOUNT_ID

salesforce:
  clientId: YOUR_CLIENT_ID
  clientSecret: YOUR_CLIENT_SECRET
  username: YOUR_USERNAME
  password: YOUR_PASSWORD
  token: YOUR_SECURITY_TOKEN

email:
  user: YOUR_EMAIL
  password: YOUR_APP_PASSWORD
```

---

### 3. Run Locally

Set environment:

```bash
-Denv=dev
```

Run from Anypoint Studio.

---

## CloudHub Deployment

### Required Properties

Set in Runtime Manager:

```plaintext
env=prod
```

---

## Security

* Sensitive data is stored outside the repository
* `.gitignore` excludes: `config-dev.yaml`, `config-prod.yaml`

---

## Email Notifications

Emails are sent via SMTP (Gmail).

Make sure:

* App password is configured
* "Less secure apps" is not required (use App Password)

---

## License

Internal / Demo project
