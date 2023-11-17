# Basic Concepts

**_Based on Plugins, Driven by Messages_**

We are trying to define common interfaces for Copilots,
aiming to making "_functional parts_" **reusable**, **replaceable**, **portable**, and **flexible**,
so that people can easily build new copilots for their specific tasks with the help of predefined common components,
just like import predefined packages during coding.

Our vision is to leverage all potential tasks and benefit everyone from the development of AI & LLM.

All data structures and configs in ConCopilot is literally a python `dict` for maximizing the extensibility.
In practice, we use the `ClassDict` which extends the python `dict` in our code.
This `ClassDict` class does nothing but makes it possible to access dict fields in the class literal,
which is more convenient in some situation.
For example, `class_dict['key']` is equivalent to `class_dict.key`.

[TOC]

## Copilot/Agent

We define a Copilot as a specific LLM based worker that can automatically accomplish tasks in some area.

This is a very **GENERAL** concept so that a specific copilot can be literally anything that can accomplish one or more tasks described below (not limited):

1. analyse information
2. coordinate tasks and functions
3. do planning, execute commands, and accomplish goals
4. interact with human

Under this definition, everything that helps human works can be a copilot.
GitHub Copilot, Office Copilot, AI Assistant like Siri, and even devices like robots and drones can be copilots.

The concept of Agent is very similar to the concept of Copilot,
they are mixed-used in most cases and the community uses Agent more.

In our view, we think "Copilot" emphasize more on the application and business,
while "Agent" emphasize more about the LLM.
This is the reason we use "Copilot" more than "Agent".
ConCopilot focus more on how to help develop a business/task based application.
But it is OK for you to mix-using these two concepts.

## Component

In practice, a component is a module that can be uniquely defined by a "group_id", "artifact_id", and "version" tuple.

This is a virtual logical concept that indicates any module that can be auto-loaded by the ConCopilot build tool.

Generally, its meaning is similar to "Plugin" (since everything is a Plugin),
but it is more about an engineering concept rather than a ConCopilot framework concept.
Its specific meaning might vary slightly according to different contexts.

### Group Id and Artifact Id

We use the same concepts of these two identifiers as Maven.

A "group_id" identifies an organization, a company, a community, or a person.
It is typically composed by a reversed domain owned by a group.
It separates components developed by different groups to eliminate the component conflict between groups.
Developers should apply access to their group ids.
See [here](../build_&_deploy/group_id.md) for details.

An "artifact_id" is the name of a component.
It usually represents the functionality of the component, and can be any arbitrary string.
An "artifact_id" uniquely identifies a component in a group (under a "group_id").
So please make sure different projects in the same group use different "artifact_id"s.

### About the Version

The format of version is `(\d+).(\d+).(\d+)(-any-string)?(-SNAPSHOT)?`, e.g., "1.0.21-linux-SNAPSHOT", "2.3.4".

The first 3 digits parts are common typical version number,
while the tailing "-SNAPSHOT" means it will be deployed to the snapshot repository during deployment for component testing.

We provide two repositories, a snapshot one, and a release one, just like the Maven.
Components without the "-SNAPSHOT" identifier will be deployed into the release repository, and are not allowed to be modified or redeployed.

If you want to test your component in a full stack, you can add the "-SNAPSHOT" identifier at the tail of your component version,
so that it will be deployed to the snapshot repository, and you can redeploy it later after some modifications.

Please note that components deployed into the snapshot repository will be removed regularly,
thus make sure to remove the "-SNAPSHOT" identifier after testing and deploy it again to the release repository.

You can add any string after the version number to convey any information you like.
But make sure the "-SNAPSHOT" identifier, if exists, always appears at the end of the version.

## Plugin

This is one of the most important concept in ConCopilot, because ConCopilot is **_Based on Plugins_**.

A plugin is a module that can work on a group of related specific tasks in an area of expertise.

All plugins should extend the `AbstractPlugin` and implement

1. the `__init__` method which receives only one `dict` parameter containing its configuration.
2. the `command` method which receives a command name `string` and a parameter `dict` and returns its response in a `dict`.

We use Plugin to encapsulate a group of reusable functions, methods, or services that some specific tasks relies on.

Thus, on the other hand, a copilot can choose its plugins for specific tasks.
This mechanism makes it easier to build and deploy a copilot,
transfer functions from one copilot to others,
and improve the developing efficiency of LLM related projects.

To maximize the flexibility and reusability, Copilots are also regarded as Plugins,
so that it is very natural to make a copilot to call and coordinate other copilots as a hierarchical structure.

