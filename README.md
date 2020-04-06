# VPaaS Virtual Classrooms Integration Guide

The new Kaltura-Newrow integration allows for live video collaboration tools to be built directly into your applications, using components of the Kaltura API. 

This guide will walk you through the steps of creating a **virtual classroom URL**, that you will embed in your applications with an iFrame. 

## Before You Begin 

To use the Kaltura API, you'll need a Kaltura account and credentials. The important credentials can be found in the Integration Settings in your KMC. 

This integration will be using your KAF endpoint, which should contain your partner ID. If you're not sure whether you have a KAF endpoint, speak to your account rep or email us at vpaas@kaltura.com.

A virtual classroom is represented as a `KalturaLocationScheduleResource`. A scheduled  session is represented by a `KalturaRecordScheduleEvent`. One resource can be used for many different events, but an event will only be associated with one resource. The classroom will be launched using an embed link that is **made up of the event ID and a Kaltura Session**, all of which will be discussed below. 

## Creating a Resource 

A schedule resource is created using the [`scheduleResource.add`](https://developer.kaltura.com/console/service/scheduleResource/action/add) action, which will return a `KalturaLocationScheduleResource` object. 
The resource *must* have a name, and it should include **tags** of `vcprovider:newrow` in order to be recognized as a virtual classroom. 

Once created, the response will return an `id`, which you should hold on to. 
However, if you've lost track of it - no worries. You can see all of your scheduled resources by calling `scheduleResource.list`. 

**Note that you should be using an ADMIN Kaltura Session for this creation.**

### Required Parameters 

- **name**: (string) classroom name 
- **tags**: `vcprovider:newrow`
  
### Optional Parameters 

- **systemName**: (string) 
- **description**: (string) 

### Tags / Adding Playlists 

Although the `tags` parameter is *technically* not required with this action, you must include tags of `vcprovider:newrow` in order to create a virtual classroom and distinguish from other types of resources.

Tags can also include additional (optional) parameters for initializing the classroom with a predefined playlist:

| Key  | Type  | Value |
|---|---|---|
| custom_playlist_id  | integer  | Playlist ID to load in this resource |
| custom_playlist_name  | string | Playlist Name to load. Used only if Playlist ID was not passed |
| custom_reset_playlist_instance | boolean | **(default) 1:** creates a new playlist instance if the ID is already loaded in the room. **0:** does not reset playlist instance |


### Example
```php
<?php
    $schedulePlugin = KalturaScheduleClientPlugin::get($client);
    $scheduleResource = new KalturaLocationScheduleResource();
    $scheduleResource->description = "My first classroom";
    $scheduleResource->name = "Classroom-1";
    $scheduleResource->tags = "vcprovider:newrow";

    $result = $schedulePlugin->scheduleResource->add($scheduleResource);
?>
```

#### Response Object 
```json
{
  "id": 1100601,
  "partnerId": 2365491,
  "name": "ClassroomA",
  "description": "This is a sample classroom",
  "status": 2,
  "tags": "vcprovider:newrow",
  "createdAt": 1586100234,
  "updatedAt": 1586100234,
  "objectType": "KalturaLocationScheduleResource"
}
```


## Creating an Event 

You'll use the [`scheduleEvent.add`](https://developer.kaltura.com/console/service/scheduleEvent/action/add) action to create an event of type `KalturaRecordScheduleEvent`. This action must include a startDate and endDate, and a recurrence type of NONE. 

### Required Parameters 

- **startDate**: (int) timestamp
- **endDate**: (int) timestamp  
- **recurrenceType**: NONE [0]
- **summary**: (string) event summary 

### Optional Parameters 

- **organizer**: (string) name of the organizer (string)
- **templateEntryId**: (string) the entry used for session recording 
- **ownerId**: (int) in the case that templateEntryId is empty, this user will now own the recording 
- **referenceId**: (string) third party's corresponding event ID 
- **location**: (string) geographical location of the event 
- **tags**: (string) see below 

The creation of the event will return an `id`. Hold on to that as well. And once again, if you've lost track of it, you can use `scheduleEvent.list` to see all of your created events. 

### Tags / Event Settings  

An event can also include room settings, such as auto-recording, by passing  **tags** as comma-separated key-value-pairs. Tags are optional. 
**Note that any settings on the event will override those of the resource.**

| Key  | Type  | Default  | Other |
|---|---|---|---|
| custom_rec_auto_start  | boolean  | **0:** Auto-start recording is disabled | **1:** Start recording automatically when instructor joins.   |
| custom_rec_set_reminder  | boolean | **0:** Recording reminder is disabled | **1:** Enable recording reminder prompt once instructor has been in the room for two minutes |  
| custom_rs_show_participant  | boolean  | **1:** Show participant list for students and guests | **0:** Hide participant list for students and guests  |
| custom_rs_show_invite  | boolean  | **1:** Show invite option for moderators and instructors  | **0:** Hide invite option for moderators and instructors |
| custom_rs_show_chat  | boolean  | **1:** Enable chat for students and guests  | **0:** Disable chat for students and guests  |
| custom_rs_enable_guests_to_join  | boolean  | **1:** Enable guests to join with invite link  | **0:** Block guests from joining by invite link  |
| custom_rs_class_mode  | string  | **webinar:** Set room mode to be in webinar mode, where users are not live | **virtual_classroom:** Set room to be in virtual classroom mod, where users are automatically set to Live and prompted to activate webcams |
| custom_rs_show_chat_moderators  | boolean  | **1:** Enable moderator chat | **0:** Disable moderator chat |
| custom_rs_show_chat_questions  | boolean  | **1:** Enable Q&A for students and guests | **0:** Disable Q&A for students and guests |

### Example 

```php
<?php
    $schedulePlugin = KalturaScheduleClientPlugin::get($client);
    $scheduleEvent = new KalturaRecordScheduleEvent();
    $scheduleEvent->recurrenceType = KalturaScheduleEventRecurrenceType::NONE;
    $scheduleEvent->startDate = 1586186820;
    $scheduleEvent->endDate = 1586190420;
    $scheduleEvent->summary = "Tomorrow's Event";
    $scheduleEvent->tags = "custom_rec_auto_start:1,custom_rs_show_invite:1";

    $result = $schedulePlugin->scheduleEvent->add($scheduleEvent);
?>
```

#### Response Object 

```json
{
  "blackoutConflicts": [],
  "id": 3371011,
  "partnerId": 0000000,
  "summary": "Tomorrow's Event ",
  "status": 2,
  "startDate": 1586186820,
  "endDate": 1586190420,
  "classificationType": 1,
  "ownerId": "vpaas@kaltura.com",
  "sequence": 11,
  "recurrenceType": 0,
  "duration": 3600,
  "createdAt": 1586100946,
  "updatedAt": 1586100946,
  "objectType": "KalturaRecordScheduleEvent"
}
```

## Creating an Event Resource 

An event *must* be associated with a classroom. As mentioned, the same resource (classroom) can be launched at many different times with different events. This allows you to keep specific resources inside a classroom; for example, for the same class that is taught numerous times with the same lesson plan. 

To associate a session with an event, use the [`scheduleEventResource.add`](https://developer.kaltura.com/console/service/scheduleEventResource/action/add) action. This is where you'll need the resource Id and the event Id that you just created. 

### Example 

```php
<?php
  $schedulePlugin = KalturaScheduleClientPlugin::get($client);
  $scheduleEventResource = new KalturaScheduleEventResource();
  $scheduleEventResource->eventId = 3371011;
  $scheduleEventResource->resourceId = 1100601;

  try {
    $result = $schedulePlugin->scheduleEventResource->add($scheduleEventResource);
    var_dump($result);
  } catch (Exception $e) {
    echo $e->getMessage();
  }
?>
```

#### Response Object 

```json
{
  "eventId": 3371011,
  "resourceId": 1100601,
  "partnerId": 2365491,
  "createdAt": 1586148953,
  "updatedAt": 1586148953,
  "objectType": "KalturaScheduleEventResource"
}
```

## Creating a Kaltura Session 

There are two ways to authenticate a virtual classroom - via LTI and by using a Kaltura Session. For the purpose of this guide, we will create and use a Kaltura Session (KS), which is an authentication string that identifies the user and contains permissions and privileges. 

One way to create a KS is by using the session.start action. 
Here, you'll need a partner ID, and a USER secret. You can find these in the integration settings in your KMC. 

