# Planning phase

The first stage of the SHARP pipeline is the planning phase.
The human and the agent agree on a reproduction plan in a collaborative manner and the `project.json` is created.

## Highlights:

The planning phase starts with the initial prompt that is given to the SHARP agent, it triggers the `\plan` skill that you can find [here](https://github.com/stanford-ai4physics/SHARP/blob/main/.claude/skills/plan/SKILL.md):
> Load the plan skill and create a PRD for replication (incl reimplementation) of arXiv:1902.08570.
> - Running in a container with 1 GPU. Ensure that the code runs on GPU, but also add CPU support as fallback.
> - Only ParticleNet-Lite, only top tagging. Reproduce accuracy, AUC, and 1/eps_b results.
> - Provide smoke-test mode (‘--n-events N‘) that runs train->evaluate->plot on a tiny subset in a short time.
> - Use PyTorch + PyTorch Lightning. Create validation plots of input feature distributions vs physical expectations.
> - Versioned output (‘--version i‘) creating separate results folders. Plots in PDF.
> - Ensure that uncertainties on the final results are handled exactly as in the paper.

After the planning session has ended, we can transform the plan into a project.json via the `\setup` skill:
> Load the setup skill and convert analysis-plan.md to project.json

We are now all set to invoke the execution phase!

## Full transcript (Run 1)

> ### Session 1 — 2026-04-12 17:51
> 
> **Human** &ensp; Load the plan skill and create a PRD for replication (incl reimplementation) of arXiv:1902.08570.
> - Running in a container with 1 GPU. Ensure that the code runs on GPU, but also add CPU support as fallback.
> - Only ParticleNet-Lite, only top tagging. Reproduce accuracy, AUC, and 1/eps_b results.
> - Implement chunked data loading. 
> - Provide smoke-test mode (`--n-events N`) that runs train→evaluate→plot on a tiny subset in a short time.
> - Use PyTorch + PyTorch Lightning. Create validation plots of input feature distributions vs physical expectations.
> - Versioned output (`--version i`) creating separate results folders. Plots in PDF.
> - Ensure that uncertainties on the final results are handled exactly as in the paper.
> 
> **Assistant** &ensp; I'll load the plan skill to create the PRD.
> 
> **Assistant** &ensp; I'll start by fetching the paper to understand the details, then ask clarifying questions.
> 
> **Assistant** &ensp; Good, I have the full paper details. Before writing the plan, here are a few clarifying questions:
> 
> **1. Top tagging dataset acquisition:**
   > - A. Download automatically from Zenodo in a law task (the dataset is ~30 GB)
   > - B. Assume it's already available in `source/` and just load it
   > - C. Provide a download task but also support a manual path override
> 
> **2. Number of training runs for uncertainty estimation:**
   > The paper reports 9 trainings with different random seeds (median ± std). Do you want to:
   > - A. Replicate exactly: 9 trainings, report median accuracy run's metrics ± std across all 9
   > - B. Use 5 trainings (you mentioned 5 — lighter on compute, still gives mean ± std)
   > - C. Support configurable `--n-runs N`, default to 5
> 
> **3. Input data format:**
   > The top tagging dataset is distributed as HDF5. For chunked loading:
   > - A. Read directly from HDF5 with chunked access (simple, no conversion step)
   > - B. Convert to a more training-friendly format first (e.g., parquet, memory-mapped numpy), then load in chunks
   > - C. No preference — pick whatever is most robust
> 
> **4. Validation plots — what level of detail?**
   > - A. Just input feature distributions (the 7 kinematic variables) for signal vs background, compared to physical expectations
   > - B. Also include training curves (loss, accuracy vs epoch) and ROC curves
   > - C. Full suite: input features, training curves, ROC, score distributions, plus the 1/ε_B vs ε_S curve as in the paper
> 
> **5. Reference comparison in plots:**
   > - A. Just overlay signal vs background from our data (sanity check)
   > - B. Also digitize/hardcode the paper's reported numbers and show them as reference points on our result plots
   > - C. No preference
> 
> **Human** &ensp; 1A, 2A, 3A, 4C, 5A
> 
> **Assistant** &ensp; Now I'll write the analysis plan.
> 
> **Assistant** &ensp; The analysis plan has been saved to `analysis-plan.md`. Here's a summary of the 9 steps:
> 
> | Step | Title | Key Deliverable |
> |------|-------|-----------------|
> | 1 | Paper Specification | `spec.md` with all numerical targets |
> | 2 | Dataset Download | HDF5 files from Zenodo (~30 GB) |
> | 3 | Data Loading & Exploration | Chunked HDF5 loader + input feature validation plots |
> | 4 | Model Implementation | ParticleNet-Lite in PyTorch (2 EdgeConv blocks, k=7, channels 32/64) |
> | 5 | Training Pipeline | Lightning module, AdamW, one-cycle LR, 20 epochs, batch 1024 |
> | 6 | Evaluation | Accuracy, AUC, 1/ε_B @ 30% and 50% signal efficiency |
> | 7 | Multi-Seed Uncertainty | 9 trainings, report median-run metrics ± std (matching paper exactly) |
> | 8 | Result Plots | Full suite: input features, training curves, ROC, scores, rejection, multi-seed summary — all PDF |
> | 9 | Smoke Test | End-to-end with `--n-events 1000` in <10 min |
> 
> Key design decisions based on your answers:
> - **Auto-download** from Zenodo in a law task
> - **9 seeds** (exact paper procedure: median accuracy run ± std across all 9)
> - **Direct HDF5 chunked reading** (no format conversion)
> - **Full plot suite** including training curves, ROC, score distributions, and 1/ε_B vs ε_S
> - **Signal vs background overlays** for sanity checks (no digitized paper reference points)
> 
> Ready to convert this into `project.json` milestones via `/setup` when you are.
> 
> **Human** &ensp; Load the setup skill and convert analysis-plan.md to project.json
> 
> **Assistant** &ensp; No existing `project.json` to archive. Now I'll create the milestone plan.
> 
> **Assistant** &ensp; `project.json` created with 9 milestones:
> 
> | ID | Title | Subagents | Checkpoint |
> |----|-------|-----------|:----------:|
> | M-001 | Paper specification | paper-analyst | yes |
> | M-002 | Dataset download | coder, tester | |
> | M-003 | Data loading and feature exploration | coder, tester | yes |
> | M-004 | ParticleNet-Lite model implementation | coder, tester | |
> | M-005 | Training pipeline | coder, tester | yes |
> | M-006 | Evaluation pipeline | coder, tester | |
> | M-007 | Smoke test end-to-end validation | coder, tester | yes |
> | M-008 | Multi-seed uncertainty estimation (9 seeds) | coder, statistician, tester | yes |
> | M-009 | Full result plots and final comparison | coder, tester | yes |
> 
> Key design choices:
> - **6 checkpoints** at natural human-review points: after spec, after data exploration plots, after training pipeline works, after smoke test passes, after 9-seed runs complete, and after final plots
> - **M-007 (smoke test) before M-008 (9 seeds)** — validates the full pipeline on tiny data before committing to 9×20-epoch training runs
> - **Statistician** involved in M-008 to verify the uncertainty aggregation matches the paper's median ± std methodology exactly
> - Each milestone is scoped to 2–3 law tasks max, fitting one Overwatcher iteration