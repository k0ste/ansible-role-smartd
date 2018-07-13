# ansible-smartd

`smartd` is a daemon that monitors the Self-Monitoring, Analysis and Reporting
Technology (SMART) system built into most ATA/SATA and SCSI/SAS hard drives and
solid-state drives. The purpose of SMART is to monitor the reliability of the
hard drive and predict drive failures, and to carry out different types of
drive self-tests.

## Requirements

* Ansible 2.5+;

## Example configuration

```yaml
---
# smartd attribute information (smartd CSV smart logs).
smartd_attributelog_path: '/var/log/smartd'
# Create or not smartd_attributelog_path.
smartd_create_attrbutelog_path: 'false'
# Install smartmontools package or not.
smartd_install_package: 'false'
# Restart after successful deploy of configuration.
smartd_restart: 'false'

smartd:
- smartd_devices:
# Example - enable experimenatl nvme support and set
  - name: '/dev/nvme0n1'
# 'true' - device scan enabled, options are parsed.
# 'false' - device scan disabled.
# 'all' - equivalent to turning on all of the following Directives:
#
# 'H' to check the SMART health status.
# 'f' to report failures of Usage (rather than Prefail) Attributes.
# 't' to track changes in both Prefailure and Usage Attributes.
# 'l error' to report increases in the number of ATA errors.
# 'l selftest' to report increases in the number of Self-Test Log errors.
# 'l selfteststs' to report changes of Self-Test execution status.
# 'C 197' to report nonzero values of the current pending sector count.
# 'U 198' to report nonzero values of the offline pending sector count.
    mode_all: 'true'
    d: 'nvme'
# Special name 'all': default for all devices (consult with 'smartd.conf' man).
  - name: 'all'
# smartd will try to open all available devices (see 'smartd' man page).
    mode_all: 'false'
# Specifies the type of the device. Please consult with man page smartd.conf to
# all possible values.
    d: 'auto'
# [ATA only] This 'nocheck' Directive is used to prevent a disk from being
# spun-up when it is periodically polled by smartd.
# ATA disks have five different power states. In order of increasing power
# consumption they are: 'OFF', 'SLEEP', 'STANDBY', 'IDLE', and 'ACTIVE'.
# Typically in the OFF, SLEEP, and STANDBY modes the disk's platters are not
# spinning. But usually, in response to SMART commands issued by smartd, the
# disk platters are spun up. So if this option is not used, then a disk which
# is in a low-power mode may be spun up and put into a higher-power mode when
# it is periodically polled by smartd. Note that if the disk is in SLEEP mode
# when smartd is started, then it won't respond to smartd commands, and so the
# disk won't be registered as a device for smartd to monitor. If a disk is in
# any other low-power mode, then the commands issued by smartd to register the
# disk will probably cause it to spin-up.
# The 'nocheck' directive specifies if smartd's periodic checks should still be
# carried out when the device is in a low-power mode. It may be used to prevent
# a disk from being spun-up by periodic smartd polling. The allowed values are:
#
# never - smartd will poll (check) the device regardless of its power mode. This
# may cause a disk which is spun-down to be spun-up when smartd checks it. This
# is the default behavior.
# sleep - check the device unless it is in SLEEP mode.
# standby - check the device unless it is in SLEEP or STANDBY mode. In these
# modes most disks are not spinning, so if you want to prevent a laptop disk
# from spinning up each time that smartd polls, this is probably what you want.
# idle - check the device unless it is in SLEEP, STANDBY or IDLE mode. In the
# IDLE state, most disks are still spinning, so this is probably not what you
# want.
# Maximum number of skipped checks (in a row) can be specified by appending
# positive number ',N' to POWERMODE (like 'standby,15'). After N checks are
# skipped in a row, powermode is ignored and the check is performed anyway.
# When a periodic test is skipped, smartd normally writes an informal log
# message. The message can be suppressed by appending the option ',q' to
# POWERMODE (like 'standby,q'). This prevents a laptop disk from spinning up due
# to this message. Both ',N' and ',q' can be specified together.
    n: 'never'
# Specifies how tolerant smartd should be of SMART command failures. The valid
# arguments to this Directive are:
# normal - do not try to monitor the disk if a mandatory SMART command fails,
# but continue if an optional SMART command fails. This is the default.
# permissive - try to monitor the disk even if it appears to lack SMART
# capabilities. This may be required for some old disks (prior to ATA-3
# revision 4) that implemented SMART before the SMART standards were
# incorporated into the ATA/ATAPI Specifications.
    T: 'normal'
# [ATA only] Enables or disables SMART Automatic Offline Testing when smartd
# starts up and has no further effect. The delay between tests is
# vendor-specific, but is typically four hours. Note that SMART Automatic
# Offline Testing is not part of the ATA Specification. The valid arguments to
# this Directive are on and off.
    o: 'on'
# Enables or disables Attribute Autosave when smartd starts up and has no
# further effect. Also affects SCSI devices. The valid arguments to this
# directive are on and off.
    S: 'on'
# [ATA] Check the health status of the disk with the SMART RETURN STATUS
# command. If this command reports a failing health status, then disk failure
# is predicted in less than 24 hours, and a message at loglevel 'LOG_CRIT' will
# be logged to syslog.
# [NVMe] Checks the "Critical Warning" byte from the SMART/Health Information
# log. If any warning bit is set, a message at loglevel 'LOG_CRIT' will be
# logged to syslog.
    H: 'true'
# Reports increases in the number of errors in one of three SMART logs. The
# valid arguments to this Directive are:
# error - [ATA] report if the number of ATA errors reported in the Summary
# SMART error log has increased since the last check.
# error - [NVMe] report if the "Number of Error Information Log Entries" from
# the SMART/Health Information log has increased since the last check.
# xerror - [ATA] report if the number of ATA errors reported in the Extended
# Comprehensive SMART error log has increased since the last check.
# If both 'error' and 'xerror' are specified, smartd checks the maximum of both
# values.
# xerror - [NVMe] same as 'error'.
# selftest - report if the number of failed tests reported in the SMART
# Self-Test Log has increased since the last check, or if the timestamp
# associated with the most recent failed test has increased. Note that such
# errors will only be logged if you run self-tests on the disk (and it fails a
# test!). Self-Tests can be run automatically by smartd: please see the 's'.
# [ATA only] Failed self-tests outdated by a newer successful extended
# self-test are ignored. The warning email counter is reset if the number of
# failed self tests dropped to 0. This typically happens when an extended
# self-test is run after all bad sectors have been reallocated.
# offlinests[,ns] - [ATA only] report if the Offline Data Collection status
# has changed since the last check. The report will be logged as LOG_CRIT if
# the new status indicates an error. With some drives the status often changes,
# therefore "l: offlinests" is not enabled by 'a' Directive. Appending ',ns'
# (no standby) to this directive is not implemented on Linux.
# selfteststs[,ns] - [ATA only] report if the Self-Test execution status has
# changed since the last check. The report will be logged as LOG_CRIT if the
# new status indicates an error. Appending ',ns' (no standby) to this directive
# is not implemented on Linux.
# scterc,READTIME,WRITETIME - [ATA only] sets the SCT Error Recovery Control
# settings to the specified values (deciseconds) when smartd starts up and has
# no further effect. Values of 0 disable the feature, other values less than 65
# are probably not supported.
# For RAID configurations, this is typically set to 70,70 deciseconds.
    l:
    - 'error'
    - 'selftest'
    - 'selfteststs'
# Sets non-SMART device settings when smartd starts up and has no further
# effect. Valid arguments are:
#
# aam,[N|off] - [ATA only] Sets the Automatic Acoustic Management (AAM) feature.
# apm,[N|off] - [ATA only] Sets the Advanced Power Management (APM) feature.
# lookahead,[on|off] - [ATA only] Sets the read look-ahead feature.
# security-freeze - [ATA only] Sets ATA Security feature to frozen mode.
# standby,[N|off] - [ATA only] Sets the standby (spindown) timer and places the
# drive in the IDLE mode.
# wcache,[on|off] - [ATA only] Sets the volatile write cache feature.
# dsn,[on|off] - [ATA only] [NEW EXPERIMENTAL SMARTD FEATURE] Sets the DSN
# feature.
    e: ''
# Run Self-Tests or Offline Immediate Tests, at scheduled times. A Self or
# Offline Immediate Test will be run at the end of periodic device polling, if
# all 12 characters of the string T/MM/DD/d/HH match the extended regular
# expression. Here:
#
# T - is the type of the test. The values that smartd will try to match (in
# turn) are: 'L' for a Long Self-Test, 'S' for a Short Self-Test, 'C' for a
# Conveyance Self-Test (ATA only), and 'O' for an Offline Immediate Test
# (ATA only). As soon as a match is found, the test will be started and no
# additional matches will be sought for that device and that polling cycle.
# To run scheduled Selective Self-Tests, use 'n' for next span, 'r' to redo last
# span, or 'c' to continue with next span or redo last span based on status of
# last test. The LBA range is based on the first span from the last test.
# Some disks (e.g. WD) do not preserve the selective self test log across power
# cycles. If state persistence ('s' option) is enabled, the last test span is
# preserved by smartd and used if (and only if) the selective self test log is
# empty.
# MM - is the month of the year, expressed with two decimal digits. The range
# is from 01 (January) to 12 (December) inclusive. Do not use a single decimal
# digit or the match will always fail!
# DD - is the day of the month, expressed with two decimal digits. The range is
# from 01 to 31 inclusive. Do not use a single decimal digit or the match will
# always fail!
# d - is the day of the week, expressed with one decimal digit. The range is
# from 1 (Monday) to 7 (Sunday) inclusive.
# HH is the hour of the day, written with two decimal digits, and given in
# hours after midnight. The range is 00 (midnight to just before 1 am) to 23
# (11pm to just before midnight) inclusive. Do not use a single decimal digit or
# the match will always fail!
    s: ''
# Send a warning email to the email address ADD if the detect a failure or a
# new error, or if a SMART command to the disk fails.
# To prevent your email inbox from getting filled up with warning messages, by
# default only a single warning and (depending on 's' option) daily reminder
# emails will be sent for each of the enabled alert types.
    m:
    - 'user1@add1'
    - 'user2@add2'
# These Directives modify the behavior of the smartd email warnings. Directives
# only work in conjunction with the 'm' and can not be used without it.
# The valid arguments to the are (one of the following three):
#
# once - send only one warning email for each type of disk problem detected.
# This is the default unless state persistence ('s' option) is enabled.
# daily - send additional warning reminder emails, once per day, for each type
# of disk problem detected. This is the default if state persistence
# ('s' option) is enabled.
# diminishing - send additional warning reminder emails, after a one-day
# interval, then a two-day interval, then a four-day interval, and so on for
# each type of disk problem detected. Each interval is twice as long as the
# previous interval. If a disk problem is no longer detected, the internal email
# counter is reset. If the problem reappears a new warning email is sent
# immediately.
# In addition, one may add zero or more of the following Directives:
# test - send a single test email immediately upon smartd startup. This allows
# one to verify that email is delivered correctly. Note that if this Directive
# is used, smartd will also send the normal email warnings that were enabled
# with the 'm' Directive, in addition to the single test email!
# exec /path/to/script - run the executable path instead of the default mail
# command, when smartd needs to send email. Path must point to an executable
# binary file or script.
# By setting /path to point to a customized script, you can make smartd perform
# useful tricks when a disk problem is detected (beeping the console, shutting
# down the machine, broadcasting warnings to all logged-in users, etc). But
# please be careful. smartd will block until the executable path returns, so if
# your executable hangs, then smartd will also hang.
# The exit status of the executable is recorded by smartd in syslog. The
# executable is not expected to write to stdout or stderr. If it does, then
# this is interpreted as indicating that something is going wrong with your
# executable, and a fragment of this output is logged to syslog to help you to
# understand the problem. Normally, if you wish to leave some record behind,
# the executable should send mail or write to a file or device.
# Before running the executable, smartd sets a number of environment variables.
# These environment variables may be used to control the executable's behavior.
    M:
    - 'once'
    - 'daily'
# [ATA only] Check for 'failure' of any Usage Attributes. If these Attributes
# are less than or equal to the threshold, it does NOT indicate imminent disk
# failure. It "indicates an advisory condition where the usage or age of the
# device has exceeded its intended design life period."
    f: 'true'
# [ATA only] Report anytime that a Prefail Attribute has changed its value
# since the last check.
    p: 'true'
# [ATA only] Report anytime that a Usage Attribute has changed its value since
# the last check.
    u: 'true'
# [ATA only] Equivalent to turning on the two previous flags 'p' and 'u'.
# Tracks changes in all device Attributes (both Prefailure and Usage).
    t: 'true'
# [ATA only] Ignore device Attribute number ID when checking for failure of
# Usage Attributes. ID must be a decimal integer in the range from 1 to 255.
# This Directive modifies the behavior of the 'f' Directive and has no effect
# without it. This is useful, for example, if you have a very old disk and
# don't want to keep getting messages about the hours-on-lifetime Attribute
# (usually Attribute 9) failing. This Directive may appear multiple times for a
# single device, if you want to ignore multiple Attributes.
    i:
    - '9'
    - '241'
# [ATA only] Ignore device Attribute ID when tracking changes in the Attribute
# values. ID must be a decimal integer in the range from 1 to 255. This
# Directive modifies the behavior of the 'p', 'u', and 't' tracking Directives
# and has no effect without one of them. This is useful, for example, if one of
# the device Attributes is the disk temperature (usually Attribute 194 or 231).
# It's annoying to get reports each time the temperature changes. This
# Directive may appear multiple times for a single device, if you want to
# ignore multiple Attributes.
    I:
    - '194'
    - '231'
# [ATA only] When tracking, report the Raw value of Attribute ID along with its
# (normally reported) Normalized value. ID must be a decimal integer in the
# range from 1 to 255. This Directive modifies the behavior of the 'p', 'u',
# and 't' tracking Directives and has no effect without one of them. This
# Directive may be given multiple times. A common use of this Directive is to
# track the device Temperature (often ID=194 or 231). If the optional flag '!'
# is appended, a change of the Normalized value is considered critical. The
# report will be logged as LOG_CRIT and a warning email will be sent if 'm'
# is specified.
    r:
    - '194'
    - '231!'
# [ATA only] When tracking, report whenever the Raw value of Attribute ID
# changes. Normally smartd only tracks/reports changes of the Normalized
# Attribute values. ID must be a decimal integer in the range from 1 to 255.
# This Directive modifies the behavior of the 'p', 'u', and 't' tracking
# Directives and has no effect without one of them. This Directive may be given
# multiple times. If this Directive is given, it automatically implies the 'r'
# Directive for the same Attribute, so that the Raw value of the Attribute is
# reported. A common use of this Directive is to track the device Temperature
# (often ID=194 or 231). It is also useful for understanding how different
# types of system behavior affects the values of certain Attributes. If the
# optional flag '!' is appended, a change of the Raw value is considered
# critical. The report will be logged as LOG_CRIT and a warning email will be
# sent if 'm' is specified. An example is '5!' to warn when new sectors are
# reallocated.
    R:
    - '5!'
# [ATA only] Report if the current number of pending sectors is non-zero. Here
# ID is the id number of the Attribute whose raw value is the
# "Current Pending Sector" count. The allowed range of ID is 0 to 255 inclusive.
# To turn off this reporting, set it to '0'. If the 'C' ID option is not given,
# then it defaults to 'C=197' (since Attribute 197 is generally used to monitor
# pending sectors). If '+' is specified, a report is only printed if the number
# of sectors has increased between two check cycles. Some disks do not reset
# this attribute when a bad sector is reallocated. The warning email counter is
# reset if the number of pending sectors dropped to 0. This typically happens
# when all pending sectors have been reallocated or could be read again. A
# pending sector is a disk sector (containing 512 bytes of your data) which the
# device would like to mark as "bad" and reallocate. Typically this is because
# your computer tried to read that sector, and the read failed because the data
# on it has been corrupted and has inconsistent Error Checking and Correction
# (ECC) codes. This is important to know, because it means that there is some
# unreadable data on the disk. The problem of figuring out what file this data
# belongs to is operating system and file system specific. You can typically
# force the sector to reallocate by writing to it (translation: make the device
# substitute a spare good sector for the bad one) but at the price of losing
# the 512 bytes of data stored there.
    C: '197+'
# [ATA only] Report if the number of offline uncorrectable sectors is non-zero.
# Here ID is the id number of the Attribute whose raw value is the Offline
# Uncorrectable Sector count. The allowed range of ID is 0 to 255 inclusive. To
# turn off this reporting, set value to '0'. If the option is not given, then it
# defaults to '198' (since Attribute 198 is generally used to monitor offline
# uncorrectable sectors). If '+' is specified, a report is only printed if the
# number of sectors has increased since the last check cycle. Some disks do not
# reset this attribute when a bad sector is reallocated. The warning email
# counter is reset if the number of offline uncorrectable sectors dropped to 0.
# This typically happens when all offline uncorrectable sectors have been
# reallocated or could be read again. An offline uncorrectable sector is a disk
# sector which was not readable during an off-line scan or a self-test. This is
# important to know, because if you have data stored in this disk sector, and
# you need to read it, the read will fail.
    U: '198+'
# Report if the current temperature had changed by at least DIFF degrees since
# last report, or if new min or max temperature is detected. Report or Warn if
# the temperature is greater or equal than one of INFO or CRIT degrees Celsius.
# If the limit CRIT is reached, a message with loglevel 'LOG_CRIT' will be
# logged to syslog and a warning email will be send if 'm' is specified. If
# only the limit INFO is reached, a message with loglevel 'LOG_INFO' will be
# logged. The warning email counter is reset if the temperature dropped below
# INFO or CRIT-5 if INFO is not specified. If this directive is used in
# conjunction with state persistence ('s' option), the min and max temperature
# values are preserved across boot cycles. The minimum temperature value is not
# updated during the first 30 minutes after startup. To disable any of the 3
# reports, set the corresponding limit to 0. Trailing zero arguments may be
# omitted. By default, all temperature reports are disabled.
# To track temperature changes of at least 2 degrees, use:
# W: '2'
# To log informal messages on temperatures of at least 40 degrees, use:
# W: '0,40'
# For warning messages/mails on temperatures of at least 45 degrees, use:
# W: '0,0,45'
# To combine all of the above reports, use:
# W: '2,40,45'
# For ATA devices, smartd interprets Attribute 194 or 190 as Temperature
# Celsius by default. For NVMe devices, smartd checks the maximum of the
# Composite Temperature value and all Temperature Sensor values reported by
# SMART/Health Information log.
    W: '0'
# [ATA only] Modifies the behavior of smartd to compensate for some known and
# understood device firmware bug. This directive may be used multiple times.
# The valid arguments are:
#
# none - Assume that the device firmware obeys the ATA specifications. This is
# the default.
# nologdir - Suppresses read attempts of SMART or GP Log Directory. Support for
# all standard logs is assumed without an actual check. Some Intel SSDs may
# freeze if log address 0 is read.
# samsung - In some Samsung disks (example: model SV4012H
# Firmware Version: RM100-08) some of the two- and four-byte quantities in the
# SMART data structures are byte-swapped (relative to the ATA specification).
# Enabling this option tells smartd to evaluate these quantities in
# byte-reversed order. Some signs that your disk needs this option are no
# self-test log printed, even though you have run self-tests;
# very large numbers of ATA errors reported in the ATA error log;
# strange and impossible values for the ATA error log timestamps.
# samsung2 - In some Samsung disks the number of ATA errors reported is byte
# swapped. Enabling this option tells smartd to evaluate this quantity in
# byte-reversed order.
# samsung3 - Some Samsung disks (at least SP2514N with Firmware VF100-37)
# report a self-test still in progress with 0% remaining when the test was
# already completed. If this directive is specified, smartd will not skip the
# next scheduled self-test (see Directive 's' above) in this case.
# xerrorlba - This only affects smartctl.
    F: 'none'
# [ATA only] Sets a vendor-specific raw value print FORMAT, an optional
# BYTEORDER and an optional NAME for Attribute ID. This directive may be used
# multiple times. The following arguments affect smartd warning output:
#
# '197,increasing' - Raw Attribute number 197 (Current Pending Sector Count) is
# not reset if uncorrectable sectors are reallocated. This sets '-C 197+' if no
# other 'C' directive is specified.
# '198,increasing' - Raw Attribute number 198 (Offline Uncorrectable Sector
# Count) is not reset if uncorrectable sectors are reallocated. This sets
# "U: '198+'" if no other 'U' directive is specified.
    v: ''
# [ATA only] Specifies whether smartd should use any preset options that are
# available for this drive. The valid arguments to this Directive are:
#
# use - use any presets that are available for this drive. This is the default.
# ignore - do not use any presets for this drive.
# show - show the presets listed for this drive in the database.
# showall - show the presets that are available for all drives and then exit.
    P: 'use'
# With this options smartd will be started.
  smartd_options:
# Writes smartd attribute information (normalized and raw attribute values) to
# files 'path/MODEL-SERIAL.ata.csv' or 'path/VEN‐DOR-MODEL-SERIAL.scsi.csv'.
# At each check cycle attributes are logged as a line of semicolon separated
# triplets of the form "attribute-ID;attribute-norm-value;attribute-raw-value;".
# For SCSI devices error counters and temperature recorded in the form
# "counter-name;counter-value;". Each line is led by a date string of the form
# "yyyy-mm-dd HH:MM:SS" (in UTC). MODEL and SERIAL are build from drive identify
# information, invalid characters are replaced by underline. If the path has
# the form '/path/dir/' (e.g. '/var/lib/smartd/'), then files
# 'MODEL-SERIAL.ata.csv' are created in directory '/path/dir'. If the path
# has the form '/path/name' (e.g. '/var/lib/misc/attrlog-'), then files
# 'nameMODEL-SERIAL.ata.csv' are created in directory '/path/'. The path must
# be absolute, except if debug mode is enabled.
  - attributelog: '/var/log/smartd/'
# [ATA only] Read the drive database from file. The new database replaces the
# built in database by default. If '+' is specified, then the new entries
# prepend the built in entries. Please see the smartctl(8) man page for further
# details.
    drivedb: ''
# Read smartd configuration Directives from file, instead of from the default
# location '/etc/smartd.conf'. If file does not exist, then smartd will print
# an error message and exit with nonzero status. If '-' is specified, the
# configuration is read from standard input.
    configfile: ''
# Use libcap-ng to drop unneeded Linux process capabilities. The following
# capabilities are kept: CAP_SYS_ADMIN, CAP_SYS_RAWIO, CAP_MKNOD. Warning: Mail
# notification does not work when used.
    capabilities: ''
# Runs smartd in "debug" mode. In this mode, it displays status information to
# STDOUT rather than logging it to SYSLOG and does not fork into the background
# and detach from the controlling terminal. In this mode, smartd also prints
# more verbose information about what it is doing than when operating in
# daemon" mode. In this mode, the INT signal (normally generated from a terminal
# with CONTROL-C) makes smartd reload its configuration file. Please use
# CONTROL-\ to exit.
    debug: 'false'
# Prints a list (to STDOUT) of all the possible Directives which may appear in
# the configuration file /etc/smartd.conf, and then exits. These Directives are
# described in the smartd.conf(5) man page. They may appear in the configuration
# file following the device name.
    showdirectives: 'false'
# Sets the interval between disk checks to N seconds, where N is a decimal
# integer. The minimum allowed value is ten and the maximum is the largest
# positive integer that can be represented on your system (often 2^31-1).
# The default is 1800 seconds.
    interval: '1800'
# Uses syslog facility to log the messages from smartd. Default is: 'daemon'.
    logfacility: 'daemon'
# Do not fork into background. This is useful when executed from modern init
# methods like initng, minit, supervise or systemd.
    nofork: 'true'
# Writes pidfile containing the smartd Process ID number (PID). To avoid symlink
# attacks make sure the directory to which pidfile is written is only writable
# for root. Without this option, or if the '--debug' option is given, no PID
# file is written on startup. If smartd is killed with a maskable signal then
# the pidfile is removed.
    pidfile: ''
# Specifies when, if ever, smartd should exit. The valid arguments are to this
# option are:
# nodev - Exit if there are no devices to monitor, or if any errors are found at
# startup in the configuration file. This is the default.
# errors - Exit if there are no devices to monitor, or if any errors are found
# in the configuration file /etc/smartd.conf at startup or whenever it is
# reloaded.
# nodevstartup - Exit if there are no devices to monitor at startup. But
# continue to run if no devices are found whenever the configuration file is
# reloaded.
# never - Only exit if a fatal error occurs (no remaining system memory,
# invalid command line arguments). In this mode, even if there are no devices to
# monitor, or if the configuration file /etc/smartd.conf has errors, smartd will
# continue to run, waiting to load a configuration file listing valid devices.
# onecheck - Start smartd in debug mode, then register devices, then check
# device's SMART status once, and then exit with zero exit status if all of
# these steps worked correctly.
#
# This last option is intended for 'distribution-writers' who want to create
# automated scripts to determine whether or not to automatically start up smartd
# after installing smartmontools. After starting smartd with this command-line
# option, the distribution's install scripts should wait a reasonable length of
# time (say ten seconds). If smartd has not exited with zero status by that
# time, the script should send smartd a SIGTERM or SIGKILL and assume that
# smartd will not operate correctly on the host. Conversely, if smartd exits
# with zero status, then it is safe to run smartd in normal daemon mode. If
# smartd is unable to monitor any devices or encounters other problems then it
# will return with non-zero exit status.
# showtests - Start smartd in debug mode, then register devices, then write a
# list of future scheduled self tests to stdout, and then exit with zero exit
# status if all of these steps worked correctly. Device's SMART status is not
# checked. This option is intended to test whether the '-s REGEX' directives
# in smartd.conf will have the desired effect. The output lists the next test
# schedules, limited to 5 tests per type and device. This is followed by a
# summary of all tests of each device within the next 90 days.
    quit: 'nodev'
# Intended primarily to help smartmontools developers understand the behavior
# of smartmontools on non-conforming or poorly-conforming hardware. This option
# reports details of smartd transactions with the device. The option can be
# used multiple times. When used just once, it shows a record of the ioctl()
# transactions with the device. When used more than once, the detail of these
# ioctl() transactions are reported in greater detail. The valid arguments to
# this option are:
# ioctl - report all ioctl() transactions.
# ataioctl - report only ioctl() transactions with ATA devices.
# scsiioctl - report only ioctl() transactions with SCSI devices.
# nvmeioctl - report only ioctl() transactions with NVMe devices.
#
# Any argument may include a positive integer to specify the level of detail
# that should be reported. The argument should be followed by a comma then the
# integer with no spaces. For example: 'ataioctl,2'. The default level is 1, so
# 'ataioctl,1' and 'ataioctl' are equivalent.
    report: ''
# Reads/writes smartd state information from/to files
# 'path/MODEL-SERIAL.ata.state' or 'path/VENDOR-MODEL-SERIAL.scsi.state'.
# This preserves SMART attributes, drive min and max temperatures
# ('-W' directive of smartd.conf), info about last sent warning email
# ('-m' directiveof smartd.conf), and the time of next check of the self-test
# REGEXP (-s directive) across boot cycles. MODEL and SERIAL are build from
# drive identify information, invalid characters are replaced by underline.
# If the path has the form '/path/dir/' (e.g. '/var/lib/smartd/'), then files
# 'MODEL-SERIAL.ata.state' are created in directory '/path/dir'. If the path
# has the form '/path/name' (e.g. '/var/lib/misc/smartd-'), then files
# 'nameMODEL-SERIAL.ata.state' are created in directory '/path/'. The path must
# be absolute, except if debug mode is enabled. The state information files are
# read on smartd startup. The files are always (re)written after reading the
# configuration file, before rereading the configuration file (SIGHUP), before
# smartd shutdown, and after a check forced by SIGUSR1. After a normal check
# cycle, a file is only rewritten if an important change (which usually results
# in a SYSLOG output) occurred.
    savestates: ''
# Run the executable path instead of the default script when smartd needs to
# send warning messages. Path must point to an executable binary file or script.
# The default script is /usr/share/smartmontools/smartd_warning.sh.
    warnexec: ''
```
