# 🧠 FlowMind AI
### AI-Powered Workflow Intelligence Platform on ServiceNow

[![ServiceNow](https://img.shields.io/badge/ServiceNow-Enterprise-00A591?style=flat-square&logo=servicenow&logoColor=white)](https://www.servicenow.com/)
[![Platform](https://img.shields.io/badge/Platform-Scoped_App-0072C6?style=flat-square)](https://developer.servicenow.com/)
[![Gamification](https://img.shields.io/badge/Feature-Gamification-F5A623?style=flat-square)](#gamification-engine)
[![AI](https://img.shields.io/badge/Feature-AI_Insights-8B5CF6?style=flat-square)](#ai--insights)

---

## 📖 Overview

FlowMind AI is a production-grade, scoped ServiceNow application built for enterprise environments. It combines **smart task management**, **gamification**, **AI-powered insights**, and **team analytics** into a single cohesive platform — making it suitable for internship portfolios, hackathon submissions, and real-world deployment.

The application is designed to feel like a modern SaaS product running natively on the ServiceNow platform, leveraging its full power: GlideRecord, Business Rules, Script Includes, ACLs, Flow Designer, and UI Builder.

---

## ✨ Features

| Feature | Description |
|---|---|
| 📋 Smart Task Management | Create, assign, prioritize, and track tasks with deadlines and status workflows |
| 🎮 XP & Gamification | Earn XP on task completion, level up, track streaks, unlock achievements |
| 🏆 Achievement Badges | Auto-awarded badges based on milestones (task count, XP threshold, streak days) |
| 📊 Team Analytics | Team XP leaderboards, member management, department tracking |
| 🤖 AI Insights | AI-generated productivity tips, burnout warnings, and workflow recommendations |
| 🔐 Role-Based Security | Three-tier role hierarchy with 18 granular ACL rules |
| 📜 XP Transaction Log | Full audit trail of every XP event with source tracking |
| 🔄 Automation | Business rules auto-calculate XP, update levels, and award achievements on task completion |

---

## 🏗️ Architecture

FlowMind AI is built as a **scoped ServiceNow application**, giving it an isolated namespace (`x_[scope]_flowmind`) that prevents conflicts with other apps on the same instance. All metadata lives within this scope.

```
FlowMind AI (Scoped App)
├── Data Layer          → 8 tables, 70+ fields
├── Security Layer      → 3 roles, 18 ACLs
├── Navigation Layer    → 1 app menu, 11 modules
├── Logic Layer         → 3 Script Includes, 4 Business Rules
└── Gamification Engine → XP, Levels, Streaks, Achievements
```

### Why Scoped?

In enterprise environments, multiple teams build applications on the same ServiceNow instance. Scoping prevents accidental collisions in table names, script names, and global variables. It also makes deployment, update sets, and source control significantly cleaner.

---

## 📊 Data Model

### Tables (8 Total)

```
FM User Profile ──────────────────────────────────────
  Fields: user (ref:sys_user), display_name, total_xp,
          current_level, productivity_score, streak_count,
          status, bio, last_active
  Purpose: One gamification profile per user

FM Task (Auto-number: FMT0001001) ────────────────────
  Fields: title, description, assigned_to, created_by,
          priority, status, category, due_date,
          completed_date, xp_reward, estimated_hours,
          actual_hours, notes, ai_summary
  Purpose: Core task management with XP integration

FM Achievement (Auto-number: ACH10001) ───────────────
  Fields: name, description, badge_icon, xp_value,
          category, criteria_type, criteria_value, is_active
  Purpose: Defines achievement conditions and rewards

FM User Achievement ──────────────────────────────────
  Fields: user_profile, achievement, earned_date, awarded_by
  Purpose: Junction table — tracks which users earned which badges

FM XP Log (Auto-number: XPL0001001) ──────────────────
  Fields: user_profile, xp_amount, source_type, source_task,
          source_achievement, description, awarded_date
  Purpose: Full audit trail of every XP transaction

FM Team (Auto-number: TM00101) ───────────────────────
  Fields: name, description, team_lead, department,
          is_active, total_xp, member_count
  Purpose: Team management and leaderboards

FM Team Member ───────────────────────────────────────
  Fields: team, user_profile, role, joined_date
  Purpose: Many-to-many: users belong to teams

FM AI Insight (Auto-number: AI10001) ─────────────────
  Fields: user_profile, insight_type, title, content,
          priority, status, generated_date, is_active
  Purpose: Stores AI-generated recommendations per user
```

### Entity Relationship Diagram

```
sys_user ◄──────────── FM User Profile ◄──── FM XP Log
                              │                    │
                              │                    └── FM Task
                              │
                              ├──── FM User Achievement ──► FM Achievement
                              │
                              └──── FM Team Member ────────► FM Team
```

---

## 🔐 Security Model

### Role Hierarchy

```
x_[scope].admin
    └── x_[scope].manager
            └── x_[scope].user
```

Role hierarchy in ServiceNow means higher roles **inherit all permissions** from lower ones — an admin automatically has everything a manager and user can do.

| Role | Permissions |
|---|---|
| `user` | Read all tables; create/update own tasks and profile |
| `manager` | + Write to teams, XP logs, achievements, insights |
| `admin` | + Create and delete achievement definitions; full CRUD |

### ACL Coverage (18 Rules)

| Table | Read | Write | Create | Delete |
|---|---|---|---|---|
| FM Task | user | user | — | — |
| FM User Profile | user | user | — | — |
| FM XP Log | user | manager | — | — |
| FM User Achievement | user | manager | — | — |
| FM AI Insight | user | manager | — | — |
| FM Team | user | manager | — | — |
| FM Team Member | user | manager | — | — |
| FM Achievement | user | manager | admin | admin |

---

## 🎮 Gamification Engine

### XP Calculation Formula

When a user completes a task, the XP awarded is calculated as:

```
Total XP = (task.xp_reward × priority_multiplier) + streak_bonus
```

**Priority Multipliers:**

| Priority | Multiplier |
|---|---|
| Critical | 2.0× |
| High | 1.5× |
| Medium | 1.0× |
| Low | 0.5× |

**Streak Bonuses:**

| Consecutive Days | Bonus XP |
|---|---|
| 1–2 | +0 |
| 3–6 | +5 |
| 7–13 | +10 |
| 14–29 | +20 |
| 30+ | +50 |

### Level Progression

| Level Range | XP Per Level | Total XP to Reach |
|---|---|---|
| 1–10 | 100 XP | 0 → 1,000 |
| 11–20 | 200 XP | 1,000 → 3,000 |
| 21+ | 500 XP | 3,000+ |

### Achievement Criteria Types

| Type | Triggers When |
|---|---|
| `task_count` | User completes N total tasks |
| `xp_threshold` | User's total XP reaches N |
| `streak_days` | User's streak count reaches N days |
| `manual` | Admin manually awards the badge |

### Gamification Flow

```
User marks task → "Completed"
        │
        ▼
Business Rule fires (after update)
        │
        ├─ Find user's FM User Profile
        ├─ FMXPCalculator.awardXP()
        │       ├─ Create XP Log record
        │       ├─ Update total_xp on profile
        │       └─ Recalculate + update current_level
        │
        ├─ FMStreakTracker.updateStreak()
        │       └─ Increment streak_count
        │
        ├─ Set completed_date timestamp
        │
        └─ FMAchievementChecker.checkAchievements()
                ├─ Evaluate all active achievements
                ├─ If criteria met: create FM User Achievement
                └─ Award bonus XP for earned badge
```

---

## ⚙️ Script Includes

Script Includes are reusable server-side JavaScript classes callable from Business Rules, other Script Includes, or client code via GlideAjax.

### FMXPCalculator
```javascript
// Core XP engine
xpCalc.awardXP(userProfileSysId, xpAmount, sourceType, sourceTaskSysId, description)
xpCalc.calculateLevelFromXP(totalXP)
xpCalc.updateUserLevel(userProfileSysId)
xpCalc.getXPForPriority(priority)  // Returns multiplier: 0.5–2.0
```

### FMStreakTracker
```javascript
// Streak management
tracker.updateStreak(userProfileSysId)         // Increment by 1
tracker.resetStreakIfMissed(userProfileSysId)  // Zero out if no task yesterday
tracker.getStreakBonus(streakCount)            // Returns bonus XP: 0–50
```

### FMAchievementChecker
```javascript
// Achievement evaluation
checker.checkAchievements(userProfileSysId)              // Evaluate all
checker.awardAchievement(userProfileSysId, achievementSysId)
checker.hasEarnedAchievement(userProfileSysId, achievementSysId)
checker.getCompletedTaskCount(userProfileSysId)
```

---

## 🤖 Business Rules

| Rule Name | Table | Trigger | When |
|---|---|---|---|
| Award XP on Task Completion | FM Task | `status` changes to `completed` | After update |
| Set Task Defaults on Create | FM Task | New record created | Before insert |
| Update Team XP | FM User Profile | `total_xp` value changes | After update |
| Update Last Active | FM Task | Record created or updated | After insert/update |

**Why "Before" vs "After":**
- **Before rules** modify the current record _before_ it's written to the database — efficient for setting defaults.
- **After rules** trigger cascading updates on related records _after_ the save is confirmed — used for XP, team scores, and achievements.

---

## 🧭 Navigation

```
FlowMind AI (Application Menu)
├── Tasks                    → List view of FM Task
├── Create Task              → New record form
├── ─── Gamification ───     (separator)
├── My Profile               → FM User Profile list
├── Achievements             → FM Achievement list
├── XP History               → FM XP Log list
├── ─── Teams ───            (separator)
├── Teams                    → FM Team list
├── Team Members             → FM Team Member list
├── ─── AI & Insights ───   (separator)
└── AI Insights              → FM AI Insight list
```

---

## 🚀 Setup & Installation

### Prerequisites

- ServiceNow Personal Developer Instance (PDI) — free at [developer.servicenow.com](https://developer.servicenow.com/)
- Access to App Engine Studio or classic Studio
- Admin rights on the instance

### Installation Steps

1. **Clone this repository**
   ```bash
   git clone https://github.com/yourusername/flowmind-ai.git
   cd flowmind-ai
   ```

2. **Create a Scoped Application**
   - Navigate to: `All > App Engine Studio > Create app`
   - Name: `FlowMind AI`
   - Note the auto-generated scope prefix (e.g., `x_12345_flowmind`)

3. **Replace scope prefix**
   - Find all instances of `x_yourscope` in the codebase
   - Replace with your actual scope prefix

4. **Deploy using Fluent DSL** (if using SDK)
   ```bash
   npm install
   now-cli build
   now-cli deploy
   ```

5. **Or manually recreate** using the step-by-step guide in [`/docs/manual-build-guide.md`](./docs/manual-build-guide.md)

6. **Assign roles** to your admin user:
   - Navigate to: `All > User Administration > Users > [your user]`
   - Add role: `x_[scope].admin`

### First Test

1. Go to `FlowMind AI > My Profile > New` — create a profile linked to your user
2. Go to `Achievements > New` — create "First Blood" (criteria_type: `task_count`, criteria_value: `1`)
3. Go to `Create Task` — fill it out and submit
4. Open the task and change status to `Completed`
5. Check `XP History` — you should see a new XP log entry
6. Check `My Profile` — total_xp and current_level should have updated
7. Check `Achievements` — "First Blood" should now appear in your earned achievements

---

## 📁 Project Structure

```
flowmind-ai/
├── src/
│   └── fluent/
│       ├── tables/
│       │   ├── fm-task.now.ts
│       │   ├── fm-achievement.now.ts
│       │   ├── fm-user-achievement.now.ts
│       │   ├── fm-xp-log.now.ts
│       │   ├── fm-team.now.ts
│       │   ├── fm-team-member.now.ts
│       │   └── fm-ai-insight.now.ts
│       ├── generated/data/table/
│       │   └── fm-user-profile.now.ts
│       ├── security/
│       │   ├── roles.now.ts
│       │   └── acls.now.ts
│       ├── navigation/
│       │   └── menu.now.ts
│       ├── script-includes/
│       │   ├── FMXPCalculator.now.ts
│       │   ├── FMStreakTracker.now.ts
│       │   └── FMAchievementChecker.now.ts
│       └── business-rules/
│           ├── award-xp-on-completion.now.ts
│           ├── set-task-defaults.now.ts
│           ├── update-team-xp.now.ts
│           └── update-last-active.now.ts
├── docs/
│   └── manual-build-guide.md
├── package.json
├── now.config.json
└── README.md
```

---

## 🗺️ Roadmap

### Phase 1 — Foundation ✅
- Scoped application, tables, fields, security, navigation

### Phase 2 — Gamification Engine ✅
- XP system, levels, streaks, achievements, business rules

### Phase 3 — Automation 🔜
- Flow Designer workflows, email notifications, escalation rules, scheduled jobs (daily streak reset)

### Phase 4 — AI Layer 🔜
- OpenAI / Claude API integration for task summarization
- AI-generated productivity insights
- Burnout detection based on streak patterns
- Workflow bottleneck analysis

### Phase 5 — UI/UX 🔜
- UI Builder workspace with modern dashboards
- Service Portal for a consumer-grade experience
- Leaderboard component
- XP progress bars and level badges

### Phase 6 — Analytics 🔜
- Performance Analytics dashboards
- Team vs individual productivity reports
- SLA tracking and breach alerts

### Phase 7 — Enterprise Readiness 🔜
- Update sets for deployment
- Source control integration (GitHub)
- ATF automated test framework
- Documentation site

---

## 💡 Key Concepts Learned

| Concept | What It Is | Why It Matters |
|---|---|---|
| Scoped App | Isolated namespace for app metadata | Prevents conflicts in multi-team instances |
| Reference Field | Foreign key to another table | Creates relational data integrity |
| Choice Field | Controlled dropdown vocabulary | Prevents dirty data, enables reporting |
| Auto-Number | System-generated record IDs (FMT0001001) | Human-readable identifiers for tickets |
| Role Hierarchy | Roles inheriting other roles | Admin inherits manager + user automatically |
| ACL | Rule: who can do what on which table | Enterprise security enforcement |
| Script Include | Reusable server-side JS class | Write once, call from anywhere (DRY) |
| Business Rule | DB trigger: auto-runs on insert/update | Automates workflows without user clicks |
| Before Rule | Runs before record is saved | Modify current record efficiently |
| After Rule | Runs after record is confirmed saved | Trigger cascading updates safely |
| GlideRecord | ServiceNow's ORM for DB queries | The foundation of all server-side scripting |
| GlideAggregate | Optimized aggregate queries (COUNT, SUM) | Efficient for counting/summing without loading all records |

---


---

## 🛠️ Tech Stack

- **Platform:** ServiceNow (Tokyo / Utah / Vancouver or later)
- **Language:** Server-side JavaScript (ES5 compatible)
- **ORM:** GlideRecord, GlideAggregate
- **SDK:** ServiceNow Fluent DSL (`@servicenow/sdk`)
- **Security:** ACL framework, scoped roles
- **Automation:** Business Rules, (upcoming) Flow Designer
- **UI:** (upcoming) UI Builder, Service Portal

---

## 📄 License

MIT License — free to use, modify, and distribute. Attribution appreciated.

---

## 🙋 Contributing

Pull requests welcome! If you're extending this for your own PDI or hackathon project:

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/ai-summarization`
3. Commit your changes: `git commit -m 'Add OpenAI task summarization'`
4. Push and open a PR

---

## 👤 Author

Built as an enterprise-grade portfolio project demonstrating ServiceNow development best practices — suitable for developer internship applications, hackathons, and certification study.

---

> *"The best way to learn enterprise platforms is to build something real on them."*
