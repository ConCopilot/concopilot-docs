# Find Component Source Code

Sometimes people may be curious about how a ConCopilot component works,
or want to refer to an existed component for any purpose.

Find the source code for any ConCopilot component is very easy.
Remember we have a `setup` section in any component's "config.yaml" file.

Thus, whenever you see a component section in anywhere of any config Yaml file like below:

```yaml
group_id: <group_id>
artifact_id: <artifact_id>
version: <version>
```

You can find the component "config.yaml" in either your local repository (default to "~/.concopilot/repository") or your working directory's ".runtime" folder,
according to its `group_id`, `artifact_id`, and `version`.

Eg.:

For local repository: `~/.concopilot/repository/<path/to/the/group/id>/<artifact_id>/<version>/config.yaml`

For ".runtime" folder: `<working_directory>/.runtime/<path/to/the/group/id>/<artifact_id>/<version>/<instance_id>/config.yaml`

Then open the "config.yaml" and find the `setup` section like below:

```yaml
setup:
  pip:
    # - ...
  package: package.path.that.contains.the.__init__.py
```

The value of the `package` field is the component import path.
So open a new python file in any IDE, and import the package:

```python
import package.path.that.contains.the.__init__.py
```

Then you can follow your IDE's prompt to the source code file,
usually by clicking the import path with your ctrl button down.

Another way is to open the package directly from your python environment's "site-packages" folder if you don't want to use an IDE.

There is an obvious prerequisite for find the source code.
That is you need to install the package first.
Run the `conpack build` or `conpack run` command first to let ConCopilot do this for you.
