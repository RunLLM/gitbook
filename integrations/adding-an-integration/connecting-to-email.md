# Connecting to Email

You can connect aqueduct to your email account in order to receive automated notifications on your workflow execution status. You can simply click the email icon on your [integrations page](../../integrations.md). Aqueduct will ask for [#sender](connecting-to-email.md#configuring-sender), [#receivers](connecting-to-email.md#configuring-receivers), and [#workflow](connecting-to-email#configuring-global-workflow-settings) information to connect to an email integration.

## Configuring Sender
You need to provide an email account for Aqueduct to send notifications on your behalf. Please refer to the guide based on your email providers:
* [Gmail](connecting-to-email.md#gmail)
* [Microsoft Exchange](connecting-to-email.md#microsoft-exchange)
* [Other Email Providers](connecting-to-email.md#other-email-providers)
### Gmail
* Host: `smtp.gmail.com`
* Port: 587
* Sender Address: Your Gmail account address (e.g. myemail@gmail.com).
* Password: Your gmail account's **Application password**. This password is **different** from your regular password. To create one, follow these steps:
    * Go to your Google account's [security management](https://myaccount.google.com/security) page.
    * Go to **Signing in to Google** section.
    * Select **App passwords**.
    * Select **Generate** and create the password.
    <figure><img src="../../.gitbook/assets/connecting_email_google_security_management.png" alt=""><figcaption></figcaption></figure>
### Microsoft Exchange
* Host: `smtp.office365.com`
* Port: 587
* Sender Address: Your Office 365 address (e.g. myemail@mydomain.com).
* Password: Your Office 365 password.
### Other Email Providers
* Host: The SMTP server address, excluding the port. You should check with your email provider on this.
* Port: The SMPT server port. You should check with your email provider on this.
* Sender Address: The account identifier to sign-in, typically the email address.
* Password: The credentials to the above account, typically the log-in password.
## Configuring Receivers
Aqueduct supports sending notifications to multiple receivers. To do so, simply put all receivers' email addresses as a comma-separated-list in the **Receivers** section.

## Configuring Default Workflow Settings
You can configure the default behavior on how the notification applies to all workflows. If you choose to enable the notification, you also need to select levels at which you would like to be notified. You can also modify this setting in your account settings page.

For individual workflows where you would like to be notified differently, you can override this default settings in [workflow settings tab](../../workflows/editing-a-workflow.md).