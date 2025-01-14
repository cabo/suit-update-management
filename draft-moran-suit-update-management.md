---
title: Update Management Extensions for Software Updates for Internet of Things (SUIT) Manifests
abbrev: SUIT Manifest Update Management Extensions
docname: draft-moran-suit-update-management-00
category: std

ipr: trust200902
area: Security
workgroup: SUIT
keyword: Internet-Draft

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: -o*+
  docmapping: yes
  toc_levels: 4

author:
 -
      ins: B. Moran
      name: Brendan Moran
      organization: Arm Limited
      email: Brendan.Moran@arm.com

normative:
  I-D.ietf-sacm-coswid:
  I-D.ietf-suit-manifest:
  RFC9019:

informative:
  I-D.ietf-suit-architecture:
  I-D.ietf-suit-information-model:
  I-D.ietf-teep-architecture:
  I-D.ietf-cbor-tags-oid:
  RFC7932:
  RFC1950:
  RFC8392:
  RFC7228:
  RFC8747:
  RFC8878:
  YAML:
    title: "YAML Ain't Markup Language"
    author:
    date: 2020
    target: https://yaml.org/
  HEX:
    title: "Intel HEX"
    author:
    -
      ins: "Wikipedia"
    date: 2020
    target: https://en.wikipedia.org/wiki/Intel_HEX
  SREC:
    title: "SREC (file format)"
    author:
    -
      ins: "Wikipedia"
    date: 2020
    target: https://en.wikipedia.org/wiki/SREC_(file_format)
  ELF:
    title: "Executable and Linkable Format (ELF)"
    author:
    -
      ins: "Wikipedia"
    date: 2020
    target: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
  COFF:
    title: "Common Object File Format (COFF)"
    author:
    -
      ins: "Wikipedia"
    date: 2020
    target: https://en.wikipedia.org/wiki/COFF

--- abstract
This specification describes extensions to the SUIT manifest format
defined in {{I-D.ietf-suit-manifest}}. These extensions allow an update
author, update distributor or device operator to more precisely control
the distribution and installation of updates to IoT devices. These
extensions also provide a mechanism to inform a management system of
Software Identifier and Software Bill Of Materials information about an
updated device.

--- middle

#  Introduction

Full management of software updates for unattended, connected devices, such as Internet of Things devices requires a cooperation between the update author(s) and management, distribution, policy enforcement, and auditing systems. This specification provides the extensions to the SUIT manifest ({{I-D.ietf-suit-manifest}}) that enable an author to coordinate with these other systems. These extensions enable authors to instruct devices to examine update priority, local update authorisation, update lifetime, and system properties. They also enable devices to report and distributors to collect Software Bill of Materials information.

Extensions in this specification are OPTIONAL to implment and OPTIONAL to include in manifests unless otherwise designated.

#  Conventions and Terminology

{::boilerplate bcp14}

Additionally, the following terminology is used throughout this document:

* SUIT: Software Update for the Internet of Things, also the IETF working group for this standard.

# Extension Metadata

Some additional metadata makes management of SUIT updates easier: a CoSWID can enable a 

## suit-coswid {#manifest-digest-coswid}

suit-coswid is a member of the suit-manifest. It contains a Concise Software Identifier (CoSWID) as defined in {{I-D.ietf-sacm-coswid}}. This element SHOULD be made severable so that it can be discarded by the Recipient or an intermediary if it is not required by the Recipient.

suit-coswid typically requires no processing by the Recipient. However all Recipients MUST NOT fail if a suit-coswid is present.

suit-coswid is RECOMMENDED to implement and RECOMMENDED to include in manifests.

TODO: CoRIM might be a preferable alternative to CoSWID.
TODO: Should CoMID be offered as an alternative to Vendor ID/Class ID?
TODO: Should there be a CoMID namespace identifier for UUIDs?

## text-version-required {#text-version-required}

suit-text-version-required is used to represent a version-based dependency on suit-parameter-version as described in {{suit-parameter-version}} and {{suit-condition-version}}. To describe a version dependency, a Manifest Author SHOULD populate the suit-text map with a SUIT_Component_Identifier key for the dependency component, and place in the corresponding map a suit-text-version-required key with a free text expression that is representative of the version constraints placed on the dependency. This text SHOULD be expressive enough that a device operator can be expected to understand the dependency. This is a free text field and there are no specific formatting rules.

By way of example only, to express a dependency on a component "\['x', 'y'\]", where the version should be any v1.x later than v1.2.5, but not v2.0 or above, the author would add the following structure to the suit-text element. Note that this text is in cbor-diag notation.

~~~
[h'78',h'79'] : {
    7 : ">=1.2.5,<2"
}
~~~

# Extension Parameters {#extension-parameters}

Several parameters are needed to define the behaviour of the commands specified in {{extension-commands}}. These parameters follow the same considerations as defined in {{secparameters(Parameters)<I-D.ietf-suit-manifest}} in {{I-D.ietf-suit-manifest}}

