# eSim Winter Internship 2026 — Task 4 Report

**Contributor:** Poonam Gupta  
**Date:** 23 November 2026  
**Environment:** Ubuntu 25.04 (WSL2)

## 1. Objective

Investigate eSim 2.5 installer and dependency compatibility issues on Ubuntu 25.04, identify their root causes, implement fixes, and validate the changes.

The investigation also covered the NGHDL dependency installer because its GHDL build process depends on compatible LLVM and Clang versions.

## 2. Issue 1 — eSim Ubuntu version detection and installer dispatch

### Observed problem

The original installer failed to correctly detect Ubuntu 25.04.

The previous version-detection logic extracted a version using a pattern expecting three numeric components. Ubuntu 25.04 reports a two-component version, so the detected version string was empty.

The installer consequently reported an unsupported Ubuntu version and could not dispatch to the Ubuntu 25.04 installer.

### Root cause

The version detection relied on parsing the human-readable output of `lsb_release` with a regular expression that assumed a three-component version format.

The dispatch logic also lacked a dedicated Ubuntu 25.04 case.

### Changes implemented

File modified: `Ubuntu/install-eSim.sh`

- Replaced fragile version parsing with `/etc/os-release`.
- Added Ubuntu distribution validation.
- Added support for Ubuntu 25.04.
- Added explicit installer script selection for supported Ubuntu versions.
- Added a check to ensure the selected installer script exists.
- Improved unsupported-version error handling.
- Preserved the existing special handling for Ubuntu 22.04.4 and other 22.04 releases.

### Validation

The following checks were performed:

- Bash syntax validation.
- ShellCheck validation.
- Git diff whitespace validation.
- Controlled tests for Ubuntu 25.04 detection and dispatch.
- Unsupported-version handling.
- Child installer failure exit-code propagation.
- Version mapping checks for Ubuntu 22.04.4, 22.04.5 and 24.04.

The complete eSim GUI installation was not verified end-to-end.

## 3. Issue 2 — NGHDL LLVM and Clang compatibility

### Observed problem

On Ubuntu 25.04, the generic LLVM packages resolve to LLVM 20.

The GHDL 4.1.0 LLVM backend configuration does not support LLVM 20 and reports:

`Unhandled version llvm 20.0.0`

This prevents the expected GHDL LLVM backend build from proceeding with the generic LLVM configuration.

### Root cause

The NGHDL installer used generic LLVM and Clang package names rather than selecting versions compatible with the GHDL build.

### Changes implemented

File modified: `Ubuntu/install-nghdl.sh`

- Added Ubuntu 25.04-specific LLVM 18 and Clang 18 selection.
- Added installation of `llvm-18-dev` and `clang-18` for Ubuntu 25.04.
- Configured GHDL using the explicit LLVM 18 configuration executable.
- Passed the matching Clang 18 compiler to the GHDL build.
- Updated the uninstall package list to match the selected LLVM and Clang packages.
- Added stricter shell error handling to the installer flow.

Other Ubuntu versions retain the generic LLVM and Clang selection.

### Validation

- Bash syntax validation.
- ShellCheck validation.
- Git diff whitespace validation.
- Confirmed LLVM 20 incompatibility with GHDL 4.1.0.
- Successfully configured GHDL with LLVM 18.
- Successfully built GHDL using Clang 18.
- Executed a VHDL smoke test successfully.
- Checked the LLVM 18 package purge operation using APT simulation.

The complete NGHDL installer was not verified end-to-end.

## 4. Validation summary

| Validation | eSim | NGHDL |
|---|---|---|
| Bash syntax | PASS | PASS |
| ShellCheck | PASS | PASS |
| Git diff check | PASS | PASS |
| Functional checks | PASS | PASS |
| Full installer integration | Not verified | Not verified |

## 5. GitHub changes

### eSim

Repository: https://github.com/poonamgupta17/eSim

Branch: `task4/ubuntu-25.04-installer-fix`

Commit: `694a087b495e7e965b9e4dccd24f61fe9da40131`

### NGHDL

Repository: https://github.com/poonamgupta17/nghdl

Branch: `task4/ubuntu-25.04-llvm-fix`

Commit: `5067f905d95c2fb4c5f24396c896ee92bdb423a6`

## 6. Evidence

Static validation output is saved in:

`evidence/static-validation.txt`

Additional investigation and build evidence should be retained alongside this report where available.

## 7. Limitations

The changes were validated through shell checks, controlled installer tests, dependency compatibility checks, a GHDL build, and a VHDL smoke test.

A complete end-to-end installation of the eSim GUI and the complete NGHDL installer workflow was not performed. Therefore, this report does not claim that the entire eSim application installation was successfully completed on Ubuntu 25.04.

## 8. Conclusion

The investigation identified and addressed two Ubuntu 25.04 compatibility problems:

1. Fragile Ubuntu version detection and missing installer dispatch in eSim.
2. Incompatible generic LLVM and Clang package selection in NGHDL.

The corresponding changes have been committed and pushed to the contributor's GitHub repositories. The validation results and limitations are documented above.
