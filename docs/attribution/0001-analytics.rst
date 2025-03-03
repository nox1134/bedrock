.. This Source Code Form is subject to the terms of the Mozilla Public
.. License, v. 2.0. If a copy of the MPL was not distributed with this
.. file, You can obtain one at https://mozilla.org/MPL/2.0/.

.. _analytics:

=================
Mozorg analytics
=================

Google Tag Manager (GTM)
************************

In mozorg mode, bedrock uses `Google Tag Manager (GTM)`_ to manage and organize
its `Google Analytics`_ solution.

:abbr:`GTM (Google Tag Manager)` is a tag management system that allows for
easy implementation of Google Analytics (GA) tags and other 3rd party marketing
tags in a nice :abbr:`GUI (Graphical User Interface)` experience. Tags can be
added, updated, or removed directly from the GUI. GTM allows for a "one
source of truth" approach to managing an analytics solution in that all
analytics tracking can be inside GTM.

Bedrock's GTM solution is :abbr:`CSP (Content Security Policy)` compliant and
does not allow for the injection of custom HTML or JavaScript but all tags use
built in templates to minimize any chance of introducing a bug into Bedrock.

The GTM DataLayer
-----------------

How an application communicates with GTM is via the ``dataLayer`` object, which
is  a simple JavaScript array GTM instantiates on the page. Bedrock will send
messages to the ``dataLayer`` object by means of pushing an object literal onto
the ``dataLayer``. GTM creates an abstract data model from these pushed objects
that consists of the most recent value for all keys that have been pushed to
the ``dataLayer``.

The only reserved key in an object pushed to the ``dataLayer`` is ``event`` which
will cause GTM to evaluate the firing conditions for all tag triggers.

DataLayer push example
----------------------

If we wanted to track clicks on a carousel and capture what the image was that
was clicked, we might write a dataLayer push like this:

.. code-block:: javascript

    dataLayer.push({
        'event': 'carousel-click',
        'image': 'house'
    });

In the dataLayer push there is an event value to have GTM evaluate the firing
conditions for tag triggers, making it possible to fire a tag off the dataLayer
push. The event value is descriptive to the user action so it's clear to someone
coming in later what the dataLayer push signifies. There is also an image property
to capture the image that is clicked, in this example it's the house picture.

In GTM, a tag could be setup to fire when the event ``carousel-click`` is pushed
to the dataLayer and could consume the image value to pass on what image was clicked.

The Core DataLayer object
-------------------------

For the passing of contextual data on the user and page to GTM, we've created what we
call the Core DataLayer Object. This object passes as soon as all required API calls
for contextual data have completed. Unless there is a significant delay to when data
will be available, please pass all contextual or meta data on the user or page here
that you want to make available to GTM.


Conditional banners
~~~~~~~~~~~~~~~~~~~

When a banner is shown:

.. code-block:: javascript

    // UA
    window.dataLayer.push({
        'eLabel': 'Banner Impression',
        'data-banner-name': '<banner name>', //ex. Fb-Video-Compat
        'data-banner-impression': '1',
        'event': 'non-interaction'
    });
    // GA4
    window.dataLayer.push({
        event: 'widget_action',
        type: 'banner',
        action: 'display',
        name: '<banner name>', //ex. Fb-Video-Compat
        non_interaction: true
    });

When an element in the banner is clicked:

.. code-block:: javascript

    // UA
    window.dataLayer.push({
        'eLabel': 'Banner Click (OK)',
        'data-banner-name': '<banner name>', //ex. Fb-Video-Compat
        'data-banner-click': '1',
        'event': 'in-page-interaction'
    });
    // GA4
    window.dataLayer.push({
        event: 'widget_action',
        type: 'banner',
        action: 'clickthrough',
        name: '<banner name>', //ex. Fb-Video-Compat
    });

When a banner is dismissed:

.. code-block:: javascript

    // UA
    dataLayer.push({
        'eLabel': 'Banner Dismissal',
        'data-banner-name': '<banner name>', //ex. Fb-Video-Compat
        'data-banner-dismissal': '1',
        'event': 'in-page-interaction'
    });
    // GA4
    window.dataLayer.push({
        event: 'widget_action',
        type: 'banner',
        action: 'dismiss',
        name: '<banner name>' //ex. Fb-Video-Compat
    });


A/B tests
~~~~~~~~~

