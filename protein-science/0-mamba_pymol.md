# Installing Mamba (Miniforge) + PyMOL on Ubuntu

Context: Ubuntu's apt `pymol` package (2.5.0+dfsg) is built against an old
Python and breaks on Python 3.12+ (`ModuleNotFoundError: No module named 'imp'`,
since `imp` was removed in Python 3.12). The fix is to install the actively
maintained `pymol-open-source` build from conda-forge instead, isolated in its
own mamba environment.

## 1. Remove the broken apt package (if installed)

```bash
sudo apt-get remove --purge pymol pymol-data python3-pymol
sudo apt-get autoremove
```

## 2. Check architecture

```bash
uname -m
# x86_64 = standard Intel/AMD 64-bit
# aarch64 = ARM 64-bit
```

## 3. Download and install Miniforge

Miniforge is the conda-forge-first distribution (defaults to the conda-forge
channel, unlike classic Anaconda). Ships `conda` + `mamba`.

```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"
bash Miniforge3-Linux-x86_64.sh
```
- `-L` = follow redirects (GitHub's "latest" link redirects to a versioned URL)
- `-O` = save using the remote filename
- Accept the license, keep the default install path (`~/miniforge3`) unless you
  have a reason not to, and say **yes** when asked to run `conda init`.

```bash
source ~/.bashrc     # reload shell config to pick up conda/mamba
mamba --version       # verify install
```

Docs: https://github.com/conda-forge/miniforge

## 4. Create an isolated environment for PyMOL

```bash
mamba create -n pymol-env python=3.11
mamba activate pymol-env
```
- Isolating in its own env avoids dependency conflicts with other projects.
- Python 3.11 is currently the best-tested version for the
  `pymol-open-source` conda-forge build.

Docs: https://mamba.readthedocs.io/en/latest/user_guide/mamba.html

## 5. Install PyMOL

```bash
mamba install -c conda-forge pymol-open-source
```
- `-c conda-forge` = pull specifically from the conda-forge channel, where the
  maintained open-source build lives.

Docs: https://github.com/schrodinger/pymol-open-source

## 6. Launch and verify

```bash
mamba activate pymol-env
pymol
```

Quick smoke test inside PyMOL's command line:
```
fetch 1lyz, async=0      # download a small test structure from RCSB PDB
show cartoon
hide lines
spectrum count, rainbow
png ~/pymol_test.png, width=800, height=600, dpi=150
```
Then from a regular terminal: `ls -lh ~/pymol_test.png` to confirm the file
was written.

Docs:
- Fetch: https://pymolwiki.org/index.php/Fetch
- Spectrum: https://pymolwiki.org/index.php/Spectrum
- Png export: https://pymolwiki.org/index.php/Png
- Full command reference: https://pymolwiki.org/index.php/Category:Commands
- Scripting/Python API: https://pymolwiki.org/index.php/Python

## Notes on managing base env alongside existing venv projects

If most other projects already use Python's built-in `venv` and you only want
mamba for PyMOL, stop `base` from auto-activating on every new terminal:

```bash
conda config --set auto_activate_base false
```
- Edits `~/.condarc`. The `conda`/`mamba` commands remain available — this
  only stops the `base` *environment* from auto-activating.
- Existing `venv` workflows (`python3 -m venv venv`, `source venv/bin/activate`)
  are unaffected.
- To re-enable: `conda config --set auto_activate_base true`

Day-to-day commands:
```bash
mamba env list           # list all environments, marks active with *
mamba activate <name>    # switch into an environment
mamba deactivate          # step back out (e.g. pymol-env -> base, or fully out)
mamba create -n <name> python=<version>   # new env for a future project
```