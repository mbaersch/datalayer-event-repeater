# DataLayer Event Repeater 

**Custom Variable Template for Google Tag Manager**

Climbs up the dataLayer to a specific event and repeats all events that match a list of event keys.

![Template Status](https://img.shields.io/badge/Community%20Template%20Gallery%20Status-Submitted-orange) ![Repo Size](https://img.shields.io/github/repo-size/mbaersch/datalayer-event-repeater) ![License](https://img.shields.io/github/license/mbaersch/datalayer-event-repeater)

---

This template is meant to help with early dataLayer pushes for ecommerce tracking and other events that may not be processed until a consent manager adds the current consent state to the dataLayer. Usually a site initializes a dataLayer with ecommerce events that occur when a page is loaded. Product detail views, checkout event, purchases and other pushes might me too far down the dataLayer to be used to fire certain tags that need consent. 

Trigger groups are the usual way to "pull event up the dataLayer" once a consent manager is loaded and communicates the current consent conditions via dataLayer events and / or variables. 

Use this tag template to repeat all events that occured before a certain (consent) condition was met if you want to avoid  trigger groups in your setup. 

## What it does
This tag template can be used to re-push events with specific `event` keys. When fired, the tag gets a copy of the current dataLayer and compares every `event` value for every push with all event names specified in the list, starting with the first event that occured in the dataLayer. 

## Usage

### Define events to repeat
Add at least one event name to the list of events to process. Every dataLayer object with an `event` key will be compared when this tag runs. 

If event names match, the former event...

- is (optionally) changed so that the existing `gtm.uniqueEventId` is not published again (GTM will add a new id). The old id will be preseved as `repeater.originalEventId`. Note: if you deactivate this option, you will not be able to see repeated events in Google Tag Assistant!
- optionally gets an additional key `repeater.isRepeatedEvent` with value of `true` 
- gets pushed to the dataLayer again  

**Clear ecommerce**: If a repeated dataLayer object has an `ecommerce` key, it can be reset before the event gets pushed to the dataLayer again. This is recommended in order to jave a clean `ecommerce` dataLayer when tags fire.  

By default, all events that exist in the dataLayer at the time of triggering this tag are re-processed if they match one of the defined event names. If events with the same name are present more than once, all occurrences will be repeated by this tag. 

### Optional "stop marker" event 
Limit this process by defining a specific event name if neccessary. When this event is found in the copy of previous dataLayer events, the tag stops processing all following events. 

You can optionally define a key name that has to be part of the copied dataLayer push and needs to contain a specific value. Using this, a "stop marker" event can occur several times and will be ignored until the desired key and value are set with this event (a specific consent information for example). 

**Note**: If the "stop marker" cannot be found, all events will be processed that were already present in the dataLayer at the moment of tag execution. Which would be the same result like picking *None* as *Stop condition* (which is default). 

**Important**: Note that this tag should be set to triggered **"once per page"** if your setup might repeat an event that is used to trigger this tag, resulting in an endless loop or events! 
