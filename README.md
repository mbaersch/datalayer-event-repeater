# DataLayer Event Repeater 

**Custom Variable Template for Google Tag Manager**

Climbs up the dataLayer to a specific event and repeats all events that match a list of desired event keys.

---

Use this variable to repeat all events that occured before a certain condition in the dataLayer was met. Meant to help with eraly dataLayer pushes for ecommerce tracking and other events that could not be processed until a consent manager added the current consent state to the dataLayer.   

## Usage
This tag template ca be used to re-push events with specific `event` keys. When fired, the tag gets a copy of the current dataLayer and compares every `event` value for every push with all event names specified in the list, starting with the first event that occured in the dataLayer. 

### Define events to repeat
If event names match, the former event...

- is changed so that the existing `gtm.uniqueEventId` is not published again (GTM will add a new id)
- optionally gets an additional key `isRepeatedEvent` with value of `true` 
- gets pushed to the dataLayer again  

### Define a "marker" event to stop searching
... until an event with a given name is found in the dataLayer. You can optionally define a key name that has to be part of the current dataLayer push and contain a specific value. Using this, a "marker" event can occur several times and is ignored until the desired key and value are set with this event (a specific consent information for example). 

**Note**: If the "marker" cannot be found, all events will be processed that were already present in the dataLayer at the moment of tag execution. 
  