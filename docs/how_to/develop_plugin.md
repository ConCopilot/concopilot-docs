# Develop a Plugin

All components in ConCopilot are regarded as plugins,
and extended the `Plugin` interface.

This guide provides an overview about how to develop a plugin.

[TOC]

## Create a package

As mentioned in the [General Guide](../introduction/general_guide.md),
you can add as many component packages as you want in your project.
We will use the simplest structure as shown below in this example.

```
component_root
|-- .config
|    |-- config.yaml
|    |-- ... (other config files)
|
|-- __init__.py (expose a "constructor" method to construct your plugin)
|-- ...
```

Notes:

1. The ".config" folder.
    1. Any file in this folder will be pushed to our component repository during deployment.
    2. A "config.yaml" containing all information about this component is **REQUIRED**. A full template can be found [here](https://github.com/ConCopilot/concopilot/blob/main/config/plugin/config.yaml)
    3. A "readme.md" to describe this component is **recommended**.
    4. Do **NOT** put anything too large (like a model weights file) here.
2. The "\_\_init\_\_.py": expose a "constructor" method like below to make ConCopilot is able to construct your plugin.

```python
from typing import Dict

from package.to.your.plugin import YourPlugin


def constructor(config: Dict):
    return YourPlugin(config)


__all__=[
    'constructor'
]

```

## Implement the `Plugin` Interface & Edit the "config.yaml"

There is no specific order of these two steps.
It is more convenient to do them simultaneously since the "code" need to use the "config" to make the component work.

You should extend the `AbstractPlugin` class like below for your own plugin.

```python
from concopilot.framework.plugin import AbstractPlugin


class YourPlugin(AbstractPlugin):
    def __init__(self, config: Dict):
        super(YourPlugin, self).__init__(config)
        # ...

    def command(self, command_name: str, param: Dict, **kwargs) -> Dict:
        # ...
```

The `AbstractPlugin` class already implemented most useful methods for a plugin,
and you only need to implement the `__init__` and `command` methods in most cases.
You can also add more method to this class for special usage.

The `config` parameter in the `__init__` method contains all information from the plugin's "config.yaml".
But you don't need to use it because the `AbstractPlugin` has dealt with most jobs.
Instead, you can directly use `self.config` to access the data anywhere in the class.

The `config` section in the "config.yaml" is a good place to store parameters that are supposed to be modified in different tasks.
Because they can be overriden by outer scope such like a copilot's "config.yaml".
Use `self.config.config` to access the data.

### The `info` and `commands` sections in the "config.yaml", and the plugin `command` method

If you want your Plugin to be called by the LLM rather than called via hard coding,
you must fill both the `info` and `commands` sections, as well as implement the `command` method.

Below is the detailed description for the `info` and `commands` sections:

```yaml
group_id: <group_id>
artifact_id: <artifact_id>
version: <version>

type: plugin
as_plugin: true

info: # The basic information of the plugin. Must exist if `as_plugin` is `true`.
  title: <title> # the title of the plugin
  description: <description> # description of the plugin
  description_for_human: <description_for_human> # optional, description for human to read
  description_for_model: <description_for_model> # optional, description for LLMs to read
  prompt: <optional_prompt> # if exists, it is the instruction to prompt LLMs
  prompt_file_name: <optional_prompt_file_name> # if exists, it is the file located in the same folder of this "config.yaml" that contains the instructions to prompt LLMs
  prompt_file_path: <optional_prompt_file_path> # if exists, it is the full file path that indicate the file contains the instructions to prompt LLMs
commands: # API list that the plugin provided. Must exist if `as_plugin` is `true`.
  -
    command_name: <command_name_1> # the name of the first API
    description: <command_description_1> # the API description
    parameters: # parameters that the API need to be passed
      -
        name: <param_name_1> # the name of the first parameter
        type: string # the type of the first parameter
        description: <description_1> # the parameter description
        enum: # optional, entire possible values of this parameter
          - <enum_1>
          - <enum_2>
        required: true # if true, the parameter must be provided when calling the API, and if false, the parameter is optional.
        example: <example> # optional, an optional field gives an example of the parameter.
      -
        name: <param_name_2>
        type: string
        description: <description_2>
        required: true
        example: <example>
    response: # fields in the API response
      -
        name: <response_field_name_1> # the name of the first response field
        type: string # the type of the first response field
        description: <response_field_description_1> # the response field description.
        optional: false # if false, this response field will always be included in the response, and if true, this response field can be absent in the response.
        example: <response_field_example_1> # an optional field gives an example of this response field.
      -
        name: <response_field_name_2>
        type: string
        description: <response_field_description_2>
        optional: false
        example: <response_field_example_2>
  -
    command_name: <command_name_2>
    description: <command_description_2>
    parameters: 
      # ...
    response:
      # ...
  # ...
```

The `def command(self, command_name: str, param: Dict, **kwargs) -> Dict:` method which receives a command name string and a parameter dict and returns its response in another dict should be implemented strictly according to the `commands` section above.

The `command_name` makes it possible for a plugin to provide multiple commands,
and each command should receive its `param` and return its "response" with a format respectively according to the `parameters` and `response` sections under the item with the same `command_name` in the `commands` list.

### Using Resources

Please note that do **NOT** do any resource allocation,
including but not limited to model loading, DB connection, HTTP pools creation, memory allocation, etc., in the `__init__` method.
Instead, please [make Resource components](./develop_resource.md) to do these stuffs and add them to the `config.resources` section in your plugin's "config.yaml" like below,
and ConCopilot will do the initialization and finalization for you.

```yaml
group_id: <group_id>
artifact_id: <artifact_id>
version: <version>

# ...

config:
  # ...

  resources:
    -
      type: model
      id: default
      # name: null
      config:
        # the resource configs
    # -
    #   another resource
```

You can access your resources by using any of the `resources`, `resource_id_map`, `resource_name_map`, `resource_type_map` properties,
or the `get_resource` method.
Learn more in [Plugin Resources](../framework_docs/plugin.md#plugin-resources)

Resources created by other method may result memory leak or other negative effects due to incorrect initialization and finalization.

Please also note that those resources are not initialized in your plugins `__init__` method.
You can only use them in other parts of your class.

It is a good practice to use a property to store a frequently used resource in your plugin class like below:

```python
class YourPlugin(AbstractPlugin):
    # ...

    @property
    def your_resource(self) -> Resource:
        if self._your_resource is None:
            # cache the resource into a property
            self._your_resource=self.resources[0]
        return self._your_resource

    def any_method(self):
        self.your_resource # use the resource
```

### Fill the `setup` section in the "config.yaml"

This section controls how the plugin to be loaded in a copilot or agent.

It provides two parts of information:
1. The pip packages the plugin depends on
2. The python package path where the plugin's "\_\_init\_\_.py" with the `constructor` method exists.

```yaml
setup:
  pip: # put all your pip dependencies here
    - <one_dependency>
    - <another_dependency>
    # - ...
  package: package.path.that.contains.the.__init__.py
```

Please make sure the package path to the "\_\_init\_\_.py" is consistence between local test and release.

### Create another component in a plugin

Sometimes people may need to manually create a component by code.
Just use the `create_component` method.

```python
from concopilot.util.initializer import component

plugin=component.create_component({
    'group_id': '<group_id>',
    'artifact_id': '<artifact_id>',
    'version': '<version>'
})
```

If their is a section in your plugin's config contains the sub-components information,
you can just pass it to the `create_component` method.
For example, if the "config.yaml" contains this piece of data:

```yaml
group_id: <group_id>
artifact_id: <artifact_id>
version: <version>

# ...

config:
  # ...

  a_component:
    group_id: <a_component_group_id>
    artifact_id: <a_component_artifact_id>
    version: <a_component_version>
    # ...
```

You can directly pass this information to the `create_component` method like below:

```python
from concopilot.framework.plugin import AbstractPlugin
from concopilot.util.initializer import component


class YourPlugin(AbstractPlugin):
    def __init__(self, config: Dict):
        super(YourPlugin, self).__init__(config)
        a_component=component.create_component(self.config.config.a_component)
        # ...

    def command(self, command_name: str, param: Dict, **kwargs) -> Dict:
        # ...
```

## Test your plugin

### In-project test

### Test in Local Repository

### Snapshot Deployment

## Release your plugin

### Release Deployment

