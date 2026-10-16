---
name: stacked-diff-discipline
description: Use when splitting dependent work into reviewable PRs, continuing above an unmerged prerequisite, converting existing branches into a stack, or merging a GitHub stack together. Also use when serial PR admission or repeated rebases delay dependent work.
---

# Stacked Diff Discipline

Build dependent changes as small reviewable layers and merge the ready prefix
of the stack together. Continue authoring above an unmerged parent while it is
reviewed. Keep independent work on separate branches or separate stacks.

## Choose the dependency shape

| Observed relationship | Shape |
| --- | --- |
| A later change needs an earlier change's code or contract | Stack, prerequisite at the bottom |
| Changes merely share a tracker, reviewer, CI runner, or ledger file | Separate PRs; use the repository's merge queue for integration |
| Related commits contain distinct dependent concerns | One concern per layer, each tested with its ancestors |
| A change needs an unmerged member of another stack | Coordinate one explicit dependency chain with its owner |

Prefer short stacks, usually two to five layers. Review the decomposition when
the stack grows; this is a planning heuristic, not a platform limit. A layer
may depend on lower layers, never on an unmerged layer above it. Include the
tests needed to establish that layer's behavior with the layer.

## Use native stack support when available

For GitHub, use the official `github/gh-stack` extension and its
[upstream agent skill](https://github.com/github/gh-stack/tree/main/skills/gh-stack)
for command details. Reuse upstream guidance rather than maintaining a second
command catalog. Check installed help when flags or behavior differ.

1. Verify the authenticated repository, trunk, extension version, permissions,
   and native stack availability. All branches must be in the same repository;
   cross-fork stacks are unsupported.
2. Create the chain before implementing every concern on one branch. Each PR
   targets its immediate parent's branch; the bottom targets trunk. Submit and
   verify native membership, order, bases, and heads.
3. Validate each layer against its actual parent and review its incremental
   diff. An upper layer's passing tests do not prove an intermediate layer.
4. Assign one owner for stack mutations. Authors may use separate worktrees;
   serialize rebases, syncs, submissions, and merges across the shared clone.
   Check affected worktrees before a cascade; preserve uncommitted work.
5. Edit the layer that owns a change, then restack affected descendants.
   Record new heads/bases and refresh affected review and CI evidence.

Read [the operating reference](references/github-stacks.md) for an executable
example, migration, CI considerations, and unsupported-host fallback.

## Merge a ready contiguous prefix

Record the intended prefix: repository, trunk SHA, ordered PR URLs, each
base/head SHA, required approvals/checks, chosen method, and highest included
PR. Re-read current GitHub state before execution.

- Every selected PR and every unmerged PR below it must satisfy trunk rules,
  be ready for review, and form a linear stack. A mid-stack PR cannot merge
  alone. If an upper member is blocked, merge only the ready lower prefix.
- Use `gh stack merge <highest-included-pr> --yes` with the repository's
  approved method. Verify the resolved target and selected members; stack and
  PR numbers share the command's target syntax.
- Native stacks do **not** support auto-merge. `submit --auto` generates PR
  metadata; it does not arm auto-merge. Use native stack merge or its queue
  path instead of a loop of `gh pr merge --auto` calls. API clients must use
  GitHub's asynchronous stack merge API.
- Without a merge queue, the selected prefix lands together in one operation.
  With a queue, record queued state and inspect actual resulting groups;
  queue admission is not merge completion.
- A lower-head, base, order, or membership change invalidates the merge plan.
  Reconcile and refresh evidence before retrying. Platform-retained approvals
  do not establish that local reviews/tests apply to changed code.
  While queued, let the queue handle normal trunk advancement and verify its
  current merge-group SHA/checks; do not race it with branch rewrites.

## Separate capacity controls from merge order

Drafts communicate unfinished work. They suppress CI only when the actual
workflow implements draft-aware conditions. Inspect triggers, concurrency,
required checks, runner limits, and `merge_group` coverage. Native stacks apply
trunk protections and PR workflow filtering to every member; upper PRs are not
exempt because their immediate base differs.

Use bounded concurrency, cancellation of superseded runs, and verified
stack-aware job selection to reduce repeated expensive checks. Preserve every
required check and its meaning. When the intended prefix is ready, promote and
qualify all its members within capacity; requiring every parent to merge before
promoting its child defeats group merge. Preserve genuine repository approval,
deployment, and resource gates; identify an obsolete serial policy as a
separate source change instead of bypassing it.

## Verify the result

Read back every included PR's merge state, resulting trunk SHA, and required
post-merge checks. Map each PR to its resulting commit/group and verify the
combined artifact and required deployment or user behavior. Reuse combined
certification only when repository policy allows it, with every member mapped
to that evidence. Check remaining members' new bases/heads after restacking.

Preserve existing PRs, reviews, and recovery refs during conversion; do not
close, squash together, or replace them merely to manufacture a stack.
Distinguish planned, submitted, queued, merged, and verified states.

Sources: [GitHub stack model](https://docs.github.com/en/pull-requests/get-started/about-stacked-prs),
[group merge requirements](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests),
[stack CI](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/optimizing-ci-for-stacked-pull-requests).
