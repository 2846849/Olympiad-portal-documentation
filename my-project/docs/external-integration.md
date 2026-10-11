# 6. External Integration — EmailJS

EmailJS is the external service integration required of the project, tied to the
round reminder, follow-up, and results notification features.

Because these notifications must fire automatically when a round opens or closes,
independent of any user action in a browser, EmailJS is called from the **Express
scheduling module** using its Node.js SDK and a private key, rather than from the
React frontend using its browser SDK.

!!! warning "Configuration notes"
    - Non-browser API access must be enabled in the EmailJS account's security
      settings for backend calls to succeed.
    - The send method is limited to **one request per second**, so notifications
      sent to multiple schools at once are queued and sent sequentially rather
      than issued in a single loop.

# Nager.Date API Integration

## Purpose

Olympiad Portal integrates the **Nager.Date API** to identify South African public holidays when organisers schedule Olympiad rounds.

The integration helps organisers avoid unintentionally scheduling rounds on public holidays. If a selected date falls on a public holiday, the system displays a warning. Organisers can then change the date or proceed with the original schedule.

## Where to Find It in the Application

1. Log in to Olympiad Portal as an **Organiser**.
2. Navigate to the **Organiser Dashboard**.
3. Select **Create Round**.
4. Choose an Olympiad and enter the round's opening and closing dates.
5. If either date falls on a South African public holiday, the system displays a warning identifying the holiday.
6. The organiser can select **Change Dates** or **Create Anyway**.

## Example

To demonstrate the integration, select **16 December 2026 (Day of Reconciliation)** as a round opening or closing date.

The system should display a **"Public holiday clashes detected"** warning, allowing the organiser to reconsider the schedule.

**External API:** https://date.nager.at

**Integration Type:** Third-party REST API