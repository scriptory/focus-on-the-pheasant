# Whistle API

## General API Information

The term `customer` refers to an OpenStack installation, grouping one or more DUs.

The term `du` refers to a Deployment Unit, an FQDN which corresponds to an independently deployed controller in OpenStack,
or any other type of system which is specifically monitored, such as a DNS server. It does not refer to OpenStack hosts or VMs.

The term `host` refers to OpenStack hosts.

The terms `instance` and `vm` are used interchangeably, referring to virtual machines running on hosts.

Whistle is the component which serves as API backend and UI server.

Omnibus is a component deployed on DUs, which report back to Whistle.

Muster is a component deployed on hosts. Muster relays telemetry from hosts, to Omnibus, thence to Whistle.

All communication between components is done via APIs using JSON.

APIs might be serviced over HTTP, Websockets, or AMQP.

All timestamps are given in seconds since 1970.

Whistle, Omnibus, and Muster share similar architecture and are written in Golang. All three operate asynchronous
tasks, to which commands may be sent. Tasks run for the lifetime of the component.

<div style="page-break-after: always;"></div>

## API Endpoint

All Whistle APIs are available through the websocket interface. Certain APIs are also available via HTTP query.
The endpoint for those regarding events (alerts) and the dashboard are kept in the "analytics" task. API communication
with this task can be done through its endpoint:

        curl -X POST --data-binary
            '{ "task": "analytics", "command": "event-summary", "since": 0, "class": "prod" }'
            'https://whistle.example.com/api/analytics'

For the Websocket API, send the above JSON only. The Websocket endpoint is:

        wss://whistle.example.com/ws

When using the Websocket interface, before sending other transactions a client should first send its identity:

	{
	"command": "hello",
	"client": "ui-dashboard",
	"identifier": "anonymous"
	}

If the user is known to the client (for example, email address or a username), use the `identifier` field.

<div style="page-break-after: always;"></div>

## Alertables Object

Describes each type of possible event. Dictionary entry has format:

    {
    "compute_down":
        {
        alert_group: "tier2",
        event: "compute-down",
        id: 3,
        roles: "support",
        timeless: 0,
        tooltip: "Nova compute service is down"
        }
    }

`alert_group`: Which tier this event resides in. See ALERT GROUPS.

`event`: The event type.

`roles`: To which roles this event applies. Can be "support" or "devops". Mainly of interest to a client UI.

`timeless`: Most events will vanish off the default dashboard after 24h. Certain critical events, like subsystem failures, will be displayed regardless of the requested time period. Such events have timeless set to 1.

`tooltip`: A short English description of the event.

## Alert Groups Object

An array containing dictionaries describing the various alert tiers. This data is used to group events
together in a UI.

    [
        {
        id: 1,
        alert_group: "tier1",
        title: "Tier 1 Alerts",
        ordering: 1
        },
        ...
    ]

The default configuration contains 3 tiers.

<div style="page-break-after: always;"></div>

## Event Status

May be any of:

Event Type | Description
--- | ---
open | event is currently an ongoing issue
reopened | event was open, closed recently, and has now recurred
notify | event was a one-time occurrence, such as a single Nova API call failure
tentative-fix | an open event has resolved itself, but might be reopened should it recur
fixed-itself | an open event has resolved itself, but should the same recur, a new event will open
fresh-open | event is a current issue, but not yet worthy of notice
fresh-open-fixed | a fresh-open event has resolved itself
fresh-notify | a notification potentially worthy of attention later
fresh-notify-fixed | a fresh-notify event has resolved itself

Likely state transitions:

	open -> tentative-fix -> reopened
	open -> tentative-fix -> fixed-itself

	fresh-open -> open
	fresh-open -> fresh-open-fixed

	fresh-notify -> notify
	fresh-notify -> fresh-notify-fixed

Generally, escalated alerts concern mainly those with status `open` or `reopened`. Events with `notify`
will have higher order events triggered should they occur in sufficient quantities.

<div style="page-break-after: always;"></div>

## Target Types

A `target` is a first-class object monitored by Whistle. A target will generally have an ID, a name, and a type. The possible types are:

Target Type | Description
--- | ---
api | An API call, in which case the target is generally a request-id
du | Deployment unit
host | Platform9 Host
image | OpenStack Glance image
instance | OpenStack Nova virtual machine
snapshot | OpenStack Cinder snapshot
stack | OpenStack Heat stack
volume | OpenStack Cinder volume

<div style="page-break-after: always;"></div>

## event-summary API

gives an overview of events.

### REQUEST

	{
	task: "analytics",
	command: "event-summary",
	class: "prod",
	since: 1487034468
	}

`task`: "analytics"

`command`: "event-summary"

`class`: Which types of deployments (`prod`, `dev`, `test`)

`since`: Time restriction of events, in seconds since 1970. Omit (or use 0) to list all events


