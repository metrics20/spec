# Metrics 2.0 spec

This specification lives at [github.com/metrics20/spec](https://github.com/metrics20/spec), that's where change requests can be made

## Table of Contents
1. [Role of the spec](#role-of-the-spec)
2. [Glossary](#glossary)
3. [Data model](#data-model)
4. [Tags](#tags)
    * [Tag keys](#tag-keys)
    * [Tag values: unit](#tag-values-unit)
    * [Tag values: stat](#tag-values-stat)
    * [Tag values: mtype](#tag-values-mtype)
5. [Examples](#examples)
    * [Example 1: Swift proxy server timings](#example-1-swift-proxy-server-timings)
    * [Example 2: Disk space](#example-2-disk-space)

## Role of the spec

* foster adoption of shared terminology and accurate classification
* make metrics more self-describing

With the end goal of tooling interoperability, correctness and being more user friendly.  see http://metrics20.org/ for more details.

It does not dictate transport protocols or storage mechanisms (except it imposes minimum requirements to support the spec),
since that's an area in heavy flux and spans a broad technical spectrum where
varying tradeoffs make sense (e.g. simplicity vs high performance), though the metrics2.0 project and website
also aims to bring projects together under shared implementations and formats (see http://metrics20.org/implementations/)

## Glossary

Tag
: pieces of text that describe or identify a metric. Can either be in key=value form (e.g. 'env=prod') or just value without a key. (e.g. 'prod') which can be more convenient if you don't do key based searches.  You should be able to mix both styles even for a given metric, But it is highly encouraged to use the key=value format as much as possible.  If not explicitly specified, a tag can be assumed to be intrinsic.  Tag values should always be non-empty strings, except for unit.

(Tag) Key
: describes the dimension or property being measured.  Can be very useful for specifying aggregations, grouping, filtering and searching.

Intrinsic tag
: A tag value that describes the thing being measured in a fundamental way.  Changing this tag means we're talking about measuring something else or in a different way, and means the timeseries identifier changes, so we're talking about a different series. (e.g. mtype, unit).

Extrinsic tag 
: A tag value that provides information about the thing being measured, or the data, but that can change over time without meaning a changing in the timeseries identifier (e.g. line, agent).
This is optional and implementation specific (e.g. are changes over time tracked or just the current state)

Meta tag
: Synonym for extrinsic tag

Key
: In many systems, a string is also used to uniquely identify a timeseries, in addition to tags.


## Data model

* metrics have tags and optionally meta tags and optionally a key
* the metric (timeseries) identity is directly tied to the metric key (if any) and tags values (but not their order).
* this means that for a different key or a different set of tags we're talking about a different stream of data.
* meta tags can change without affecting the metric identity.
* tags should be chosen to make the metric as self-describing as possible, a key can be used for information that is hard to fit in tags.  It is OK for there to be semantical overlap between tags and the key. (e.g. to put information in the key that is already also in tags)
* characters that should be allowed: (at least) alphanumerics (case sensitive), underscore, hyphen, dot, forward slash.  (units can contain slash, ip/hostname can contain dot)


## Tags

* the following tags are **mandatory**: unit, mtype.
* try to add as many tags to your metrics as possible, using all tags in the below table as appropriate or coming up with your own as needed.
* if the tag key you're looking for is not in the spec yet, open a ticket to try to get it added.
* keys should be fitting, descriptive and short, in that order of importance. 
* try to keep everything lowercase except where upper casing is sensible (e.g. units and prefixes that use capitals (M, B,...) or commonly capitalized terms such as http verbs (GET/PUT)


### Tag keys

Tag key     | use
------------|-----:
host        | physical or virtual machine
http_method | the http method. like PUT, GET, etc.
http_code   | 200, 404, etc
device      | block device, network device, ...
unit        | the unit something is expressed in (b/s, MB, etc). See below.
what        | the thing being measured, if the other tags don't suffice. often same as metric key.
type        | further describe the metric.  type is a very generic word, only use it if you really don't know anything better.
result      | values: ok, fail, ... (for http requests, http_code is probably more useful)
stat        | to clarify the statistical view
bin_max     | if your metrics are separated into bins by some numeric value, upper limit of a bin (like (statsd) histograms)
direction   | in/out (not 'tx' or 'rx', more consistent)
mtype       | type of metric in terms of how the data should be interpreted. See below.
unit        | in what is the magnititude being measured.  see below
file        | file (that generated a metric)
line        | line (that generated a metric)
env         | environment


### Special tag values

Value   | Meaning
--------|-----:
`_sum_` | represents the sum of all other (would-be) metrics summed across this tag. ( equivalence)
`_avg_` | represents the avg of all other (would-be) metrics averaged across this tag. (equivalence)



### Tag values: unit

* Units in metrics 2.0 are the union of [all SI units and prefixes](http://en.wikipedia.org/wiki/International_System_of_Units) and [IEC prefixes](http://en.wikipedia.org/wiki/Binary_prefix), extended with units commonly used in IT (the "extensions").
* The extensions are designed to be intuitive (i.e. as commonly used by [strftime](http://strftime.org), however, are to never conflict with SI or IEC.
* Note that unit can be empty string for unitless data (e.g. the fraction of two series with the same unit, or a probability.  See "open question" further down)

#### Commonly used SI units

Unit  | Meaning
------|-----:
s     | second (time)
Hz    | frequency (1/s)

For the full listing, see the SI website

#### SI and IEC prefixes

* [SI decimal prefixes](http://en.wikipedia.org/wiki/SI_prefix)
* [IEC binary prefixes](http://en.wikipedia.org/wiki/Binary_prefix)

The most common ones are in the table below:

Unit  | Meaning
------|---------:
n     | nano, 10^-9
μ     | micro, 10^-6
m     | milli, 10^-3
c     | centi, 10^-2
d     | deci, 10^-1
k     | kilo, 10^3
M     | mega, 10^6
G     | giga, 10^9
T     | tera, 10^12
P     | peta, 10^15
Ki    | kibi 1024
Mi    | mebi, 1024^2
Gi    | gibi, 1024^3
Ti    | tebi, 1024^4
Pi    | pebi, 1024^5
Ei    | exbi, 1024^6

#### Extensions

Symbol  | Meaning
--------|-----:
b       | [bit](http://en.wikipedia.org/wiki/Bit#Unit_and_symbol)
B       | [byte](http://en.wikipedia.org/wiki/Bit#Unit_and_symbol)
M       | minute (strftime)
h       | hour (strftime)
d       | day (strftime) 
w       | week (strftime)
mo      | month (not 'm' like in strftime because that would be SI conflict)
err     | errors
warn    | warnings
conn    | connections
event   | events (TCP events etc)
ino     | inodes
email   | email messages
jiff    | jiffies (i.e. for cpu usage)
job     | job (as in job queue)
file    | (not 'F' that's farad)
load    | cpu load
metric  | a metric line like in the statsd or graphite protocol
msg     | message (like in message queues)
P       | probability (between 0 and 1)
page    | page (as in memory segment)
pckt    | network packet
process | process
req     | http requests, database queries, etc
sock    | sockets
thread  | thread
ticket  | upload tickets, kerberos tickets, ..

Any combination of a prefix with any of the unit is supported. I.e. kHz, MB/s, etc.  
Note that out of consistency, and for clarity 'Mb/s' should be used instead of 'Mbps', and so forth for similar network metrics. [^Mbps]

### Tag values: stat

Symbol  | Meaning
--------|-----:
min     | lowest value seen
max     | highest value seen
mean    | standard mean
std     | standard deviation
*_NUM   | the NUM percentile of the stat

### Tag values: mtype

Symbol    | Meaning
----------|-----:
rate      | a number per second (implies that unit ends on '/s')
count     | a number per a given interval (such as a statsd flushInterval)
gauge     | values at each point in time
counter   | keeps increasing over time (but might wrap/reset at some point) i.e. a gauge with the added notion of "i usually want to derive this to see the rate"
timestamp | value represents a unix timestamp. so basically a gauge or counter but we know we can also render the "age" at each point.

### Open question: representing things that alter the meaning of unit and mtype

* something in percent. do we add _pct to the unit or mtype tag? or normalized=100?
* probability, commonly expressed as P is not really a unit, a probability has no unit. How do we specify that the number is a probability or that the range is between 0 and 1 ?
* cpu load can also be considered unitless.


## Examples

### Example 1: Swift proxy-server timings

This comes from the structured_metrics toolkit which upgrades a metric from the traditional form:
```
stats.timers.dfsproxy1.proxy-server.object.GET.206.timing.upper_90
```

Into: [^target_type]
```
{
    what=response_time
    http_code=206
    http_method=GET
    host=dfsproxy1
    service=proxy-server
    stat=upper_90
    swift_type=object
    target_type=gauge
    unit=ms
}
```


### Example 2: Disk space

A hypothetical monitoring agent "diamond2" could submit native metrics 2.0 to track used disk space on a given mountpoint (file system) on a given server, like so: [^target_type]

```
{
    mountpoint=/srv/node/dfs3
    what=disk_space
    host=dfs4
    target_type=gauge
    type=used
    unit=B
}
meta: {
    agent=diamond2
}
```

A hypothetical storage system could hence use something like this as the id for the corresponding series:
```
id=mountpoint=/srv/node/dfs3,what=disk_space,host=dfs4,target_type=gauge,type=used,unit=B
```

Note that if we switch to a different agent, the id will stay the same because meta tags are not used to generate the identifier for the storage system.

[^Mbps]: Carbon-tagger and structured_metrics will set the unit to Mb/s whether the unit in the serialized key is Mbps or Mb/s.  Also Graph-Explorer does not support the Mbps form, it does support the Mb/s form.
[^target_type]: `target_type` was the old name for `mtype`, still used by tools such as structured_metrics and graph-explorer.  They should be updated. They also still use 'server' instead of 'host', and lower/upper instead of min/max.