.. code-block:: javascript

    if (href.indexOf('v=a') !== -1) {
        // UA
        window.dataLayer.push({
            'data-ex-variant': 'de-page',
            'data-ex-name': 'Berlin-Campaign-Landing-Page'
        });
        // GA4
        window.dataLayer.push({
            event: 'experiment_view',
            id: 'Berlin-Campaign-Landing-Page',
            variant: 'de-page',
        });
    } else if (href.indexOf('v=b') !== -1) {
        // UA
        window.dataLayer.push({
            'data-ex-variant': 'campaign-page',
            'data-ex-name': 'Berlin-Campaign-Landing-Page'
        });
        // GA4
        window.dataLayer.push({
            event: 'experiment_view',
            id: 'Berlin-Campaign-Landing-Page',
            variant: 'campaign-page',
        });
    }

GTM listeners & data attributes
-------------------------------

GTM also uses click and form submit listeners to gather context on what is happening
on the page. Listeners push to the dataLayer data on the specific element that
triggered the event, along with the element object itself.

Since GTM listeners pass the interacted element object to the dataLayer, the use of
data attributes works very well when trying to identify key elements that you want to
be tracked and for storing data on that element to be passed into Google Analytics. We
use data attributes to track clicks on all downloads, buttons elements, and nav, footer,
and :abbr:`CTA (Call To Action)`/button link elements.

.. Important::

    When adding any new elements to a Bedrock page, please follow the below guidelines
    to ensure accurate analytics tracking.

Generic CTAs
~~~~~~~~~~~~

For all generic CTA links and ``<button>`` elements, add these data attributes
(* indicates a required attribute):

+-----------------------+---------------------------------------------------------------------------+
| Data Attribute        | Expected Value (lowercase)                                                |
+=======================+===========================================================================+
| ``data-cta-type`` *   | Link type (e.g. ``navigation``, ``footer``, or ``button``)                |
+-----------------------+---------------------------------------------------------------------------+
| ``data-cta-text``     | name or text of the link                                                  |
+-----------------------+---------------------------------------------------------------------------+
| ``data-cta-position`` | Location of CTA on the page (e.g. ``primary``, ``secondary``, ``header``) |
+-----------------------+---------------------------------------------------------------------------+

For all links to accounts.firefox.com use these data attributes (* indicates a required attribute):

+-----------------------+----------------------------------------------------------------------------------+
| Data Attribute        | Expected Value                                                                   |
+=======================+==================================================================================+
| ``data-cta-type`` *   | fxa-servicename (e.g. ``fxa-sync``, ``fxa-monitor``)                             |
+-----------------------+----------------------------------------------------------------------------------+
| ``data-cta-text``     | Name or text of the link (e.g. ``Sign Up``, ``Join Now``, ``Start Here``).       |
|                       |                                                                                  |
|                       | We use this when the link text is not useful, as is the case with many           |
|                       | account forms that say, ``Continue``. We replace ``Continue`` with ``Register``. |
+-----------------------+----------------------------------------------------------------------------------+
| ``data-cta-position`` | Location of CTA on the page (e.g. ``primary``, ``secondary``, ``header``)        |
+-----------------------+----------------------------------------------------------------------------------+

Links identified with ``data-cta-type`` become UA events with the following format:

| **Category:** ``cta click``
| **Action:** ``cta: {{data-cta-type}}``
| **Label:** ``{{data-cta-text}}``
| **CD Index 9 - CTA Position:** ``{{data-cta-position}}``


Download Links
~~~~~~~~~~~~~~

For Firefox download buttons, add these data attributes (* indicates a required attribute).
Note that ``data-download-name`` and ``data-download-version`` should be included for download
buttons that serve multiple platforms. For mobile specific store badges, they are not strictly
required.

+----------------------------+-------------------------------------------------------------------------------------------------------------+
| Data Attribute             | Expected Value                                                                                              |
+============================+=============================================================================================================+
| ``data-link-type`` *       | ``download``                                                                                                |
+----------------------------+-------------------------------------------------------------------------------------------------------------+
| ``data-download-os`` *     | ``Desktop``, ``Android``, ``iOS``                                                                           |
+----------------------------+-------------------------------------------------------------------------------------------------------------+
| ``data-download-name``     | ``Windows 32-bit``, ``Windows 64-bit``, ``macOS``, ``Linux 32-bit``, ``Linux 64-bit``, ``iOS``, ``Android`` |
+----------------------------+-------------------------------------------------------------------------------------------------------------+
| ``data-download-version``  | ``win``, ``win64``, ``osx``, ``linux``, ``linux64``, ``ios``, ``android``                                   |
+----------------------------+-------------------------------------------------------------------------------------------------------------+
| ``data-download-location`` | ``primary``, ``secondary``, ``nav``, ``other``                                                              |
+----------------------------+-------------------------------------------------------------------------------------------------------------+


