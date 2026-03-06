Review the current branch changes against the main branch.

If a PR number is provided as $ARGUMENTS, review that PR's diff instead:
`command gh pr diff $ARGUMENTS`

Otherwise, get the local diff:
`git diff $(git merge-base HEAD origin/main)...HEAD`

Read `.claude/review-criteria.md` for the quality bar and what to look for.

For each potential finding, read the relevant source file to verify it's a real issue
before reporting. Only report problems you've confirmed with full context.

If no problems found, say "No issues found."
