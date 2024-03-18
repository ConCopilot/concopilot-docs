# Assets

Assets are designed to store complex objects in memory.
They are used to store data that cannot be serialized into chat histories, such as images.

An Asset is represented as a Python `dict` with the following format:

```yaml
asset_type: <asset_type> # optional, a task-related string to classify assets
asset_id: <asset_id> # the asset ID
asset_name: <asset_name> # optional, the asset name
description: <description> # optional, the asset description
content_type: <content_type> # the content type of the "content" value below
content: <content> # the asset content, can be anything
```

The `content_type` and `content` fields have the same meaning as those in a Message,
but can store data in any type and structure.

## Asset Meta (AssetMeta)

An Asset Meta is a JSON object that describes the structure and data types of an asset.
It has the same structure as an Asset, but all complex data types are replaced with their Python type string (`str(type(obj))`).

For example, let's say we have an asset that stores an image with the following structure:

```json
{
  "asset_type": "asset_example",
  "asset_id": "<asset_id>",
  "content_type": "<class 'dict'>",
  "content": {
    "image": <a_complex_numpy_ndarray>,
    "source_url": "<the_image_source_url>",
    "labels": [
      "<label_1>",
      "<label_2>"
    ]
  }
}
```

Its corresponding asset meta will be:

```json
{
  "asset_type": "asset_example",
  "asset_id": "<asset_id>",
  "content_type": "<class 'dict'>",
  "content": {
    "image": "<class 'numpy.ndarray'>",
    "source_url": "<the_image_source_url>",
    "labels": [
      "<label_1>",
      "<label_2>"
    ]
  }
}
```

Sometimes, LLMs may struggle to capture the hierarchical structure of the asset meta.
In such cases, a flattened asset meta with type information of its sub-containers can be easier for LLMs to understand.
Developers can use the `asset_ref_obj.flatten(sep='/', keep_container_type=True)` method to retrieve the flattened asset meta with container type information.

For the example above, its flattened asset meta with container type information will be:

```json
{
  "asset_type": "asset_example",
  "asset_id": "<asset_id>",
  "content_type": "<class 'dict'>",
  "content": "<class 'dict'>",
  "content/image": "<class 'numpy.ndarray'>",
  "content/source_url": "<the_image_source_url>",
  "content/labels": "<class 'list'>",
  "content/labels/0": "<label_1>",
  "content/labels/1": "<label_2>"
}
```

Please note that the `"content": "<class 'dict'>"` and `"content.labels": "<class 'list'>"` are sub-container type information.

In `Cerebrum` developments,
it is recommended to provide a flattened AssetMeta list that contains the meta information for each asset to the LLM.
This allows the LLM to review the Asset Meta in each round of chat and connect data in assets to the tasks and goals.

## Asset Reference (AssetRef)

An Asset Reference is used to reference an asset (or asset field) instead of using the actual asset (or asset field) object.
It is represented as a Python `dict` with the following structure:

```json
{
  "asset_id": "the_id_to_the_referenced_asset",
  "field_path": ["the", "hierarchical", "path", "starting", "from", "the", "asset", "json", "root", "to", "the", "field", "being", "retrieved"]
}
```

An Asset Reference can also be represented as an Asset Reference URL::

```text
asset://the_id_to_the_reference_asset/the/hierarchical/path/starting/from/the/asset/json/root/to/the/field/being/retrieved
```

The `"field_path"` is a list of keys that represents the hierarchical path to the field you want to retrieve.
Please note that a `"field_path"` should always start from the first level of the asset JSON.

An integer number can also be used in the `"field_path"` to reference an element in a JSON array.

Please note that the `"field_path"` to a field in a certain Asset and its corresponding AssetRef are identical.

### Passing AssetRef as Plugin Call Parameters

Sometimes, a plugin call command explicitly declares that it accepts an asset reference as its parameters by setting `asset_ref_acceptable: true` in its parameter description sections in the plugin config YAML.
In this case, an asset reference can be passed to the command as if passed the referencing object to it.

When passing an Asset Reference to a plugin command,
substitute the real parameter value with the asset reference JSON object or asset reference URL string.

Please note that an asset reference URL should be regarded as an object that is identical to its corresponding `dict` format and not as a URL.
Therefore, do NOT confuse asset reference URLs to a common URLs.

For example, passing an object stored in some asset to a command:

The JSON object version:

```json
{
  "receiver": {
    "role": "plugin",
    "name": "<plugin_name>"
  },
  "content_type": "command",
  "content": {
    "command": "<command_name>",
    "param": {
      "<param_name>": {
        "asset_id": "<asset_id>",
        "field_path": ["<path>", "<to>", "<the>", "<object>"]
      }
    }
  }
}
```

The Asset Reference URL version:

```json
{
  "receiver": {
    "role": "plugin",
    "name": "<plugin_name>"
  },
  "content_type": "command",
  "content": {
    "command": "<command_name>",
    "param": {
      "<param_name>": "asset://<asset_id>/<path>/<to>/<the>/<object>"
    }
  }
}
```

### Using AssetRef in Message Contents Text

Sometimes, it is needs to embed an asset object in a message text.
For example, there might need to embed an image in a Markdown text.
Use the asset reference to the object to achieve this.
Both the JSON object version and Asset Reference URL version are acceptable.
However, when embedding the asset reference into the message text, make sure to surround it with "<|" and "|>".
This is necessary only in this case and is not required when passing AssetRef to Plugin Calls.

Sometimes, it is needs to reference some asset object in a message.
Use the asset reference to the object to achieve this.
Both the JSON object version and Asset Reference URL version are acceptable.
Unlike passing AssetRef to Plugin Calls,
Please surround the asset reference with "<|" and "|>" in this case.

For example, inserting an image object by using asset reference in a Markdown text:

The JSON object version:

```markdown
![image_alt](<|{"asset_id":"the_asset_id","field_path":["path", "to", "the", "image", "object"]}|>)
```

The Asset Reference URL version:

```markdown
![image_alt](<|asset://the_asset_id/path/to/the/image/object|>)
```

Make sure the referenced asset object can be correctly interpreted and displayed according to the corresponding message `"content_type"`.
