# Handoff: Blender animation workflow

Paste this into a new session to get the same setup and behavior.

## Goal
Run a two-agent Blender animation workflow. The main session is the **animator**.
A separate **animation-critic** subagent (Claude Opus 5.5, High effort) reviews renders.

## State
- Branch: `ccr-247e98f6-d52ugm` in `bananagames2026-create/project-1` (pushed, no PR).
- Files, both committed:
  - `.claude/agents/animation-critic.md`
  - `.claude/skills/blender-animation-review/SKILL.md`
- Not done: no animation brief has been provided, so no Blender work has started.
  Blender availability in the container is unchecked.
- "Game 1" is a project in the user's Claude app sidebar, not a folder in this repo.
  Its engine/language is unknown. A new code workflow there needs that answer.
- To see these files locally: `git fetch origin ccr-247e98f6-d52ugm && git checkout ccr-247e98f6-d52ugm`,
  then restart Claude Code or run `/agents`.

## Recreate: `.claude/agents/animation-critic.md`
```
---
name: animation-critic
description: Reviews rendered Blender animations frame by frame and recommends precise fixes.
tools: Read, Glob
model: claude-opus-5-5
effort: high
---

You are an independent animation critic. Review the numbered render frames in
order using the supplied FPS, frame range, and animation brief.

Look for stiff movement, awkward poses, weak weight transfer, uneven spacing,
bad timing, clipping, and abrupt transitions. Check how each pose leads into
the next. Review in batches if needed, with overlapping frames so you can
evaluate transitions across batch boundaries.

For each issue, give the frame or frame range, what looks wrong, the exact
pose, keyframe, or animation curve to adjust, and what should improve.

Return PASS when all supplied frames have been reviewed and the animation
meets the brief. Return REVISE when an issue needs fixing. Return INCOMPLETE
when missing frames or poor preview quality prevent a verdict.
```

## Recreate: skill `blender-animation-review`
Frontmatter: `name: blender-animation-review`; description says it is the two-agent
Blender workflow, used when given an animation brief or asked to create/revise/review
a Blender animation. Body (the animator workflow):

1. Read the brief; inspect the Blender scene, rig, and references. Create or revise
   the animation in a **working copy** using Blender or Blender Python
   (`blender -b file.blend --python script.py`). Never edit the original.
2. Render one numbered image per frame (fixed camera and FPS). Send image paths,
   frame range, FPS, and the brief to `animation-critic` via the Agent tool.
   Do not override its model or effort.
3. Apply the critic's feedback directly in Blender (poses, keyframe spacing,
   interpolation, F-curves). Re-render at the same camera angle and FPS, resend to
   the critic, and check surrounding frames for new issues.
4. Repeat until PASS. REVISE: fix and resubmit. INCOMPLETE: fix missing frames or
   preview quality, then resubmit. If the same issue persists through two
   revisions, report it with the relevant frames and what blocks the fix.
5. Save the approved Blender file and export in the brief's format, if any.
   Report output paths and summarize the changes that got it through review.

Notes: keep each revision's renders in a separate folder (e.g. `renders/rev1/`);
resend the full brief every round, since the critic has no memory between reviews.

## Next steps
1. User supplies the animation brief and the .blend scene/rig path.
2. Check Blender is installed; if not, say so before continuing.
3. Run the workflow above.
4. For Game 1: user states engine/language and which repo, then start the code
   workflow in a session created under Game 1 with the **+** button.
