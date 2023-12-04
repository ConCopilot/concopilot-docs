# Develop a Plugin

All components in ConCopilot are regarded as plugins,
and extended the `Plugin` interface.

This guide provides an overview about how to develop a plugin.

[TOC]

## Examples

You can refer [these examples](https://github.com/ConCopilot/concopilot/blob/main/concopilot_examples/plugin) during developing.
This folder contains several developed plugins, click into sub-folders for details.

## Create a package

As mentioned in the [General Guide](../introduction/general_guide.md#component-package-structure),
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
    2. A "config.yaml" containing all information about this component is **REQUIRED**. A full template can be found [here](https://github.com/ConCopilot/concopilot/blob/main/config/plugin/config.yaml).
    3. A "readme.md" to describe this component is **recommended**.
    4. Do **NOT** put anything too large (like a model weights file) here.
2. The "\_\_init\_\_.py": expose a "constructor" method like below to make ConCopilot able to construct your plugin.

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
Because they can be overriden by outer scopes such like a copilot's "config.yaml".
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

Please note that do **NOT** do any resource allocation in the `__init__` method,
including but not limited to model loading, DB connection, HTTP pools creation, memory allocation, etc.
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

This mechanism has a great advantage for cross-component resources sharing.
For example, some plugins may want to use the LLM that mainly initialized for the cerebrum.
Another example is that one HTTP client can be shared for all plugins that need to request URLs.
Learn more about the resource initialization life cycle from the [Copilot Framework execution steps](../framework_docs/copilot.md#framework).
You can also code your own copilot to modify these steps if the general-purposed one cannot match your work.

You can access your resources by using any of the `resources`, `resource_id_map`, `resource_name_map`, `resource_type_map` properties,
or the `get_resource` method in your plugin scope.
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

This section controls how the plugin to be loaded into a copilot or agent.

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

We recommend to use the `ClassDict` to create mappings in ConCopilot,
as shown below,
because it allows the class style field accessing (`obj.attr`).

```python
from concopilot.util.initializer import component
from concopilot.util import ClassDict

plugin=component.create_component(ClassDict(
    group_id='<group_id>',
    artifact_id='<artifact_id>',
    version='<version>'
))
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

    # ...
```

## Test your plugin

Generally, there are 3 phase of testing stages in a strict developing pipeline:

1. In-project: testing your component in your project.
2. Local: testing your component in your local environment (with or without packaged python codes).
3. Snapshot: testing your component with remote snapshot repository (with packaged python codes and deployed component snapshots).

It is OK but not recommend to skip any of these 3 stages.
This step-by-step testing is necessary to ensure a high quality component delivery.

### In-project test

You can build/run you developing component in the project developing environment.
Run below command in your project root directory to do this.

```shell
conpack [build/run]
        --config-file=<config-file> # Your component config.yaml file path
        --add-current-folder-to-path # Add the current folder to path, if absent the python interpreter may not be able to load your project code.
        [--working-directory=<working_directory>] # The working directory (where to place the ".runtime" folder), default to current directory
        [--skip-setup] # add this if you don't need to install any pip package mentioned in the "setup.pip" section in the "config.yaml"
        [--pip-params=<pip_params>] # Additional parameters to be passed to pip when installing a python package
```

The `conpack build` command will exit after successfully initialized your component,
while the `conpack run` command will run your component.
But please note that only a Copilot can be run, run other component will raise an error.

You can also run this command in a python script and use the component instance for further test:

```python
from concopilot import conpack

plugin=conpack(['build', '--config-file=<config-file>', '--skip-setup', '--add-current-folder-to-path'])

plugin.some_method(...)
```

### Test in Local Repository

You can do very limit things in the in-project test.
A higher stage is to test your component in a well-built Copilot to evaluate its performance.

The only work to do is to install your component into your local repository (default to "~/.concopilot/repository").
Run below command in your project root directory to do this.

```shell
conpack install
        --src-folder=. # The source folder (where the ".config" folder exists), default to current directory
        [--recursive] # check all sub-folder of the <src_folder> for possible config files
        [--skip-setup] # add this if you don't want to build your component
        [--add-current-folder-to-path] # Add the current folder to path if you want to build your component
        [--pip-params=<pip_params>] # Additional parameters to be passed to pip when installing a python package
```

After this step, you will find your components installed into your local repository located at `~/.concopilot/repository`,
with the paths consisted of their `group_id`, `artifact_id`, and `version`.

Now, you can use your plugin in any copilot by adding your plugin into the copilot's plugin list.
For example, here is the "[config.yaml](https://github.com/ConCopilot/concopilot/blob/main/concopilot_examples/copilot/auto/.config/config.yaml)" of an AutoGPT like copilot/agent.
Download this file,
and you can add your plugin by adding your plugin's `group_id`, `artifact_id`,
and `version` into the `plugin_manager.config.plugins` section in the file. 

```yaml
# ...

  plugin_manager:
    group_id: org.concopilot.basic.plugin.manager
    artifact_id: basic-plugin-manager
    version: 0.0.0
    config:
      plugin_prompt_generator:
        group_id: org.concopilot.example
        artifact_id: lm-plugin-prompt-gen
        version: 0.0.0
      plugins:
        -
          #...
        -
          group_id: <your_plugin_group_id>
          artifact_id: <your_plugin_artifact_id>
          version: <your_plugin_version>
```

Now, run the `conpack run` command to see the performance.

```shell
conpack run
        --config-file=<your_downloaded_file_path>
        --add-current-folder-to-path
        [--skip-setup]
        [--working-directory=<your_working_directory>]
```

You can also try other copilot or write your own to test your plugins or other components.

You can choose to package your python code (.whl) in this testing stage.
Read [this](https://packaging.python.org/en/latest/tutorials/packaging-projects/) and [this](https://setuptools.pypa.io/en/latest/userguide/quickstart.html) about how to package python project.

After you installed the component into your local repository and installed your pip whl into another venv,
you can build/run your component by substitute the `--config-file` parameter to your component `group_id`, `artifact_id`, and `version` like below:

```shell
conpack build
        --group-id=<group-id> --artifact-id=<artifact-id> --version=<version>
        [--skip-setup]
```

You also do not need to add the `--add-current-folder-to-path` parameter now,
because your codes have been already installed into your venv python "site-packages" folder.

One thing to be emphasized,
remember to remove your component folder under the ".runtime" folder,
if you re-installed your component and want to use the new version.

ConCopilot allows users to modify configs under the ".runtime" folder for special tasks,
thus ConCopilot will never override files under this folder.

### Snapshot Deployment

Theoretically, you can test your package anywhere on your local machine after you installed it into your local repository,
but it is not able to test it in any other places.
This would be a big problem in some cases,
especially you are developing a big project and are cooperating with others.

It is obvious that you need to deliver both your components and your pip whl to your teammates.
It is easy to send you whl file to them, but how about your components?

Don't worry. You can deploy a snapshot version of your component so that others can use it just as it has been released.
(See [this](../introduction/concepts.md#about-the-version) for more information about component versions.)
The only difference between a snapshot version and a formal release version is that a snapshot component can be re-deployed.
So that you can modify your components again and again before the formal release.
If you have work with Maven, you will feel very familiar with this.

All you need to do is to add a "-SNAPSHOT" suffix to you component version, and deploy it using the `conpack deploy` command.

```shell
conpack deploy
        --src-folder=<src_folder> # The source folder (where the ".config" folder exists), default to current directory
        --repo-user-name=<repo_user_name> --repo-user-pwd=<repo_user_pwd>
        [--recursive] # check all sub-folder of the <src_folder> for possible config files
        [--gpg-passphrase=<gpg_passphrase>] [--gnupg-home=<gnupg_home>]
        [--skip-setup] # add this if you don't want to build your component
        [--pip-params=<pip_params>] # Additional parameters to be passed to pip when installing a python package
        [--add-current-folder-to-path] # Whether add the current folder to path
        [--add-working-directory-to-path] # Whether add the working directory to path
        [--add-src-folder-to-path] # Whether add the source folder to path
```

Now, your colleagues and teammates can directly access your component by their `group_id`, `artifact_id`, and `version`.
You are also able to modify and re-deploy your component during the testing.

To request an eligibility to deploy your component, for both snapshot and release versions,
you need to register an account on [concopilot.org](https://concopilot.org),
and apply the accessibility of a `group_id`.
See [this](../build_&_deploy/group_id.md) for the details.

Like the previous section,
ask your teammates to remove the component folders under both their local repository and their ".runtime" folder to use the new version.

One last thing to be emphasized,
the snapshot deployments will be regularly removed from the repository.
If you found it has been removed, just re-deploy it if necessary.

## Release your plugin

For a release deployment,
you should release your python package first.
See [here](https://packaging.python.org/en/latest/tutorials/packaging-projects/) and [here](https://setuptools.pypa.io/en/latest/userguide/quickstart.html) for details.

Then, like the snapshot deployment, you can deploy your release version by using the same `conpack deploy` command.
Please register your account and applied the accessibility of your `group_id` first,
and remember to remove the "-SNAPSHOT" suffix before you release.
Please also note that you are not allowed to re-deploy a released component.

Now, others can find your components on our [website](https://concopilot.org) and access them by the (`group_id`, `artifact_id`, `version`) tuples.
