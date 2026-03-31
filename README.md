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
