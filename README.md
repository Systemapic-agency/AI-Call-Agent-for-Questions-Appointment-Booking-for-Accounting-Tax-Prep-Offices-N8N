# AI-Call-Agent-for-Questions-Appointment-Booking-for-Accounting-Tax-Prep-Offices-N8N

An **n8n appointment availability checking workflow** designed for **AI voice agents**, inbound/outbound support systems, and scheduling assistants. This workflow receives a requested appointment date/time through a webhook, checks available time slots from **GoHighLevel calendar**, and returns whether the requested time is available or suggests nearby alternatives. :contentReference[oaicite:1]{index=1}

---

## Features

- Receives scheduling requests via **Webhook**
- Designed to work with **AI support / voice agents**
- Accepts a requested appointment date and time
- Checks free slots from **GoHighLevel Calendar**
- Returns:
  - exact availability confirmation, or
  - nearest available alternatives
- Includes date normalization and timestamp conversion
- Supports real-time appointment checking for service businesses :contentReference[oaicite:2]{index=2}

---

## Workflow Overview

This workflow follows the process below:

1. An AI assistant or external system sends a requested appointment date/time to an **n8n Webhook**
2. The workflow normalizes the incoming appointment date
3. It converts the requested date into a timestamp format
4. It calculates the **next date** to create a valid availability search window
5. It queries **GoHighLevel free calendar slots**
6. A JavaScript logic block checks whether the exact requested slot exists
7. If available, it returns a confirmation
8. If unavailable, it returns the closest available time slots
9. The result is sent back as a webhook response for the AI assistant or calling system to use in conversation :contentReference[oaicite:3]{index=3}

---

## Nodes Used

### 1. Webhook
The workflow begins with a **Webhook** node that listens for **POST requests**.

This is typically triggered by:

- AI voice agents
- Retell AI tools
- support bots
- scheduling assistants
- outbound appointment systems
- custom booking tools :contentReference[oaicite:4]{index=4}

---

### 2. Code in JavaScript1
This node adjusts the incoming appointment datetime to the **current year**.

Why this is useful:
- some AI systems or tools may pass dates with mismatched years
- this ensures the request is aligned with the current scheduling context

It specifically updates:

- `body.args.appointment_datetime` :contentReference[oaicite:5]{index=5}

---

### 3. Get Start Date
This **Date & Time** node converts the requested date into a Unix timestamp format.

This becomes the **startDate** used in the calendar availability check. :contentReference[oaicite:6]{index=6}

---

### 4. Get Next Date
This node adds **1 day** to the requested date.

This is used to define the availability checking window so the system can query all relevant free slots for the target day. :contentReference[oaicite:7]{index=7}

---

### 5. Get End Date
This **Date & Time** node converts the incremented date into a Unix timestamp.

This becomes the **endDate** used for the GoHighLevel availability API request. :contentReference[oaicite:8]{index=8}

---

### 6. Get Free Slots from GHL
This **HTTP Request** node sends a request to the **GoHighLevel Calendar Free Slots API**.

It checks availability using:

- `startDate`
- `endDate`

It also includes:
- authorization header
- API version header
- timezone header

The configured timezone is:

- `America/New_York` :contentReference[oaicite:9]{index=9}

---

### 7. Code in JavaScript
This is the main **availability decision logic**.

It performs the following:

- collects all free slots from the GoHighLevel response
- checks if the exact requested datetime exists
- if available:
  - returns a human-readable confirmation like  
    `"Thursday at 9:00 AM is available."`
- if unavailable:
  - finds the closest available slots before/after the requested time
  - returns alternative booking options :contentReference[oaicite:10]{index=10}

---

### 8. Respond to Webhook
This node returns the result back to the caller.

This allows:
- AI phone agents
- support bots
- frontend booking apps

to immediately respond to the user in real time. :contentReference[oaicite:11]{index=11}

---

## Expected Input

This workflow expects a webhook payload containing a requested appointment date.

Example:

