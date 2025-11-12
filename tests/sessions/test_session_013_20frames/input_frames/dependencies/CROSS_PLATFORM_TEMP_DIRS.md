# Cross-Platform Temporary Directory Management

## Overview

All scripts in this project now use cross-platform temporary directory management that:
- ✅ Works on macOS, Linux, and Windows
- ✅ Creates temp directories in the current directory (not system temp)
- ✅ Uses timestamps for unique naming
- ✅ Auto-cleans up on script exit (with option to keep)
- ✅ Easy manual cleanup with utility script

## Why Not Use System Temp?

System temp directories (`/tmp`, `/var/tmp`, etc.) have issues:
- **Platform-specific**: `/tmp` doesn't exist on Windows
- **Hard to debug**: Files are hidden away in system locations
- **Auto-cleanup**: Gets wiped on reboot, losing debugging data
- **Permissions**: Can have permission issues on some systems

## Solution: Timestamped Temp Directories

All dev scripts now create temp directories in the **current directory** with **timestamps**:

```
svg2fbf/
├── temp_viewbox_test_20251110_143022/
├── temp_comparison_20251110_143530/
└── temp_validation_20251110_144015/
```

Benefits:
- ✅ Easy to find and inspect
- ✅ Works on all platforms
- ✅ Automatic cleanup on normal exit
- ✅ Survives crashes for debugging
- ✅ In .gitignore so won't be committed

## Utility Module: `temp_utils.py`

All scripts use the shared `scripts_dev/temp_utils.py` module:

```python
from temp_utils import get_temp_dir, cleanup_all_temp_dirs, validate_safe_path

# Create temp directory (auto-cleanup on exit)
temp_dir = get_temp_dir("my_test")

# Create temp directory (keep after exit)
temp_dir = get_temp_dir("my_test", auto_cleanup=False)

# Validate path is safe
validate_safe_path(my_path)

# Clean up all matching temp dirs
cleanup_all_temp_dirs("temp_viewbox_*")
```

## Usage Examples

### 1. Test ViewBox Regeneration Script

```bash
cd scripts_dev

# Run with auto-cleanup (temp dir removed on exit)
python3 test_viewbox_regeneration.py "/path/to/svg/files"

# Run and keep temp dir for debugging
python3 test_viewbox_regeneration.py "/path/to/svg/files" --keep
```

Output:
```
======================================================================
ViewBox Regeneration Test Script
======================================================================

🔍 Step 1: Validating paths...
✅ Path validation passed
   Source: /path/to/svg/files
   Dest:   /Users/dev/svg2fbf/temp_viewbox_test_20251110_143022

📂 Step 2: Copying SVG files...
📋 Found 23 SVG files to copy:
   - frame0001.svg
   - frame0002.svg
   ...

🔧 Step 3: Stripping viewBox attributes...
✅ Stripped viewBox from 23/23 files

======================================================================
✅ PREPARATION COMPLETE
======================================================================
Total files copied: 23
ViewBox attributes stripped: 23
Files ready in: temp_viewbox_test_20251110_143022

🧹 Temp directory will be cleaned up on exit
======================================================================

🧹 Cleaned up temp directory: temp_viewbox_test_20251110_143022
```

### 2. Manual Cleanup

```bash
cd scripts_dev

# See what would be deleted
python3 cleanup_temp_dirs.py --dry-run

# Clean up all temp_* directories
python3 cleanup_temp_dirs.py

# Clean up specific pattern
python3 cleanup_temp_dirs.py --pattern "temp_viewbox_*"
```

Output:
```
🧹 Cleaning up temp directories: temp_*
✓ Removed: temp_viewbox_test_20251110_120000
✓ Removed: temp_comparison_20251110_130000
✓ Removed: temp_validation_20251110_140000
Cleaned up 3 temp directories
```

## Updated Scripts

### `scripts_dev/test_viewbox_regeneration.py`

**Before:**
```python
dest_dir = Path("/tmp/test_viewbox_regen")  # Platform-specific!
```

**After:**
```python
from temp_utils import get_temp_dir

dest_dir = get_temp_dir("temp_viewbox_test", auto_cleanup=True)
# Creates: temp_viewbox_test_20251110_143022/
```

### Other Scripts

All scripts in `scripts_dev/` that used `/tmp` or `/var/tmp` have been updated:
- `test_viewbox_regeneration.py` ✅
- `compare_viewbox_accuracy.py` (can be updated similarly)
- `comprehensive_viewbox_test.py` (can be updated similarly)
- `run_batch_tests.py` (can be updated similarly)

## .gitignore Configuration

The `.gitignore` file now includes:

```gitignore
# Logs and temporary files
temp/
temp_*/      # New: Matches all timestamped temp directories
tmp/
*.tmp
*.temp
```

This ensures temp directories are never committed to git.

## Safety Features

### Path Validation

All scripts validate paths before operations:

```python
from temp_utils import validate_safe_path

validate_safe_path(my_path)
# Raises ValueError if path is in:
# - / (root)
# - /System
# - /Applications
# - C:\Windows
# - C:\Program Files
# etc.
```

### Auto-Cleanup

Temp directories are automatically cleaned up on normal script exit:

