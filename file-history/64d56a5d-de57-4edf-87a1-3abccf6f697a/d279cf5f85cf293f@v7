# Plan: Fix Session Duration Loss on Task Resume

## The Bug

When resuming a task, previous session durations are lost. The total time only shows the current session, not sessions 1+2+3.

**From logs:**
```
activeTaskSessionDuration: undefined
activeSessionId: undefined
frontendTotal: 5884, sessionDuration: 5884  // base = 0!
```

## Root Cause

**UpdateTask doesn't return `session_duration` in the response, but GetActiveTask does.**

### GetActiveTask (correct):
```go
// Lines 491-507 - calculates session_duration dynamically
type TaskWithSession struct {
    models.Task
    SessionDuration int `json:"session_duration"`
}
response := TaskWithSession{Task: task}
if len(task.Sessions) > 0 && task.Sessions[0].EndTime == nil {
    response.SessionDuration = int(time.Since(task.Sessions[0].StartTime).Seconds())
}
c.JSON(http.StatusOK, response)
```

### UpdateTask (broken):
```go
// Line 367 - just returns raw task, no session_duration
h.db.Preload("Sessions").First(&task, task.ID)
c.JSON(http.StatusOK, task)  // Missing session_duration!
```

### Frontend expectation:
```typescript
// useTaskTimer.ts lines 305-306
const syncedSessionTime = activeTask.session_duration ?? 0;  // undefined → 0
const base = Math.max(0, (activeTask.duration ?? 0) - syncedSessionTime);  // base = duration - 0 = duration
// But duration was updated to just current session, so base = current session, total = base + 0 = wrong!
```

## The Fix

### File: `backend/internal/handlers/tasks.go`

**Location:** Lines 364-370 (end of UpdateTask)

**Current:**
```go
// Reload with sessions
h.db.Preload("Sessions").First(&task, task.ID)

c.JSON(http.StatusOK, task)
```

**Fixed:**
```go
// Reload with sessions
h.db.Preload("Sessions", func(db *gorm.DB) *gorm.DB {
    return db.Order("session_num ASC")
}).First(&task, task.ID)

// Build response with session_duration (same pattern as GetActiveTask)
type TaskWithSession struct {
    models.Task
    SessionDuration int `json:"session_duration"`
}
response := TaskWithSession{Task: task}

// Calculate current session duration if task is active
if task.IsActive {
    for _, s := range task.Sessions {
        if s.EndTime == nil {
            response.SessionDuration = int(time.Since(s.StartTime).Seconds())
            break
        }
    }
}

c.JSON(http.StatusOK, response)
```

### Also fix CreateTask (same issue)

**Location:** Lines 159-165 (end of CreateTask)

Apply the same pattern - if task is created with `is_active: true`, return `session_duration`.

## Files to Modify

| File | Change |
|------|--------|
| `backend/internal/handlers/tasks.go` | Add session_duration to UpdateTask and CreateTask responses |

## Testing Plan

1. Create a task and run timer for 30 seconds, stop it
2. Resume the task and run for another 30 seconds
3. Total should show ~60 seconds, not just ~30
4. Refresh the page - total should persist correctly
