<%* const today = tp.date.now("YYYY-MM-DD"); const dayOfWeek = moment().day(); // 0=Sun, 1=Mon, 2=Tue, 3=Wed, 4=Thu, 5=Fri, 6=Sat

// Therapy Tuesday check (every 2 weeks from 2026-03-10) const therapyStart = moment("2026-03-10"); const weeksDiff = moment().diff(therapyStart, "weeks"); const isTherapyTuesday = dayOfWeek === 2 && weeksDiff >= 0 && weeksDiff % 2 === 0;

let schedule = "";

if (dayOfWeek === 1 || dayOfWeek === 5) { // Monday & Friday schedule = `## 🗓 Today's Schedule

- 06:30 Wake Up
- 06:30 Quality Time with Baby
- 07:00 Morning Journal + Stretch
- 07:15 Prep Breakfast
- 07:30 Breakfast + Vitamins
- 08:00 Quality Time with Baby (Julia eats)
- 09:00 Work starts
- 13:00 Lunch + Vitamins
- 15:45 Review Tomorrow's Calendar
- 16:00 Housekeeping
- 16:30 Career Growth (Read / Listen / LinkedIn)
- 17:00 Free block / ad hoc tasks
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time / baby routine
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`;

} else if (dayOfWeek === 2 && isTherapyTuesday) { // Therapy Tuesday schedule = `## 🗓 Today's Schedule (Therapy Day 🧠)

- 06:30 Wake Up
- 06:30 Quality Time with Baby
- 07:00 Morning Journal + Stretch
- 07:15 Prep Breakfast
- 07:30 Breakfast + Vitamins
- 08:00 Quality Time with Baby (Julia eats)
- 09:00 Work starts
- 13:00 Lunch + Vitamins
- 15:45 Review Tomorrow's Calendar
- 16:00 Housekeeping
- 16:45 Leave for Therapy
- 17:00 Therapy
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time / baby routine
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`;

} else if (dayOfWeek === 2) { // Regular Tuesday schedule = `## 🗓 Today's Schedule

- 06:30 Wake Up
- 06:30 Quality Time with Baby
- 07:00 Morning Journal + Stretch
- 07:15 Prep Breakfast
- 07:30 Breakfast + Vitamins
- 08:00 Quality Time with Baby (Julia eats)
- 09:00 Work starts
- 13:00 Lunch + Vitamins
- 15:45 Review Tomorrow's Calendar
- 16:00 Housekeeping
- 16:30 Run 🏃
- 17:00 Career Growth (Read / Listen / LinkedIn)
- 17:30 Free block / ad hoc tasks
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time / baby routine
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`;

} else if (dayOfWeek === 3 || dayOfWeek === 4) { // Wednesday & Thursday schedule = `## 🗓 Today's Schedule

- 06:30 Wake Up
- 06:30 Quality Time with Baby
- 07:00 Morning Journal + Stretch
- 07:15 Prep Breakfast
- 07:30 Breakfast + Vitamins
- 08:00 Quality Time with Baby (Julia eats)
- 09:00 Work starts
- 13:00 Lunch + Vitamins
- 15:45 Review Tomorrow's Calendar
- 16:00 Housekeeping
- 16:30 Workout 💪
- 17:00 Career Growth (Read / Listen / LinkedIn)
- 17:30 Free block / ad hoc tasks
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time / baby routine
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`;

} else if (dayOfWeek === 6) { // Saturday schedule = `## 🗓 Today's Schedule

- 07:00 Wake Up
- 07:30 Breakfast + Vitamins
- 08:30 Housekeeping batch (Laundry, Trash, Mop, etc.)
- 11:00 Workout 💪
- 12:30 Lunch + Vitamins
- 14:00 Free block / family time
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time / Date Night with Julia (biweekly)
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`;

} else if (dayOfWeek === 0) { // Sunday schedule = `## 🗓 Today's Schedule

- 07:00 Wake Up
- 07:30 Breakfast + Vitamins
- 08:30 Housekeeping batch (Deep cleans, Meal Prep, etc.)
- 11:00 Weekly Review + Budget Check
- 12:30 Lunch + Vitamins
- 14:00 Portfolio / Side Project
- 17:00 Call Mother + Massagem Julia
- 18:00 Dinner prep
- 18:30 Dinner + Vitamins
- 19:00 Family time
- 21:00 Walk Tofu
- 21:20 Floss + Wind down`; } %>

# 📅 <% tp.date.now("dddd, MMMM DD YYYY") %>

---

<% schedule %>

---

## 🙏 Gratitude

---

## ✅ Tasks

### 💼 Work Tasks

- [ ]
- [ ]
- [ ]

### 📋 Today's Habits & To-Dos

```tasks
not done
tags include #td
due on <% today %>
sort by tags
```