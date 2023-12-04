# Special Plugins

Special Plugins are defined to deal with information pipelines and task coordinating in a Copilot.
Generally, they will not be called as common Plugins by the Copilot,
but by the predefined working flow instead just like traditionally coding.

All special plugins extends the `Plugin` interface and contains all the plugin methods.
This doc list their special methods only.

[TOC]

## Cerebrum

Source code: [cerebrum.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/cerebrum/cerebrum.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/cerebrum/config.yaml)
<br>
Implement example: [OpenAICerebrum](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot_examples/cerebrum/openaicerebrum)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the Cerebrum without initialization.
    <br>
    Make sure the `type` in the config file is set to "cerebrum".

- `def setup_plugins(self, plugin_manager: PluginManager)`

    This method will be called by the Interactor to expose the plugin_manager (and all plugins in it) to the cerebrum.
    <br>
    Special work can be done here with the plugins, such as config OpenAI function calls from plugins, if necessary.

- `def model(self) -> LLM`

    Return the backed LLM.
    <br>
    Retrieve the LLM from the plugin's resource list for convenient use.

- `def interact(self, param: InteractParameter, **kwargs) -> InteractResponse`

    This method will be called by the Interactor to interact copilot materials (`InteractParameter`) with the backed LLM.
    <br>
    A standard `InteractResponse` object will be constructed and returned to the Interactor by reading the LLM response,
    so that the Interactor and other parts of the copilot need not know any detail of the LLM.

- the `role` property

    The getter and setter of the Cerebrum role.

### The `AbstractCerebrum`

```python
from concopilot.framework.cerebrum import AbstractCerebrum
```

This class implemented the cerebrum `role` which will be initialized from the `config.role` field of the "config.yaml".

## Interactor

Source code: [interactor.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/interactor/interactor.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/interactor/config.yaml)
<br>
Implement example: [AutoInteractor](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot_examples/interactor/auto)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the Interactor without initialization.
    <br>
    Make sure the `type` in the config file is set to "interactor".

- `def setup_prompts(self)`

    Setup the interaction prompts if necessary.
    <br>
    Setup all subcomponents' prompts here, if necessary.

- `def setup_plugins(self)`

    Setup plugins under its own scope if necessary.
    <br>
    Setup all subcomponents' plugins here, if necessary.

- `def interact_loop(self)`

    The main interaction loop.
    <br>
    All main procedures of the interactor/copilot tasks should be defined here.

### The `BasicInteractor`

```python
from concopilot.framework.interactor import BasicInteractor
```

This class implemented the `__init__` with passing the `resource_manager`, `cerebrum`, `plugin_manager`, and `message_manager`.

The `setup_prompts` and the `setup_plugins` methods are also defined to call the `plugin_manager.generate_prompt` and `cerebrum.setup_plugins`, respectively.

## User Interface

Source code: [interface.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/interface/interface.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/interface/config.yaml)
<br>
Implement example: [CmdUserInterface](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot/basic/interface/cmd)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the UserInterface without initialization.
    <br>
    Make sure the `type` in the config file is set to "user_interface".

- `def send_user_msg(self, msg: Message)`

    Send a message to the user.

- `def on_user_msg(self, msg: Message) -> Optional[Message]`

    Send a message to the user and wait the user response.

- `def has_user_msg(self) -> bool`

    Check if user has sent any message.

- `def get_user_msg(self) -> Optional[Message]`

    Retrieve a new user message if any.
    <br>
    Use this with `has_user_msg` for async user interaction pipelines.

- `def wait_user_msg(self) -> Optional[Message]`

    Wait a new user message if until there is one.
    <br>
    Use this for async user interaction pipelines.

## Message Manager

Source code: [manager.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/message/manager.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/message/manager/config.yaml)
<br>
Implement example: [BasicJsonMessageManager](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot/basic/message/manager/jsonmanager)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the MessageManager without initialization.
    <br>
    Make sure the `type` in the config file is set to "message_manager".

- `def parse(self, response: InteractResponse) -> Message`

    Parse the input `InteractResponse` object into a `Message` object.
    <br>
    Usually called by an interactor after receiving a cerebrum response.

