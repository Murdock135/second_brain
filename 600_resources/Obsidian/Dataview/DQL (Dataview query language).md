A DQL Query consists of several parts:
- Exactly one [**Query Type**](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/) that determines what your Query Output looks like
- None or one [**FROM statement**](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands#from) to pick a specific tag or folder (or another [source](https://blacksmithgu.github.io/obsidian-dataview/reference/sources/)) to look at
- None to multiple [**other Data Commands**](https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/) that help you filter, group and sort your wanted output
A very simple example:
```
LIST
```
which lists all files in your vault.
A more restricted query may look like the following (example taken from the [documentation]([Dataview](https://blacksmithgu.github.io/obsidian-dataview/#data-querying)))
```
LIST FROM #poems WHERE author = "Edgar Allan Poe"
```

which lists all files in your vault that have the tag `#poems` and a [field](https://blacksmithgu.github.io/obsidian-dataview/annotation/add-metadata/) named `author` with the value `Edgar Allan Poe`. This query would find our example page from above.
- `LIST` is only one out of four [Query Types](https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/) you can use. For example, with a `TABLE`, we could add some more information to our output:
```
TABLE author, published, file.inlinks AS "Mentions" 
FROM #poems
```
This will return:
![[Screenshot from 2025-06-20 13-38-53.png]]
# Where to next
- You can operate on your data using [**functions**](https://blacksmithgu.github.io/obsidian-dataview/reference/functions/)
- Find more examples [here](https://blacksmithgu.github.io/obsidian-dataview/resources/examples/).