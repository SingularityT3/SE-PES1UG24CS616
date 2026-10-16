# Lab 5: Issues Found

**Name:** Sathvik Karthik Malali
**SRN:** PES1UG24CS616

| Issue | Type | Line(s) | Description | Fix Approach |
|-------|------|---------|-------------|--------------|
| Mutable default argument (Pylint W0102) | Bug | 9 | `zone_records=[]` is created once and shared across every call, so records leak between calls. | Default to `None` and only append when a list is passed in. |
| Bare `except:` (Pylint W0702, Flake8 E722) | Bug / Security | 16, 22, 29, 43 | Catches everything, including `KeyboardInterrupt`, and hides real errors (`register_permit` silently returns `None`). | Removed the try blocks where no error can occur; catch specific exceptions (`KeyError`, `ValueError`, `OSError`) elsewhere. |
| Try / except / pass (Bandit B110) | Security | 19-23 | `revoke_permit` swallows every error silently. | Replaced with `permits.pop(vehicle_plate, None)`. |
| Weak PRNG for permit numbers (Bandit B311) | Security | 11 | `random.randint` is predictable, so permit numbers can be guessed. | Use `secrets.randbelow` for permit numbers. |
| Unvalidated zone and plate (logic) | Bug | 9-17, 56-59 | Zone `"Z"`, an empty plate and `None` are accepted as valid permits (3 permits on file instead of 1), and `None` becomes a `"null"` key in the JSON. | Reject empty plates and zones not in `ZONES`; return `None`. |
| Wrong date format accepted (logic) | Bug | 51-52, 60 | `setExpiry("...", "01/01/2020")` stores a string `is_expired` cannot parse, so the bare except returns False and an expired permit looks valid. | Validate with `strptime("%Y-%m-%d")`, return False on bad input; demo uses `2020-01-01`. |
| `setExpiry` on unknown plate (logic) | Bug | 51-52 | Raises `KeyError` if the plate has no permit. | Check membership first and return False. |
| Unclosed file handles (Pylint R1732) | Bug | 42, 47 | `open()` without `with` can leak the handle if an error occurs. | Use `with open(...)`. |
| Missing encoding on `open` (Pylint W1514) | Bug | 42, 47 | Uses the platform default encoding, which varies between systems. | Pass `encoding="utf-8"`. |
| Global state / `global` statement (Pylint W0603) | Design | 6, 40 | Module-level `permits` is rebound inside `load_permit_database`, which makes functions hard to test. | `load_permit_database` returns the dict and all functions take `permits` as a parameter. |
| `main()` runs on import | Design | 68 | Importing the module executes the demo. | Guard with `if __name__ == "__main__":`. |
| Invalid function name `setExpiry` (Pylint C0103) | Style | 51 | Not snake_case. | Renamed to `set_expiry`. |
| Missing docstrings (Pylint C0114, C0116) | Style | 1, all functions | No module or function documentation. | Added a docstring to the module and every function. |
| Line too long (Pylint C0301, Flake8 E501) | Style | 13, 27, 62 | Lines exceed 79 characters. | Wrapped the long lines. |
| Blank lines around functions (Flake8 E302, E305) | Style | 9-68 | Needs 2 blank lines between top-level definitions. | Added the blank lines. |

## Results

| Tool | Before | After |
|------|--------|-------|
| Pylint | 6.32/10 | 10.00/10 |
| Bandit | 2 issues (Low) | 0 issues |
| Flake8 | 16 warnings | 0 warnings |
