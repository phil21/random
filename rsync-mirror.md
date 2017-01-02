Mirror script
=============
Currently mirroring linux distributions and other popular open source projects is rather
dated and hodge-podge.  Particularly, reliable alerting and error handling is nowhere to
be seen.  Since most distributions utilize rsync for their mirror network, it should be
possible to write a generic script that can mirror most open source projects with minor
configuration options.

This script should ship with a simple configuration file that any experienced admin can 
trivially add mirror support for any common open source project.  These defaults should
ship with project-approved default settings when possible, but otherwise be as plug and
play as possible.

The primary asset mirror admins should be providing is disk space and bandwidth - not
man-hours battling unreliable upstream mirrors and poor alerting.  The goal of
this project is to install this script, confirm it's working/logging/alerting/graphing
appropriately and then forget it exists.  It will ask for help if it ever needs it.

Architecture
===========
Perl is the language used to implement this project.  Web components (if applicable) will
be written using the Mojolicious Perl framework.


General Requirements:
=====================
The list of general requirements is below.  Further scope work is required to properly
scope out all options.  In general the ideas of portability and simplicity should be kept
in mind.  The least external dependencies the better.

In no particular order:
* Rsync-based
* Two-stage rsync supported (download everything but a regex match of files first, then download indexes afterwards)
* Stats collected each run
  * Time the sync took
  * Bytes transferred
  * Bitrate
  * # of total files
  * # of files transferred
  * In-progress indictor and % done (best estimate)
  * Errors
* Stats should support collectd/graphite style streaming
* Smart error handling
  * Alert based on severity
  * re-tries supported
  * fallback to alternate upstream mirror supported
  * Intelligent exit code handling - not all errors are equal
    * Heuristic-based alerting for certain error codes:  Some error codes such as
      permission denied and other transient errors are expected under normal operation.
      However, sometimes these error codes mean quite a bit.  If 10 out of 50,000 files 
      cannot be transfered, generally this is not something to alert on.  If 26,000 of 
      50,000 cannot be transferred a human should be immediately let know - after a retry
      on an alternate upstream first.
  * Re-try a set number of times prior to alerting a human, using alternate upstream 
    targets when available.
  * Primary goal is to reduce false positives while retaining 0 false negative hitrate. 
    This can be achieved by starting small and tweaking as new false positives come in. A
    alerting framework should be built to make this easy.
* Alerts fire external plugin
* Alerts highly dependent on severity - many INFO level alerts will be generated for later
  review, while crital alerts will be thrown for when the autoation cannot fix itself.
* Up to the alert consumer to decide what to do in each case - probably log to logstash
  and syslog everything, and e-mail/nagios/etc. any critical alerts passed on to it.
* Default shipped alerting script should focus on graphite, nagios, and e-mail
  integration.
* Once an alert is critical (sev0) it means there is absolutely no reasonable way to fix
  it in an automated fashion.  These should be heavily investigated.
* A monitoring target for Nagios should be included for all enabled mirrors.  This should
  possibly be implemented either via individual checks per-enabled-mirror or a generic
  covering alert that provides additional information
* As with stats, logging should be properly implemented and log too much data rather than
  too little.

Nice to haves
=============
* Round robin based selection of mirror in a list
* Better integration with distribution mirror tooling
* engineered in an extensible enough fashion to be suitable for open sores
* Look at integrating incremental rsync support for fedora mirroring
* Parrellelize rsync utiziling file lists - potentially detect first-seed and multiplex
  multiple mirrors
* Smart update detection for distributions that support it - rapidly poll lightweight
  files/systems to see if an update is available.  If so, run a rsync immediately ahead
  of regular schedule.
* ZFS integration - allow for snapshot management.