GA4
---

.. Note::

    The migration to GA4 has begun but is incomplete.

Enhanced Event Measurement
~~~~~~~~~~~~~~~~~~~~~~~~~~

Pageviews, video events, and external link clicks are being collected using GA4's
`enhanced event measurement`_.

Some form submissions are also being collected but newsletter signups are not.
`(See Bug #13348)`_


Begin Checkout
~~~~~~~~~~~~~~

We are using GA4's recommended eCommerce event `begin_checkout`_ for VPN and Relay
referrals to the FxA Subscription Platform with purchase intent.

.. Note::

    Any link to Mozilla accounts should also be using :ref:`mozilla accounts attribution<mozilla-accounts-attribution>`


``datalayer-begincheckout.es6.js`` contains generic functions
that can be called on to push the appropriate information to the dataLayer. The
script is expecting the following values:

- item_id: Stripe Plan ID
- brand: ``relay``, ``vpn``, or ``monitor``
- plan:
   - ``vpn-monthly``
   - ``vpn-yearly``
   - ``vpn-relay-yearly``
   - ``relay-email-monthly``
   - ``relay-email-yearly``
   - ``relay-phone-monthly``
   - ``relay-phone-yearly``
   - ``monitor-monthly``
   - ``monitor-yearly``
- period: ``monthly`` or ``yearly``
- price: cost displayed at checkout, pre tax (example: 119.88)
- currency: in `3-letter ISO 4217 format`_ (examples: USD, EUR)
- discount: value of the discount in the same currency as price (example: 60.00)


There are two ways to use TrackBeginCheckout:

1) Call the function passing the values directly.

.. code-block:: javascript

    TrackBeginCheckout.getEventObjectAndSend(item_id, brand, plan, period, price, currency, discount)

2) Pass the values as a data attribute.

The ``vpn_subscribe_link`` will automatically generate a ``data-ga-item`` object
and add the ``ga-begin-checkout`` class to links they create -- as long as there is analytics information
associated with the plan in its lookup table.

To use this method you will need to include ``datalayer-begincheckout-init.es6.js`` in the page bundle.

.. code-block:: html

    <a href="{{ fxa link }}"
        class="ga-begin-checkout"
        data-ga-item="{
            'id' : 'price_1Iw7qSJNcmPzuWtRMUZpOwLm',
            'brand' : 'vpn',
            'plan' : 'vpn',
            'period' : 'monthly',
            'price' : '9.99',
            'discount' : '0',
            'currency' : 'USD'
        }"
    >
        Get monthly plan
    </a>


CTA Click
~~~~~~~~~

Like our UA implementation (documented above) the implementation of ``cta_click`` for GA4 is based of
the existence of certain data-attributes on an element.

Only one of the following data-attributes is necessary to log the event:

- data-cta-type (examples: fxa-sync, fxa-monitor, fxa-vpn, monitor, relay, pocket)
  - This is to group CTAs by their destination
  - Do not use this to identify the element (ie. link, button)
- data-cta-position (examples: banner, pricing, primary, secondary)
- data-cta-text
  - If no value is provided the text of the clicked element will be used
  - Please use this when the link text is not useful.
  - Also, if it's not useful to us we might be failing our users as well! Don't use text like
  "click here" or "learn more"

.. code-block:: html

    <a href="https://monitor.firefox.com/&entrypoint={{ _entrypoint }}" data-cta-type="fxa-monitor">Check for breaches</a>

    <a href="{{ url('firefox.browsers.mobile.get-app') }}" data-cta-position="banner" data-cta-text="Get It Now">Send me a link</a>

    <a href="{{ url('firefox.browsers.mobile.ios') }}" data-cta-text="Firefox for iOS">Firefox for iOS</a>


For all links to accounts.firefox.com use these data attributes (* indicates a required attribute):

