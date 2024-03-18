# General Guide

The purpose of this guide is to provide a general philosophy about the design idea to developers and users.
We wish these ideas can help developers make more general components that can be used more conveniently and comfortably by end users.

Our vision is always that to make each part of a copilot **reusable**, **replaceable**, **portable**, and **flexible**,
and benefit everyone from the development of AI & LLM.
Thus, we wish developers can follow this guide during their development, and leave more freedoms to end users.

Contributions to our framework will be restricted under the idea of **reusable**, **replaceable**, **portable**, and **flexible**.
We regard these ideas as the first priority, more important than framework features.
We also do **NOT** want our framework become too complicated to developers and end users.
Although we encourage contributors to contribute ideas,
we also highly recommend contributors take more discussions with each other and with developers and users to make sure that
_**a)** the change is really necessary and cannot be substituted by features originally existed_, and
_**b)** it is simple enough and easy to use_.

On the other hand, developers are more free to make their components.
Developers basically can do whatever they want to make their components,
only need to follow some necessary constraints (see [Contracts](#contracts) below).
The purpose of these constrains is also to "make each part of a copilot **reusable**, **replaceable**, **portable**, and **flexible**",
so that end users can be benefited more.

As a result, end users are totally free to deal with their config files like `config.yaml`.
Anything could be changed to run a copilot.
Try to find something different by adding plugins, change prompts, try different models, select resources like DB, etc.

Finally, please feel free to post issues to our [git](https://github.com/ConCopilot/concopilot) if you have any questions or come up with any ideas.

Enjoy.

[TOC]

## Contracts

Generally, developers are free on how to use this framework, but we recommend developers to follow the constraints and contracts in this section.

These constraints and contracts are necessary to make things easy to use and make plugins **reusable**, **replaceable**, **portable**, and **flexible**.

We are also trying to make things simple, so feel relax to check them.

### For LLM

`LLM` is a type of resources that handle the connection and calling of Large Language Model.
They are mainly used by a `Cerebrum` instance to provide central coordinating and task management in a copilot.
But they can also be used by other plugins if they provide any method backed on an LLM.

Thus, we use `LLM.LLMParameter` and `LLM.LLMResponse` to represent the LLM input and LLM output data structures, respectively.
They are both basically `ClassDict` object with their predefined fields shown below:

```yaml
LLMParameter:
  prompt: the input prompt string
  max_tokens: the max tokens
  temperature: temperature
  require_token_len: require the LLM return input and output token length if true
  require_cost: require the LLM return the calling cost if true
```

```yaml
LLMResponse:
  content: the output result string (Required)
  input_token_len: the input token length
  output_token_len: the output token length
  cost: the calling cost
```

Most of the fields described above are optional, but we require developers to implement the input `prompt` and output `content` fields.
In other word, make the `LLM` functions well when it received a `prompt` and return a `content`.

Please note that this requirement does **NOT** mean the input always contains a `prompt` field
(e.g. for OpenAI GPTs, usually a `message` rather than a `prompt`),
instead, it means that **"it must run correctly when the `LLM` received a `prompt`"**.
This is necessary for any arbitrary plugin to call an `LLM` without knowing what it backed on.
Meanwhile, the requirement **DOES** mean that an `LLM` should always return a `content` with string type.

As both the data structures are backed on `ClassDict`, any extra parameters can be added and returned.

### For Cerebrum

A cerebrum takes the responsibility to translate current stage,
including but not limited to message history, long/short term memory, file, etc.,
to what an LLM can understand, and then returns the LLM output.

Thus, the contract for Cerebrums is that **"do NOT expose any LLM related stuff out of its own scope"**,
so that a copilot can easily substitute a cerebrum with another without knowing the detail of LLM.

A copilot's interactor will call its cerebrum by the `interact` method, providing below information:

```yaml
InteractParameter:
  instructions: a list of strings contains system instruction prompts
  command: a prompt string represents the current command
  message_history: a list of Messages represents the interacting history
  assets: a list of Assets which can be anything that is necessary to current round of interaction
  content: a content string contains extra information

  require_token_len: require the LLM return input and output token length if true
  require_cost: require the LLM return the calling cost if true
```

None the information fields above is required,
and it is the cerebrum's duty to decide how to use those information.

Then the cerebrum's `interact` method should return below information to the copilot's interactor:

```yaml
InteractResponse:
  content: the output result string (Required)
  plugin_call: a PluginCall object if the LLM has an ability to directly return a function call (like OpenAI Function Call)
  input_token_len: the input token length
  output_token_len: the output token length
  cost: the calling cost
```

### For Message creation

Messages are used to transfer information and data between copilot, plugins, and users.
It contains several predefined fields:

```yaml
Message:
  sender: # the message sender
    role: the message sender role
    id: the message sender id
    name: the message sender name
  receiver: # the message receiver
    role: the message receiver role
    id: the message receiver id
    name: the message receiver name
  content_type: the content type of the message content
  content: the message content
  time: a string indicates the message time
  id: an optional id of this message
  thrd_id: an optional id for a message thread that this message belongs to.
```

The meaning of the message content is natural: the `sender` sent the `content` within the format of `content_type` to the `receiver` at the `time`.

We want components can understand messages sent to them without considering the sender or the current task.
Thus, the contracts below are necessary:

1. for `message.sender.role` and `message.receiver.role`
    1. must be "cerebrum" if the message sender/receiver is the cerebrum
    2. must be "plugin" if the message sender/receiver is a plugin
    3. must be "user" if the message sender/receiver is the user
    4. must be "system" if the message sender/receiver is the "system"
        (We use "system" role to transfer system messages such as "error" and "exit")
2. for `content_type`
    1. use Multipurpose Internet Mail Extensions type (MIME type) for common serialized `content` (e.g.: text/markdown)
    2. set to "command" if the `content` is a plugin call command or response
    3. set to `content` Python object type string for a common Python object, (e.g.: "<class 'dict'>")
3. for `message.content.command`, `message.content.param`, and `message.content.response`
   These three fields are reserved for plugin calls.
   `command` is the plugin command name, `param` is the related parameters, and `response` is the response.
   Set the `content_type` to "command" in this case.

All of the above strings are case-sensitive.

For example, when the message is delivering a plugin call command or plugin call response,
the `content_type` and `content` should look like below:

```yaml
Message:
  content_type: command
  content:
    command: <plugin command, exists in both calling and response>
    param: <plugin call parameter, exists only in calling a command>
    response: <plugin call response, exists only in a command response>
```

### For "config.yaml"

The `__init__` method of all the components accept a `config` dict (`ClassDict`) as their first parameter,
and this `config` is mainly directly read from the component's "config.yaml".

We use the Maven format for our component package management,
if you are familiar with Maven, that will be easy to understand.
if you are not familiar with Maven, don't worry, just remember each (`group_id`, `artifact_id`, `version`) tuple uniquely define a component.

Below are the core fields of a "config.yaml" file:

```yaml
group_id: <group_id> # required, the component group id
artifact_id: <artifact_id> # required, the component artifact id
version: <version> # required, the component version

type: plugin # required, the component type
as_plugin: true # required, if the component can be called as a plugin

info:
  # optional, plugin information, only required if `as_plugin` is true
commands:
  # optional, plugin command list, only required if `as_plugin` is true

setup:
  pip:
    # optional, python dependents list, if any

  package: <python_package> # required, the full package path where the plugin package __init__.py exist,
                            # an absence of this may result your component not able to be loaded.

url: <project_url> # optional but recommended

developers:
  # optional but recommended, developer list

licenses:
  # optional but recommended, license list

config:
  # optional, plugin config
```

Templates of the "config.yaml" files for each component can be found in out [git](https://github.com/ConCopilot/concopilot/tree/v0.0.3/config).
We only give the constraints here.

1. The `type` field indicates the plugin type which can be any arbitrary string.
   In most cases, just fill it to "plugin" if you are developing a plugin without any additional function.
   Each _Special Plugin_ has its own `type`, see the [Framework Docs](../framework_docs/index.md) for detail.
2. The `as_plugin` must be `true` if you are developing a plugin for others to use.
   Only set it to `false` if you are developing _Special Plugins_ which have their specific roles in a copilot and not necessary to be called as plugins.
   We encourage developers to set it to `true` even if you are developing _Special Plugins_,
   as long as you think your _Special Plugins_ can act as plugins in some other situation.
   Don't forget to fill the `info` and `commands` fields in the "config.yaml" and implement the plugin's `command` method in this case.

#### The `config` field in "config.yaml"

The `config` field in any "config.yaml" is special.
Generally, you can add any data in your "config.yaml", as long as your component `__init__` can understand,
but data under the `config` field can be overridden by outer scope during the copilot initialization while data out of the `config` field cannot.
This is convenient for users to modify component configs directly in the copilot's "config.yaml" like below:

```yaml
# copilot config.yaml

# ...

config:
  interactor:
    group_id: <interactor_group_id>
    artifact_id: <interactor_artifact_id>
    version: <interactor_version>

    # make sure this `config` section is at the same level of the `group_id`, `artifact_id`, and `version`
    config: 
      key1: <value_in_copilot> # override the "<value_in_interactor>" in the interactor config.yaml

# ...
```

```yaml
# interactor config.yaml

group_id: <interactor_group_id>
artifact_id: <interactor_artifact_id>
version: <interactor_version>

# ...

# also make sure this `config` section is at the same level of the `group_id`, `artifact_id`, and `version`
config:
  key1: <value_in_interactor>

# ...
```

The final value of the `key1` in the interactor will be `<value_in_copilot>`.

## Component Package Structure

Below is the simplest package structure of a component.
All files in the ".config" directory should be pushed to our component repository,
while other python codes should be deployed to python repository like PyPi.
Don't forget to fill the `setup.package` in the "config.yaml" with the full package path of the `__init__.py`

```
component_root
|-- .config
|    |-- config.yaml
|    |-- ... (other config files)
|
|-- __init__.py (expose a "constructor" method to construct your plugin)
|-- ...
```

Below is a more general package structure:

```
project_root
|
|-- component_root_1
|    |
|    |-- .config
|    |    |-- config.yaml
|    |    |-- ... (other config files)
|    |
|    |-- __init__.py (expose a "constructor" method to construct your plugin)
|    |-- ...
|
|-- ...
|    |
|    |-- component_root_2
|    |    |-- .config
|    |    |    |-- config.yaml
|    |    |    |-- ... (other config files)
|    |    |
|    |    |-- __init__.py (expose a "constructor" method to construct your plugin)
|    |    |-- ...
|    |
|    |-- ...
|
|-- ...
```

This means you can build as many components as you need in your project,
so that you can upload more than one component with only one python package deployment.

In this case, execute the `conpack deploy` command in your "project_root" with the `recursive` flag on.

```shell
conpack deploy
   --repo-user-name=<your_repository_user_name>
   --repo-user-pwd=<your_repository_password>
   --gpg-passphrase=<your_gpg_passphrase>
   --recursive
   # --src-folder=<your_project_root> # default to current directory if absent
```

## Copilot vs. Interactor

As mentioned in the [Concepts](concepts.md) section,
a Copilot is the final APP while its Interactor take the responsibility of data and information communication, task coordinating, and workflow control.

We provided a general `BasicCopilot` class that implemented all necessary component, framework, structure, and initialization method in the `Copilot` interface.
Thus, in most cases, developers only need to rewrite an Interactor to implement their own special tasks,
since all important task specific logics are controlled by the interactor.
This will be more convenient in practices,
but don't forget to modify the default copilot's "config.yaml" with your own interactor.

However, you are also welcome to code your own Copilot class if you find the default one cannot satisfy your requirement.

## About Plugin

Developers can build a plugin by extent the AbstractPlugin class like below:

```python
from concopilot.framework.plugin import AbstractPlugin


class YourPlugin(AbstractPlugin):
    def __init__(self, config: Dict):
        super(YourPlugin, self).__init__(config)
        # ...

    def command(self, command_name: str, param: Any, **kwargs) -> Any:
        # ...
```

Remember to implement its `command` method which receives a command name string and a parameter (recommend to use a Python `dict`) and returns its response (also recommend to use a Python `dict`).

The `command_name` makes it possible for a plugin to provide multiple commands,
such as a disk plugin provides both the "read" and "write" command.

### The `info` and `commands` sections in the "config.yaml"

These two sections are required for a Plugin's "config.yaml".
Below is the detailed description for them:

```yaml
group_id: <group_id>
artifact_id: <artifact_id>
version: <version>

type: plugin
as_plugin: true

info: # The basic information of the plugin. This section must exist if `as_plugin` is `true`.
  title: <title> # The title of the plugin
  description: <description> # The description of the plugin
  description_for_human: <description_for_human> # Optional. Description for human readers
  description_for_model: <description_for_model> # Optional. Description for Large Language Models (LLMs) to read
  prompt: <optional_prompt> # If exists, this is the instruction to prompt LLMs
  prompt_file_name: <optional_prompt_file_name> # If exists, this is the name of the file located in the same folder as the "config.yaml" file that contains the instructions to prompt LLMs
  prompt_file_path: <optional_prompt_file_path> # If exists, this is the full file path that indicates the file contains the instructions to prompt LLMs
commands: # The list of APIs provided by the plugin. This section must exist if `as_plugin` is `true`.
  -
    command_name: <command_name_1> # The name of the first API
    description: <command_description_1> # The description of the API
    parameter: # The `parameter` required by the API
      type: # The type of the `parameter`, recommend to use a Python `Dict`
            # Use the YAML mapping/object syntax to describe the `Dict` items.
            # The YAML mapping/object syntax indicates that the type is a `Dict`,
            # and each item in the YAML mapping/object describes one key-value pair in the `Dict`.
            # The keys in the `Dict` are always string type, and their names are described as the YAML mapping/object keys.
            # The type of the value for each key is described under that key using the parameter type description syntax.
        <param_name_1>: # The name of the first parameter
          type: string # The type of the first parameter
          description: <description_1> # The description of this parameter
          enum: # Optional. The entire possible values of this parameter
            - <enum_1>
            - <enum_2>
          required: true # If true, the parameter must be provided when calling the API. If false, the parameter is optional. Defaults to true.
          asset_ref_acceptable: false # If true, an `AssetRef` object (or URL) representing an `Asset` field with the acceptable data type can be passed instead of the real parameter object.
                                      # The plugin will read the corresponding asset from the Copilot context in this case.
                                      # If false, only the real parameter object is acceptable. No `AssetRef` should be passed to this parameter.
                                      # This field defaults to false if not provided.
          example: <example> # Optional. An example of the parameter.
        <param_name_2>:
          type: List # Set the type to `List` if the parameter is a list type and the list elements are simple.
                     # It is better to attach the element type, e.g., `List[int]`.
          description: <description_2>
          required: true
          example: <example>
        <param_name_3>:
          type: # Use the YAML list syntax to describe the list element object when the list element type is complex.
                # The YAML list syntax indicates that the type is a `List`,
                # and each element in the YAML list describes the key name and value type of one field in the list element object using the parameter type description syntax.
                # Note the additional YAML `name` field that describes the key name.
            -
              name: <element_field_name1>
              type: int
              description: <element_field_description_1>
              required: true
              example: <example>
            -
              name: <element_field_name2>
              type: string
              description: <element_field_description_2>
              required: true
              example: <example>
          description: <description_3>
          required: true
          example: <example>
        <param_name_4>:
          type: Dict # Set the type to `Dict` or `Mapping` if the keys and values are simple.
                     # It is better to attach the key and value types, e.g., `Dict[str, int]`.
                     # Please note that the `Dict` keys should always be in string type.
          description: <description_4>
          required: true
          example: <example>
        <param_name_5>:
          type: # Use the YAML mapping/object syntax to describe the `Dict` items.
                # Just like the type description of the `parameter` field
            _type_ref: <type_ref_name> # Optional. A complex type can have a type reference that other parts of the YAML file can refer to for the entire type description.
            <key_name_1>:
              type: int
              description: <value_description_1>
              required: true
              example: <example>
            <key_name_2>:
              type: string
              description: <value_description_2>
              required: true
              example: <example>
          description: <description_5>
          required: true
          example: <example>
        <param_name_6>:
          type: <type_ref_name> # Use the value of the `_type_ref` defined above to indicate that this parameter has the same type as the above parameter
          description: <description_6>
          required: false
          example: <example>
      description: <description_1> # An optional description.
      required: true # An optional `required` field. Can be omitted because it defaults to true.
      asset_ref_acceptable: false # This field can also appear here to indicate that an asset reference can be passed to represent the entire input parameter dictionary.
    response: # The API response.
      type:
        <response_field_name_1>: # The name of the first response field
          type: string # The type of the first response field
          description: <response_field_description_1> # The description of the response field.
          optional: false # If false, this response field will always be included in the response. If true, this response field can be absent in the response. Defaults to false.
          example: <response_field_example_1> # Optional. An example of this response field.
        <response_field_name_2>:
          type: string
          description: <response_field_description_2>
          optional: false
          example: <response_field_example_2>
  -
    command_name: <command_name_2>
    description: <command_description_2>
    parameter:
      type: string # Can also be a simple type.
      # ...
    response:
      # ...
  # ...
```

### Resources for components to use

Remember do **NOT** make any resource acquire or release in a component, such as a DB connection or an HTTP connection pool,
do this in `Resource` class, and add the resource config into your component's `config.yaml` like below
so that the Copilot framework can deal with the resource initialization and finalization for you
as well as sharing resources between plugins:

```yaml
# in the plugin's config.yaml

group_id: <interactor_group_id>
artifact_id: <interactor_artifact_id>
version: <interactor_version>

# ...

config:
  resources:
    -
      type: <resource_type>
      id: default
      # name: null
      config:
        # the resource configs
    # -
    #   another resource
```

Users can access those resources by the plugin's `get_resource` method, or `resources`, `resource_id_map`, `resource_name_map`, and `resource_type_map` attributes.

See details in the [Plugin Docs](../framework_docs/plugin.md) and [Resource Docs](../framework_docs/resource.md).

## Interfaces

ConCopilot defines interfaces for each kind of components.
The most fundamental one is the Plugin interface.

Interfaces of _Special Plugins_ extend the Plugin interface,
and are very similar to the Plugin interface with some special methods added.

_Special Plugins_ "config.yaml" are also slightly different to the Plugin's by the value of the `type` and `as_plugin` fields,
and the `info` and `commands` sections, depending on what component it is.

We provided template "config.yaml" for each ConCopilot predefined components,
see the [ConCopilot Component Config Templates](https://github.com/ConCopilot/concopilot/tree/v0.0.3/config) on our GitHub.

See also the [Framework Docs](../framework_docs/index.md) for details.

## The running directory

A running directory must be specified when run a copilot.
Users should pay attention that all components can only access files under this directory for security reason
(We are planning to make this as a mandatory restriction in the future).

Users can pass a `--working-directory` (which default to ".") to the `conpack` cli command like below:

```shell
conpack run
        --working-directory=<your_working_directory>
        ...
```

We also recommend to restrict components from access folders starts with a "." under the working directory,
reserve them only for special usage only,
such as the ".runtime" folder.

### The ".runtime" folder

ConCopilot will copy all related components config to the working directory for users to modify.
This can protect the local repository and also easier for component reusable.

By default, ConCopilot regards components with the same (`group_id`, `artifact_id`, `version`) tuple sharing the same config files.
Users can also create multiple instances with different config files by adding an `instance_id` field under the component section in any of its outer `config.yaml` that refers it:

```yaml
# in the copilot's config.yaml

group_id: <group_id>
artifact_id: <artifact_id>
version: <version>

# ...

config:
  resource_manager:
    group_id: <resource_manager_group_id>
    artifact_id: <resource_manager_artifact_id>
    version: <version>
    config:
      resources:
        -
          group_id: <a_specific_mysql_group_id>
          artifact_id: <a_specific_mysql_artifact_id>
          version: <a_specific_mysql_version>
          instance_id: 0
        -
          group_id: <a_specific_mysql_group_id>
          artifact_id: <a_specific_mysql_artifact_id>
          version: <a_specific_mysql_version>
          instance_id: <some_string>
```

Make sure the `instance_id` is at the same level of the `group_id`, `artifact_id`, and `version`.

ConCopilot will make each instance a dedicated folder for its config files at
`<working_directory>/.runtime/path/to/the/group_id/artifact_id/version/instance_id`.

Modify each instance's config folder allows reusable of the same component logics but different configs,
such as different DB connections.