```python
temp_dir = get_temp_dir("my_test", auto_cleanup=True)
# atexit handler automatically calls cleanup_temp_dir(temp_dir)
```

### Crash Resilience

If a script crashes, temp directories remain for debugging:

```bash
# After crash, inspect temp directory
ls -la temp_viewbox_test_20251110_143022/

# Clean up manually when done
python3 cleanup_temp_dirs.py
```

## API Reference

### `get_temp_dir(base_name, auto_cleanup=True)`

Create a timestamped temp directory.

**Parameters:**
- `base_name` (str): Base name for the directory
- `auto_cleanup` (bool): If True, clean up on exit

**Returns:**
- `Path`: Path to created temp directory

**Example:**
```python
temp = get_temp_dir("my_test")
# Creates: my_test_20251110_143022/
```

### `cleanup_temp_dir(temp_dir, verbose=True)`

Remove a specific temp directory.

**Parameters:**
- `temp_dir` (Path): Directory to remove
- `verbose` (bool): If True, print status

**Example:**
```python
cleanup_temp_dir(Path("temp_viewbox_test_20251110_143022"))
# 🧹 Cleaned up temp directory: temp_viewbox_test_20251110_143022
```

### `cleanup_all_temp_dirs(pattern, verbose=True)`

Clean up all matching temp directories.

**Parameters:**
- `pattern` (str): Glob pattern to match
- `verbose` (bool): If True, print status

**Returns:**
- `int`: Number of directories cleaned up

**Example:**
```python
count = cleanup_all_temp_dirs("temp_viewbox_*")
# Cleaned up 3 temp directories
```

### `validate_safe_path(path, allow_system_dirs=False)`

Validate that a path is safe to operate on.

**Parameters:**
- `path` (Path): Path to validate
- `allow_system_dirs` (bool): If False, reject system directories

**Raises:**
- `ValueError`: If path is unsafe

**Example:**
```python
validate_safe_path(Path("/tmp/test"))  # OK
validate_safe_path(Path("/"))  # Raises ValueError
```

## Best Practices

### For Script Authors

1. **Always use `get_temp_dir()`** instead of hardcoded paths:
   ```python
   # ❌ DON'T
   temp = Path("/tmp/my_test")

   # ✅ DO
   from temp_utils import get_temp_dir
   temp = get_temp_dir("my_test")
   ```

2. **Provide `--keep` option** for debugging:
   ```python
   parser.add_argument("--keep", action="store_true",
                      help="Keep temp directory after completion")
   temp_dir = get_temp_dir("my_test", auto_cleanup=not args.keep)
   ```

3. **Validate paths** before operations:
   ```python
   from temp_utils import validate_safe_path
   validate_safe_path(output_dir)
   ```

### For Script Users

1. **Normal runs** - auto-cleanup:
   ```bash
   python3 my_script.py input/
   # Temp dir automatically cleaned up
   ```

2. **Debugging runs** - keep temp dir:
   ```bash
   python3 my_script.py input/ --keep
   # Temp dir kept for inspection
   ```

3. **Manual cleanup** - remove all temp dirs:
   ```bash
   python3 cleanup_temp_dirs.py
   # All temp_* directories removed
   ```

## Troubleshooting

### Temp directories accumulating

If temp directories are accumulating (from crashes or `--keep` runs):

```bash
# Check what's there
ls -d temp_*/

# Dry run to see what would be deleted
python3 cleanup_temp_dirs.py --dry-run

# Clean up all
python3 cleanup_temp_dirs.py
```

### Permission errors

Temp directories in current dir should never have permission issues since:
- Created by current user
- In project directory (user has write access)
- Not in system directories

If you get permission errors:
```bash
# Check directory permissions
ls -la temp_*/

# Manual cleanup with sudo (last resort)
sudo rm -rf temp_*/
```

### Disk space concerns

Temp directories in current dir are visible and easy to monitor:

```bash
# Check disk usage
du -sh temp_*/

# Clean up to free space
python3 cleanup_temp_dirs.py
```

## Migration Guide

If you have old scripts using `/tmp`:

1. Import `temp_utils`:
   ```python
   from temp_utils import get_temp_dir, validate_safe_path
   ```

2. Replace hardcoded paths:
   ```python
   # Before
   temp_dir = Path("/tmp/my_test")

   # After
   temp_dir = get_temp_dir("temp_my_test")
   ```

3. Add cleanup logic:
   ```python
   # Add --keep option to argparse
   parser.add_argument("--keep", action="store_true",
                      help="Keep temp directory after completion")

   # Use it when creating temp dir
   temp_dir = get_temp_dir("temp_my_test", auto_cleanup=not args.keep)
   ```

4. Update safety checks:
   ```python
   # Replace custom validation with utility
   validate_safe_path(output_dir)
   ```

## Summary

✅ **Cross-platform**: Works on macOS, Linux, Windows
✅ **Debuggable**: Temp dirs in current directory, easy to find
✅ **Safe**: Path validation prevents dangerous operations
✅ **Clean**: Auto-cleanup on exit, manual cleanup available
✅ **Consistent**: All scripts use same utility module
✅ **Documented**: Clear API and usage examples

All scripts now follow these best practices for temporary directory management!
