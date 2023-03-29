# How to Maximize User Privacy When Using Google Analytics 4

## What is Google Analytics 4?

Web analytics is the practice of gathering information about how your users are using your websites, for the purposes of marketing, sales, improving product offerings, etc...

Google Analytics is one such tool that aids in this practice. It is by far the most popular one.

Google Analytics 4 is the latest iteration of the tool. It was created in large part as a response to GDPR (General Data Protection Regulation) privacy legislation in the EU (European Union) that Universal Analytics ("Google Analytics 3") couldn't support.

## How does Google Analytics 4 Treat Privacy By Default?

Despite the increased focus on privacy, it doesn't look great. 

It has several defaults that are unnecessary for ordinary website analytics, exposing way more information than needed back to Google. 

## Steps to Take to Maximize Your Users' Privacy

### Turn Off All Account Data Sharing Settings

By default, you're opted in to sharing your users' analytics data with these 4 other entities

- Google Products & Services
- Modeling Contributions & Business Insights
- Technical Support
- Account Specialists (aka Google salespeople)

Uncheck all of them during the onboarding flow.

### Use "Device-based" for Reporting Identity

Reporting Identity refers to how Google Analytics tracks your users across different websites and different devices.

By default this is set to "Blended", which includes

- Google signals (aka tracking your users based on their having logged into their Google account on any browser or device)
- Modeling ([aka using machine learning based on users who accepted tracking to infer behaviors of users who declined tracking, so they can be tracked...](https://support.google.com/analytics/answer/11161109))

Both of these are overkill for basic website analytics, and overreach into your users' privacy.

The best alternative is actually hidden. You have to hit "Show All" to uncover the "Device-based" option.

Go to Admin, then Account Access Management, then Reporting Identity, then hit "Show All", then select "Device-based".  

#### Actually Make the "Device-based" Choice Useful

Even with this choice, [Google is still able to track your users across your site by setting a first-party cookie](https://support.google.com/analytics/answer/11593727). 

While the cookie may be first-party, Google is most certainly not. Any tracking at this level should at best be done by solutions that let you be the steward of your users' data, not Google.

To stop this tracking, you need to effectively deny tracking automatically on behalf of your users (as if they'd automatically denied all such tracking via your cookie notice)

The details vary by platform and integration, but [seem to be eventually findable in the docs (e.g. for web)](https://developers.google.com/tag-platform/devguides/consent#implementation_example).

## What You're Left With

After all that, Google Analytics 4 should be a tool that

- Captures anonymous analytics about how your users are using your site
- Lets you add your own custom anonymous instrumentation to capture events it doesn't by default

Or at least I hope so...

## Summary

Google Analytics 4 is not an analytics tool. It's an advertising and marketing tool.

That's the only framing I can make that explains why its defaults are the way they are.

P.S.

For web, here's what that final step looks like. You need to add these script tags so they're the first children of the head element.

```html
        <!-- Google tag (gtag.js) -->
        <script async src="https://www.googletagmanager.com/gtag/js?id=G-AAAAAAAAA"></script>
        <script>
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}

          gtag('consent', 'default', {
            'analytics_storage': 'denied'
          });

          gtag('js', new Date());

          gtag('config', 'AAAAAAAAA');
        </script>
```

The key edit to what Google Analytics gave me was to add that gtag call denying consent for analytics immediately after the gtag function was declared. That prevents all subsequent calls from setting cookies. It also seems to prevent any data from being saved at all in Google Analytics (though data can be sent there), so you'll unfortunately have to conditionally re-enable it to make Google Analytics be of any use.

I replaced the unique id Google Analytics gave me with a sequence of 9 A's because might as well avoid abuse