---
title: "I18n Frontend guideline"
seoTitle: "I18n Frontend guideline"
seoDescription: "To support multiple languages and locales, the application needs to have the necessary content available for each locale..."
datePublished: Fri Mar 31 2023 04:09:12 GMT+0000 (Coordinated Universal Time)
cuid: clfw0zfue000709mhhhh1gtau
slug: i18n-frontend-guideline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680235676681/a134c9d9-3801-45e2-a8fc-e2ae011b3803.png
tags: reactjs, i18n, frontend-development, nextjs, localization

---

In a front-end web application, locales are used to determine the language and geographic location of the user and to display the appropriate content to them.

When a user visits a web page, the application will first detect their locale, then use this information to determine which version of the content to display.

To support multiple languages and locales, the application needs to have the necessary content available for each locale. This can be achieved by creating separate files or resources for each locale or by using a localization library like `i18next` that allows for dynamic localization based on the user's detected locale.

In this frontend guideline, we will discuss general principles and provide examples using `i18next` with `React.js`

## **Locale detection**

There are several ways to detect the user's locale (i.e., their language and geographic location) in a front-end application:

* **Browser settings**: The user's browser settings may indicate their preferred language. This information can be accessed through the `navigator.language` or `navigator.userLanguage` property in JavaScript.
    
* **IP geolocation**: The user's IP address can be used to determine their geographic location. This can be done by making a request to a geolocation service, such as MaxMind or IP-API, which returns the user's location based on their IP address.
    
* **Server-side detection**: An HTTP header that relays these language preferences to the server with each request. This is the `Accept-Language` header, and it often looks something like this: `Accept-Language: en-CA,ar-EG;q=0.5`.
    
* **Use of cookies or web storage**: The user's locale preference can be stored in a cookie or HTML5 web storage when the user first interacts with the application. The locale would then be read from this location for all subsequent requests.
    

There are a few libraries where that offer locale detection based on the settings listed above. For instance, In React.js, there is the `i18next-browser-languageDetector` library that detects language based on:

* Cookie
    
* LocalStorage
    
* Navigator
    
* Query(`?lng=LANGUAGE`)
    
* HtmlTag
    
* Path
    
* Subdomain.
    

Another example would be Next.js, where the locale will be automatically detected based on the `Accept-Language` header and the current domain. Locale detection is enabled by default.

## **Internationalized routing**

Internationalized routing is a way to handle different URLs for the same page based on the user's detected locale. There are two types of URL routing:

* **Sub-path routing** (e.g. [example.com/en/home](http://example.com/en/home), [example.com/fr/home](http://example.com/fr/home))
    
* **Domain routing** (e.g. example.en, [example.fr](http://example.fr))
    

For example in React.js, the routing process can be implemented like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680235566765/4526dce5-5ca4-42c3-8ba6-a96025eb6d90.png align="center")

With Next.js, there is built-in support for internationalized routing since `v10.0.0`

```jsx
// next.config.js
module.exports = {
  i18n: {
    locales: ['en-US', 'fr', 'nl-NL', 'nl-BE'],
    defaultLocale: 'en-US',
    domains: [
      {
        // Note: subdomains must be included in the domain value to be matched
        // e.g. www.example.com should be used if that is the expected hostname
        domain: 'example.com',
        defaultLocale: 'en-US',
      },
      {
        domain: 'example.fr',
        defaultLocale: 'fr',
      },
      {
        domain: 'example.nl',
        defaultLocale: 'nl-NL',
        // specify other locales that should be redirected to this domain
        locales: ['nl-BE'],
      },
    ],
  },
}
```

## Supports LTR and RTL text

For some languages, such as Arabic, the letters are arranged from right to left. To ensure that your application supports Right-To-Left (**RTL**) layout rendering for such languages, you need to add **LTR** or **RTL** support.

To add **LTR** or **RTL** support to the application, we will set the `dir` attribute on the `body` element dynamically in the `index.html` file.

You can also set the `dir` attribute on global components such as `Header` and `Footer`.

Here's an example of setting the `dir` attribute dynamically in the `App` component:

```jsx
import React from 'react';
import { useTranslation } from 'react-i18next';
import './App.css';

function App()  {
    const { t, i18n } = useTranslation();
    document.body.dir = i18n.dir(); // return ltr or rtl of current language
    return (
      <div className="App">
        {t('welcome')}
      </div>
    );
}

export default App;
```

Here's an example of setting the `dir` attribute on a global `Header` component:

```jsx
import { useEffect } from 'react';
import { useTranslation } from 'next-i18next';

const Header = () => {
  const { t, i18n } = useTranslation('common');
  const localeDir = i18n.dir(); // return ltr or rtl of current language

  useEffect(() => {
    document.querySelector('html')?.setAttribute('dir', localeDir);
  }, [localeDir]);

  return (
    <header>
				{...}
    </header>
  );
};

export default Header;
```

## Formatting

Starting from **i18next version 21.3.0**, you can take advantage of the built-in formatting functions based on the [**Intl API**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) for the following formats:

* **Number**
    
* **Currency**
    
* **DateTime**
    
* **RelativeTime**
    
* **List**
    

With these built-in formatting functions, you can easily format and localize various types of data in your application to match the user's preferred language and regional settings, improving the overall user experience of your application.

Here's an example of how you can use them:

```jsx
// Translation JSON
{
  "number": "Number: {{val, number}}",
  "currency": "Currency: {{val, currency(USD)}}",
  "dateTime": "Date/Time: {{val, datetime}}",
  "relativeTime": "Relative Time: {{val, relativetime}}",
  "list": "List: {{val, list}}",
  "weekdays": [
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday"
  ]
}

// Number
i18next.t('number', { val: 1000 }); // --> Number: 1,000
i18next.t('number', { val: 1000.1, formatParams: { val: { minimumFractionDigits: 3 } } }); // --> Number: 1,000.100

// Currency
i18next.t('currency', { val: 2000 }); // --> Currency: $2,000.00
i18next.t('currency', {
  val: 2000.12,
  currency: 'CAD',
  locale: 'fr-CA'
}); // --> Currency: 2 000,12 $ CA

// DateTime
i18next.t('dateTime', { val: new Date(Date.UTC(2012, 11, 20, 3, 0, 0)) }); // --> Date/Time: 12/20/2012
i18next.t('dateTime', {
  val: new Date(Date.UTC(2012, 11, 20, 3, 0, 0)),
  formatParams: {
    val: { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' },
  },
}); // --> Date/Time: Thursday, December 20, 2012

// RelativeTime
i18next.t('relativeTime', { val: 3 }); // --> Relative Time: in 3 days

// List
i18next.t('list', {
  val: i18next.t('weekdays', { returnObjects: true }),
}); // --> List: Monday, Tuesday, Wednesday, Thursday, and Friday
```

## Conclusion

Multilingual is a very important function for web or mobile applications nowadays, Users will come from all over the world and always ask for support for their language. Above is a guide to help you install multi-language for your application or website, You can implement it according to the instructions to automatically find the user's language, setup multi-language by domain or subpath, support LTR and RTL text, and finally format of number, currency, time, list…

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)