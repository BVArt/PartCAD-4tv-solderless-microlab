# PartCAD Troubleshooting Guide

Common issues and solutions for developing and using PartCAD.

## VTK Missing in Sandbox Environments

### Symptom
When inspecting PartCAD assemblies, you see errors like:
```
ERROR: fastener/raisedcheesehead-iso7045: No module named 'vtkmodules.vtkCommonDataModel'
```

The error appears in a sandbox environment path:
```
C:\Users\User\.partcad\sandbox\pc-py-conda-3.11\v-env-*\Scripts\python.exe
```

### Root Cause
PartCAD's isolated sandbox environments (used for executing CAD scripts) don't have VTK properly installed, even if it exists in the main project environment.

### Solution

⚠️ **WARNING: This is a lengthy operation (5–15 minutes)**

1. **Close VS Code completely**
   - The PartCAD extension daemon can interfere with reinstallation

2. **Reinstall PartCAD from source** (in development mode):
   ```powershell
   cd C:\_Seekbirdy_\PartCAD\partcad
   $PythonExe = "C:\_Seekbirdy_\PartCAD\PartCAD-4tv-solderless-microlab\.conda\python.exe"
   & $PythonExe -m pip install -e . --force-reinstall --no-cache-dir
   ```

3. **Clear PartCAD sandbox cache:**
   ```powershell
   Remove-Item "C:\Users\User\.partcad\sandbox\pc-py-conda-3.11" -Recurse -Force -ErrorAction SilentlyContinue
   ```

4. **Reopen VS Code**
   - The extension will restart with fresh sandbox environments

5. **Verify the fix**
   - The next time you inspect assemblies, PartCAD will rebuild sandboxes with all dependencies correctly installed

### Why This Works
- Reinstalling PartCAD from source ensures all dependencies (including VTK) are properly declared
- Clearing the sandbox cache forces PartCAD to recreate isolated environments with correct dependencies
- Fresh VS Code session avoids daemon state conflicts

### Why It's Lengthy
- `pip install -e .` with `--force-reinstall --no-cache-dir` rebuilds everything from source
- VS Code extension needs to fully restart and reinitialize the PartCAD daemon

### Faster Alternative
If you encounter this issue again, try clearing just the sandbox cache first (faster but may not always work):
```powershell
Remove-Item "C:\Users\User\.partcad\sandbox\pc-py-conda-3.11" -Recurse -Force -ErrorAction SilentlyContinue
```
Only do the full reinstall if cache clearing doesn't resolve the issue.

---

## Environment Notes

### PartCAD Sandbox vs. Project venv
**PartCAD Sandbox** (`C:\Users\User\.partcad\sandbox\...`)
- Isolated runtime environment for executing CAD scripts (CadQuery, build123d, etc.)
- Created automatically by PartCAD daemon on-demand
- Ephemeral: safe to clear/recreate without breaking anything

**Project venv** (e.g., `C:\_Seekbirdy_\PartCAD\PartCAD-4tv-solderless-microlab\.conda`)
- Main working environment for the PartCAD project
- Contains PartCAD CLI, daemon service, and project management tools
- Persistent: do not clear; contains your working installation

---

## External STEP File Geometry Not Loading in Renders

### Symptom
When rendering assemblies, PartCAD fails with:
```
EmptyShapesError: No shapes found to render. Please specify valid sketches, parts, or assemblies.
```

This happens even though:
- Individual cadquery parts render fine (e.g., `mason-jar-32oz-wide-mouth.py`)
- External STEP files are accessible on GitHub (HTTP 200 OK)
- Parts correctly reference files via `fileFrom: url` in partcad.yaml

### Root Cause
**Unknown** - Under investigation as of 2026-10-05

Possible causes:
1. PartCAD configuration missing for external file fetching
2. `fileFrom: url` not implemented for rendering (only inspection)
3. Cache/download mechanism not working
4. Geometry files need pre-download or local storage

### What Works
- Rendering cadquery-generated parts (Python CAD scripts)
- URL fetching verification (GitHub files are accessible)
- `pc update` command (completes successfully)

### What Doesn't Work
- Rendering assemblies with external STEP files
- `pc render` with `--with-ports` for coordinate verification
- `pc update` fetching/caching STEP geometry

### Workarounds (Not Yet Tested)

**Option 1: Manual File Download**
1. Download STEP files from GitHub manually
2. Place in PartCAD cache directory (location TBD)
3. Test if local rendering works

**Option 2: Coordinate Tracing Without Rendering**
- Manually trace assembly connections from `.assy` files
- Use interface positions from `partcad.yaml`
- Apply 3D transformation matrices at each connection
- Verify results by comparing known measurements
- Example: reactor-core assembly verified using coordinate tracing to Z=-75

### Investigation Completed
- ✅ Verified GitHub URLs are accessible
- ✅ Confirmed parts use correct `fileFrom: url` syntax
- ✅ Tested `pc update` command
- ✅ Checked cache directories
- ✅ Rendered cadquery parts successfully
- ⏳ Still needed: PartCAD config/docs investigation

### See Also
- Memory file: `.claude/projects/.../memory/4tv_geometry_loading_issue.md`
- Related issue: Assembly coordinate transformation (solved via manual tracing)
- PartCAD version: 0.8.142

---

## Additional Resources
- PartCAD Documentation: `docs/source/contributing.rst`
- Project Instructions: `CLAUDE.md`
- Development Setup: `.devcontainer/devcontainer.json`
