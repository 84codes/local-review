Review and fix the current branch changes in a loop until clean.

Maximum 5 iterations. Each iteration:

1. Get the diff: `git diff $(git merge-base HEAD origin/main)...HEAD`
2. If `.claude/review-criteria.md` exists, read it. Otherwise use these defaults:
   - Report only: bugs, security vulnerabilities, performance issues, missing error handling, language anti-patterns
   - Only issues you'd block a PR for. Skip stylistic nits.
   - Each finding needs a file:line reference and clear impact description.
3. For each potential finding, read the source file to verify it's real.
4. If "No issues found", stop and report success.
5. Otherwise, fix each confirmed issue. Read the relevant files, make the edits.
6. After fixing, go back to step 1 with the updated diff.

Rules:
- Do not ask for confirmation between iterations. Fix and re-review automatically.
- Do not fix stylistic issues or make improvements beyond what the review found.
- If the same issue persists after 2 fix attempts, skip it and note it in the summary.
- After the loop ends, show a summary: what was found, what was fixed, what remains.

If a PR number is provided as $ARGUMENTS, check out that PR branch first:
`command gh pr checkout $ARGUMENTS`
