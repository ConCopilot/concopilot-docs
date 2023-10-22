# Build & Deploy

ConCopilot uses a Maven like repository to manage components.
That is, a (`group_id`, `artifact_id`, `version`) tuple uniquely identifies a component.

We provide a build and deploy tool named `conpack` to build, run, install, and deploy copilots and components.
It can be used both in code and in CLI.

We also provide a lower level `run` API for build and run a copilot.

[The `conpack` CLI](conpack.md)
<br>
[The `run` API](run.md)

## Concepts

### settings

Like Maven, we use a "setting.yaml" to configure the build and deploy environment.

Details of this are still under developing. Please just use the default settings for now.

### working directory

We use a `--working-directory` indicator to specify the working directory of a running copilot.

For task specific component configuration, a ".runtime" directory will be created under the working directory.
The ".runtime" directory will save all component configs under a tree formatted path structure just like Maven.
All component config files in this directory are initially copied from the component local repository.
With these copies, users can modify any of these configs for their specific tasks without worrying conflicts between tasks.
Do NOT directly modify configs in your local repository.

Meanwhile, for safety and security issues,
we recommend developers to check the disk access authority for their plugins if applicable,
making they can only access files and folders under the working directory.

During developing, the working directory can be accessed by this:

```python
from concopilot import Settings

settings=Settings()
settings.working_directory
```

We are going to make some mechanism such as a container to make it possible to constraint a running copilot can only access resources under the working directory in the future.

### The Group ID

Like Maven, developers need to register a group_id to upload their components.
This is to eliminate component conflicts between different organizations.
See [Apply a Group ID](group_id.md) for detail.
