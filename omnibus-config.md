# Omnibus Configuration

## General Configuration Information

The Omnibus configuration file is a nonhierarchical list of key/value parameters.

Most are `task` directives, that run an asynchronous task for the lifetime of the daemon. Some of these tasks require additional paramters for their configuration.

## General Directives

### port

The port number which Omnibus will offer for local API requests.

	port 4455

### is_master_du

Whether this controller is a master (1), or a slave (0).

	is_master_du 1

## OpenStack Configuration

### os.tenant.name

OpenStack tenant name, for use by probes.

	os.tenant.name admin

Use only one of *os.tenant.name* or *os.tenant.id*.

### os.tenant.id

OpenStack tenant id, for use by probes.

	os.tenant.id bef02e62-1f4f-4ecf-9e91-8fedffac44ed

Use only one of *os.tenant.name* or *os.tenant.id*.

### os.username

OpenStack username, for use by probes.

	os.username service

### os.password

OpenStack password, for use by probes.

	os.password swordfish

### os.region.name

OpenStack Region Name, for use by probes.

	os.region.name primary

### os.domain.name

OpenStack Domain Name, for use by probes.

	os.domain.name default

Use only one of *os.domain.name* or *os.domain.id*.

### os.domain.id

Openstack Domain id, for use by probes.

	os.domain.id 6b74966e-129c-4bf6-b603-cb44d4ba541d

Use only one of *os.domain.name* or *os.domain.id*.

## Tasks

### task logger

Performs logging and log rotation.

- **logger_keyroot**

	The location of the logfile base. This will be the currently open log output, and earlier logs will have `.0`, etc. appended to them.

		logger_logroot /var/log/pf9/omnibus.log

- **logger_size**

	Size, in bytes, at which the logs will be rotated.

		logger_size 1000000

- **logger_keep**

	Maximum number of rotated logs that will be saved at any one time.

		logger_keep 5

### task dispatcher

Allows tasks to register themselves and accept messages by name.

### task rabbit-rx

A task to manage receiving Rabbit messages. Used by other tasks.

- **rabbit_host**

	Host or IP number of the Rabbit server.

		rabbit_host 127.0.0.1

- **rabbit_port**

	Port of the Rabbit server.

		rabbit_port 5672

- **rabbit_vhost**

	Virtual host, if any, used by the OpenStack installation.

		rabbit_vhost openstack1

- **rabbit_user**

	Username with which to connect to Rabbit. Must be able to create queues and exchanges.

		rabbit_user guest

- **rabbit_password**

	Password for the above username.

		rabbit_password swordfish


### task rabbit-tx

A task to manage transmitting Rabbit messages. Used by other tasks. See parameters under *task rabbit-rx*.

### task upstream

A task to send messages upstream to a central server.

- **upstream_method**

	Methods by which data will be sent upstream.

		upstream_method websocket rabbit

- **upstream_websocket**

	The endpoint of a central Whistle server, to which data will be streamed.

		upstream_websocket https://whistle.example.com:4077/ws

	Used if `websocket` is a configured *upstream_method*.

- **upstream_rabbit**

	A Rabbit exchange, to which data will be posted.

		upstream_rabbit pf9-upstream

	Used if `rabbit` is a configured *upstream_method*.

### task collector

Collects various information from the Linux system (example: load average).

- **collector_fd_skips**

	Normally, a survey of file descriptor usage is taken every minute. This uses *lsof*. If this DU is constrained for resources, set this parameter to *n* to skip every *n* minutes.

		collector_fd_skips 10

### task hostmaster

Collects information regarding Platform 9 hosts and their status.

### task livestock

Manages the collection of OpenStack inventory and status from the various OpenStack databases.

### task zebra

Manages the collection of some additional OpenStack inventory and status from the various OpenStack databases.

### task tail.nova

Collect API data from the Nova API log.

### task tail.cinder

Collect API data from the Cinder API log.