### RESPONSE

	{
	task: "analytics",

	command: "event-summary",

	class: "prod",

	alertables: General event descriptions. See *Alertables Object*.

	alert_groups: The event tiers. See *Alert Groups Object*

	summary: a dictionary of events, with their counts and partial information in a dictionary:

		{
		"stack-error":
			{
			new: 4,
			total: 4,
			affected:
				{
				customer: "yoyodyne",
				count: 2,
				critical: 0,
				timestamp: 1487168797
				}
			},
		...
		}
	}

`new` -- The number of events that have not been acknowledged

`total` -- The total of new and acknowledged events

`affected` -- counts of affected customers for this type of event. In this dictionary,

`customer` -- Customer name

`count` -- Number of events

`critical` -- 0 or 1, indicating whether the event is critical

`timestamp` -- time of the most recent of these events for this customer

As the events returned in a dictionary, the ordering of events in the UI is determined by the alertables and alert_groups described above.

<div style="page-break-after: always;"></div>

## open-events API

gives a list of all open events for a specified event type, or for all events

### REQUEST

	{
	task: "analytics",
	command: "open-events",
	event_filter: "image-error"
	}

`task`: "analytics"

`command`: "open-events"

`event_filter`: "image-error" -- If omitted, returns events of all types.

### RESPONSE

	{
	task: "analytics"

	command: "open-events"

	event_filter: "image-error"

	alertables: General event descriptions. See *Alertables Object*

	events:
		[
			{
			ack_id: 0,
			customer: "yoyodyne",
			deployment_type: "kvm",
			event: "image-error",
			status: "open"
			detail: "Image entered "queued" state",
			offense: "failed"
			...
			}
		]
	}

`events` is an array of open events whose types match the above filter:

`event` -- One of the event types described in *Alertables*

`status` -- State of the event. See *Event Status*

`target` -- The ID of the first-class object to which the event applies.

`target_name` -- A human-readable moniker attached to the target, if any.

`target_type` -- What type of target this is. See *Target Types*

`offense` -- The value that triggered this event. Could be a number (load average), a status (`failed`), etc.

`detail` -- Arbitrary text showing detail regarding the error. The `detail` field will generally indicate the original state at the time the error was triggered, and the `offense` field will contain a more current status value.

`timestamp` -- The time the event was triggered, in seconds from 1970.

`customer` -- Customer. See glossary.

`du` -- Deployment unit. See glossary.

`ack_id` -- Pointer to an acknowledgement record, if this event was acknowledged through the Whistle UI.

`resolution_id` -- Identifier of the most recent state history record. All state changes to events are logged.

<div style="page-break-after: always;"></div>

## event-history API

gives a record of all status changes to a specified event.

### REQUEST

	{
	task: "analytics",
	command: "event-history",
	event: 47012
	}

### RESPONSE

	{
	task: "analytics",

	command: "event-history",

	event: 47012,

	timestamp: 1487299145,

	history:
		[
			{
			"id": 992224,
			"status": "open",
			"timestamp": 1487021452,
			"username": "monitor"
			},
			{
			"id": 994355,
			"status": "tentative-fix",
			"timestamp": 1487104232,
			"username": "monitor"
			},
			...
		]
	}

`history` consists of an array of change records. Fields of a change record:

`id` -- The change record id

`status` --The new status of the event

`timestamp` -- Seconds since 1970

`username` -- User or program which caused the status change. Those performed by the monitoring system itself show as "monitor"

## add-event API

Add an event to the events table. Normally, such events are generated by monitoring scripts, but there are cases where an event is to be managed outside of these scripts.

### REQUEST

	{
	"task": "analytics",
	"command": "add-event",
	"customer": "pf9-boson",
	"du": "pf9-boson.platform9.net",
	"event": "flux-capacitor-fail",
	"status": "open",
	"target": "91aeec63-abbc-4018-98cd-84456a0f71b3",
	"target_type": "capacitor",
	"target_name": "Main Flux Capacitor",
	"offense": "2 farads",
	"detail": "Note: This is a freeform field\n" + "Traceback: " + traceback`
	}

### RESPONSE

	{
	"task": "analytics",
	"command": "add-event",
	"customer":"pf9-boson",
	"du":"pf9-boson.platform9.net",
	"event_id":3018424,
	"event_status": "open",
	"success": true
	}

## udpate-event API

Update certain fields (status, offense, or detail) of an event.

### REQUEST

	{
	"task": "analytics",
	"command": "add-event",
	"customer": "pf9-boson",
	"du": "pf9-boson.platform9.net",
	"status": "open",
	"offense": "2 farads",
	"detail": "Note: This is a freeform field\n" + "Traceback: " + traceback`
	}

### RESPONSE

	{
	"task": "analytics",
	"command": "update-event",
	"event_id": 3018424,
	"event_status": "open",
	"success": true
	}
