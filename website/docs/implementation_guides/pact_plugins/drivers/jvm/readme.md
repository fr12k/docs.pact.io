---
title: Pact plugin driver library for the JVM
custom_edit_url: https://github.com/pact-foundation/pact-plugins/edit/main/drivers/jvm/README.md
---
<!-- This file has been synced from the pact-foundation/pact-plugins repository. Please do not edit it directly. The URL of the source file can be found in the custom_edit_url value above -->

## Source Code

https://github.com/pact-foundation/pact-plugins/tree/main/drivers

> Pact support library that provides an interface for interacting with Pact plugins

## State of implementation

* [X] The ability to find plugins.
* [X] Load plugins and extract the plugin manifests that describe what the plugin provides.
* [X] Provide a catalogue of features provided by the plugins.
* [ ] Provide a messaging bus to facilitate communication between the language implementation and the plugins.
* [X] Manage the plugin lifecycles.

## Building the JVM driver

The JVM driver is built with Gradle. The build can be run with `./gradlew build`, but there is a test `DriverPactTest`
that requires a Protobuf plugin to work. Either skip or disable that test, or install the prototype Protobuf plugin before
running the build.
