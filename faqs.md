# FAQs

**Q: I can't run run an Aqueduct workflow because whenever I run an operator, I get an error about invalid continuation bytes. Something like this:**

```bash
File "pydantic/json.py", line 93, in pydantic.json.pydantic_encoder
File "pydantic/json.py", line 51, in pydantic.json.lambda
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xed in position 10: invalid continuation byte
python-BaseException
```

**A**: Pydantic v1.8 and older have an issue with JSON deserialization in newer version of Python. Please update to Pydantic v1.9 or later to resolve this issue.

**Q: I'm getting an error about importing type hints:**

```bash
ImportError: cannot import name 'TypeGuard' from 'typing' 
```

**A**: Older versions of the `typing-extensions` library do not have support for typing features released in newer versions of Python3. Please upgrade to `typing-extensions>=4.3.0`.

**Q: When connecting a resource I get an error message about missing dependencies. How do I resolve this?**

**A:** Each resource requires specific drivers that you might not have installed. Before connecting an resource, please run `aqueduct install <resource-name>` from your command line. This ensures that the right depenendencies are installed and that can Aqueduct can connect to the relevant system. See [configuring-aqueduct.md](installation-and-configuration/configuring-aqueduct.md "mention") for more details.
