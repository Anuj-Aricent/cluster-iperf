# cluster-iperf

cluster-iperf is a docker image that runs the iperf IP network performance
tool. The same image can be used to run iperf in client or server mode, and
environment variables can be set, and command line arguments passed through,
to further customize iperf's behavior as described below.

## Documentation

### Table of Contents
* [Why](#why)
* [Pre-release](#pre-release)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
* [Configuration](#configuration)

### Why?

The idea for the cluster-iperf image arose from a desire to have an easy way of testing
bandwidth and network performance within a kubernetes or ECS cluster. A single image that
can be run as client or server, and configured through settings that are accessible to
kubernetes server/replication controller defintions, and ECS tasks, would allow network
testing of various cluster configurations. For example, on kubernetes, testing network
throughput in pod-to-pod, service-to-service, and client-to-service scenarios.

In order to be useful in a cluster environment the image had to have some convenient way
of getting report results out, and so that can be done three ways: 1) run in default mode
and view reports in stdout, which is useful for local testing; 2) run with RSYSLOG=true
which will write reports to LOG_PATH (/var/log/iperf by default); or 3) also run with
RSYSLOG_REMOTE=true and set RSYSLOG_REMOTE_IP and RSYSLOG_REMOTE_PORT to log reports
to a remote syslog server using UDP.

### Pre-release

The cluster-iperf image is pre-release and has not received extensive testing. Obviously
there is very little risk to running iperf other than bandwidth utilization.

### Prerequisites

To build and run the image you need docker installed.

### Installation

Clone the repo and use the supplied makefile to build the image. Use the supplied
kubernetes and ECS object definitions to install the image into a cluster. You
can also pull the image at markbnj/cluster-iperf.


### Configuration

The cluster-iperf image responds to a few environment variables on startup. These
control whether it runs in client or server mode, and how it logs report results.
The variables and their use are described below.

Also iperf itself responds to a large number of environment variables. See the
[iperf2 documentation](https://iperf.fr/iperf-doc.php) for more information.

To set these or the custom environment variables in the image either use the -e
argument to [docker run](https://docs.docker.com/engine/reference/run/), or the
appropriate property of a kubernetes or ECS task/pod.

Lastly, command line arguments can be passed to the entrypoint using the
--command argument to [docker run](https://docs.docker.com/engine/reference/run/),
and by using the appropriate property of a kubernetes or ECS pod/task to pass them.
Any arguments passed in the CMD will be appended to the start up script command
line and passed through to iperf at startup.

----

###### LOG_PATH=path

Example:

`docker run -d -e "LOG_PATH=/var/log/mylogs" cluster-iperf`

Determines where output from the iperf process will be logged when RSYSLOG is
set to `true` and RSYSLOG_REMOTE is set to `false`. Defaults to /var/log/iperf.

----

###### ROLE=[client || server]

Example:

`docker run -d -e "ROLE=server" cluster-iperf`

Determines whether iperf starts in client or server mode.

----

###### RSYSLOG=[true || *false]

Example:

`docker run -d -e "RSYSLOG=true" cluster-iperf`

Determines whether iperf will use the rsyslog facility local0 to output report
results. If this variable is set to `true` then rsyslogd will be configured and
started in the image before iperf is started. Defaults to `false`.

----

###### RSYSLOG_REMOTE=[true || *false]

Example:

`docker run -d -e "RSYSLOG=true" -e "RSYSLOG_REMOTE=true" -e 'RSYSLOG_REMOTE_IP=172.17.0.2'
 -e "RSYSLOG_REMOTE_PORT=5002" cluster-iperf`

Determines whether the rsyslog facility local0 will be sent to a remote server when the
RSYSLOG variable is set to `true`. Also requires RSYSLOG_REMOTE_IP and RSYSLOG_REMOTE_PORT
to be set. Defaults to `false`.

----

###### RSYSLOG_REMOTE_IP=[ip || host name]

Example:

`docker run -d -e "RSYSLOG=true" -e "RSYSLOG_REMOTE=true" -e 'RSYSLOG_REMOTE_IP=172.17.0.2'
 -e "RSYSLOG_REMOTE_PORT=5002" cluster-iperf`

Determines the IP address of the remote syslog daemon to which report lines should be sent.
Required when RSYSLOG_REMOTE is set to `true`.

----

###### RSYSLOG_REMOTE_PORT=port

Example:

`docker run -d -e "RSYSLOG=true" -e "RSYSLOG_REMOTE=true" -e 'RSYSLOG_REMOTE_IP=172.17.0.2'
 -e "RSYSLOG_REMOTE_PORT=5002" cluster-iperf`

Determines the port on the remote syslog daemon to which report lines should be sent.
Required when RSYSLOG_REMOTE is set to `true`.

