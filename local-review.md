Review the current branch changes against the main branch.

If a PR number is provided as $ARGUMENTS, review that PR's diff instead:
`command gh pr diff $ARGUMENTS`

Otherwise, get the local diff:
`git diff $(git merge-base HEAD origin/main)...HEAD`

If `.claude/review-criteria.md` exists, read it and use those criteria.
Otherwise, apply these defaults:

Report only issues you are confident about:
- Bugs and logic errors
- Security vulnerabilities
- Performance issues
- Missing error handling
- Language anti-patterns

Quality bar:
- Only report issues you'd block a PR for. Skip stylistic nits and minor suggestions.
- If you start describing an issue and realize it's not actually a problem, drop it.
- Each finding must have a concrete file:line reference and a clear explanation of the actual impact.

For each potential finding, read the relevant source file to verify it's a real issue
before reporting. Only report problems you've confirmed with full context.

If no problems found, say "No issues found."