```json
{
  "body": {
    "args": {
      "date": "2025-10-09",
      "time": "09:00:00",
      "appointment_datetime": "2025-10-09T09:00:00.000Z"
    }
  }
}


Inbound/Outbound - Book Appointment (Support AI)

An **n8n AI appointment booking workflow** designed for **voice AI assistants**, inbound/outbound support systems, and service-based businesses. This workflow receives appointment booking details through a webhook, checks whether the caller already exists in **GoHighLevel CRM**, creates the contact if needed, and books the appointment directly into the **GoHighLevel calendar**. :contentReference[oaicite:1]{index=1}

---

## Features

- Receives appointment booking requests through **Webhook**
- Built for **AI voice assistants** and support automation
- Searches for existing contacts in **GoHighLevel CRM**
- Automatically creates a new contact if none exists
- Books confirmed appointments directly in **GoHighLevel Calendar**
- Handles:
  - patient/client name
  - phone number
  - email
  - service type
  - appointment date/time
- Supports healthcare and service business booking flows
- Includes fallback error handling for failed contact lookup cases :contentReference[oaicite:2]{index=2}

---

## Workflow Overview

This workflow follows the process below:

1. A voice AI assistant or external system sends booking details to an **n8n Webhook**
2. The workflow normalizes the appointment date to the current year
3. It searches **GoHighLevel CRM** for an existing contact using the phone number
4. The workflow checks the contact search result:
   - **If contact exists** → directly books the appointment
   - **If contact does not exist** → creates a new contact first, then books the appointment
   - **If an unexpected result occurs** → returns an error response
5. The appointment is created in the **GoHighLevel Calendar** with confirmed status :contentReference[oaicite:3]{index=3}

---

## Nodes Used

### 1. Webhook
The workflow begins with a **Webhook** node that accepts **POST requests**.

This is typically triggered by:

- AI voice assistants
- Retell AI workflows
- support bots
- appointment booking assistants
- inbound call systems
- outbound scheduling flows :contentReference[oaicite:4]{index=4}

---

### 2. Code in JavaScript1
This node updates the incoming appointment date to the **current year**.

Why this is useful:
- voice AI tools may pass relative dates or dates without reliable year context
- this ensures booking data stays aligned with the active scheduling year

It updates:

- `body.args.appointment_date` :contentReference[oaicite:5]{index=5}

---

### 3. Search Contact
This **HTTP Request** node searches **GoHighLevel CRM** for an existing contact using the phone number.

The workflow checks:

- `body.args.ph_num`
- or fallback:
- `body.call.from_number`

This helps prevent duplicate contact creation. :contentReference[oaicite:6]{index=6}

---

### 4. Contact Status (Switch Node)
This **Switch** node routes the workflow based on the CRM search result.

### Possible outcomes:

#### Not Found
If no contact exists:
- the workflow moves to **contact creation flow**

#### Found
If a contact exists:
- the workflow skips creation
- and directly books the appointment

#### Error
If an unexpected response occurs:
- the workflow returns a fallback error message :contentReference[oaicite:7]{index=7}

---

## Booking Logic Paths

---

## Path A: Existing Contact Found

### 5. Book Appointment1
If the contact already exists in GoHighLevel, this node books the appointment directly.

It uses:

- existing `contactId`
- existing `locationId`
- requested `appointment_datetime`
- selected `service`

The appointment is created with:

- `appointmentStatus: confirmed`
- `ignoreFreeSlotValidation: true` :contentReference[oaicite:8]{index=8}

---

## Path B: Contact Not Found

### 6. If Node
Before creating a new contact, the workflow checks whether the provided email looks valid.

It checks whether the email contains:

- `@`

This splits the flow into two contact creation paths:

- contact creation **with email**
- contact creation **without email** :contentReference[oaicite:9]{index=9}

---

### 7. Create a Contact
If a valid email is available, the workflow creates a new contact in **GoHighLevel CRM** with:

- name
- email
- phone
- location ID
- tag: `ai_agent` :contentReference[oaicite:10]{index=10}

---

### 8. Book Appointment
After contact creation with email, the workflow books the appointment directly in the **GoHighLevel calendar**. :contentReference[oaicite:11]{index=11}

---

### 9. Create a Contact1
If no valid email is available, the workflow creates a new contact **without relying on a valid email format**. :contentReference[oaicite:12]{index=12}

---

### 10. Book Appointment2
After creating the contact without email validation, the workflow still proceeds to book the appointment. :contentReference[oaicite:13]{index=13}

---

## Error Handling

### 11. Return Error
If the contact search result is unexpected, the workflow returns this fallback message:

```text id="vkgx1v"
I apologize; it seems we are having an issue.



# Create Outbound Call When Contact Added into GHL (Support AI)

An **n8n outbound calling automation workflow** that captures lead details through a form, waits for processing, and then automatically triggers an **AI-powered outbound phone call** using **Retell AI**. This workflow is useful for support teams, appointment follow-ups, lead qualification, and AI voice outreach systems. :contentReference[oaicite:1]{index=1}

---

## Features

- Collects lead/contact details through an **n8n Form**
- Supports:
  - Name
  - Email
  - Phone Number
- Automatically triggers an **outbound AI phone call**
- Uses **Retell AI** for voice agent calling
- Passes dynamic contact variables into the AI call
- Includes a **Wait** step before call execution
- Useful for:
  - support callbacks
  - lead follow-up
  - appointment reminders
  - outbound AI assistants :contentReference[oaicite:2]{index=2}

---

## Workflow Overview

This workflow follows the process below:

1. A user or staff member submits a contact through an **n8n form**
2. The workflow captures:
   - Name
   - Email
   - Phone
3. A **Wait** node pauses execution before the outbound call
4. The workflow sends the contact details to **Retell AI**
5. Retell AI places an **outbound phone call** to the lead using a configured AI agent :contentReference[oaicite:3]{index=3}

---

## Nodes Used

### 1. On Form Submission
The workflow starts with an **n8n Form Trigger** titled:

**Outbound Calls Form**

It collects the following fields:

- **Name** (required)
- **Email**
- **Phone** :contentReference[oaicite:4]{index=4}

---

### 2. Wait
A **Wait** node pauses the workflow before initiating the call.

This can be useful for:

- delaying immediate outreach
- creating a buffer after form submission
- preparing downstream processing
- avoiding instant robotic callback behavior :contentReference[oaicite:5]{index=5}

---

### 3. HTTP Request1
This node sends a **POST request** to the **Retell AI API** endpoint:

`https://api.retellai.com/v2/create-phone-call`

It triggers the outbound AI phone call using:

- a configured **from number**
- the submitted **lead phone number**
- a selected **Retell AI agent**
- dynamic variables passed into the AI assistant :contentReference[oaicite:6]{index=6}

---

## Retell AI Call Configuration

The outbound call request includes:

- **From Number** → the business caller ID
- **To Number** → the submitted lead phone number
- **Call Type** → `phone_call`
- **Override Agent ID** → specific Retell AI voice agent
- **Dynamic Variables**:
  - Name
  - Email
  - Phone :contentReference[oaicite:7]{index=7}

---

## Dynamic Variables Passed to AI

The workflow injects lead information into the AI call:

```json id="q5n4w7"
{
  "name": "{{ $json.Name }}",
  "email": "{{ $json.Email }}",
  "phone": "{{ $json.Phone }}"
}
