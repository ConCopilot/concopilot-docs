# Resource

Source code: [resource.py](https://github.com/ConCopilot/concopilot/blob/main/concopilot/framework/resource/resource.py)
<br>
Config file example: [config.yaml](https://github.com/ConCopilot/concopilot/blob/main/config/resource/category/model/config.yaml)

[TOC]

A resource must be the `type` of "resource".
Developers should make sure that the `type` in its config file is set to "resource".

A `resource_type` field is need in the config file to indicate the type of the resource.
Such as "disk", "memory", "mongodb", etc.
Basically, this `resource_type` is for human read so that any string is allowed.

## Methods

### `def resource_type(self) -> str`

Return the `resource_type`. This method has been implemented.

### `def initialize(self)`

Initialize this resource.
Please finish all initialization and allocation here.
Your resource is expected ready to use after this method return.

### `def finalize(self)`

Do all finalization, release, and clean here.
The resource is supposed being destroyed after this method return.
