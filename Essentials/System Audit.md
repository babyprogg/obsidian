# System Audit - {{date}}

## Total Notes
```dataview
TABLE file.ctime as "Created"
WHERE file.name != "System Audit"
SORT file.ctime DESC
```

Total: [Count manually first time]

## Orphan Notes (No Links)
```dataview
TABLE file.ctime as "Created"
WHERE length(file.inlinks) = 0 
  AND length(file.outlinks) = 0
  AND file.name != "System Audit"
SORT file.ctime DESC
```

## Most Linked Notes
```dataview
TABLE 
  length(file.inlinks) as "Backlinks",
  length(file.outlinks) as "Outlinks",
  (length(file.inlinks) + length(file.outlinks)) as "Total"
WHERE file.name != "System Audit"
SORT (length(file.inlinks) + length(file.outlinks)) DESC
LIMIT 20
```

## Notes by Folder
```dataview
TABLE length(rows) as "Count"
FROM ""
WHERE file.name != "System Audit"
GROUP BY file.folder
SORT length(rows) DESC
```

## Recent Activity (Last 7 Days)
```dataview
TABLE file.mtime as "Modified"
WHERE file.mtime >= date(today) - dur(7 days)
  AND file.name != "System Audit"
SORT file.mtime DESC
```