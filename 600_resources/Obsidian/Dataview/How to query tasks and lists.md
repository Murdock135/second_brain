Recall that dataview can only query 'indexed' data or in other words, content that is associated with metadata (A 'field' aka a key-value pair). 
You can associate tasks and lists with meta data using the inline field syntax like the following:
```
- [ ] Hello, this is some [metadata:: value]! 
- [X] I finished this on [completion:: 2021-08-15].
```
