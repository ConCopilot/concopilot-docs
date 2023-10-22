# Message

Source code: [message.py](https://github.com/ConCopilot/concopilot/blob/main/concopilot/framework/message/message.py)

A message can be constructed using the `__init__` method by passing the `sender`, `receiver`, `content`, and `time`.

The `Message` class is backed on the `ClassDict` class.
Thus, theoretically it is OK to pass anything in a message as long as there is any component can process it,
otherwise those fields will be just ignored.