+-----------------------+----------------------------------------------------------------------------------+
| Data Attribute        | Expected Value                                                                   |
+=======================+==================================================================================+
| ``data-cta-type`` *   | fxa-servicename (e.g. ``fxa-sync``, ``fxa-monitor``)                             |
+-----------------------+----------------------------------------------------------------------------------+
| ``data-cta-text``     | Name or text of the link (e.g. ``Sign Up``, ``Join Now``, ``Start Here``).       |
|                       |                                                                                  |
|                       | We use this when the link text is not useful, as is the case with many           |
|                       | account forms that say, ``Continue``. We replace ``Continue`` with ``Register``. |
+-----------------------+----------------------------------------------------------------------------------+
| ``data-cta-position`` | Location of CTA on the page (e.g. ``primary``, ``secondary``, ``header``)        |
+-----------------------+----------------------------------------------------------------------------------+


Product Download
~~~~~~~~~~~~~~~~

.. Important::

    Only Firefox and Pocket are currently supported. VPN support has not been added.

We are using a the custom event `product_download` to track product downloads and app store referrals
for Firefox, Pocket, and VPN. We are not using the default GA4 event file_download for a combination of reasons:
it does not trigger for the Firefox file types, we would like to collect more information than is included with
the default events, and we would like to treat product downloads as goals but not all file downloads are goals.

