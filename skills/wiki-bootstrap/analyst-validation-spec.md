# Analyst Validation Spec

Companion to `SKILL.md` — extracted from Step 2 (Dispatch Analyst) to stay within the 250-line budget.

---

## Analyst Validation

Verify the analyst's response contains all four required sections: `## Plan`, `## Classifications`, `## Risks`, `## Research Plan`. If any section is missing, note which section(s) are absent and re-dispatch the analyst with: "Your response is missing the following required sections: [list]. Please include all four sections." If the second attempt also fails, proceed with available sections and note the gap in the completion report.

**Handle analyst risks:** If the `## Risks` section lists ambiguities, coverage gaps, or potential overlaps that need clarification (anything other than "None"), present the risks to the user and ask for guidance. Re-dispatch the analyst with the user's clarifications if needed.

**WebSearch fallback:** If the analyst reports that WebSearch calls failed or were unavailable, note affected topics in the completion report as "seeded from training knowledge — verify for accuracy." Do NOT stop the bootstrap. Proceed with available knowledge and flag articles for manual review.
