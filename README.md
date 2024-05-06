# Epochrony
A recurring task scheduling service.

## Introduction & use cases
This library covers a number of use cases where a task needs to be repeated.
For each use case, 3 tasks (`A`, `B` and `C`) that need to be completed every 3 days are used as examples and visualised with
Gantt charts.
In use cases where tasks are not completed on time, tasks `A`  and `B` indicated delayed tasks and task `C` is completed on time.

### Ideal user behaviour
Ideally, recurring tasks that must be completed within a time window are completed immediately as they become due.
> Note that the completion date of tasks `A`, `B` and `C` are on the same day as the due date in the diagram below.
```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (done): done, a, 2024-01-01, 3d
    Task A due: milestone, 2024-01-01, 0d
    Task A complete: milestone, 2024-01-01, 0d

  %% Task B
  section Task B
    Task B (done): done, b, after a, 3d
    Task B due: milestone, 2024-01-04, 0d
    Task B complete: milestone, 2024-01-04, 0d

  %% Task C
  section Task C
    Task C (done): done, c, after b, 3d
    Task C due: milestone, 2024-01-07, 0d
    Task C complete: milestone, 2024-01-07, 0d
```

### Actual user behaviour
In real-world scenarios, tasks are likely to be delayed beyond due dates.
In the simple case where tasks are completed within an allowed time window, this may look like the following.
Note how for each task `A`, `B` and `C`, the completion date is after the due date, but before the start of the next task.
Task `A` is completed 2 days after becoming due, and task `B` is completed 1 day after becoming due.

```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (wait): active, a1, 2024-01-01, 2d
    Task A (done): done, a2, after a1, 1d
    Task A due: milestone, 2024-01-01, 0d
    Task A complete: milestone, 2024-01-03, 0d

  %% Task B
  section Task B
    Task B (wait): active, b1, after a2, 1d
    Task B (done): done, b2, after b1, 2d
    Task B due: milestone, 2024-01-04, 0d
    Task B complete: milestone, 2024-01-05, 0d

  %% Task C
  section Task C
    Task C (wait): active, c1, after b2, 3d
    Task C due: milestone, 2024-01-07, 0d
    Task C complete: milestone, 2024-01-10, 0d
```

Some tasks may become overdue and depending on the nature of the task, may either be:
- `expired`: the task is dropped in place of a future recurring task, or
- `extended`: the task must still be completed, but the effect on the next occurrence may be:
  - `ignored`: the original start of the next task is kept so the tasks overlap, or
  - `deferred`: the start of the next task will wait until the current task is complete

These scenarios are visualised as follows.

#### Overdue task expires
If a task expires after the end date has passed and becomes overdue, it is dropped and can no longer be completed.
New tasks replace the expired tasks on the expected schedule.
In the following example, tasks `A` and `B` are missed, but `C` is completed.
```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (expired): crit, a1, 2024-01-01, 3d
    Task A due: milestone, 2024-01-01, 0d

  %% Task B
  section Task B
    Task B (expired): crit, b1, after a1, 3d
    Task B due: milestone, 2024-01-04, 0d

  %% Task C
  section Task C
    Task C (wait): active, c1, after b1, 3d
    Task C due: milestone, 2024-01-07, 0d
    Task C complete: milestone, 2024-01-10, 0d
```

#### Overdue task extends but does not affect next occurrence
In this use case, the task (`A`) is allowed to be completed after the start of the next occurrence of the task (`B`).
The different tasks may either be:
  - `Independent`: tasks can be completed in any order, e.g. due task `B` can be completed before overdue task `A`
  - `Ordered`: task `B` cannot be completed until task `A` has been completed, task `C` cannot be completed until task `B` has been completed, etc.
```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (wait): active, a1, 2024-01-01, 3d
    Task A (overdue): crit, a2, after a1, 2d
    Task A due: milestone, 2024-01-01, 0d
    Task A complete: milestone, 2024-01-06, 0d

  %% Task B
  section Task B
    Task B (wait): active, b1, after a1, 3d
    Task B (overdue): crit, b2, after b1, 3d
    Task B due: milestone, 2024-01-04, 0d
    Task B complete: milestone, 2024-01-10, 0d

  %% Task C
  section Task C
    Task C (wait): active, c1, after b1, 3d
    Task C due: milestone, 2024-01-07, 0d
    Task C complete: milestone, 2024-01-10, 0d
```

#### Overdue task extends and defers next occurrence
In this use case, the task (`A`) is allowed to be completed after the scheduled start of the next occurrence of the task (`B`), but the next task (`B`) does not become due until the first task is complete.
> The original start dates (had there been no overdue delay) for tasks `B` and `C` are shown at the end of the chart. Note the extended axis compared to previous examples as the 2-day delay for tasks `A` and `B` accumulate to push out the due date of task `C`.

```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (wait): active, a1, 2024-01-01, 3d
    Task A (overdue): crit, a2, after a1, 1d
    Task A due: milestone, 2024-01-01, 0d
    Task A complete: milestone, 2024-01-05, 0d

  %% Task B
  section Task B
    Task B (wait): active, b2, after a2, 3d
    Task B (overdue): crit, b3, after b2, 1d
    Task B due (defer): milestone, 2024-01-05, 0d
    Task B complete: milestone, 2024-01-09, 0d

  %% Task C
  section Task C
    Task C (wait): active, c1, after b3, 3d
    Task C due (defer): milestone, 2024-01-09, 0d
    Task C complete: milestone, 2024-01-12, 0d

  %% Task B (def)
  section Task B (original)
    Task B (wait, original): done, bs2, after a1, 3d
    Task B due (original): milestone, 2024-01-04, 0d

  %% Task C
  section Task C (original)
    Task C (wait, original): done, cs1, after bs2, 3d
    Task C due (original): milestone, 2024-01-07, 0d
```

## Task configuration
This section details how a task can be configured for the above use cases.
A single task is presented in the examples.
### Escalation
A previous use case allowed tasks to become overdue.
This is a single escalation.
A task may be configured to have multiple escalations of varying durations.
Escalations attributes are customisable.
In the example below, task `A` should initially be completed in a 3-day window.
If this is not completed, then:
- after 2 days, the task should be escalated to `highlight`
- after a further 2 days, the task should be escalated to `warning`
- after another 2 days, the task should be escalated to `danger`

The last escalation may optionally either:
- expire the task
- repeat indefinitely

```mermaid
gantt
  %% Config
  dateFormat YYYY-MM-DD
  tickInterval 1day

  %% Task A
  section Task A
    Task A (initial wait, missed): crit, a1, 2024-01-01, 3d
    Task A due: milestone, 2024-01-01, 0d

    Task A (highlight, missed): crit, b1, after a1, 2d
    Task A highlighted: milestone, 2024-01-04, 0d

    Task A (warning, missed): crit, c1, after b1, 2d
    Task A in warning: milestone, 2024-01-06, 0d

    Task A (danger, missed): crit, d1, after c1, 2d
    Task A in danger: milestone, 2024-01-08, 0d
```
