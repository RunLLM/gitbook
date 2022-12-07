# FAQs

**Q: I can't run run an Aqueduct workflow because whenever I run an operator, I get an error about invalid continuation bytes. Something like this:**&#x20;

```bash
File "pydantic/json.py", line 93, in pydantic.json.pydantic_encoder
File "pydantic/json.py", line 51, in pydantic.json.lambda
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xed in position 10: invalid continuation byte
python-BaseException
```

**A**: Pydantic v1.8 and older have an issue with JSON deserialization in newer version of Python. Please update to Pydantic v1.9 or later to resolve this issue.

**Q: I'm getting an error about importing type hints:**&#x20;

```bash
ImportError: cannot import name 'TypeGuard' from 'typing' 
```

**A**: Older versions of the `typing-extensions` library do not have support for typing features released in newer versions of Python3. Please upgrade to `typing-extensions>=4.3.0`.

**Q: I'm using different python versions for my SDK and my Aqueduct server enviroments:**&#x20;

**A**: Aqueduct provides better dependencies and python version management support through conda integration. You can refer to [this guide](./operators/using-conda.md) to learn about what we support and how to use conda integration.

