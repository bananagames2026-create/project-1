---
name: blender-animation-review
description: Two-agent Blender animation workflow. You animate in Blender (or Blender Python), render numbered frames, and the animation-critic subagent (Opus 5.5, High effort) reviews them until it returns PASS. Use when the user gives an animation brief or asks to create, revise, or review a Blender animation.
---

# Blender animation + critic workflow

You are the animator. The `animation-critic` subagent
(`.claude/agents/animation-critic.md`, model `claude-opus-5-5`, effort `high`)
is the independent critic. If that agent file is missing, recreate it from the
repo's version before starting.

The user supplies the animation brief. If none was given, ask for it first.

## Steps

1. **Read the brief and inspect** the available Blender scene, rig, and
   references. Create or revise the animation in a **working copy** of the
   .blend file, using Blender or Blender Python (`blender -b file.blend
   --python script.py`). Never edit the original.

2. **Render one numbered image per frame** (e.g. `frame_0001.png`) with a fixed
   camera and FPS. Send the critic the image paths, frame range, FPS, and the
   brief via the Agent tool (`subagent_type: animation-critic`). Do not
   override its model or effort.

3. **Apply the critic's feedback directly in Blender**: edit the relevant
   poses, keyframe spacing, interpolation, or F-curves. Re-render at the
   **same camera angle and FPS**, then send the revision back to the critic.
   Check surrounding frames for issues introduced by the fix.

4. **Repeat until the critic returns PASS.** If the same issue persists
   through two revisions, stop and report it with the relevant frames and
   what is blocking the fix.
   - `REVISE`: fix the listed issues and re-submit.
   - `INCOMPLETE`: fix missing frames or preview quality, then re-submit.

5. **Finish**: save the approved Blender file and export in the format the
   brief specifies, if any. Report the output paths and summarize the changes
   that got the animation through review.

## Notes

- Keep renders low-cost but legible; poor preview quality yields INCOMPLETE.
- Keep each revision's renders in a separate folder (e.g. `renders/rev1/`) so
  the critic never mixes old and new frames.
- Give the critic the full brief on every round; it has no memory of earlier
  reviews unless you resend context.
