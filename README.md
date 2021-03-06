# Fluent::Plugin::AutoTypecastFilter [![CircleCI](https://circleci.com/gh/s5o-c/fluent-plugin-auto-typecast-filter/tree/master.svg?style=svg)](https://circleci.com/gh/s5o-c/fluent-plugin-auto-typecast-filter/tree/master) [![Gem Version](https://badge.fury.io/rb/fluent-plugin-auto-typecast-filter.svg)](https://badge.fury.io/rb/fluent-plugin-auto-typecast-filter)

This plug-in helps to automatically cast and retransmit structured log data flowing through the Fluent network.

## Tested dependencies

* fluentd v1.7.0

## Installation

```sh
gem install fluent-plugin-auto-typecast-filter
```

or

```sh
# for `fluent-gem` users:
fluent-gem install fluent-plugin-auto-typecast-filter
```

## Usage

Basic usage in fluentd.conf:

```
<filter any.tag.**>
    @type auto_typecast
</filter>
```

## Parameters

| Parameter | Description | Type | Default |
---|---|---|---
| `maxdepth` | Maximum level of nesting that is autocast. If **`0`** is specified, all nests are targeted. | Integer | `1` |
| `ignore_key_regexp` | Regular expression to determine keys that are not subject to processing. | RegExp | `nil` |

## Examples

**Case 1:**

```plain
# CONFIG:
maxdepth 0

# INPUT:
{ 'k': { 'k': { 'k': [ { 'k': {
    'k0': 'string',
    'k1': '20',
    'k2': '1.0',
    'k3': 'NULL',
    'k4': 'true'
} } ] } } }

# FILTERED OUTPUT:
{ 'k': { 'k': { 'k': [ { 'k': {
    'k0': 'string',
    'k1': 20,
    'k2': 1.0,
    'k3': null,
    'k4': true
} } ] } } }
```

**Case 2:**

```plain
# CONFIG:
maxdepth 3

# INPUT:
{ 'k' : {
    'k0': [ '10', '20', '30' ],
    'k1': { 'k': [ '10', '20', '30' ] }
}

# FILTERED OUTPUT:
{ 'k' : {
    'k0': [ 10, 20, 30 ]
    'k1': { 'k': [ '10', '20', '30' ] }
}
```

**Case 3:**

```plain
# CONFIG:
maxdepth 0
ignore_key_regexp /^0$/

# INPUT:
{ 'k': [
    [ '10', '20' ], [
    [ '100', '200' ], [
    [ '1000', '2000' ]
] ] ] }

# FILTERED OUTPUT:
{ 'k': [
    [ '10', 20 ], [
    [ '100', 200 ], [
    [ '1000', 2000 ]
] ] ] }
```

**Case 4:**

```plain
# CONFIG:
maxdepth 0
ignore_key_regexp /^[a-z]{1}[0-9]{1,}1$/

# INPUT:
{ 'k0001': [ {
    'k0000': 'true',
    'k0002': {
    'j0001': 'null'
} } ] }

# FILTERED OUTPUT:
{ 'k0001': [ {
    'k0000': true,
    'k0002': {
    'j0001': 'null'
} } ] }
```

## Copyright

* Copyright (c) 2019- Shotaro Chiba
* Apache License, Version 2.0
