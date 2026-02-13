# aTrain Flatpak Build Guide

This document describes every step from the upstream GitHub repositories https://github.com/JuergenFleiss/aTrain and https://github.com/JuergenFleiss/atrain_core to a running aTrain Flatpak, including all manual changes required along the way.

## Overview

The build takes two upstream repositories and packages them as a Flatpak application:

- **aTrain** (GUI): https://github.com/JuergenFleiss/aTrain (branch: `feature_flatpak`)
- **aTrain_core** (backend): https://github.com/JuergenFleiss/atrain_core (branch: `feature_flatpak`)

The resulting Flatpak uses the GNOME 49 runtime (`org.gnome.Platform//49`).

---

## Prerequisites

Install the following on your build machine:

```
sudo apt install flatpak flatpak-builder
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

You also need:

- **uv** 
- **req2flatpak**:
- **Git**

---

## Step 1 — Set Up the Working Environment
Clone the flatpak packaging repository:

```
git clone git@github.com:JuergenFleiss/atrain2flatpak.git
```

---

## Step 2 — Clone the aTrain Source

Clone the aTrain application source into the `atrain/` subdirectory (the Flatpak manifest references it as a local directory source):

```
mkdir atrain
cd atrain
git clone git@github.com:JuergenFleiss/aTrain.git .
```

This gives you the upstream `pyproject.toml` which includes the backend `aTrain_core@TARGETBRANCH`.


## Step 4 — Resolve Dependencies with `uv sync`

From inside the `atrain/` directory:

```
uv sync
```

This creates a `.venv/` directory and a `uv.lock` file with fully resolved dependencies, pulling `torch==2.9.1+cu128` from the PyTorch index and everything else from PyPI.

---

## Step 5 — Generate `requirements.txt`

Generate requirements.txt form uv.lock

```
uv pip freeze > requirements.txt
```


```
aiofiles==25.1.0
...
torch==2.8.0+cu128
torchaudio==2.8.0+cu128
...
```

---

## Step 6 — MANUAL Edit `requirements.txt` (Remove Editable Lines Only)

**Manual change:** Remove all `+cu128` suffixes .

**Manual change:** remove any editable `-e file://...` lines (local paths).

**Manual change:**  Remove aTrain_core

---

## Step 7 — Generate `atrain_python_dependencies.json` with `req2flatpak`

Run `req2flatpak` to generate the Flatpak sources manifest for all pip packages:

```
cd ~/github/flatpak_venv/atrain_flatpak/atrain
req2flatpak --requirements-file requirements.txt --target-platforms 313-x86_64 > ../atrain_python_dependencies.json
```

- `313-x86_64` we build with CPython 3.13 on x86_64 (matching GNOME 49 SDK's Python version).


---

## Step 8 — Manually Edit `atrain_python_dependencies.json` (Minimal Changes) for Torch/Cuda wheels

To document still...


---


---

## Step 11 — Build the Flatpak

From the `atrain2flatpak/` directory:

```
flatpak-builder --force-clean --install --user build-dir io.github.JuergenFleiss.aTrain.yml
```