### task tail.neutron

Collect API data from the Neutron API log.

### task tail.heat

Collect API data from the Heat API log.

### task tail.secure

Collect data from `/var/log/secure`

### task tail.audit

Collect data from `/var/log/audit/audit.log`

### task tail.keystone

Collect API data from the Keystone API log.

### task tail.ksadm

Collect API data from the Keystone Admin API log.

### task loopback

Probe to perform Rabbit data ingestion and extraction tests.

### task support

Platform9-specific Remote support task.

### task sidekick

Probe to monitor the Platform9-specific Sidekick role.


- **sidekick_url**

	Endpoint of the Sidekick service.

		sidekick_url http://localhost:3011/v1/hosts

- **sidekick_interval**

	Interval, in seconds, at which to execute the probe.

		sidekick_interval 300

### task hostbus

Task to allow hosts using `muster` to communicate with Omnibus over Rabbit.

### task communion

Task to allow hosts using `muster` to send data to the a central server.

### task token

Manage Keystone tokens for use by Omnibus, as well as test that Keystone is responding to API queries. Required by many other tasks.

- **keystone_base_url**

	Endpoint of the Keystone service.

		keystone_base_url https://du.example.com/keystone

- **token_interval**

	Interval, in seconds, at which to request a fresh token and verify Keystone functionality.

		token_interval 300

### task notifications

Probe to test the Platform9-specific Notifications service.

- **notifications_url**

	Endpoint of the Notifications service.

		notifications_url http://localhost:8888/

- **notifications_interval**

	Interval, in seconds, at which to execute the probe.

		notifications_interval 60

### task nova

Probe to test that Nova is responding to API queries.

- **nova_base_url**

	Endpoint of the Nova service.

		nova_base_url https://du.example.com/nova

- **nova_interval**

	Interval, in seconds, at which to execute the probe.

		nova_interval 180

### task cinder

Probe to test that Cinder is responding to API queries.

- **cinder_base_url**

	Endpoint of the Cinder service.

		cinder_base_url https://du.example.com/cinder

- **cinder_interval**

	Interval, in seconds, at which to execute the probe.

		cinder_interval 180

### task ceilometer

Probe to test that Ceilometer is responding to API queries.

- **ceilometer_base_url**

	Endpoint of the Ceilometer service.

		ceilometer_base_url https://du.example.com/ceilometer

- **ceilometer_interval**

	Interval, in seconds, at which to execute the probe.

		ceilometer_interval 180

### task heat

Probe to test that Heat is responding to API queries.

- **heat_base_url**

	Endpoint of the Heat service.

		heat_base_url https://du.example.com/heat

- **heat_interval**

	Interval, in seconds, at which to execute the probe.

		heat_interval 180

### task murano

Probe to test that Murano is responding to API queries.

- **murano_base_url**

	Endpoint of the Murano service.

		heat_base_url https://du.example.com/murano

- **murano_interval**

	Interval, in seconds, at which to execute the probe.

		murano_interval 300

### task neutron

Probe to test that Neutron is responding to API queries.

- **neutron_interval**

	Interval, in seconds, at which to execute the probe.

		neutron_interval 180

### task keyparty

Probe to collect Keystone inventory data.

### task expiry

Platform9-specific probe to warn if there are any pending certificate expirations.

- **expiry_days**

	Number of days in adavnce to warn of certificate expirations.

		expiry_days 21

### task qbert

Platform9-specific probe to report health of Platform9 Managed Kubernetes.

- **qbert_interval**

	Interval, in seconds, at which to execute the probe.

		qbert_interval 300

### task convergence

Platform9-specific probe to report problems with host convergences.

- **convergence_interval**

	Interval, in seconds, at which to execute the probe.

		convergence_interval 300

### task jobs

Task to manage Omnibus extensions through its jobs framework.

- **jobs_interval**

	Interval, in seconds, at which to check for pending jobs.

		jobs_interval 300
