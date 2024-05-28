# DataLayer Event Repeater 

**Custom Tag Template for Google Tag Manager**

Climbs up or down the dataLayer and repeats events that match a list of desired event keys. Optionally repeats only once and / or until a specific event is found.

![Template Status](https://img.shields.io/badge/Community%20Template%20Gallery%20Status-published-green)](https://tagmanager.google.com/gallery/#/owners/mbaersch/templates/datalayer-event-repeater) ![Repo Size](https://img.shields.io/github/repo-size/mbaersch/datalayer-event-repeater) ![License](https://img.shields.io/github/license/mbaersch/datalayer-event-repeater)


**Installing the template**

As long as this template can not be installed using the Template Gallery, you can manually import the `template.tpl` file from this repository. After adding a new template in the *Templates -> Tag Templates* section (*New* button), select *Import* from the dot menu in the opper right corner of the *Templae Editor* and hit *Save*.   

---

## Release Notes
*2024-04-25*
- added options for "Direction" (up or down the dataLayer) and "Frequency" (only once or every match) to have more control when used in a single page application 

*2024-05-10*
- new options for adding a suffix to every repeated event name and a random event id in a `randomEventId` key (e.g. for deduplication)  

## Description
This template is meant to help with early dataLayer pushes for ecommerce tracking and other events that may not be processed until a consent manager adds the current consent state to the dataLayer. Usually a site initializes a dataLayer with ecommerce events that occur when a page is loaded. Product detail views, checkout event, purchases and other pushes might me too far down the dataLayer to be used to fire certain tags that need consent. 

Trigger groups are the usual way to "pull event up the dataLayer" once a consent manager is loaded and communicates the current consent conditions via dataLayer events and / or variables. 

Use this tag template to repeat all events that occured before a certain (consent) condition was met if you want to avoid  trigger groups in your set-up. 

### What it does
This tag template can be used to re-push events with specific `event` keys. When fired, the tag gets a copy of the current dataLayer and compares every `event` value for every push with all event names specified in the list, starting with the first event that occured in the dataLayer. 

### Example
A product detail page pushes a `view_item` event (7) before `consent_status` (10) occurs. It triggers no tags because there is no (known) consent available from dataLayer variables used to block tags without consent. 

![example dataLayer in Tag Assistant](https://github.com/mbaersch/datalayer-event-repeater/blob/main/res/example.png)

At `consent_status` (10), a *Data Layer Request Repeater* tag is fired that repeats all ecommerce events. Now that consent variables are populated correctly, a GA4 event tag and TikTok Pixel are fired when the repeated `view_item` event (13) enters the dataLayer again. The *Message* (12) right before the new `view_item` contains a `ecommerce:null` reset push as defined in the tag´s options (see description).   

## Usage

### Define events to repeat
Add at least one event name to the list of events to process. Every dataLayer object with an `event` key will be compared when this tag runs. 

### Frequency and Direction
You can either repeat every matching event or choose to just repeat events once (compared by event name). Along with picking "*Down*" as *Direction* you can control how far "in the past" an event may be to be repeated. Especially in a SPA ("singe page application") you would not want to repeat every event since the beginning of a session if consent is granted at a later stage.  

Defining a **Direction** matters...

- if you want to keep the order of events the same ("Up")
- if a stop marker is present 
- if you run this tag in a SPA and want to repeat just recent event ("Down")  

### Optional "stop marker" event 
By default, all events that exist in the dataLayer at the time of triggering this tag are re-processed if they match one of the defined event names. If events with the same name are present more than once, all occurrences will be repeated by this tag.

Limit this process by defining a specific event name if neccessary. When this event is found in the copy of previous dataLayer events, the tag stops processing all following events. 

You can optionally define a key name that has to be part of the copied dataLayer push and needs to contain a specific value. Using this, a "stop marker" event can occur several times and will be ignored until the desired key and value are set with this event (a specific consent information for example). 

**Note**: If the "stop marker" cannot be found, all events will be processed that were already present in the dataLayer at the moment of tag execution. Which will lead to the same result as  picking *None* as *Stop condition* (which is default). 

## Additional Settings
If event names match, the former event...

- is (optionally) changed so that the existing `gtm.uniqueEventId` is not published again (GTM will add a new id). The old id will be preseved as `repeater.originalEventId`. Note: if you deactivate this option, you will not be able to see repeated events in Google Tag Assistant!
- optionally gets an additional key `repeater.isRepeatedEvent` with value of `true` 
- optionally gets another additional key `randomEventId` with a dot-separated timestamp, a random integer, and the current event name 
- may have an altered event name with an added user-defined suffix 
- gets pushed to the dataLayer again  

**Clear ecommerce**: If a repeated dataLayer object has an `ecommerce` key, it can be reset before the event gets pushed to the dataLayer again. This is recommended in order to jave a clean `ecommerce` dataLayer when tags fire.  

### Partial match and RegEx
Instead of defining every event to repeat by entering it´s full name in a list entry, you can use partial match and regular expressions to cover several events with one entry.      

When this option in *Advanced settings* is active, the tag uses a JavaScript "match" (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) to determine what events to repeat. If not, an event name must be equal to an entry on the list to be handled by the tag. If you activate this option and want to make sure that certain event names only trigger when a full match for "examle_event" is found, use `^example_event$`.  

## Triggering
Usually, a trigger that fires on all pages whenever consent state is available or was changed will be the right choice. 

When events that trigger this tag can occur multiple times (like a change of consent settings by a user), this tag should be triggered **"once per page"**. 

### Options
Depending on your set-up (GTM loads consent tool, consent loads GTM, or both are part of the code) you should consider the following:

- if your website is a single page app (SPA) using this tag might lead to wrong data in your systems, caused by "older" events that happened on a different URL or rely on other data that has  changed in the meantime

- if your *Data Layer Request Repeater* tag set-up might repeat events that are used to trigger this tag again, you can end up in an endless loop of events! 

- it can lead to a more robust container set-up when all tags that fire on potentially repeated dataLayer pushes are set to fire *once per page*, too. This might not be appropriate in every situation (especially on SPAs) 

## Testing
Before publishing, create and test several pages with consent- and ecommerce events in different order: 

- try a page with events that have to be repeated in order to work because they occur before a consent trigger event (can be hard coded for testing). To do this on an e-commerce site, try this tag on a product detail page with a `view_item` that occurs early in the dataLayer like demonstrated in the example above.  

- create a page with opposite order of consent- and e-commerce events in the dataLayer. Use this page to test if your triggers really work and no unnecessary events are repeated

- visit and test a page wurh multiple events that should be repeated (e.g. `view_item_list`)

Make sure that not only tags are fired but also include **correct data** by looking at parameters from outgoing requests in the browser or using GTM preview!

Test all cases with... 

* existing consent and then 
* repeat with missing consent that is granted afterwards by changing consent settings    
