---
created: <% tp.date.now("YYYY-MM-DD") %>
tags:
  - daily-notes
week: <% tp.date.now("ww") %>
---

# <% tp.date.now("dddd, MMMM DD, YYYY") %>

[[<% tp.date.now("YYYY-MM-DD", -1) %>|← Yesterday]] | [[<% tp.date.now("YYYY-MM-DD", 1) %>|Tomorrow →]]

## 🌅 Morning Intentions

- **Today's Main Focus:** 
- **Top 3 Priorities:**
	1. 
	2. 
	3. 
- **Energy Level (1-10):** 

## ✅ Tasks

### Priority Tasks
- [ ] 
- [ ] 
- [ ] 

### Other Tasks
- [ ] 
- [ ] 

## 📝 Daily Log

*Quick captures, meeting notes, ideas, and observations throughout the day*

### Meeting Notes

### Quick Captures

### Ideas & Insights

## 🌙 Evening Reflection

- **What went well today?**
	- 

- **What could be improved?**
	- 

- **Key learnings:**
	- 

- **Gratitude:**
	- 

- **Tomorrow's preparation:**
	- 

## 📊 Today's Activity

### Notes Created Today
```dataview
LIST
FROM ""
WHERE file.cday = date("{{date:YYYY-MM-DD}}")
SORT file.ctime DESC
```

### Notes Modified Today
```dataview
LIST
FROM ""
WHERE file.mday = date("{{date:YYYY-MM-DD}}")
AND file.cday != date("{{date:YYYY-MM-DD}}")
SORT file.mtime DESC
```

---

**Links:** [[Weekly Notes/{{date:YYYY-[W]ww}}|This Week]] | [[Monthly Notes/{{date:YYYY-MM}}|This Month]]