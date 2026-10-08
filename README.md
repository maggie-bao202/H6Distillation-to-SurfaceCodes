# H6Distillation-to-SurfaceCodes

One notebook, [`notebooks/h6_distill_and_teleport.ipynb`](notebooks/h6_distill_and_teleport.ipynb).
It distills two `|+⟩` states with the `[[6,2,2]]` H6 code (Quantinuum's Magic-H6
"0-level distillation", arXiv:[2506.14688](https://arxiv.org/abs/2506.14688)), then
teleports each distilled logical qubit into its own rotated surface code block.

The circuits live in LightStim (`feat/h6-distillation-protocol` branch):

- `lightstim.protocols.h6_distillation`: the H6 encoder plus Bell-pair check.
- `lightstim.protocols.h6_teleport`: distill, then teleport each slot into a surface block
  through a repeated joint `X_L(H6 slot i) ⊗ X_L(S_i)` measurement.

This repo only runs and analyses them.

## What the notebook shows

1. Distillation alone: fault distance 2, post-selected error scaling as `p²`.
2. The distill + teleport circuit at `d = 3`: no detector fires without noise.
3. Fault distance 2 for the full protocol. Removing the Bell-pair check or the repeated
   joint measurement drops it to 1.
4. Per-block `X_L` error rate vs `p` at `d = 3` and `d = 5`, against two references:
   teleporting without distillation, and `|+⟩_L` prepared directly in the surface code.
5. Acceptance rate after post-selection.
6. How the number of joint-measurement repeats trades error rate against acceptance.
   The teleport, not the distillation, is what limits the output.

## Read this before using the results

- **H-state, not T-state.** The real protocol distills `|H+⟩ = cos(π/8)|0⟩ + sin(π/8)|1⟩`,
  the Hadamard +1 eigenstate. It's single-candidate verify-and-discard ("0-level
  distillation"), not many-copies-to-one.
- **Clifford proxy.** Stim can't simulate `|H+⟩`, so `|+⟩` stands in for it, as in the
  paper's Section III. This tests circuit-level fault tolerance and error scaling, not
  the fidelity of a real magic state.
- **Unflagged joint measurement.** Each teleport uses one ancilla over a weight-`3 + d`
  operator. Under the proxy the ancilla's hook faults are invisible (they're X errors),
  but a real magic state would need flags or a lattice-surgery-style merge.

## Setup

LightStim is not on PyPI.

**Local development (sibling checkout):**
```bash
python3 -m venv .venv
.venv/bin/pip install -e ../LightStim   # assumes LightStim checked out alongside this repo
.venv/bin/pip install -e . --no-deps
.venv/bin/pip install matplotlib jupyter
```

**From the pinned branch** (what `pyproject.toml` declares):
```bash
python3 -m venv .venv
.venv/bin/pip install -e ".[notebook]"
```
This pulls `lightstim` from `maggie-bao202/LightStim@feat/h6-distillation-protocol` over SSH.

Then open the notebook:
```bash
.venv/bin/jupyter lab notebooks/h6_distill_and_teleport.ipynb
```
A full run takes about a minute.

## The `[[6,2,2]]` code's canonical logical convention

```
S^X_1 = X0 X1 X2 X3      S^Z_1 = Z0 Z1 Z2 Z3
S^X_2 = X2 X3 X4 X5      S^Z_2 = Z2 Z3 Z4 Z5
X0_L  = X0 X2 X4         Z0_L  = Z0 Z2 Z4
X1_L  = X1 X3 X5         Z1_L  = Z1 Z3 Z5
```

Self-dual (`Hx == Hz`): transversal H is logical H, transversal S is logical S.
