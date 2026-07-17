# Protein Design Practice Environment — Setup Guide

**Goal:** get all the accounts, bookmarks, and local folder structure in place so we can start
running structure prediction (AlphaFold Server / ColabFold) and design (RFdiffusion /
ProteinMPNN) practice runs on free cloud GPUs, then pull the results into local PyMOL for
inspection.

No local GPU is required for anything in this guide — the heavy compute runs on Google's
infrastructure via Colab, or on DeepMind's servers via AlphaFold Server. Your machine only needs
to handle PyMOL, which is lightweight.

---

## 1. Accounts you need

| Service | Why | Link |
|---|---|---|
| Google account | Required to run any Colab notebook and to use AlphaFold Server | you likely already have one |
| AlphaFold Server | Free, no-code AF3 predictions (proteins, ligands, DNA/RNA, complexes) — 30 jobs/day | [alphafoldserver.com](https://alphafoldserver.com) |

That's it — ColabFold, ColabDesign (RFdiffusion + ProteinMPNN) don't need separate registration,
just a Google account to run the notebook in your browser.

**Action:** Log into [alphafoldserver.com](https://alphafoldserver.com) with your Google account once now,
just to confirm access works before we need it.

---

## 2. Bookmark the notebooks we'll use

We'll work through these in order over the next few sessions. Bookmark all three now so they're
one click away:

1. **ColabFold (AlphaFold2-based structure prediction)**
   [colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb)
   — paste a sequence, get a predicted structure + pLDDT/PAE confidence metrics.

2. **RFdiffusion (backbone generation) / ProteinMPNN (sequence design)**
   Both are bundled under the ColabDesign project:
   [github.com/sokrypton/ColabDesign](https://github.com/sokrypton/ColabDesign)
   (has direct Colab links for each tool in the README)

3. **AlphaFold Server (web UI, no notebook)**
   [alphafoldserver.com](https://alphafoldserver.com)
   — use this later for complexes/ligands once we've covered plain protein folding.

**Note on Colab sessions:** free-tier Colab gives you a GPU (usually an NVIDIA T4) for a limited
session window (a several-hour ceiling, and it'll disconnect if idle). This is fine for the
scale we're working at — a single small-protein fold or design run takes minutes, not hours —
but don't expect a notebook to keep running unattended overnight.

---

## 3. Local directory structure

We'll pull every structure file (`.pdb` / `.cif`) down from Colab/AlphaFold Server and store it
locally so PyMOL can load it and so we build up a reference set as we go. Suggested layout,
matching your existing kebab-case convention:

```bash
# Create the working directory tree for this practice project
# -p flag: create parent directories as needed, and don't error if they already exist
# https://man7.org/linux/man-pages/man1/mkdir.1.html
mkdir -p ~/protein-design-practice/{predictions,designs,sequences,notes}
```

- `predictions/` — outputs from ColabFold / AlphaFold Server (folded structures)
- `designs/` — outputs from RFdiffusion (backbones) and ProteinMPNN (sequences)
- `sequences/` — input FASTA files we're testing with
- `notes/` — your own observations as we go (pLDDT ranges you saw, what worked, what didn't)

Each time you download a result from Colab, save it into the matching subfolder with a
descriptive kebab-case filename, e.g. `predictions/lysozyme-colabfold-run1.pdb`.

---

## 4. Verify PyMOL is ready

You already have `pymol-env` set up via Miniforge. Quick check that it still activates cleanly
before we need it:

```bash
# Activate the existing conda environment for PyMOL
# (created previously via Miniforge — no need to recreate it)
conda activate pymol-env

# Launch PyMOL to confirm it opens without errors, then quit
pymol -c -q -d "print('pymol ok'); quit"
# -c : run in command-line/headless mode (no GUI window)
# -q : quiet startup banner
# -d : run the quoted PyMOL command string immediately, then exit
```

If that prints `pymol ok` without errors, we're set. If it fails, let me know the error and
we'll fix the environment before moving on — no point starting the fun part on a broken install.

---

## 5. What's next

Once accounts are confirmed and the folder structure exists, our first practice run will be:

1. Pick a small, well-characterized protein (something like lysozyme, ~130 residues) as a
   sanity-check target.
2. Fold it in ColabFold, save the output to `predictions/`.
3. Open it in PyMOL, color by confidence (`spectrum b`, since AF2/AF3 store pLDDT in the
   B-factor column), and get a feel for reading pLDDT/PAE before we move on to design work.

We'll do that step together in the next session — this guide just gets the groundwork in place.