.. Note::

    Most apps listed in *appstores.py* are supported but you may still want to check that the URL
    you are tracking is identified as valid in ```isValidDownloadURL``` and will be recognized by ```getEventFromUrl``.

Properties for use with `product_download` (not all products will have all options):

- product (example: firefox)
- platform (example: win64)
- method (store, site, or adjust)
- release_channel (example: nightly)
- download_language (example: en-CA)

There are two ways to use TrackProductDownload:

1) Call the function, passing it the same URL you are sending the user to:

.. code-block:: javascript

    TrackProductDownload.sendEventFromURL(downloadURL);

2) Add a class to the link:

.. code-block:: html

    <a href="{{ link }}" class="ga-product-download">Link text</a>

You do NOT need to include ``datalayer-productdownload-init.es6.js`` in the page bundle, it is already included
in the site bundle.

Widget Action
~~~~~~~~~~~~~

We are using the custom event ``widget_action`` to track the behaviour of javascript widgets.


**How do you chose between ``widget_action`` and ``cta_click``?**

+-------------------------------------------------+-------------------------------------------------+
| widget_action                                   | cta_click                                       |
+=================================================+=================================================+
| The action is specific or unique.               | The action is "click".                          |
|                                                 |                                                 |
| *(Only the language switcher changes*           |                                                 |
| *the page language.)*                           |                                                 |
+-------------------------------------------------+-------------------------------------------------+
| The user does not leave the page.               | It sends the user somewhere else.               |
+-------------------------------------------------+-------------------------------------------------+
| It requires Javascript to work.                 | No JS required.                                 |
+-------------------------------------------------+-------------------------------------------------+
| It can perform several actions.                 | It does one action.                             |
|                                                 |                                                 |
| *(A modal can be opened and closed.)*           |                                                 |
+-------------------------------------------------+-------------------------------------------------+
| There could be several on the page              | There could be several on the page              |
| doing different things.                         | doing the same thing.                           |
|                                                 |                                                 |
| *(An accordion list of FAQs)*                   | *(A download button in the header and footer.)* |
+-------------------------------------------------+-------------------------------------------------+


Properties for use with `widget_action`  (not all widgets will use all options):

- type
    - **Required.**
    - The type of widget.
    - Examples: "modal", "protection report", "affiliate notification", "help icon".
    - *Avoid “button” or “link”. If you want to track a link or button use `cta_click`.*
- action
    - **Required.**
    - The thing that happened.
    - Examples: "open", "accept", "timeout", "vote up".
    - *Avoid “click”. If you want to track a click use `cta_click`.*
- name
    - Give the widget a name.
    - This can help you group actions from the same widget, or make it easier to find
      the widget in the reports.
    - The dashes are not required but they're allowed if you want to match the element
      class or ID.
    - Examples: "dad-joke-banner", "focus-qr-code", "Join Firefox Modal"
- text
    - How is this action labeled to the user?
    - Examples: "Okay", "Check your protection report", "Get the app"
- non_interaction (boolean)
    - True if the action was triggered by something other than a user gesture.
    - If it's not included we assume the value is *false*

To use ``widget_action`` push your event to the ``dataLayer``:

.. code-block:: js

    window.dataLayer.push({
        event: 'widget_action',
        type: 'banner',
        action: 'accept',
        name: 'dad-jokes-banner'
    });

    window.dataLayer.push({
        event: 'widget_action',
        type: 'modal',
        action: 'open',
        name: 'help-icon'
        text: 'Get Browser Help'
    });

    window.dataLayer.push({
        event: 'widget_action',
        type: 'vote',
        action: 'helpful',
        name: 'vpn-resource-center'
        text: 'What is an IP address?'
    });

    window.dataLayer.push({
        event: 'widget_action',
        type: 'details',
        action: 'open',
        name: 'relay-faq'
        text: 'Where is Relay available?'
    });

Default Browser
~~~~~~~~~~~~~~~

Trigger this event when a user sets their default browser to Firefox. It's an important conversion for us!

.. code-block:: javascript

    window.dataLayer.push({
        event: 'default_browser_set',
    });


User Scoped Custom Dimensions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When using GA4 through GTM there isn’t a way to set user scoped custom dimensions without an accompanying event.
The custom event we use for this is `dimension_set`.

.. code-block:: javascript

    window.dataLayer.push({
        event: 'dimension_set',
        firefox_is_default: true
    });

User scoped custom dimensions must be configured in GA4. The list of supported custom dimensions is:

- firefox_is_default (boolean)


How can visitors opt out of GA?
-------------------------------

Visitors to the website can opt-out of loading Google Analytics on our
website by enabling `Do Not Track (DNT)`_ in their web browser. We
facilitate this by using a `DNT helper`_ that our team maintains.


Glean
*****

Currently in an evaluation phase, bedrock is now capable of running a parallel
first-party analytics implementation alongside :abbr:`GTM (Google Tag Manager)`,
using Mozilla's own `Glean`_ telemetry :abbr:`SDK (Software Development Kit)`.
See the `Glean Book`_ for more developer reference documentation.

Glean is currently behind a feature switch called ``SWITCH_GLEAN_ANALYTICS``.
When the switch is enabled pages will load the Glean JavaScript bundle,
which will do things like record page hits and link click events that we want
to measure.

Debugging pings
---------------

Glean supports debugging pings via a set of flags that can be enabled directly
in the browser's web console.

- ``window.Glean.setLogPings(true)`` (enable verbose ping logging in the web console).
- ``window.Glean.setDebugViewTag('bedrock')`` (send pings to the `Glean debug dashboard`_ with the tag name ``bedrock``).

.. Note::

    After enabling Glean debugging in the web console, it will be remembered
    when navigating across pages using ``sessionStorage``. To stop debugging,
    you need to either close the browser tab, or delete the items from
    ``sessionStorage``. You can disable ping logging by calling
    ``window.Glean.setLogPings(false)``.

Filtering out non-production pings
----------------------------------

Bedrock will also set an ``app_channel`` tag with a value of either ``prod`` or
``non-prod``, depending on the environment. This is present in all pings in the
``client_info`` section, and is useful for filtering out non-production data
in telemetry dashboards.

Defining metrics and pings
--------------------------

All of the data we send to the Glean pipeline is defined in
:abbr:`YAML (Yet Another Markup Language)` schema files in the ``./glean/``
project root directory. The ``metrics.yaml`` file defines all the different
metrics types and events we record.

.. Note::

   Before running any Glean commands locally, always make sure you have first
   activated your virtual environment by running ``pyenv activate bedrock``.

When bedrock starts, we automatically run ``npm run glean`` which parses these
schema files and then generates some JavaScript library code in
``./media/js/libs/glean/``. This library code is not committed to the repository
on purpose, in order to avoid people altering it and becoming out of sync with
the schema. This library code is then imported into our Glean analytics code in
``./media/js/glean/``, which is where we initiate page views and capture click
events.

Running ``npm run glean`` can also be performed independently of starting bedrock.
It will also first lint the schema files.

.. Important::

    All metrics and events we record using Glean must first undergo a `data review`_
    before being made active in production. Therefore anytime we make new additions
    to these files, those changes should also undergo review.

Using Glean events in individual page bundles
---------------------------------------------

Our analytics code for Glean lives in a single bundle in the base template,
which is intended to be shared across all web pages. This bundle automatically
initializes Glean and records page hit events. It also creates some helpers
that can be used across different page bundles to record interaction events
such as link clicks and form submissions.

The ``Mozilla.Glean.pageEvent()`` helper can be used to record events that are
specific to a page, such as successful form completions:

.. code-block:: javascript

    if (typeof window.Mozilla.Glean !== 'undefined') {
        window.Mozilla.Glean.pageEvent({
            label: 'newsletter-sign-up-success',
            type: 'mozilla-and-you' // type is optional
        });
    }

It can also be used to record non-interaction events that are not directly
initiated by a visitor:

.. code-block:: javascript

    if (typeof window.Mozilla.Glean !== 'undefined') {
        window.Mozilla.Glean.pageEvent({
            label: 'firefox-default',
            nonInteraction: true
        });
    }

The ``Mozilla.Glean.clickEvent()`` helper can be used to record click events
that are specific to an element in a page, such as a link or button.

.. code-block:: javascript

    if (typeof window.Mozilla.Glean !== 'undefined') {
        window.Mozilla.Glean.clickEvent({
            label: 'firefox-download',
            type: 'macOS, release, en-US', // type is optional
            position: 'primary' // position is optional
        });
    }

How can visitors opt out of Glean?
----------------------------------

Website visitors can opt out of Glean by visiting the first party `data preferences page`_,
which is linked to in the `websites privacy notice`_. Clicking opt-out will set a
cookie which Glean checks for before initializing on page load. In production, the
cookie that is set applies for all ``.mozilla.org`` domains, so other sites such as
``developer.mozilla.org`` can also make use of the opt-out mechanism.

Where can I view Glean data?
----------------------------

In early 2024 we will have an automated web property dashboard for www.mozilla.org in
Looker. This dashboard will feature things like sessions and click events, once those
are standardized in the Glean SDK. Until then, we have a temporary `STMO dashboard`_
(sql.telemetry.mozilla.org) for querying page load events. We also have a data pipeline
`event monitoring dashboard`_ in Looker that is updated hourly. This is useful for
monitoring event patterns, spikes, and errors.

It is also possible to create more complex queries for Glean events using any of our
standard Telemetry tools. The easiest way to do this is via the `Glean Dictionary`_.
For example, if you view the `events ping`_, you will see a table of links in the
"Access" section (see screenshot below) that contain different links to query the
event data.

.. image:: ../images/glean-dictionary.png
    :alt: Screenshot of the 'Access' table in the Glean Dictionary

.. _Google Tag Manager (GTM): https://tagmanager.google.com/
.. _Google Analytics: https://analytics.google.com/
.. _enhanced event measurement: https://support.google.com/analytics/answer/9216061
.. _begin_checkout: https://developers.google.com/analytics/devguides/collection/ga4/reference/events?client_type=gtm#begin_checkout
.. _3-letter ISO 4217 format: https://en.wikipedia.org/wiki/ISO_4217#Active_codes
.. _(See Bug #13348): https://github.com/mozilla/bedrock/issues/13348
.. _Do Not Track (DNT): https://support.mozilla.org/en-US/kb/how-do-i-turn-do-not-track-feature
.. _DNT helper: https://github.com/mozmeao/dnt-helper
.. _Glean: https://docs.telemetry.mozilla.org/concepts/glean/glean.html
.. _Glean Book: https://mozilla.github.io/glean/book/index.html
.. _Glean debug dashboard: https://debug-ping-preview.firebaseapp.com/
.. _data review: https://wiki.mozilla.org/Data_Collection
.. _data preferences page: https://www.mozilla.org/privacy/websites/data-preferences/
.. _websites privacy notice: https://www.mozilla.org/privacy/websites/
.. _STMO dashboard: https://sql.telemetry.mozilla.org/dashboard/bedrock-landing-page-dashboard?p_date=d_last_30_days
.. _event monitoring dashboard: https://mozilla.cloud.looker.com/dashboards/1452?Event+Name=%22page_load%22&App+Name=www.mozilla.org&Window+Start+Time=28+days&Channel=
.. _Glean Dictionary: https://dictionary.telemetry.mozilla.org/apps/bedrock
.. _events ping: https://dictionary.telemetry.mozilla.org/apps/bedrock/pings/events
