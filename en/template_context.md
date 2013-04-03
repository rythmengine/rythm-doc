# Template Context

This section elaborate the template context concept, strictly, the template runtime context. It is important to understand this concept it is used by rythm engine to figure out, for example, the proper escape scheme, locale and code type to support smart escaping, i18n and smart natural template features.

The template context is a combination of current state of the following object: 

### [codetype]Code Type

Code type is a concept to 

### [escape]Escape Scheme

1. Check to see if there are explicitly set escape scheme using [@escape()](directive.md#escape) or [@raw()](directiv.md#raw) directive
1. If no explicitly escape scheme found then check the **current** code type  
