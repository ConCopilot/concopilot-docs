# Run a Developed Copilot/Agent

## Use the `conpack` CLI

The `conpack` CLI provide mechanisms to download configurations, install python packages, and run a copilot/agent.

Users can use `conpack build` to download component configurations from ConCopilot Repository, and build the component object.

Users can use `conpack run` to run a developed copilot/agent.

The `conpack run` command wil `build` the copilot/agent first,
thus it is not necessary to run the `conpack build`.
However, there are usually some configurations need to be changed by user before running a copilot/agent,
such as API keys, service endpoints or ports,
run the `conpack build` to download the "config.yaml" files and initialize the working directory first and then modify necessary information is more convenient.

Users can find all the "config.yaml" files of components consisted of the copilot/agent under the ".runtime" folder of the "working directory".
The "working directory" can be configured by passing `--working-directory=<working_directory>` to the `conpack` command and is defaulted to current directory.

Learn more about the `conpack` CLI [here](../build_&_deploy/conpack.md).

## Run components in python

There might be requirements to construct or run a ConCopilot component or copilot/agent in a Python script.

Developers can import and use the `conpack` method in Python like below:

```python
from concopilot import conpack

# Download necessary configuration files,
# install necessary pip packages,
# then build and return the built component.
component=conpack('build', '--group-id=<group_id>', '--artifact-id=<artifact_id>', '--version=version')
# or
component=conpack('build', group_id='<group_id>', artifact_id='<artifact_id>', version='version')

# Run the copilot/agent described in config file <config_file_path>.
# Skip all pip package installation check
# Add current folder to Python path
conpack('run', '--config-file=<config_file_path>', '--skip-setup', '--add-current-folder-to-path')
# or
conpack('run', config_file='<config_file_path>', skip_setup=True, add_current_folder_to_path=True)
```

The usage of `conpack` method in python is identical to the `conpack` CLI.
