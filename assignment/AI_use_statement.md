# Generative AI use statement

| Item | Statement |
|---|---|
| Tool | Claude (Anthropic), chat and coding assistant. |
| Purpose | (1) Drafting and debugging Python code for the notebook, following the course practice notebooks for Weeks 7–8 and the clustering practice. (2) Suggesting wording for interpretations and the report. (3) Generating the five claims for the Part 7 claim audit. |
| Prompts | Code and text: requests to build each section of the notebook in the style of the course practices, using the assignment form as the specification. Claim audit: the exact prompt is reproduced in Section 11 of the notebook. |
| Output used | Code structure for the pipeline, temporal split, clustering stability and MLP learning curves; draft text for interpretations; the five claims in Section 11. |
| Verification | I ran every cell and checked each number in the text against the executed output. I checked the leakage decisions against the raw data (e.g. `closed_at` filled for all 326 closed firms). I chose the research question, positive class, threshold rule and k myself and can explain each. The five LLM claims were audited against the notebook; four were not fully supported. |
| Responsibility | All analytical decisions and conclusions are my own responsibility. |
