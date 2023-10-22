# Copilot

This is the `Copilot` interface.

```python
class Copilot(AbstractPlugin):
    """
    Define the Copilot interface.

    All copilots implement the Plugin interface, so it is easy to use a copilot as a plugin in another copilot.
    """

    def __init__(self, config: Dict):
        """
        Configure the copilot without initialization.

        Make sure the `type` in the config file is set to "copilot".

        :param config: configures read from its config file (default to "config.yaml")
        """
        super(Copilot, self).__init__(config)
        assert self.type=='copilot'

    @abc.abstractmethod
    def initialize(self):
        """
        Initialize the framework and resources
        """
        pass

    @abc.abstractmethod
    def finalize(self):
        """
        Finalize the framework and resources
        """
        pass

    @abc.abstractmethod
    def run_interaction(self):
        """
        Run the copilot interaction.

        This is the main logic of the copilot,
        so write your special task pipeline here.
        """
        pass

    @abc.abstractmethod
    def run(self):
        """
        The entrance of the copilot task pipeline.

        This method does the initialization, runs the interaction, and does the finalization.
        """
        pass

    def command(self, command_name: str, param: Dict, **kwargs) -> Dict:
        """
        The Plugin command method.

        Ignore this method if you are not expect your copilot acting as a plugin in other copilots,
        or you have to override it as the same way of which in developing Plugins.

        :param command_name: command name
        :param param: command parameters
        :param kwargs: reserved, not recommend to use
        :return: the command result
        """
        return {}
```

We provided a general purposed Copilot implementation: the `BasicCopilot`.
It is generally not necessary to change the framework or code of a Copilot,
just change its "config.yaml" adding more necessary component or replacing existed components for specific tasks.

## The General Purposed Copilot Implementation

```yaml
group_id: org.concopilot.basic.copilot
artifact_id: basic-copilot
```

### Framework

[Source code](https://github.com/ConCopilot/concopilot/tree/main/concopilot/basic/copilot)

When an instance of this Copilot is created and run, it executes below tasks step by step:

1. `__init__`

    1. `resource_manager`:
        1. construct the Resource Manager according to the `config.resource_manager` section in the "config.yaml" file.
        2. construct all resources under the `resource_manager` section (**currently the resources are not initialized**)

    2. `storage`: construct the Storage for the Copilot memery according to the `config.storage` section in the "config.yaml" file.

    3. `user_interface`: construct the User Interface according to the `config.user_interface` section in the "config.yaml" file.

    4. `cerebrum`: construct the Cerebrum instance to take advantages from LLM for task analysis according to the `config.cerebrum` section in the "config.yaml" file.

    5. `plugin_manager`:
        1. construct the Plugin Manager according to the `config.plugin_manager` section in the "config.yaml" file.
        2. construct the Plugin Prompt Generator according to the `plugin_prompt_generator` section under the `plugin_manager` section in the "config.yaml" file.
        3. construct all plugins under the `plugin_manager` section (**currently the plugins are not initialized**)

    6. `message_manager`: construct the Message Manager according to the `config.message_manager` section in the "config.yaml" file.

    7. `interactor`: construct the Interactor according to the `config.interactor` section in the "config.yaml" file.
      the `resource_manager`, `cerebrum`, `plugin_manager`, and `message_manager` will be passed to the interactor as it is the central coordinator.

2. initialize

    1. initialize the `resource_manager`, and all resources under it.
        **The resources can only be used after this step.**

    2. config resources for all components.
        **A component can only access its necessary resources after this step.**

    3. config a context for the Copilot so that every component can access below global components by using its `self.context` field:
        1. `storage`
        2. `assets` (additional materials apart from chat histories to be passed to Cerebrum LLM to analysis if necessary)
        3. `user_interface`

    4. initialize the `interactor`
        1. config general prompts by calling the `interactor.setup_prompts` method.
            This step is to config the general prompts for the interactor as well as its subcomponents' prompt if applicable.
            Details will vary for different kinds of interactor.
        2. config the `plugin_manager` and all plugins by calling the `interactor.setup_plugins` method.
            This step is for configuring plugins as well as pass all plugins to the cerebrum object for potential necessary initialization (such as the OpenAI function call).

3. **run interaction**: run the main task loop defined by the Interactor.

4. finalize: release/close all resources if necessary and exit the program.

### Config

1. `resource_manager`: Specify a Resource Manager to manager resources.
    Add all necessary resources under the resource manager config also.

2. `cerebrum`: Add a cerebrum component for task analysis.
    A cerebrum usually backed on a LLM to analyse user requirements and plugin information.

3. `interactor`: **This is the central part of a Copilot.**
    It controls the information dispatching, task coordinating, and plugin calls in a Copilot.
    Different task may need different "Interaction Framework".
    **Replace this component rather than the Copilot itself for specific tasks.**

4. `plugin_manager`: Specify a Plugin Manager to manage plugins.
    Add all necessary plugins under the plugin manager config section also.

5. `message_manager`: Specify a Message Manager to deal with Cerebrum response.

6. `storage`: Specify this for Copilot memory.

7. `user_interface`: Specify this to interact with users.

### Usage

Choose suitable component from our [website](https://concopilot.org) for your specific tasks,
and use them to modify the Copilot "config.yaml",
then use our `conpack` tool to run the Copilot.

Here is a runnable "[config.yaml](https://github.com/ConCopilot/concopilot/blob/main/concopilot_examples/config.yaml)" example.
