---
title: Riak CS Release Notes
project: riakcs
version: 1.2.0+
document: cookbook
index: true
audience: intermediate
keywords: [developer]
---

## Riak CS 2.0.0

**For a complete set of release notes, upgrade instructions, and changed
configuration settings, please see the
[Full Riak CS 2.0.0 Release Notes][riak_cs_2.0.0_release_notes]**

### General Information

- This release updates Riak CS to work with Riak 2.0.5.
- We have simplified the configuration system.
- All official patches for older versions of Riak and Riak CS have been included
  in these releases. There is no need to apply any patches released for Riak CS
  1.4.x or 1.5.x to the Riak CS 2.0.x series. Patches released for Riak CS 1.4.x
  or 1.5.x cannot be directly applied to Riak CS 2.0.x because the version of
  Erlang/OTP shipped with Riak CS has been updated in version 2.0.0.
- Please review the complete Release Notes before upgrading.

### Known Issues & Limitations

- None.

### Changes and Additions

- Changed the name of `gc_max_workers` to `gc.max_workers`, and lowered the
  default value from 5 to 2 (#1110) to reduce the workload on the cs cluster.
- Partial support of GET Location API (#1057)
- Add very preliminary AWS v4 header authentication - without query
  string authentication, object chunking and payload checksum (#1064).
  There is still a lot of work to reliably use v4 authentication.
- Put Enterprise deps into dependency graph (#1065)
- Introduce Cuttlefish (#1020, #1068, #1076, #1086, #1090)
  (Stanchion #88, #90, #91)
- Yessir Riak client to measure performance (#1072, #1083)
- Inspector improvement with usage change (#1084)
- Check signed date in S3 authentication (#1067)
- Update `cluster_info` and various dependent libraries (#1087, #1088)
  (Stanchion #85, #87, #93)
- Storage calculation optimization (#1089) With Riak >= 2.1 this works
  with `use_2i_for_storage_calc` flag might relieve disk read of
  storage calculation.

### Bugfixes

- Fix wrong webmachine log handler name (#1075)
- Fix lager crash (#1038)
- Fix hardcoded crashdump path (#1052)
- Suppress unnecessary warnings (#1053)
- Multibag simpler state transition (Multibag #21)
- GC block deletion failure after transition to multibag environment
  (Multibag #19)
- Connection closing caused errors for objects stored before the
  transition, after transition from single bag to multibag
  configuration (Multibag #18).

### Deprecation Notices

- Multi-Datacenter Replication using v2 replication support has been deprecated.
- Old list objects which required `fold_objects_for_list_keys` as `false` have
  been deprecated and *will be removed* in the next major version.
- Non-paginated GC in cases where `gc_paginated_indexes` is `false` has been
  deprecated and *will be removed* in the next major version.

### General Notes on Upgrading to Riak CS 2.0.0

Upgrading a Riak CS system involves upgrading the underlying Riak, Riak CS and
Stanchion installations. The upgrade process can be non-trivial depending on
your existing system configurations and the combination of sub-system versions.
This document contains general instructions and notices on upgrading the whole
system to Riak CS 2.0.0.

#### New Configuration System

Riak 2.0.0 introduced a new configuration system (`riak.conf`), and as of Riak
CS 2.0.0, Riak CS now supports the new configuration style. Both Riak and Riak
CS still support the older style configurations through `app.config` and
`vm.args`.

**Basho recommends moving to the new unified configuration system**, using the
files `riak.conf`, `riak-cs.conf` and `stanchion.conf`.

#### Note on Legacy app.config Usage

**If you choose to use the legacy `app.config` files for Riak CS and/or
Stanchion, some parameters have changed names and must be updated**.

In particular, for the Riak CS `app.config`:

 - `cs_ip` and `cs_port` have been combined into `listener`.
 - `riak_ip` and `riak_pb_port` have been combined into `riak_host`.
 - `stanchion_ip` and `stanchion_port` have been combined into `stanchion_host`.
 - `admin_ip` and `admin_port` have been combined into `admin_listener`.
 - `webmachine_log_handler` has become `webmachine_access_log_handler`.
 - `{max_open_files, 50}` has been deprecated and should be replaced with
   `{total_leveldb_mem_percent, 30}`.

For the Stanchion `app.config`:

 - `stanchion_ip` and `stanchion_port` have been combined into `listener`.
 - `riak_ip` and `riak_port` have been combined into `riak_host`.

Each of the above pairs follows a similar form. Where the old form used a
separate IP and Port parameter, the new form combines those as `{new_option, {
"IP", Port}}`. For example, if your legacy `app.config` configuration was
previously:

```
{riak_cs, [
    {cs_ip, "127.0.0.1"},
    {cs_port, 8080 },
    . . .
]},
```

It should now read:

```
{riak_cs, [
    {listener, {"127.0.0.1", 8080}},
    . . .
]},
```

and so on.

#### Note: Upgrading from Riak CS 1.5.3 or Older

[Some key objects changed names][riak_cs_1.5_release_notes_upgrading] after the
upgrade. Applications may need to change their behaviour due to this bugfix.

#### Note: Upgrading from Riak CS 1.5.0 or Older

[Bucket number limitation per user][riak_cs_1.5_release_notes_upgrading_1] have
been introduced in 1.5.1. Users who have more than 100 buckets cannot create any
bucket after the upgrade unless the limit is extended in the system
configuration.

#### Note: Upgrading From Riak CS 1.4.x

An operational procedure [to clean up incomplete multipart under deleted
buckets][riak_cs_1.5_release_notes_incomplete_mutipart] is needed. Otherwise new
buckets with names that used to exist can't be created. The operation will fail
with 409 Conflict.

Leeway seconds and disk space should also be carefully watched during the
upgrade, because timestamp management of garbage collection was changed in the
1.5.0 release. Consult the "[Leeway seconds and disk
space][riak_cs_1.5_release_notes_leeway_and_disk] section of 1.5 release notes
for a more detailed description.

#### Note: Upgrading From Riak CS 1.3.x or Older

Basho supports upgrading from the two previous major versions to the latest
release. Thus, this document will only cover upgrading from Riak CS versions
1.4.x and 1.5.x.

To upgrade to Riak CS 2.0.0 from versions prior to Riak CS 1.4.0, operators will
need to first upgrade their system to Riak CS version 1.4.5 or 1.5.4. Upgrading
to Riak CS 1.5.4 is recommended. The underlying Riak installation must also be
upgraded to the Riak 1.4.x series, preferably version 1.4.12.

### General Upgrade Instructions

**For a complete set of release notes, upgrade instructions, and changed
configuration settings, please see the
[Full Riak CS 2.0.0 Release Notes][riak_cs_2.0.0_release_notes]**

#### All Scenarios

We recommend updating Stanchion before all other subsystems. Be careful not to
have multiple live Stanchion nodes accessible from Riak CS nodes at the same
time.

Repeat these steps on each node running Stanchion:

1. Stop Stanchion
2. Back up all Stanchion configuration files
3. Uninstall the current Stanchion package
4. Install the new Stanchion 2.0.0 package
5. Migrate the Stanchion configuration (See below)
6. Start Stanchion

#### Scenario: If Riak CS and Riak are both running on the same host.

Repeat these steps on every host:

1. Stop Riak CS
2. Stop Riak
3. Back up all Riak and Riak CS configuration files and remove all patches
4. Uninstall the current Riak CS package
5. Uninstall the current Riak Riak packages
6. Install the new Riak package
7. Install the new Riak CS 2.0.0 package
8. Migrate the Riak configuration (See below)
9. Migrate the Riak CS configuration (See below)
10. Start Riak
11. Start Riak CS

#### Scenario: If Riak CS and Riak are running on separate hosts.

When Riak CS is not installed on the same host as Riak, Riak CS can be upgraded
at any time while the corresponding remote Riak node is alive.

Repeat these steps on every host:

1. Stop Riak CS
2. Back up all configuration files and remove all patches
3. Uninstall the current Riak CS package
4. Install the new Riak CS 2.0.0 package
5. Migrate the Riak CS configuration (See below)
6. Start Riak CS

**For a complete set of release notes, upgrade instructions, and changed
configuration settings, please see the
[Full Riak CS 2.0.0 Release Notes][riak_cs_2.0.0_release_notes]**


[riak_cs_1.5_release_notes_upgrading]: https://github.com/basho/riak_cs/blob/release/1.5/RELEASE-NOTES.md#notes-on-upgrading
[riak_cs_1.5_release_notes_upgrading_1]: https://github.com/basho/riak_cs/blob/release/1.5/RELEASE-NOTES.md#notes-on-upgrading-1
[riak_cs_1.5_release_notes_incomplete_mutipart]: https://github.com/basho/riak_cs/blob/release/1.5/RELEASE-NOTES.md#incomplete-multipart-uploads
[riak_cs_1.5_release_notes_leeway_and_disk]: https://github.com/basho/riak_cs/blob/release/1.5/RELEASE-NOTES.md#leeway-seconds-and-disk-space
[riak_cs_2.0.0_release_notes]: https://github.com/basho/riak_cs/blob/develop/RELEASE-NOTES.md


## Riak CS 1.5.4

### Fixes

* Disable backpressure sleep
    [riak_cs/#1041](https://github.com/basho/riak_cs/pull/1041)
  * **Problem**: When backpressure sleep is triggered due to the
      presence of many siblings, this can lead to even more siblings.
  * **Solution**: This change prevents unnecessary siblings growth in
      cases where (a) backpressure is triggered under high upload
      concurrency and (b) uploads are interleaved during backpressure
      sleep. This issue does not affect multipart uploads.
* Fix an incorrect path rewrite in the S3 API caused by unnecessary URL
    decoding [riak_cs/#1040](https://github.com/basho/riak_cs/pull/1040)
    * **Problem**: Due to the incorrect handling of URL
        encoding/decoding, object keys including
        `%[0-9a-fA-F][0-9a-fA-F]` (as a regular expression) or `+` had
        been mistakenly decoded. As a consequence, the former case was
        decoded to some other binary, while in the latter case `+` was
        replaced with a space. In both cases, there was a possibility of
        an implicit data overwrite. For the latter case, an overwrite
        occurs for an object including `+` in its key, e.g. `foo+bar`,
        by a different object with a name that is largely similar but
        replaced with space, e.g. `foo bar`, and vice versa.
    * **Solution**: Fix the incorrect handling of URL encoding/decoding.
        This fix also addresses
        [riak_cs/#910](https://github.com/basho/riak_cs/pull/910) and
        [riak_cs/#977](https://github.com/basho/riak_cs/pull/977).

### Notes on Upgrading

After upgrading to Riak CS 1.5.4, objects including
`%[0-9a-fA-F][0-9a-fA-F]` (as a regular expression) or `+` in their key,
e.g.`foo+bar`, become invisible and can be seen as objects with a
different name. For the former case, objects will be referred to with
the unnecessarily decoded key; in the latter case, those objects will
be referred to with keys in which `+` is replaced with a space, e.g.
`foo bar`, by default.

The table below provides examples for URLs including
`%[0-9a-fA-F][0-9a-fA-F]` and how they will work before and after the
upgrade.


 | Before upgrade | After upgrade
:--|:---------------|:-------------
written as | `a%2Fkey` | `-`
read as | `a%2Fkey` or `a/key` | `a/key`
listed as | `a/key` | `a/key`

Examples for unique objects including `+` or an empty space through
upgrade:

  | Before upgrade | After upgrade
:--|:---------------|:-------------
written as | `a+key` | `-`
read as | `a+key` or `a key` | `a key`
listed as | `a key` | `a key`

Examples for unique objects with an empty space in the URL:

  | Before upgrade | After upgrade
:--|:---------------|:-------------
written as | `a key` | `-`
read as | `a+key` or `a key` | `a key`
listed as | `a key` | `a key`

This fix also changes the path format in access logs from the
single-URL-encoded style to the doubly-encoded URL style. Below is an
example of the old style:

```
127.0.0.1 - - [07/Jan/2015:08:27:07 +0000] "PUT /buckets/test/objects/path1%2Fpath2%2Fte%2Bst.txt HTTP/1.1" 200 0 "" """
```

And here is the analogous URL in the new style:

```
127.0.0.1 - - [07/Jan/2015:08:27:07 +0000] "PUT /buckets/test/objects/path1%2Fpath2%2Fte%252Bst.txt HTTP/1.1" 200 0 "" ""
```

Note that the object path has changed from `path1%2Fpath2%2Fte%2Bst.txt`
to `path1%2Fpath2%2Fte%252Bst.txt` between the two examples above.

If the old behavior is preferred, e.g. because applications using Riak
CS have been written to use the older style, you can retain that
behavior on upgrade by modifying your Riak CS configuration. Change the
`rewrite_module` setting as follows:

```appconfig
{riak_cs, [
           %% Other settings
           {rewrite_module, riak_cs_s3_rewrite_legacy},
           %% Other settings
]}
```

**Note**: The old behavior is technically incorrect and implicitly
overwrites data in the ways described above. Retain the old behavior
with caution.

## Riak CS 1.5.3

### Changes

* Add `read_before_last_manifest_write` option to help avoid sibling
  explosion for use cases involving high churn and concurrency on a
  fixed set of keys. When sibling explosion occurs, the objects stored in
  Riak can become very large and severely impair the functioning of the
  system. The trade-off in enabling this option is a latency penalty of
  doing an extra read before the final write of an object's manifest to
  Riak. However, for use cases matching the description, the minor
  latency penalty is preferable to consequences of sibling explosion.
  [riak_cs/#1011](https://github.com/basho/riak_cs/pull/1011)
* Add configurable timeouts for all Riak CS interactions with Riak to
  provide more flexibility in operational situations.
  [riak_cs/#1021](https://github.com/basho/riak_cs/pull/1021)

### Fixes

* Fix storage calculation
    [riak_cs/#996](https://github.com/basho/riak_cs/pull/996)
  * **Problem**: Data for deleted buckets would be included in the
      calculation results
  * **Solution**: Storage calculations no longer include deleted buckets

### Known Issues

None

### Download

Please see the [Riak CS Downloads
Page](http://docs.basho.com/riakcs/latest/riakcs-downloads/).

### Feedback

We would love to hear from you. You can reach us in any of the following
venues:

* [Basho mailing
  list](http://lists.basho.com/mailman/listinfo/riak-users_lists.basho.com)
* [The official Basho docs](https://github.com/basho/basho_docs)
* [Riak CS on GitHub](https://github.com/basho/riak_cs)
* Via email at **info@basho.com**

## Riak CS 1.5.2

### Changes

* Improve logging around failures with Riak
  [riak_cs/#987](http://docs.basho.com/riak/latest/dev/using/libraries/)
* Add amendment log output when storing access stats into Riak failed
  [riak_cs/#988](https://github.com/basho/riak_cs/pull/988). This change
  prevents losing access stats logs in cases of temporary connection
  failure between Riak and Riak CS. Access logs are stored in
  `console.log` at the `warning` level.
* Add script to repair invalid garbage collection manifests
  [riak_cs/#983](https://github.com/basho/riak_cs/pull/983). There is a
  known issue where an active manifest would be stored in the GC bucket.
  This script changes invalid state to valid state.

### Fixes

* Fix Protocol Buffers connection pool (`pbc_pool_master`) leak
  [riak_cs/#986](https://github.com/basho/riak_cs/pull/986).
  * **Problem**: Requests for non-existent buckets without an
  authorization header and requests for listing users make connections
  leak from the pool, causing the pool to eventually go empty. This bug
  was introduced in release 1.5.0.
  * **Solution**: Fix the leak by properly releasing connections.

### Known Issues

None

### Download

Please see the [Riak CS Downloads
Page](http://docs.basho.com/riakcs/latest/riakcs-downloads)

### Feedback

We would love to hear from you. You can reach us in any of the following
venues:

* [Basho mailing
  list](http://lists.basho.com/mailman/listinfo/riak-users_lists.basho.com)
* [The official Basho docs](https://github.com/basho/basho_docs)
* [Riak CS on GitHub](https://github.com/basho/riak_cs)
* Via email at **info@basho.com**

## Riak CS 1.5.1

### Additions

* Bucket restrictions --- Similar to S3, you can now limit the number of buckets created per user to prevent users from creating an unusually large number of buckets. More details are included [here](http://docs.basho.com/riakcs/latest/cookbooks/configuration/Configuring-Riak-CS/).

### Changes

* Add sleep interval after updating manifests to suppress sibling explosion [riak_cs/#959](https://github.com/basho/riak_cs/pull/959). In order to suppress sibling explosion, a sleep interval is added after updating manifests. The duration of the sleep interval depends on the number of siblings. The problem is documented in more detail [here](https://github.com/basho/riak_cs/pull/959).
* Update `riak-cs-debug` to include information about bags in a multibag environment [riak_cs/#930](https://github.com/basho/riak_cs/issues/882). Bag listing and weight information are now included in the output of the `riak-cs-debug` command in order to help in investigating issues in a multibag environment.
* More efficient bucket resolution [riak_cs/#951](https://github.com/basho/riak_cs/pull/951). Previously, sibling resolution logic was inefficient in cases where users had many buckets (> 1000). For the sake of optimization, resolution is now skipped entirely when no siblings are present (i.e. when there is a single value).
* Similar to S3, add a limitation on the part number in a multipart upload [riak_cs/#957](https://github.com/basho/riak_cs/pull/957). Part numbers can now range from 1 to 10,000 (inclusive).

### Fixes

* GC may stall due to `riak_cs_delete_fsm` deadlock [riak_cs/#949](https://github.com/basho/riak_cs/pull/949)
  * **Problem** --- Garbage collection can stall when a `riak_cs_delete_fsm` worker process encounters a deadlock condition.
  * **Solution** --- One of the requirements in an internal data structure was violated. This fix satisfies the requirement so that deadlock does not happen.
* Fix wrong log directory for gathering logs on `riak-cs-debug` [riak_cs/#953](https://github.com/basho/riak_cs/pull/953)
  * **Problem** --- Directory structure of log files gathered by `riak-cs-debug` was different from `riak-debug`.
  * **Solution** --- The directory structure is now the same as that of `riak-debug`.
* Avoid DST-aware translation from local time to GMT [riak_cs/#954](https://github.com/basho/riak_cs/pull/954)
  * **Problem** --- Transformation from local time to GMT is slow, especially when performed by multiple threads. One such transformation was in the path of the `GET Object` API call.
  * **Solution** --- Eliminate the transformation.
* Use new UUID for seed of canonical ID instead of secret [riak_cs/#956](https://github.com/basho/riak_cs/pull/956)
  * **Problem** --- MD5-hashed value of secret access key was used in order to generate a canonical ID, which is public information. Although MD5 reverse is not realistic, it is unnecessary and avoidable to use a secret access key for canonical ID generation.
  * **Solution** --- Use newly generated UUID for canonical ID.
* Set timeout as `infinity` to replace the default of `5000ms` [riak_cs/#963](https://github.com/basho/riak_cs/pull/963)
  * **Problem** --- In Riak CS 1.5.0, middleman process wrappers for Protocol Buffers sockets were introduced and call timeout to them was incorrectly set to a default of 5000 milliseconds.
  * **Solution** --- Change the call timeout to `infinity` and actual timeout is controlled by Protocol Buffers processes.
* Skip invalid state manifests in GC bucket [riak_cs/#964](https://github.com/basho/riak_cs/pull/964)
  * **Problem** --- If there were active state manifests in the GC bucket the GC process crashed.
  * **Solution** --- Skip active state manifests and make the GC process collect valid manifests.

### Known Issues

None

### Platforms Tested

* Ubuntu GNU / Linux 12.04

### Installation and Upgrade Notes

#### Per-user bucket creation restrictions

Beginning with Riak CS 1.5.1, you can limit the number of buckets that can be created per user. The default maximum number is 100. While this limitation prohibits the creation of new buckets by users, users that exceed the limit can still perform other operations, including bucket deletion. To change the default limit, add the following line to the `riak_cs` section of `app.config`:

```appconfig
{riak_cs, [
    %% ...
    {max_buckets_per_user, 5000},
    %% ...
]}
```

To avoid having a limit, set `max_buckets_per_user_user` to `unlimited`.


### Download

Please see the [Riak CS Downloads Page](http://docs.basho.com/riakcs/latest/riakcs-downloads/).

### Feedback

We would love to hear from you. You can reach us at any of the following links:

* http://lists.basho.com/mailman/listinfo/riak-users_lists.basho.com
* https://github.com/basho/basho_docs
* https://github.com/basho/riak_cs

Or via email at **info@basho.com**.

## Riak CS 1.5.0

### Additions

* Added Multibag Technical Preview to Riak CS. More info is available [here](http://docs.basho.com/riakcs/latest/cookbooks/multibag/)
* A new command `riak-cs-debug` including `cluster-info` [riak_cs/#769](https://github.com/basho/riak_cs/pull/769), [riak_cs/#832](https://github.com/basho/riak_cs/pull/832)
* Tie up all existing commands into a new command `riak-cs-admin` [riak_cs/#839](https://github.com/basho/riak_cs/pull/839)
* Add a command `riak-cs-admin stanchion` to switch Stanchion IP and port manually [riak_cs/#657](https://github.com/basho/riak_cs/pull/657)
* Performance of garbage collection has been improved via Concurrent GC [riak_cs/#830](https://github.com/basho/riak_cs/pull/830)
* Iterator refresh [riak_cs/#805](https://github.com/basho/riak_cs/pull/805)
* `fold_objects_for_list_keys` made default in Riak CS [riak_cs/#737](https://github.com/basho/riak_cs/pull/737), [riak_cs/#785](https://github.com/basho/riak_cs/pull/785)
* Add support for Cache-Control header [riak_cs/#821](https://github.com/basho/riak_cs/pull/821)
* Allow objects to be reaped sooner than leeway interval. [riak_cs/#470](https://github.com/basho/riak_cs/pull/470)
* PUT Copy on both objects and upload parts [riak_cs/#548](https://github.com/basho/riak_cs/pull/548)
* Update to lager 2.0.3
* Compiles with R16B0x (Releases still by R15B01)
* Change default value of `gc_paginated_index` to `true` [riak_cs/#881](https://github.com/basho/riak_cs/issues/881)
* Add new API: Delete Multiple Objects [riak_cs/#728](https://github.com/basho/riak_cs/pull/728)
* Add warning logs for manifests, siblings, bytes and history [riak_cs/#915](https://github.com/basho/riak_cs/pull/915)

### Bugs Fixed

* Align `ERL_MAX_PORTS` with Riak default: 64000 [riak_cs/#636](https://github.com/basho/riak_cs/pull/636)
* Allow Riak CS admin resources to be used with OpenStack API [riak_cs/#666](https://github.com/basho/riak_cs/pull/666)
* Fix path substitution code to fix Solaris source builds [riak_cs/#733](https://github.com/basho/riak_cs/pull/733)
* `sanity_check(true,false)` logs invalid error on `riakc_pb_socket` error [riak_cs/#683](https://github.com/basho/riak_cs/pull/683)
* Riak-CS-GC timestamp for scheduler is in the year 0043, not 2013. [riak_cs/#713](https://github.com/basho/riak_cs/pull/713) fixed by [riak_cs/#676](https://github.com/basho/riak_cs/pull/676)
* Excessive calls to OTP code_server process #669 fixed by [riak_cs/#675](https://github.com/basho/riak_cs/pull/675)
* Return HTTP 400 if content-md5 does not match [riak_cs/#596](https://github.com/basho/riak_cs/pull/596)
* `/riak-cs/stats` and `admin_auth_enabled=false` don't work together correctly. [riak_cs/#719](https://github.com/basho/riak_cs/pull/719)
* Storage calculation doesn't handle tombstones, nor handle undefined manifest.props [riak_cs/#849](https://github.com/basho/riak_cs/pull/849)
* MP initiated objects remains after delete/create buckets #475 fixed by [riak_cs/#857](https://github.com/basho/riak_cs/pull/857) and [stanchion/#78](https://github.com/basho/stanchion/pull/78)
* handling empty query string on list multipart upload [riak_cs/#843](https://github.com/basho/riak_cs/pull/843)
* Setting ACLs via headers at PUT Object creation [riak_cs/#631](https://github.com/basho/riak_cs/pull/631)
* Improve handling of poolboy timeouts during ping requests [riak_cs/#763](https://github.com/basho/riak_cs/pull/763)
* Remove unnecessary log message on anonymous access [riak_cs/#876](https://github.com/basho/riak_cs/issues/876)
* Fix inconsistent ETag on objects uploaded by multipart [riak_cs/#855](https://github.com/basho/riak_cs/issues/855)
* Fix policy version validation in PUT Bucket Policy [riak_cs/#911](https://github.com/basho/riak_cs/issues/911)
* Fix return code of several commands, to return 0 for success [riak_cs/#908](https://github.com/basho/riak_cs/issues/908)
* Fix `{error, disconnected}` repainted with notfound [riak_cs/#929](https://github.com/basho/riak_cs/issues/929)

### Notes on Upgrading

#### Incomplete multipart uploads

[riak_cs/#475](https://github.com/basho/riak_cs/issues/475) was a
security issue where a newly created bucket may include unaborted or
incomplete multipart uploads which was created in previous epoch of
the bucket with same name. This was fixed by:

- on creating buckets; checking if live multipart exists and if
  exists, return 500 failure to client.

- on deleting buckets; trying to clean up all live multipart remains,
  and checking if live multipart remains (in stanchion). if exists,
  return 409 failure to client.

Note that a few operations are needed after upgrading from 1.4.x (or
former) to 1.5.0.

- run `riak-cs-admin cleanup-orphan-multipart` to cleanup all
  buckets. It would be safer to specify timestamp with ISO 8601 format
  like `2014-07-30T11:09:30.000Z` as an argument. For example, in
  which time all CS nodes upgrade has finished. Then the cleaner does
  not clean up multipart uploads newer than that timestamp. Some
  corner cases can be prevented where multipart uploads conflicting
  with bucket deletion and this cleanup.

- there might be a time period until above cleanup finished, where no
  client can create bucket if unfinished multipart upload remains
  under deleted bucket. You can find [critical] log if such bucket
  creation is attempted.

#### Leeway seconds and disk space

[riak_cs/#470](https://github.com/basho/riak_cs/pull/470) changed the
behaviour of object deletion and garbage collection. The timestamps in
garbage collection bucket were changed from the future time when the
object is to be deleted, to the current time when the object is
deleted, Garbage collector was also changed to collect objects until
'now - leeway seconds', from collecting objects until 'now' previously.

Before (-1.4.x):

```
           t1                         t2
-----------+--------------------------+------------------->
           DELETE object:             GC triggered:
           marked as                  collects objects
           "t1+leeway"                marked as "t2"
```

After (1.5.0-):

```
           t1                         t2
-----------+--------------------------+------------------->
           DELETE object:             GC triggered:
           marked as "t1"             collects objects
           in GC bucket               marked as "t2 - leeway"
```

This leads that there exists a period where no objects are collected
right after upgrade to 1.5.0, say, `t0`, until `t0 + leeway` . And
objects deleted just before `t0` won't be collected until `t0 +
2*leeway` .

Also, all CS nodes which run GC should be upgraded *first.* CS nodes
which do not run GC should be upgraded later, to let leeway second
system work properly. Or stop GC while upgrading whole cluster, by
running `riak-cs-admin gc set-interval infinity` .

Multi data center cluster should be upgraded more carefully, as to
make sure GC is not running while upgrading.

#### Riak CS Multibag

Multibag, the ability to store object manifests and blocks in separate
clusters or groups of clusters, has been added as an Enterprise feature,
but it is in early preview status. `proxy_get` has not yet been
implemented for this preview feature, so multibag is intended for a
single DC only at this time. More information on Multibag is available
[[here|Riak CS Multibag Support]].

### Known Issues and Limitations

* If a client sends another request in the same connection while
  waiting for copy finish, the copy also will be aborted.  This is a
  side effect of client disconnect detection in case of object copy.
  See [#932](https://github.com/basho/riak_cs/pull/932) for further
  information.

* Copying objects in OOS interface is not implemented.

* Multibag is added as Enterprise feature, but it is in early preview
  status. `proxy_get` setup among clusters multibag on is not
  implemented yet.


## Riak CS 1.4.5

#### Bugs Fixed

* Fix several 'data hiding' bugs with the v2 list objects FSM [riak_cs/788](https://github.com/basho/riak_cs/issues/788) .
* Don't treat HEAD requests toward BytesOut in access statistics [riak_cs/791](https://github.com/basho/riak_cs/issues/791) .
* Handle whitespace in POST/PUT XML documents [riak_cs/795](https://github.com/basho/riak_cs/issues/795) .
* Handle unicode user-names and XML [riak_cs/807](https://github.com/basho/riak_cs/issues/807) .
* Fix missing XML fields on storage usage [riak_cs/808](https://github.com/basho/riak_cs/issues/808) .
* Adjust fold-objects timeout [riak_cs/811](https://github.com/basho/riak_cs/issues/811) .
* Prune deleted buckets from user record [riak_cs/812](https://github.com/basho/riak_cs/issues/812) .
* Fix bad bucketname in storage usage [riak_cs/800](https://github.com/basho/riak_cs/issues/800) .

Riak CS 1.4.4 introduced
[a bug (#800)](https://github.com/basho/riak_cs/issues/800) where
storage calculations made while running that version would have the
bucket-name replaced by the string "struct". This version fixes the
bug, but can't go back and retroactively fix the old storage
calculations. Aggregations on an entire user-account should still be
accurate, but you won't be able to break-down storage by bucket, as
they will all share the name "struct".


#### Additions

* Optimize the list objects v2 FSM for prefix requests [riak_cs/804](https://github.com/basho/riak_cs/issues/804) .

## Riak CS 1.4.4

[Riak CS 1.4.4 Release Notes](https://github.com/basho/riak_cs/blob/1.4.4/RELEASE-NOTES.md)

#### Bugs Fixed

* Create basho-patches directory [riak_cs/775](https://github.com/basho/riak_cs/issues/775) .
* `sum_bucket` timeout crashes all storage calculation is fixed by [riak_cs/759](https://github.com/basho/riak_cs/issues/759) .
* Failure to throttle access archiver is fixed by [riak_cs/758](https://github.com/basho/riak_cs/issues/758) .
* Access archiver crash is fixed by [riak_cs/747](https://github.com/basho/riak_cs/issues/747) .

## Riak CS 1.4.3

[Riak CS 1.4.3 Release Notes](https://github.com/basho/riak_cs/blob/1.4.3/RELEASE-NOTES.org)

#### Bugs Fixed

* Fix bug that reverted manifests in the `scheduled_delete` state to the `pending_delete` or active state.
* Don't count already deleted manifests as overwritten
* Don't delete current object version on overwrite with incorrect md5

#### Additions

* Improve performance of manifest pruning
* Optionally use paginated 2i for the GC daemon. This is to help prevent timeouts when collecting data that can be garbage collected.
* Improve handling of Riak disconnects on block fetches
* Update to Lager 2.0.1
* Optionally prune manifests based on count, in addition to time
* Allow multiple access archiver processes to run concurrently

## Riak CS 1.4.2

#### Bugs Fixed

* Fix issue with Enterprise build on Debian Linux distributions.
* Fix source tarball build.
* Fix access statistics bug that caused all accesses to be treated as errors.
* Make logging in bucket listing map phase function lager version agnostic to avoid issues when using versions of Riak older than 1.4.
* Handle undefined `props` field in manifests to fix issue accessing objects written with a version of Riak CS older than 1.3.0.

#### Additions

* Add option to delay initial GC sweep on a node using the `initial_gc_delay` configuration option.
* Append random suffix to GC bucket keys to avoid hot keys and improve performance during periods of frequent deletion.
* Add `default_proxy_cluster_id` option to provide a way to specify a default cluster id to be used when the cluster id is undefined. This is to facilitate migration from the OSS version to the Enterprise version.

## Riak CS 1.4.1

#### Bugs Fixed
* Fix list objects crash when more than the first 1001 keys are in the pending delete state.
* Fix crash in garbage collection daemon.
* Fix packaging bug by updating `node_package` dependency.

## Riak CS 1.4.0

#### Additions

* Add preliminary support for the Swift API and Keystone authentication.
* Improve performance of object listing when using Riak 1.4.0 or greater.
* Add ability to edit user account name and email address.
* Add support for v3 multi-data-center replication.
* Add configurable Riak connection timeouts.
* Add syslog support via Lager.
* Only contact one vnode for immutable block requests.

#### Bugs Fixed

* Remove unnecessary keys in GC bucket.
* Fix query-string authentication for multi-part uploads.
* Fix Storage Class for multi-part uploaded objects.
* Fix etags for multi-part uploads.
* Support reformat indexes in the Riak CS multi-backend.
* Fix unbounded memory-growth on `GET` requests with a slow connection.
* Reduce access-archiver memory use.
* Fix 500 on object ACL `HEAD` request.
* Fix semantics for concurrent upload and delete of the same key with a   multipart upload.
* Verify `content-md5` header if supplied.
* Handle transient Riak connection failures.

## Riak CS 1.3.1

#### Bugs Fixed

* Fix bug in handling of active object manifests in the case of overwrite or delete that could lead to old object versions being resurrected.
* Fix improper capitalization of user metadata header names.
* Fix issue where the S3 rewrite module omits any query parameters that are not S3 subresources. Also correct handling of query parameters so that parameter values are not URL decoded twice. This primarily affects pre-signed URLs because the access key and request signature are included as query parameters.
* Fix for issue with init script stop.

## Riak CS 1.3.0

#### Additions

* Support for multipart file uploads. Parts must be in the range of 5MB-5GB.
* Support for bucket policies using a restricted set of principals and conditions.
* Support for returning bytes ranges of a file using the `Range` header.
* Administrative commands may be segregated onto a separate interface.
* Authentication for administrative commands may be disabled.
* Performance and stability improvements for listing the contents of buckets.
* Support for the prefix, delimiter, and marker options when listing the contents of a bucket.
* Support for using Webmachine's access logging features in conjunction with the Riak CS internal access logging mechanism.
* Moved all administrative resources under `/riak-cs`.
* Riak CS now supports packaging for FreeBSD, SmartOS, and Solaris.

#### Bugs Fixed

* Fix handling of cases where buckets have siblings. Previously this resulted in 500 errors returned to the client.
* Reduce likelihood of sibling creation when creating a bucket.
* Return a 404 instead of a 403 when accessing a deleted object.
* Unquote URLs to accommodate clients that URL encode `/` characters in URLs.
* Deny anonymous service-level requests to avoid unnecessary error messages trying to list the buckets owned by an undefined user.

## Riak CS 1.2.2

#### Additions

* Full support for MDC replication

#### Bugs Fixed

* Fix problem where objects with `utf-8` unicode key can be neither listed nor fetched.
* Speed up `bucket_empty` check and fix process leak. This bug was originally found when a user was having trouble with `s3cmd rb :s3//foo --recursive`. The operation first tries to delete the (potentially large) bucket, which triggers our bucket empty check. If the bucket has more than 32k items, we run out of processes unless `+P` is set higher (because of the leak).

## Riak CS 1.2.1

#### Additions

* Add reduce phase for listing bucket contents to provide backpressure when executing the MapReduce job.
* Use prereduce during storage calculations.
* Return 403 instead of 404 when a user attempts to list contents of nonexistent bucket.

#### Bugs Fixed

* Return 403 instead of 404 when a user attempts to list contents of nonexistent bucket.
* Do not do bucket list for `HEAD` or `?versioning` or `?location` request.

## Riak CS 1.2.0

#### Additions

* Add preliminary support for MDC replication
* Quickcheck test to exercise the erlcloud library against Riak CS
* Basic support for riak_test integration

#### Bugs Fixed

* Do not expose stack traces to users on 500 errors
* Fix issue with sibling creation on user record updates
* Fix crash in terminate state when fsm state is not fully populated
* Script fixes and updates in response to node_package updates

## Riak CS 1.1.0

#### Additions

* Update user creation to accept a JSON or XML document for user creation instead of URL encoded text string.
* Configuration option to allow anonymous users to create accounts. In the default mode, only the administrator is allowed to create accounts.
* Ping resource for health checks.
* Support for user-specified metadata headers.
* User accounts may be disabled by the administrator.
* A new key_secret can be issued for a user by the administrator.
* Administrator can now list all system users and optionally filter by enabled or disabled account status.
* Garbage collection for deleted and overwritten objects.
* Separate connection pool for object listings with a default of 5 connections.
* Improved performance for listing all objects in a bucket.
* Statistics collection and querying.
* DTrace probing.

#### Bugs Fixed

* Check for timeout when checking out a connection from poolboy.
* PUT object now returns 200 instead of 204.
* Fixes for Dialyzer errors and warnings.
* Return readable error message with 500 errors instead of large webmachine backtraces.

## Riak CS 1.0.2

#### Additions

* Support query parameter authentication as specified in [http://docs.amazonwebservices.com/AmazonS3/latest/dev/RESTAuthentication.html](Signing and Authenticating REST Requests).

## Riak CS 1.0.1

#### Bugs Fixed

* Default `content-type` is not passed into function to handle `PUT` request body
* Requests hang when a node in the Riak cluster is unavailable
* Correct inappropriate use of `riak_cs_utils:get_user` by `riak_moss_acl_utils:get_owner_data`

## Riak CS 1.0.0

#### Additions

* Subsystem for calculating user access and storage usage
* Fixed-size connection pool of Riak connections
* Use a single Riak connection per request to avoid deadlock conditions
* Object ACLs
* Management for multiple versions of a file manifest
* Configurable block size and max content length
* Support specifying non-default ACL at bucket creation time

#### Bugs Fixed
* Fix PUTs for zero-byte files
* Fix fsm initialization race conditions
* Canonicalize the entire path if there is no host header, but there are tokens
* Fix process and socket leaks in get fsm

## Riak CS 0.1.2

#### Bugs Fixed
* Return 403 instead of 503 for invalid anonymous or signed requests.
* Properly clean up processes and connections on object requests.

## Riak CS 0.1.1

#### Bugs Fixed
* HEAD requests always result in a `403 Forbidden`.
* `s3cmd info` on a bucket object results in an error due to missing ACL document.
* Incorrect atom specified in `riak_moss_wm_utils:parse_auth_header`.
* Bad match condition used in `riak_moss_acl:has_permission/2`.

## Riak CS 0.1.0

#### Additions
* Bucket-level access control lists
* User records have been modified so that an system-wide unique email address is required to create a user.
* User creation requests are serialized through `stanchion` to be certain the email address is unique.
* Bucket creation and deletion requests are serialized through `stanchion` to ensure bucket names are unique in the system.
* The `stanchion` serialization service is now required to be installed and running for the system to be fully operational.
* The concept of an administrative user has been added to the system. The credentials of the administrative user must be added to the `app.config` files for `moss` and `stanchion`.
* User credentials are now created using a url-safe base64 encoding module.

#### Bugs Fixed

* `s3cmd info` fails due to missing `last-modified` key in return document.
* `s3cmd get` of 0 byte file fails.
* Bucket creation fails with status code `415` using the AWS Java SDK.

#### Known Issues

* Object-level access control lists have not yet been implemented.

## Riak CS 0.0.3

#### Additions

* Support for the `s3cmd` subcommands `sync`, `du`, and `rb`
* Return valid size and checksum for each object when listing bucket objects.
* Changes so that a bucket may be deleted if it is empty.
* Changes so a subdirectory path can be specified when storing or retrieving files.
* Make buckets private by default
* Support the prefix query parameter
* Enhance process dependencies for improved failure handling

#### Bugs Fixed

* URL decode keys on put so they are represented correctly. This eliminates confusion when objects with spaces in their names are listed and when attempting to access them.
* Properly handle zero-byte files
* Reap all processes during file puts

#### Known Issues

* Buckets are marked as `/private/` by default, but globally unique bucket names are not enforced. This means that two users may create the same bucket and this could result in unauthorized access and unintentional overwriting of files. This will be addressed in a future release by ensuring that bucket names are unique across the system.
