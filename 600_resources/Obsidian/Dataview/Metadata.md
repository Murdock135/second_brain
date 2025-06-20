Dataview cannot query all content. It can only query *indexed content* (Some content are indexed automatically like bullet points or task lists). Indexed content is simply any content that is associated with **metadata.**
**Metadata**: Also called 'fields', these are key value pairs. 
Metadata can come in two forms
1. [[YAML front matter]].
2. [[Inline fields]]
# Keys have 'sanitized' versions
Suppose you used the following inline field.
```
Basic Field:: Some random Value # key::value
```
The key `Basic Field` will also have a 'sanitized' version `basic-field` (lower case, no spaces).
>[!important]
>If a key contains spaces, you cannot query it as is. You have to use the sanitized version.

# Implicit fields
Obsidian indexes some data automatically. For example some file properties like 
- `file.cday`: File creation date
- `file.outlinks`: Links in the file
- `file.etags`: Tags inside the file
And more. Find the full list of available implicit fields on  [Metadata on pages](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-pages/) and [Metadata on Tasks and Lists](https://blacksmithgu.github.io/obsidian-dataview/annotation/metadata-tasks/).