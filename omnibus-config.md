# Omnibus Configuration

## General Configuration Information

The Omnibus configuration file is a nonhierarchical list of key/value parameters.

Most are `task` directives, that run an asynchronous task for the lifetime of the daemon. Some of these tasks require additional paramters for their configuration.

## Tasks

### task logger

Performs logging and log rotation.

#### logger_keyroot

The location of the logfile base. This will be the currently open log output, and earlier logs will have `.0`, etc. appended to them.

	logger_logroot /var/log/pf9/omnibus.log

*logger_size

Size, in bytes, at which the logs will be rotated.

	logger_size 1000000

#### logger_keep

Maximum number of rotated logs that will be saved at any one time.

	logger_keep 5

### task dispatcher

Allows tasks to register themselves and accept messages by name.

### task rabbit-rx

A task to manage receiving Rabbit messages. Used by other tasks.

### task rabbit-tx

A task to manage transmitting Rabbit messages. Used by other tasks.

### task upstream

A task to send messages upstream to a central server.

### task collector

Collects various information from the Linux system (example: load average).

### task hostmaster

Collects information regarding Platform 9 hosts and their status.

### task livestock

Manages the collection of OpenStack inventory and status from the various OpenStack databases.

### task zebra

Manages the collection of some additional OpenStack inventory and status from the various OpenStack databases.

task tail.nova
task tail.cinder
task tail.neutron
task tail.secure
task tail.keystone
task tail.ksadm
task loopback
task support
task sidekick
task hostbus
task communion
task token
task notifications
task nova
task cinder
task ceilometer
task heat
task neutron
task keyparty
task expiry
task qbert
task convergence
task jobs

# webserver API access
port 4455

rabbit_user {{ rabbit_userid }}
rabbit_password {{ rabbit_password }}

os.tenant.name admin
os.username {{ admin_keystone_username }}
os.password {{ admin_keystone_password }}
os.region.name {{ region }}

notifications_url http://localhost:8888/
sidekick_url http://localhost:3011/v1/hosts

upstream_method websocket rabbit

upstream_rabbit pf9-upstream
upstream_websocket {{ whistle_server }}

expiry_days 21


token_interval 300
notifications_interval 60
ceilometer_interval 180
heat_interval 180
neutron_interval 180
nova_interval 180
cinder_interval 180
sidekick_interval 300
qbert_interval 300
convergence_interval 300
jobs_interval 300


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