There are several _Special Plugins_ that are defined as special roles in a Copilot,
people may not want to make them as common Plugins for others to call, since this introduce extra works.
We left this choice to developers by exposing an `as_plugin` config fields in the "config.yaml" of the components.
Anyway, we encourage developers to make everything a real Plugin if permitted, even they are special plugins.

Please Note that setting the `as_plugin` to `false` to a simple plugin (not the special ones) will make the plugin useless because nothing can really call it.

### Special Plugins

Special Plugins are defined to deal with information pipelines and task coordinating in a Copilot.
Generally, they will not be called as common Plugins by the Copilot,
but by the predefined working flow instead, just like traditionally coding.

We encourage developers to make their special plugins a real Plugin if you find them a possibility to be part of a working flow of other copilots.

#### Cerebrum

A Cerebrum is the brain of a copilot, since it interact user requirements and plugins with an LLM,
just like a human brain.

Cerebrums are also responsible to isolate LLM to other parts of a copilot,
making it easily to change LLMs without side effects.

#### Interactor

An Interactor is the core of a copilot,
since it controls the information dispatching, task coordinating, and function calls in a copilot.
It defines and controls the main working pipeline of specific copilots.
All task specific logics should be defined in the interactor.

During developing, generally,
there's no need to implement `Copilot` interface for a new copilot, implement an `Interactor` interface instead
(extend the `BasicInteractor` class).
This makes the Interactor interface more important than the CoPilot interface in practice.

#### User Interface

This is defined as the common interface to deal with interactions with user. 

#### Message Manager

This is defined to translate the LLM natural language text from the Cerebrum response (`InteractResponse` type) into structured data (generally a `Dict`),
and encapsulate it with other information in the `InteractResponse` response into a `Message` for the main task pipeline.

The most popular form of this is to deserialize LLM json string into python `dict`.

Choose different message manager for different LLM response format (json, yaml, or others).
The LLM response format is generally depended on prompt, not on tasks or LLMs.

#### Plugin Manager

A Plugin Manager is created to manager all plugins in a copilot,
including adding, holding, indexing, initializing plugins, and generate prompts for those plugins if necessary.

##### Plugin Prompt Generator

A Plugin Prompt Generator will be called by a Plugin Manager to generate prompts for plugins that the copilot may use.
This is necessary because in many cases prompts varying with LLMs, but working flow remains unchanged.

#### Resource Manager

A Resource Manager is defined to manage and maintain [Resources](#resource),
including adding and holding resources, as well as initialize and finalize resources.

Any specific resource can only be used by plugins after being initialized in a resource manager.

#### Storage

This is created as a long term memory of a copilot,
as well as storing other kind of materials (in the form of [Asset](#asset)) for a specific working flow.

A storage can be designed basing on any kind of storage system [resources](#resource), depending on the specific implementation,
like hard disk, memory, redis, etc.

## Message

Message is another most important concept in ConCopilot, as ConCopilot is **_Driven by Messages_**.

Messages is the medium of interaction in a copilot.
They contain and transfer information and even some parts of working data between plugins, LLMs, users, and the copilot.

Each message contains

1. the "sender" who sent the message out.
2. the "receiver" who is going to receive the message.
3. the "content" that contains what to be transferred, specifically, the command information for the receiver plugin to execute.
4. the "time" that the message happened.

## Resource

A Resource is an infrastructure, hardware, software, or service that some plugins rely on.
Such as a disk, a memory, a model, a DB connection, a service, or anything else that needs to be "initialized" before used by others,
and/or needs to be "released" or "finalized" before the copilot exits.

Plugins in the same copilot can share the resources, e.g., share the LLM with the cerebrum,
or use their dedicated resource, e.g., two plugins respectively rely on two different DB connection.
This can be configured in the copilot's (or the plugins') `config.yaml`.

## Context

The Context of a copilot contains the global endpoints of the copilot. Currently including

1. the storage for the long term memory and data persistence of the copilot.
2. the assets that contains any other data for the copilot's task.
3. the user interface.

### Asset

The Asset is defined to contain any kind of copilot task data.
This data will be store in the memory without persistence.
Any plugin can fully access those assets by access the context.

An asset mainly contains these parts:

1. type: an arbitrary string represents the asset type.
2. id: the id of the asset.
3. name: the name of the asset.
4. description: the description of the asset.
5. content: any arbitrary string represents the content of this asset.
6. data: any arbitrary data this asset contains.
