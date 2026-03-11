<%* const dow = moment().day(); const therapyBase = moment("2026-03-10"); const diff = moment().diff(therapyBase, "weeks"); const isTherapy = (dow === 2) && (diff >= 0) && (diff % 2 === 0);

let sched;

if (dow === 1 || dow === 5) { sched = [ "- 06:30 Wake Up", "- 06:30 Quality Time with Baby", "- 07:00 Morning Journal + Stretch", "- 07:15 Prep Breakfast", "- 07:30 Breakfast + Vitamins", "- 08:00 Quality Time with Baby (Julia eats)", "- 09:00 Work starts", "- 13:00 Lunch + Vitamins", "- 15:45 Review Tomorrow's Calendar + Create Tomorrow's Note", "- 16:00 Housekeeping", "- 16:30 Career Growth", "- 17:00 Free block", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); } else if (dow === 2 && isTherapy) { sched = [ "- 06:30 Wake Up", "- 06:30 Quality Time with Baby", "- 07:00 Morning Journal + Stretch", "- 07:15 Prep Breakfast", "- 07:30 Breakfast + Vitamins", "- 08:00 Quality Time with Baby (Julia eats)", "- 09:00 Work starts", "- 13:00 Lunch + Vitamins", "- 15:45 Review Tomorrow's Calendar + Create Tomorrow's Note", "- 16:00 Housekeeping", "- 16:45 Leave for Therapy", "- 17:00 Therapy", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); } else if (dow === 2) { sched = [ "- 06:30 Wake Up", "- 06:30 Quality Time with Baby", "- 07:00 Morning Journal + Stretch", "- 07:15 Prep Breakfast", "- 07:30 Breakfast + Vitamins", "- 08:00 Quality Time with Baby (Julia eats)", "- 09:00 Work starts", "- 13:00 Lunch + Vitamins", "- 15:45 Review Tomorrow's Calendar + Create Tomorrow's Note", "- 16:00 Housekeeping", "- 16:30 Run", "- 17:00 Career Growth", "- 17:30 Free block", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); } else if (dow === 3 || dow === 4) { sched = [ "- 06:30 Wake Up", "- 06:30 Quality Time with Baby", "- 07:00 Morning Journal + Stretch", "- 07:15 Prep Breakfast", "- 07:30 Breakfast + Vitamins", "- 08:00 Quality Time with Baby (Julia eats)", "- 09:00 Work starts", "- 13:00 Lunch + Vitamins", "- 15:45 Review Tomorrow's Calendar + Create Tomorrow's Note", "- 16:00 Housekeeping", "- 16:30 Workout", "- 17:00 Career Growth", "- 17:30 Free block", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); } else if (dow === 6) { sched = [ "- 07:00 Wake Up", "- 07:30 Breakfast + Vitamins", "- 08:30 Housekeeping batch (Laundry, Trash, Mop, etc.)", "- 11:00 Workout", "- 12:30 Lunch + Vitamins", "- 14:00 Free block / family time", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time / Date Night with Julia", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); } else { sched = [ "- 07:00 Wake Up", "- 07:30 Breakfast + Vitamins", "- 08:30 Housekeeping batch (Deep cleans, Meal Prep, etc.)", "- 11:00 Weekly Review + Budget Check", "- 12:30 Lunch + Vitamins", "- 14:00 Portfolio / Side Project", "- 17:00 Call Mother + Massagem Julia", "- 18:00 Dinner prep", "- 18:30 Dinner + Vitamins", "- 19:00 Family time", "- 21:00 Walk Tofu", "- 21:20 Floss + Wind down" ].join("\n"); }
tR += "# 📅 " + tp.date.now("dddd, MMMM DD YYYY") + "\n\n---\n\n## 🗓 Today's Schedule\n\n" + sched + "\n\n---\n\n## 🙏 Gratitude\n- \n- \n- \n\n---\n\n## ✅ Tasks\n\n### 💼 Work Tasks\n";
%>
```dataview
TASK
FROM #work
WHERE !completed AND contains(tags, "#work")
SORT due ASC
```
## 📋 Today's Habits & To-Dos

```dataview
TASK
FROM #td
WHERE !completed AND due = date(today)
SORT due ASC
```
## ⏰📋 Past Activities
```dataview
TASK
FROM #td
WHERE !completed AND due < date(today) AND !contains(tags, "#work")
SORT due ASC
```