# Workflow 6: Pre-Call SMS Follow-up

## Overview

This workflow reduces no-shows by sending a series of SMS reminders after someone books a call via Cal.com. When a booking event fires, the workflow calculates the time until the appointment and sends strategically timed SMS messages: a confirmation immediately after booking, a reminder a few days before, another the day before, and a final one an hour before the call. Each message is AI-generated to feel personal and includes the meeting link.

---

## Required Services

- [ ] **Cal.com** -- https://cal.com/ (booking platform with webhook support)
- [ ] **Contiguity** -- https://contiguity.co/ (SMS sending API)
- [ ] **OpenAI** -- https://platform.openai.com/ (message generation)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Cal.com Webhook | Cal.com > Settings > Developer > Webhooks | (webhook URL from n8n) |
| Contiguity API Key | Contiguity Dashboard > API Keys | HTTP Header Auth |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_CONTIGUITY_API_KEY` | Contiguity SMS API key | Authorization header in HTTP Request |
| `YOUR_OPENAI_API_KEY` | OpenAI key | AI message generation |
| `YOUR_COMPANY_NAME` | Your company name | SMS messages |
| `YOUR_NAME` | Your name (or closer name) | SMS signatures |
| `YOUR_CALCOM_WEBHOOK_PATH` | n8n webhook path | Cal.com webhook configuration |
| `YOUR_SENDER_PHONE` | Phone number to send SMS from (Contiguity) | SMS request body |

---

## Node-by-Node Configuration

### Trigger
1. **Webhook** -- Receives Cal.com booking events via POST.
   - After importing, copy the **Production Webhook URL** from this node.
   - Go to Cal.com > Settings > Developer > Webhooks > Add Webhook.
   - Paste the URL and select the `BOOKING_CREATED` event trigger.

### Booking Data Extraction
2. **Filter Booking Created** -- Confirms the event is a `BOOKING_CREATED` type (ignores cancellations/reschedules).
3. **Extract Booking Info** -- Parses the webhook payload:
   - Attendee name, email, phone
   - Meeting start time, end time
   - Meeting URL (video call link)
   - Days until appointment
   - Hours until appointment

### Timing Logic
4. **Calculate Time Until Appointment** -- Determines how many days/hours until the call.
5. **7+ Days Route** -- If appointment is 7+ days away: sends immediate confirmation, then schedules reminders at 3 days, 1 day, and 1 hour before.
6. **3-7 Days Route** -- If 3-7 days away: sends immediate confirmation, then day-before and hour-before reminders.
7. **Same Day Route** -- If appointment is today: sends immediate confirmation and hour-before reminder only.

### SMS Sending (per message)
8. **AI Agent (generate message)** -- Writes a short, friendly SMS. Includes:
   - Attendee's first name
   - Meeting date/time
   - Meeting link
   - A warm touch (excitement about the call, what to expect)
9. **Wait** -- Waits the calculated duration before sending (e.g., waits until 1 day before).
10. **HTTP Request (Contiguity)** -- Sends the SMS:
    - URL: `https://api.contiguity.co/send/text`
    - Method: POST
    - Headers: `Authorization: Token YOUR_CONTIGUITY_API_KEY`, `Content-Type: application/json`
    - Body:
    ```json
    {
      "to": "ATTENDEE_PHONE_NUMBER",
      "message": "AI_GENERATED_MESSAGE"
    }
    ```

---

## Cal.com Webhook Setup

1. Log in to Cal.com
2. Go to **Settings** > **Developer** > **Webhooks**
3. Click **New Webhook**
4. Paste your n8n webhook URL (from the Webhook node after import)
5. Select event: **Booking Created**
6. Set payload format to JSON
7. Save

Make sure your Cal.com event type collects a **phone number** in the booking form. Go to your event type > Advanced > Add a "Phone Number" field as required.

---

## Contiguity SMS Setup

1. Sign up at https://contiguity.co/
2. Get your API key from the dashboard
3. You may need to verify your sending phone number
4. Check their documentation for country-specific requirements

**Alternative SMS providers:** If you prefer Twilio, Vonage, or another SMS API, replace the HTTP Request node URL and body format with the equivalent API call. The logic stays the same.

---

## Testing Steps

1. **Create a test booking** on your Cal.com event page (book a call with yourself)
2. **Check the webhook node:** It should show the incoming payload
3. **Check the Extract Booking Info node:** Verify all fields parsed correctly
4. **Check timing logic:** Verify the correct route was taken based on appointment distance
5. **Check SMS delivery:** Verify you received the confirmation SMS
6. **Wait for reminders:** If appointment is far out, you can temporarily reduce wait times in the Wait nodes for testing

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Webhook never fires | Cal.com webhook URL incorrect | Re-copy the Production URL from n8n (not the Test URL) |
| Phone number missing | Cal.com booking form does not require phone | Add a required Phone Number field to your Cal.com event type |
| SMS not delivered | Contiguity API key wrong or phone format invalid | Verify API key; ensure phone has country code (+1 for US) |
| Wait node too long for testing | Appointment is days away | Temporarily set Wait to 10 seconds for testing, then restore |
| Duplicate SMS sent | Workflow triggered multiple times for same booking | Add dedup logic: check booking ID against a "sent" list |
| Wrong timezone in messages | Cal.com and n8n timezone mismatch | Set n8n instance timezone and check Cal.com timezone settings |
