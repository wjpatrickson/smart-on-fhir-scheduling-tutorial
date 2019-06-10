---
title: SMART on FHIR Scheduling App Tutorial

language_tabs:
  - code

search: true
---

# Introduction

This tutorial will walk you through creating an app in Cerner's SMART on FHIR ecosystem. In particular, your app will focus on FHIR scheduling workflows.

After completing this tutorial you will know how to:

* Create a basic SMART on FHIR app that interacts with scheduling resources.
* Self register an app with Cerner.
* Run an app in Cerner's SMART on FHIR sandbox.

# Prerequisites

* A public [GitHub](http://www.github.com) account

# Project Setup

First, you'll want to fork this tutorial from [smart-on-fhir-scheduling-tutorial](https://github.com/MaxPhilips/smart-on-fhir-scheduling-tutorial) to your GitHub account.

The `smart-on-fhir-scheduling-tutorial/source/scheduling-app` folder contains the example SMART app which you'll be using throughout this tutorial. Let's take a look at some of the notable files contained within:

**fhir-client-v0.1.12.js**

Located in the lib/js folder, this is a version of [`fhir-client.js`](https://github.com/smart-on-fhir/client-js) which is an open source library designed to assist with calling a FHIR API and handling the SMART on FHIR authorization workflow. This tutorial uses this library when walking you through building your first SMART app.

Additional documentation on `fhir-client.js` can be found [here](http://docs.smarthealthit.org/clients/javascript/).

<aside class="notice">
This tutorial is designed to have a minimal footprint so we made the decision to directly include `fhir-client.js` for simplicity. For your production applications we'd recommend pulling in `fhir-client.js` using npm or some other package manager to easily keep your application up to date.
</aside>

**launch.html**

launch.html is the SMART app's initial entry point and in a real production environment, would be invoked by the application launching your SMART app (for instance, the EHR or patient portal). In the [SMART documentation](http://docs.smarthealthit.org/), this is your app's "launch URL". In this tutorial, this page will be invoked when you launch your app from Cerner's [code console](https://code.cerner.com/developer/smart-on-fhir/apps).

As the entry point into your SMART app, this page will kick-off the SMART authorization workflow.

**index.html**

This page will be invoked via redirect from the Authorization server at the conclusion of the SMART authorization workflow. When this page is invoked, your SMART app will have everything it needs to run and access the FHIR API.

The other content you see in the source folder is the site for this tutorial. We used [Slate](https://github.com/lord/slate) to create the documentation for this tutorial.

# 1.) Trigger a GitHub Pages Deployment

> /scheduling-app/index.html

```html
<!DOCTYPE html>
<html lang='en' hidden>
  <head>

...

  </head>
  <body>
    <div class='container'>
      <div class='row'>
        <div class='col'>
->        <h1>[GITHUB USERNAME] Scheduling App</h1>
        </div>
      </div>

...
```

> Go to your GitHub account, select the Repositories tab and select the smart-on-fhir-scheduling-tutorial repository. Select the Branch button and switch to the gh-pages branch. Directly edit `/scheduling-app/index.html` by clicking on the pencil icon.  Once you've made the change, commit directly to the gh-pages branch.

For the purposes of this tutorial we will be hosting our SMART app through [GitHub Pages](https://help.github.com/articles/what-is-github-pages). GitHub Pages is a convenient way to host static or client rendered web sites.

Setting up GitHub pages is easy, so easy in fact that it's already done for you. GitHub pages works by hosting content from a repository branch named "gh-pages". Since you forked the tutorial, the gh-pages branch has already been created, however, GitHub won't publish your site until you make a change to that branch, so let's make a change. Modify the index.html page to include your GitHub username in the app's title, and commit directly to the gh-pages branch.

Use the GitHub UI to directly edit `index.html`. Simply switch the branch to gh-pages, navigate to `/scheduling-app/index.html` and click the pencil icon. Commit your changes to deploy.

Once the app has been deployed go to `https://<github username>.github.io/smart-on-fhir-scheduling-tutorial/scheduling-app/health` to ensure your app is available.

<aside class="notice">
GitHub Pages sites have a limit of 10 builds per hour, so if your page isn't updating, this could be the reason.
</aside>

# 2.) Register Your App with Cerner

Now that we have deployed a SMART app, let's register it to access Cerner's FHIR resources. Cerner has created a self-registration console to allow developers to run SMART apps in a sandbox environment. Navigate to Cerner's [code console](https://code.cerner.com/developer/smart-on-fhir/apps). If you don't have a Cerner Care Account, go ahead and sign up for one (it's free!). Once logged into the console, click on the "+ New App" button in the top right toolbar and fill in the following details:

 Field            | Value                                                                                              | Description
------------------|----------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------
 App Name         | Cool App                                                                                           | Any name your heart desires.
 SMART Launch URI | `https://<github username>.github.io/smart-on-fhir-scheduling-tutorial/scheduling-app/launch.html` | The page that launches your app.
 Redirect URI     | `https://<github username>.github.io/smart-on-fhir-scheduling-tutorial/scheduling-app/`            | The main logic for your app.
 App Type         | Provider                                                                                           | You are creating a provider-facing facing app.
 FHIR Spec        | dstu2                                                                                              | Your app will run against Cerner's implementation of the DSTU 2 FHIR specification.
 Authorized       | Yes                                                                                                | Authorized apps will go through secured OAuth 2 login.
 Standard Scopes  | online_access                                                                                      | The already-checked standard scopes are required to launch your SMART app. Offline access is for system-facing apps, so we need online access.
 User Scopes      | Appointment write, Patient read, Patient write, and Slot read                                      | Your app needs to be able to write appointments, read and write patients, and read slots.
 Patient Scopes   | None                                                                                               | Your app does not need any patient scopes.

Click "Register" to complete the process. This will add the app to your account and create a client id for app authorization.

The new OAuth 2 client id for your app will be displayed in a banner at the top of the page and can be viewed at any time by clicking on the application icon to view more details.

<aside class="notice">
After initially registering your SMART app, it can take up to 10 minutes for your app details to propogate throughout our sandbox. So, please wait 10 minutes before trying to launch your app - you can continue working through this tutorial in the mean time.
</aside>

# 3.) Request Authorization

> /scheduling-app/launch.html

```html
<!DOCTYPE html>
<html lang='en' hidden>
  <head>

...

  </head>
  <body>

...

    <script>
      FHIR.oauth2.authorize({
->      'client_id': '[CLIENT ID]',
        'scope': 'user/Appointment.write user/Patient.read user/Patient.write user/Slot.read launch online_access openid profile'
      });
    </script>

    <div class='spinner'>
      <div class='bounce1'></div>
      <div class='bounce2'></div>
      <div class='bounce3'></div>
    </div>
  </body>
</html>
```

> Make sure to use the client id generated by the code console for your `FHIR.oauth2.authorize` call and redeploy your site.

The responsibility of `launch.html` is to redirect to the appropriate FHIR authorization server. As you can see in the code, `fhir-client.js` makes our job pretty easy. All we have to do is call `FHIR.oauth2.authorize` supplying the client id generated by Cerner's code console during registration and the scopes we registered.

Your app's client id is found on the app details page that can be accessed by clicking on the application icon in the [code console](https://code.cerner.com/developer/smart-on-fhir/apps). Copy the client id into the authorize call in `launch.html`, commit the changes back to your repo and redeploy your site.

For the purposes of this tutorial you don't need to modify the OAuth 2 scopes. This list should match the scopes that you registered the application with.

Please refer to Cerner's [base SMART on FHIR tutorial](https://engineering.cerner.com/smart-on-fhir-tutorial/#request-authorization) for more information on request authorization.

# 4.) How the App Works

## Requesting Access to FHIR Resources

> /scheduling-app/src/js/scheduling-app.js

```javascript
...

  FHIR.oauth2.ready(function(smart) {
    // Query the FHIR server for Slots
    smart.api.fetchAll({type: 'Slot', query: slotParams}).then(

      // Display Appointment information if the call succeeded
      function(slots) {

...

      },

      // Display 'Failed to read Slots from FHIR server' if the call failed
      function() {

...

      }
    );
  });

...
```

> No code changes are required for this step.

Now that the app has successfully been authenticated, it's time to call a FHIR resource, but first we need to obtain an OAuth2 access token. We have an authorization code that was passed as a query param to the redirect URI (`index.html`) by the authorization server. The authorization code is exchanged for an access token through POST to the authorization server. Again, `fhir-client.js` makes this easy for us.

The `scheduling-app.js` file registers a form on submit event which uses the `FHIR.oauth2.ready()` function to exchange the authorization code for the access token and stores it in session storage for later use.

## Accessing FHIR Resources

> /scheduling-app/src/js/scheduling-app.js - onReady

```javascript
...

    FHIR.oauth2.ready(function(smart) {
    // Query the FHIR server for Slots
    smart.api.fetchAll({type: 'Slot', query: slotParams}).then(

      // Display Appointment information if the call succeeded
      function(slots) {
        // If any Slots matched the criteria, display them
        if (slots.length) {
          var slotsHTML = '';

          slots.forEach(function(slot) {
            slotsHTML = slotsHTML + slotHTML(slot.id, slot.type.text, slot.start, slot.end);
          });

          renderSlots(slotsHTML);
        }
        // If no Slots matched the criteria, inform the user
        else {
          renderSlots('<p>No Slots found for the selected query parameters.</p>');
        }
      },

      // Display 'Failed to read Slots from FHIR server' if the call failed
      function() {
        clearUI();
        $('#errors').html('<p>Failed to read Slots from FHIR server</p>');
        $('#errors-row').show();
      }
    );
  });

...
```

> No code changes are required for this step.

With access token in hand we're ready to request a FHIR resource and again, we will be using `fhir-client.js`.

For the purposes of this tutorial we'll be retrieving information that allows the user to book appointments.

The `fhir-client.js` library defines a useful function we can use to retrieve this information: `smart.api.fetchAll()`. This uses the [fhir.js](https://github.com/FHIR/fhir.js) API to retrieve resources and returns a JavaScript promise. We then set up functions to handle success and failure cases.

On a success, we're taking the response from the FHIR server and building HTML strings to display to our users.

The `fhir-client.js` library defines several more APIs that will come in handy while developing smart app. Read about them [here](http://docs.smarthealthit.org/clients/javascript/).

## Displaying FHIR Resources

> /scheduling-app/index.html

```html
...

      <div id='slots-holder-row' class='row mt-5'>
        <div class='col'>
          <div id='slots-holder'>
            <div class='row'>
              <div class='col'>
                <h2>Slots</h2>
              </div>
            </div>
            <div class='row'>
              <div class='col'>
                <button class='btn btn-danger' id='clear-slots'>Clear Results</button>
              </div>
            </div>
            <div class='row'>
              <div class='col'>
                <div id='slots'></div>
              </div>
            </div>
          </div>
        </div>
      </div>

...
```

> scheduling-app.js - renderSlots

```javascript
...

function renderSlots(slotsHTML) {
  clearUI();
  $('#slots').html(slotsHTML);
  $('#slots-holder-row').show();
}

...
```

> No code changes are required for this step.

The last remaining task for our application is displaying the resource information we've retrieved. In `index.html` we define a hidden div element to hold Slot information. On a successful FHIR server call we'll invoke `renderSlots` which will show the div as well as update its HTML to display Slots.

# 5.) Test your App

> Any time you need to redeploy your app via GitHub Pages, commit changes to your gh-pages branch.

Now that we have a snazzy SMART app, it's time to test it.

Return to the [code console](https://code.cerner.com/developer/smart-on-fhir/apps) and click on the app you registered earlier. To launch your app through the code console click the "Begin Testing" button. The console will ask if the app you're launching requires a patient in context - your app does not, so select "No" then "Next". Please note the Millennium username and password displayed in the launch dialog, you'll need those credentials to log in and authorize your app. Finally, click "Launch" and the console will redirect to your application.

Log in with the credentials you received (you'll only need to do this once per browser session) and play with your app!

<aside class="notice">
You will find that this application is preloaded with Practitioners, Locations, and Slot types to search by. In a production scenario, you would need to work with Cerner to gather this information before deploying your app to a client site.
</aside>

In particular, some limitations to be aware of are:

* Cerner's implementation of the Practitioner resource only allows search by id, so you need to know Practitioners ahead of time to use them in your app.
* Slot types can use standard (LOINC, SNOMED, etc.) or proprietary codes. A single standard code may represent multiple proprietary codes, e.g. a standard code for Office Visit may correspond to multiple proprietary codes like Office Visit Doctor A and Office Visit Doctor B.
* Cerner's FHIR server does not implement the ValueSet resource, so again, you need to know Slot types ahead of time to use them in your app.
* Cerner's FHIR server does not implement the Location resource, so finally, you need to know Locations ahead of time to use them in your app.

Cerner will work with you before a go-live to ensure you have all the information for your app to succeed!

<aside class="notice">
You may have noticed a date range search is included in the Slot query. It is strongly recommended to use as small of a date range as you are able to - scheduling data involves a ton of database rows.
</aside>

One Practitioner may have sixteen half-hour slots on a given day. Multiply that number by each Slot type and Location they are available at. Then multiply that by the number of Practitioners a practice may contain. Even for a small office of five Practitioners with only a handful of Slot types, there could be as many as 1,600 Slots for one week. For larger offices, it is entirely possible to see numbers like ten million Slots per month! That will take a long time for your app to request.

# 6.) Add Appointment Booking

Now that we've tested searching for Slots, let's use the Slots to book Appointments.

# 7.) Add Patient Search and Create

In the previous step, all Appointments were booked for a single test patient. Let's add functionality to our app that searches for different Patients or even registers new Patients.

# Summary

Through this tutorial we have:

* Created a basic SMART on FHIR app.
* Registered that app with Cerner.
* Run the app in Cerner's SMART on FHIR sandbox.

We've created a very basic application that meets the base requirements of being a SMART app. This application would require a fair amount of polish before being ready to be deployed in a production environment. A couple of next steps you could look at are:

* Try adding more scheduling resources like Schedule to your workflow. You may want to do this by relaxing the requirement to search for Slots by Practitioner, and follow each Slot's Schedule reference to show its Practitioner to your user.
* Write unit tests for the application.
* Pull in `fhir-client.js` through a package manager like webpack.
* Localize and Internationalize your application.

We're excited to see what you'll build next!