## Plugin Manager

Source code: [manager.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/plugin/manager.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/plugin/manager/config.yaml)
<br>
Implement example: [BasicPluginManager](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot/basic/plugin/manager)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the PluginManager without initialization.
    <br>
    Make sure the `type` in the config file is set to "plugin_manager".

- `def generate_prompt(self)`

    Generate prompt for itself and managed plugins if necessary.

- `def get_combined_prompt(self) -> str`

    Return the combined prompt of itself and all plugins managed.

- `plugin_prompt_generator` property

    Getter and setter of the plugin_prompt_generator object for plugin prompt generation.

- `plugins` property

    Get the full list of managed plugins.

- `plugin_id_map` property

    Get the map (`Dict[Union[uuid.UUID, str], Plugin]`) of managed plugins arranged by the plugin id.

- `plugin_name_map` property

    Get the map (`Dict[str, Plugin]`) of managed plugins arranged by the plugin name.

- `def add_plugin(self, plugin: Plugin)`

    Add a new plugin to the plugin pool.

- `def get_plugin(self, *, id: Union[uuid.UUID, str] = None, name: str = None) -> Optional[Plugin]`

    Retrieve a plugin with its id and name.

This class already implemented the `get_plugin` method,
as well as the `config_resources` and `config_context` method of the `Plugin` interface to configure resources and context for each plugin, respectively.

### The `BasicPluginManager`

```python
from concopilot.framework.plugin import BasicPluginManager
```

This class implemented all method in `PluginManager` and can be directly used.

## Plugin Prompt Generator

Source code: [promptgenerator.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/plugin/promptgenerator.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/plugin/promptgenerator/config.yaml) 
<br>
Implement example: [LanguageModelPluginPromptGenerator](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot_examples/plugin/promptgenerator)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the PluginPromptGenerator without initialization.
    <br>
    Make sure the `type` in the config file is set to "plugin_prompt_generator".

- `def generate_prompt(self, plugin: Plugin) -> str`

    Generate a prompt for the passed plugin.

## Resource Manager

Source code: [manager.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/resource/manager.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/resource/manager/config.yaml)
<br>
Implement example: [BasicResourceManager](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot/basic/resource/manager)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the ResourceManager without initialization.
    <br>
    Make sure the `type` in the config file is set to "resource_manager".

- `def add_resource(self, resource: Resource)`

    Add a new resource to the resource pool.

- `def initialize(self)`

    Initialize all resources.

- `def finalize(self)`

    Finalize all resources.

### The `BasicResourceManager`

```python
from concopilot.framework.resource import BasicResourceManager
```

This class implemented all method in `ResourceManager` and can be directly used.

## Storage

Source code: [storage.py](https://github.com/ConCopilot/concopilot/blob/v0.0.1/concopilot/framework/storage/storage.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/v0.0.1/config/storage/config.yaml)
<br>
Implement example: [DiskStorage](https://github.com/ConCopilot/concopilot/tree/v0.0.1/concopilot/basic/storage/disk)

### Special methods

- `def __init__(self, config: Dict)`

    Configure the Storage without initialization.
    <br>
    Make sure the `type` in the config file is set to "storage".

- `def get(self, key: str) -> Any`

    Get the stored object by the give `key`.

- `def get_or_default(self, key: str, default: Any) -> Any`

    Get the stored object by the give `key`, or the `default` if not found.

- `def put(self, key: str, value: Any)`

    Put the `value` to the storage, relate it with the given `key`.

- `def remove(self, key: str) -> Any`

    Remove the stored object related to the given `key`.

- `def get_sub_storage(self, key: str) -> 'Storage'`

    Return a sub-storage space of this storage.
    <br>
    A sub-storage is a dedicated area in this storage for special use.
    <br>
    Developers must make sure that:

    1. a sub-storage must be a subset of this parent storage,
    2. a sub-storage cannot access data in other part of the parent storage,
    3. it is recommended that the parent storage should not access data in even its own sub-storage.

- `def remove_sub_storage(self, key: str) -> bool`

    Remove a sub-storage relating to the specified `key`.
    Return `True` if success, and `False` if failed.
