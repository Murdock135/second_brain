# Personal Projects
## Ongoing Tasks
```dataview
TASK
FROM "400_projects/410_personal_projects" or "300_disciplines"
WHERE contains(tags, "#ongoing") and !contains(tags, "#milestone")
WHERE !fullyCompleted
```
## Planned Tasks
```dataview
TASK
FROM "400_projects/410_personal_projects" or "300_disciplines"
WHERE contains(tags, "#planned") and contains(tags, "#task")
WHERE !fullyCompleted
```

## All Tasks
```dataview
TASK
FROM "400_projects/410_personal_projects" or "300_disciplines"
WHERE contains(tags, "#task")
WHERE !fullyCompleted
```

# Professional Projects
## Ongoing Tasks
```dataview
TASK
FROM "400_projects/420_professional_projects"
WHERE contains(tags, "#ongoing") and contains(tags, "#task")
WHERE !fullyCompleted
```

## Planned Tasks
```dataview
TASK
FROM "400_projects/420_professional_projects"
WHERE contains(tags, "#planned") and contains(tags, "#task")
WHERE !fullyCompleted
```
## All tasks
```dataview
TASK
FROM "400_projects/420_professional_projects"
WHERE !fullyCompleted
WHERE contains(tags, "#task")
```