You can use Amazon SES, SendGrid, EmailJS.<br>
Using SendGrid:<br>

- Create a SendGrid account
- Authenticate your sender
- Create a api key
- npm i @sendgrid/mail

```ts
import sgMail from '@sendgrid/mail'

sgMail.setApiKey(process.env.SENDGRID_API_KEY)

const msg = {
  to: 'test@example.com',
  from: {
    name: 'test',
    email: 'test@example.com', // Use the email address or domain you verified
  },
  subject: 'Sending with Twilio SendGrid is Fun',
  text: 'and easy to do anywhere, even with Node.js',
  html: '<strong>and easy to do anywhere, even with Node.js</strong>',
  attachments: [
    {
      content: attachment,
      filename: 'attachment.pdf',
      type: 'application/pdf',
      disposition: 'attachment',
    },
  ],
}

try {
  sgMail.send(msg)
} catch (error) {
  console.error(error)
}
```