Name | CDDL Structure | Reference
---|---|---
Use Before | suit-parameter-use-before | {{suit-parameter-use-before}}
Minimum Battery | suit-parameter-minimum-battery | {{suit-parameter-minimum-battery}}
Update Priority | suit-parameter-update-priority | {{suit-parameter-update-priority}}
Version | suit-parameter-version | {{suit-parameter-version}}
Wait Info | suit-parameter-wait-info | {{suit-parameter-wait-info}}

## suit-parameter-use-before {#suit-parameter-use-before}

An expiry date for the use of the manifest encoded as the positive integer number of seconds since 1970-01-01. Implementations that use this parameter MUST use a 64-bit internal representation of the integer. Used with {{suit-condition-use-before}}

## suit-parameter-minimum-battery

This parameter sets the minimum battery level in mWh. This parameter is encoded as a positive integer. Used with suit-condition-minimum-battery ({{suit-condition-minimum-battery}}).

## suit-parameter-update-priority

This parameter sets the priority of the update. This parameter is encoded as an integer. It is used along with suit-condition-update-authorized ({{suit-condition-update-authorized}}) to ask an application for permission to initiate an update. This does not constitute a privilege inversion because an explicit request for authorization has been provided by the Update Authority in the form of the suit-condition-update-authorized command.

Applications MAY define their own meanings for the update priority. For example, critical reliability & vulnerability fixes MAY be given negative numbers, while bug fixes MAY be given small positive numbers, and feature additions MAY be given larger positive numbers, which allows an application to make an informed decision about whether and when to allow an update to proceed.

## suit-parameter-version {#suit-parameter-version}

Indicates allowable versions for the specified component. Allowable versions can be specified, either with a list or with range matching. This parameter is compared with version asserted by the current component when suit-condition-version ({{suit-condition-version}}) is invoked. The current component may assert the current version in many ways, including storage in a parameter storage database, in a metadata object, or in a known location within the component itself.

The component version can be compared as:

* Greater.
* Greater or Equal.
* Equal.
* Lesser or Equal.
* Lesser.

Versions are encoded as a CBOR list of integers. Comparisons are done on each integer in sequence. Comparison stops after all integers in the list defined by the manifest have been consumed OR after a non-equal match has occurred. For example, if the manifest defines a comparison, "Equal \[1\]", then this will match all version sequences starting with 1. If a manifest defines both "Greater or Equal \[1,0\]" and "Lesser \[1,10\]", then it will match versions 1.0.x up to, but not including 1.10.

While the exact encoding of versions is application-defined, semantic versions map conveniently. For example,

* 1.2.3 = \[1,2,3\].
* 1.2-rc3 = \[1,2,-1,3\].
* 1.2-beta = \[1,2,-2\].
* 1.2-alpha = \[1,2,-3\].
* 1.2-alpha4 = \[1,2,-3,4\].

suit-condition-version is OPTIONAL to implement.

Versions SHOULD be provided as follows:

1. The first integer represents the major number. This indicates breaking changes to the component.
2. The second integer represents the minor number. This is typically reserved for new features or large, non-breaking changes.
3. The third integer is the patch version. This is typically reserved for bug fixes.
4. The fourth integer is the build number.

Where Alpha (-3), Beta (-2), and Release Candidate (-1) are used, they are inserted as a negative number between Minor and Patch numbers. This allows these releases to compare correctly with final releases. For example, Version 2.0, RC1 should be lower than Version 2.0.0 and higher than any Version 1.x. By encoding RC as -1, this works correctly: \[2,0,-1,1\] compares as lower than \[2,0,0\]. Similarly, beta (-2) is lower than RC and alpha (-3) is lower than RC.

## suit-parameter-wait-info

suit-directive-wait ({{suit-directive-wait}}) directs the manifest processor to pause until a specified event occurs. The suit-parameter-wait-info encodes the parameters needed for the directive.

The exact implementation of the pause is implementation-defined. For example, this could be done by blocking on a semaphore, registering an event handler and suspending the manifest processor, polling for a notification, or aborting the update entirely, then restarting when a notification is received.

suit-parameter-wait-info is encoded as a map of wait events. When ALL wait events are satisfied, the Manifest Processor continues. The wait events currently defined are described in the following table.

Name | Encoding | Description
---|---|---
suit-wait-event-authorization | int | Same as suit-parameter-update-priority
suit-wait-event-power | int | Wait until power state
suit-wait-event-network | int | Wait until network state
suit-wait-event-other-device-version | See below | Wait for other device to match version
suit-wait-event-time | uint | Wait until time (seconds since 1970-01-01)
suit-wait-event-time-of-day | uint | Wait until seconds since 00:00:00
suit-wait-event-time-of-day-utc | uint | Wait until seconds since 00:00:00 UTC
suit-wait-event-day-of-week | uint | Wait until days since Sunday
suit-wait-event-day-of-week-utc | uint | Wait until days since Sunday UTC

