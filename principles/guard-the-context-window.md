# Guard the Context Window

**Why:** Context overflow degrades reasoning quality, creates compression artifacts, and halts progress.

**Pattern:**
- **Size phases and cap scope.** Limit files per phase, set turn budgets, account for mechanism costs.
- **Keep bulk out of the conversation.** Read excerpts with search and line ranges, write large outputs to files, and carry summaries forward instead of raw payloads. Hand bulk to subagents only when the user asked you to dispatch.
