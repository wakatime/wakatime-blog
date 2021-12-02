---
Title: How to Automate Responding to GDPR Data Removal Request Emails
Date: 2018-10-03
Image: https://wakatime.com/static/img/blog/gdpr-automated.jpg
Description: Using Gmail Canned Responses and Auto-Respond Filter to automate GDPR data removal request emails.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorAvatar: https://wakatime.com/photo/@alan?size=420
Category: Engineering
Tags: gdpr
---

<img src="https://wakatime.com/static/img/blog/gdpr-automated.jpg" alt="Automate responding to GDPR Data Removal Request emails" style="width:100%" />

Do you receive emails from your website users with the subject “Data Removal Request” that look like this?

    I hereby withdraw my consent for you to collect, process or store any
    personal data related to, and belonging to gdprfanboy@gmail.co.eu.
    Pursuant to my rights under Article 17 GDPR I request that you delete
    any and all related data.

    Thank you!

These data removal request emails are sent on behalf of your users when they sign up for a third party “unsubscribe” tool.
This tool works by scraping your emails for signup confirmations from different services providers.
I’ll ignore the huge problem with giving a random tool full access to read your emails.
However what happens when you, as a website operator, receive one of these automated data removal request emails?

If you’re like [WakaTime][wakatime], you’ve already provided a way for users to delete their account and all related data even before GDPR.
However, you still have to manually read and reply to automated GDPR emails even if it’s just to point them to your data removal page.

Let’s fix this by auto-responding to GDPR emails using Gmail Filters and Canned Responses!

*Note: I’m not a lawyer, and it’s possibly safer to just never respond. Use at your own risk.*

Here’s the email WakaTime auto-replies with after receiving a GDPR Data Removal request:

    Hi there,

    WakaTime cares about your privacy, but we can not comply to delete accounts
    via email as that is very insecure. Deleting your WakaTime account removes
    all personal data related to your account, sending us an email does not.

    To delete your account and all related personal data, go here:
    https://wakatime.com/settings/delete_account

    Best Regards,
    Your WakaTime Team

To set this up for your website, first enable Canned Responses in your Gmail.

<img src="https://wakatime.com/static/img/blog/35-gdpr-canned-response-1.png" class="img-thumbnail" alt="Gmail canned responses setting" style="width:80%" />

Next, compose an email as if you were replying to a GDPR request.
Instead of sending the email, save it as a Canned Response.

<img src="https://wakatime.com/static/img/blog/35-gdpr-canned-response-2.png" class="img-thumbnail" alt="Gmail save canned response" style="width:80%" />

Now that you have the canned response, search your Gmail for:

`article 17 gdpr I request that you delete any and all data related to`

<img src="https://wakatime.com/static/img/blog/35-gdpr-filter-step-1.png" class="img-thumbnail" alt="Create a Gmail filter" style="width:80%" />

Finally, configure your filter to skip your inbox and send your canned GDPR response.

<img src="https://wakatime.com/static/img/blog/35-gdpr-filter-step-2.png" class="img-thumbnail" alt="Auto respond with canned response" style="width:80%" />

From now on you won’t be bothered by GDPR Data Removal Request emails so you can focus your time and energy on building a useful product!


[wakatime]: https://wakatime.com
