# PyMOL Cheat Sheet

Reference: full command index at https://pymolwiki.org/index.php/Category:Commands

## Loading & Fetching Structures

```
fetch 1lyz, async=0        # download structure from RCSB PDB by 4-char ID
load /path/to/file.pdb     # load a local file (.pdb, .cif, .mol2, .sdf, etc.)
save output.pdb, 1lyz      # save an object to a file
```
Docs: https://pymolwiki.org/index.php/Fetch | https://pymolwiki.org/index.php/Load

## Object & Selection Basics

```
delete all                 # remove everything from the session
delete 1lyz                # remove a specific object
select mysel, chain A      # name a selection for reuse
select active_site, resi 45+52+87   # select specific residue numbers
select near_lig, byres (polymer within 5 of resn LIG)  # residues within 5A of a ligand
deselect                   # clear active selection highlighting
```
Selection algebra docs: https://pymolwiki.org/index.php/Selection_Algebra

Common selection keywords: `chain`, `resi` (residue number), `resn` (residue name),
`name` (atom name), `polymer`, `solvent`, `hetatm`, `byres`, `within`, `around`

## Representations (Show/Hide)

```
show cartoon
show sticks, resn LIG      # sticks for a specific ligand
show surface, chain A
hide lines
hide everything
```
Available reps: `lines`, `sticks`, `cartoon`, `surface`, `spheres`, `ribbon`, `mesh`, `dots`
Docs: https://pymolwiki.org/index.php/Show

## Coloring

```
color skyblue, chain A
color red, resn LIG
spectrum count, rainbow            # N-to-C rainbow gradient
spectrum b, blue_white_red         # color by B-factor
util.cbc                           # color chains distinctly (by chain)
```
Docs: https://pymolwiki.org/index.php/Color | https://pymolwiki.org/index.php/Spectrum

## View & Camera

```
zoom                        # zoom to fit everything in view
zoom mysel, 5                # zoom to a selection with 5A buffer
orient                      # orient view along principal axes
center mysel                # center view on a selection
turn y, 90                   # rotate view 90 degrees around y-axis
reset                       # reset to default view
```
Docs: https://pymolwiki.org/index.php/Zoom | https://pymolwiki.org/index.php/Orient

## Measurements

```
distance d1, /1lyz//A/45/CA, /1lyz//A/52/CA   # distance between two atoms
angle a1, sele1, sele2, sele3
dihedral dh1, sele1, sele2, sele3, sele4
get_area mysel               # solvent accessible surface area
```
Docs: https://pymolwiki.org/index.php/Distance

## Alignment & Comparison

```
align mobile_obj, target_obj          # sequence + structure alignment
super mobile_obj, target_obj          # structure-only alignment (good for low seq similarity)
cealign target_obj, mobile_obj        # CE algorithm, robust for distant homologs
rms_cur mobile_obj, target_obj        # RMSD without re-aligning
```
Docs: https://pymolwiki.org/index.php/Align | https://pymolwiki.org/index.php/Cealign

## Mutagenesis (Wizard)

```
wizard mutagenesis
cmd.get_wizard().do_select("resi 45 and chain A")
cmd.get_wizard().set_mode("ALA")     # choose target residue
cmd.get_wizard().apply()             # apply the mutation
set_wizard done
```
Docs: https://pymolwiki.org/index.php/Mutagenesis

## Rendering & Export

```
bg_color white
set ray_opaque_background, 0        # transparent background on export
ray 1600, 1200                      # high-quality ray-traced render
png ~/figure.png, width=1600, height=1200, dpi=300
```
Docs: https://pymolwiki.org/index.php/Png | https://pymolwiki.org/index.php/Ray

## Sessions & Scripting

```
save session.pse            # save full session (objects, views, settings)
load session.pse             # reload a saved session
run script.py                 # execute a Python script inside PyMOL
```
Docs: https://pymolwiki.org/index.php/Python (PyMOL's Python API — useful since
you're already comfortable scripting; most commands above have a `cmd.<name>()`
Python equivalent, e.g. `cmd.fetch("1lyz")`, `cmd.show("cartoon")`)

## Useful Settings

```
set cartoon_transparency, 0.5
set stick_radius, 0.15
set sphere_scale, 0.25
set antialias, 2
set ray_trace_mode, 1        # cartoon outline / illustrative style
```
Full settings list: https://pymolwiki.org/index.php/Category:Settings