suit-wait-event-other-device-version reuses the encoding of suit-parameter-version-match. It is encoded as a sequence that contains an implementation-defined bstr identifier for the other device, and a list of one or more SUIT_Parameter_Version_Match.


# Extension Commands

The following table defines the semantics of the commands defined in this specification in the same way as {{I-D.ietf-suit-manifest#command-behavior}}.

| Command Name | CDDL Identifier | Semantic of the Operation
|------|---|----
| Use Before  | suit-condition-use-before | assert(now() < current.params\[use-before\])
| Check Image Not Match | suit-condition-image-not-match | assert(not binary-match(digest(current), current.params\[digest\]))
| Check Minimum Battery | suit-condition-minimum-battery | assert(battery >= current.params\[minimum-battery\])
| Check Update Authorized | suit-condition-update-authorized | assert(isAuthorized(current.params\[priority\]))
| Check Version | suit-condition-version | assert(version_check(current, current.params\[version\]))
| Wait For Event |suit-directive-wait | until event(arg), wait


## suit-condition-use-before

Verify that the current time is BEFORE the specified time. suit-condition-use-before is used to specify the last time at which an update should be installed. The recipient evaluates the current time against the suit-parameter-use-before parameter ({{suit-parameter-use-before}}), which must have already been set as a parameter, encoded as seconds after 1970-01-01 00:00:00 UTC. Timestamp conditions MUST be evaluated in 64 bits, regardless of encoded CBOR size. suit-condition-use-before is OPTIONAL to implement.

## suit-condition-image-not-match

Verify that the current component does not match the suit-parameter-image-digest ({{I-D.ietf-suit-manifest#suit-parameter-image-digest}}). If no digest is specified, the condition fails. suit-condition-image-not-match is OPTIONAL to implement.

## suit-condition-minimum-battery

suit-condition-minimum-battery provides a mechanism to test a Recipient's battery level before installing an update. This condition is primarily for use in primary-cell applications, where the battery is only ever discharged. For batteries that are charged, suit-directive-wait is more appropriate, since it defines a "wait" until the battery level is sufficient to install the update. suit-condition-minimum-battery is specified in mWh. suit-condition-minimum-battery is OPTIONAL to implement. suit-condition-minimum-battery consumes suit-parameter-minimum-battery ({{suit-parameter-minimum-battery}}).

## suit-condition-update-authorized

Request Authorization from the application and fail if not authorized. This can allow a user to decline an update. suit-parameter-update-priority ({{suit-parameter-update-priority}}) provides an integer priority level that the application can use to determine whether or not to authorize the update. Priorities are application defined. suit-condition-update-authorized is OPTIONAL to implement.

## suit-condition-version

suit-condition-version allows comparing versions of firmware. Verifying image digests is preferred to version checks because digests are more precise. suit-condition-version examines a component's version against the version info specified in suit-parameter-version ({{suit-parameter-version}})

## suit-directive-wait {#suit-directive-wait}

suit-directive-wait directs the manifest processor to pause until a specified event occurs. Some possible events include:

1. Authorization
2. External Power
3. Network availability
4. Other Device Firmware Version
5. Time
6. Time of Day
7. Day of Week


#  IANA Considerations {#iana}

IANA is requested to:

* allocate key 14 in the SUIT Envelope registry for suit-coswid
* allocate key 14 in the SUIT Manifest registry for suit-coswid
* allocate key 7 in the SUIT Component Text registry for suit-text-version-required
* allocate the commands and parameters as shown in the following tables

## SUIT Commands

Label | Name | Reference
---|---|---
4 | Use Before | {{suit-condition-use-before}}
25 | Image Not Match | {{suit-condition-image-not-match}}
26 | Minimum Battery | {{suit-condition-minimum-battery}}
27 | Update Authorized | {{suit-condition-update-authorized}}
28 | Version | {{suit-condition-version}}
29 | Wait For Event | {{suit-directive-wait}}

## SUIT Parameters

Label | Name | Reference
---|---|---
4 | Use Before | {{suit-parameter-use-before}}
26 | Minimum Battery | {{suit-parameter-minimum-battery}}
27 | Update Priority | {{suit-parameter-update-priority}}
28 | Version | {{suit-parameter-version}}
29 | Wait Info | {{suit-parameter-wait-info}}

#  Security Considerations

This document extends the SUIT manifest specification. A detailed security treatment can be found in the architecture {{RFC9019}} and in the information model {{I-D.ietf-suit-information-model}} documents.


--- back

# A. Full CDDL {#full-cddl}

To be valid, the following CDDL MUST have the SUIT Manifest CDDL prepended to it. The SUIT CDDL is defined in {{I-D.ietf-suit-manifest#full-cddl}}

~~~ CDDL
{::include draft-moran-suit-update-management.cddl}
~~~

