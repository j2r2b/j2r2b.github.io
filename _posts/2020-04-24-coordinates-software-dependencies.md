---
title: "Software dependencies coordinates"
---

# Software dependencies coordinates

I need create the list of our software dependencies.

If this task is easy when you are using a specific build tool (npm, maven, gradle, ...) because they provide a way to identify libraries in the remote repository. I was looking for a coordinate system that works with several repositories (Maven Central, Npm, CocoaPods, GitHub, …)

The [ClearlyDefined](https://clearlydefined.io/) project has proposed something based on 5 attributes:

* type
* provider
* namespace
* name
* revision
	
Their idea is to map the coordinates of other repositories to those values.

## Maven Central:

* **type** : `"maven"`
* **provider** : `"mavencentral"`
* **namespace** : `<groupId>`
* **name** : `<artifactId>`
* **revision** : `<version>`

## NPM:

* **type** : `"npm"`
* **provider** : `"npmjs"`
* **namespace** : `"-"`
* **name** : `<id>`
* **revision** : `<version>`


## CocoaPods:

* **type** : `"pod"`
* **provider** : `"cocoapods"`
* **namespace** : `"-"`
* **name** : `<id>`
* **revision** : `<version>`

## GitHub:

* **type** : `"git"`
* **provider** : `"github"`
* **namespace** : `<owner-name>`
* **name** : `<repository-name>`
* **revision** : `<commit-sha>`
