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