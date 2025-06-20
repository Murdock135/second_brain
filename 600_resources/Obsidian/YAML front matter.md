Frontmatter is a common Markdown extension which allows for YAML metadata to be added to the top of a page. It is natively supported by Obsidian and explained in its [official documentation](https://help.obsidian.md/Advanced+topics/YAML+front+matter). All YAML Frontmatter fields will be automatically available as Dataview fields.
Example:
```
--- 
alias: "document" 
last-reviewed: 2021-08-17 
thoughts: 
	rating: 8 
	reviewable: false 
---
```
With this your note has metadata fields named `alias`, `last-reviewed`, and `thoughts`. Each of these have different **data types**:

- `alias` is a [text](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#text), because its wrapped in ""
- `last-reviewed` is a [date](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#date), because it follows the ISO date format
- `thoughts` is a [object](https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/#object) field, because it uses the YAML Frontmatter object syntax