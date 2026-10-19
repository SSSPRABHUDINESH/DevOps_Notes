## Ansible

1. **force_handlers:** Forces notified handlers to run even if a later task in the play fails.
2. **flush_handlers:** An action that immediately triggers and runs any accumulated notified handlers right at that specific point in the task list.
3. **ignore_errors, changed_when, failed_when:** Keywords used to override task outcomes by bypassing failures (ignore_errors), custom-defining what counts as a change (changed_when), or custom-defining what constitutes a task failure (failed_when).
4. **block, rescue:** Logical groupings where block runs a set of tasks, and rescue executes error-handling tasks only if a failure occurs inside that block.
5. **always:** A block section containing tasks that are guaranteed to execute regardless of whether the previous tasks succeeded or failed.
6. **max_fail_percentage:** Specifies the maximum percentage of host failures allowed in a batch before Ansible aborts the entire execution for the remaining hosts.
7. **any_errors_fatal:** Immediately halts the entire play execution across all hosts if even a single host encounters a task failure.
8. **serial:** Controls execution pacing by forcing Ansible to process the inventory in small, rolling batches of hosts (e.g., 2 at a time) rather than all at once.
9. **forks:** A configuration setting that defines the maximum number of parallel processes Ansible spawns to connect to target hosts simultaneously.

