---
name: pumascan
description: A command line interface (CLI) tool for scanning C# projects and solutions for security vulnerabilities using Puma Scan Professional.
allowed-tools: Bash, PowerShell, Grep, Glob, Read, WebFetch
---

# pumascan CLI

`pumascan` is a .NET C# code analysis tool that analyzes C# projects and solutions for security vulnerabilities.

## Commands

### scan (default)

Run a security scan against one or more .NET projects or solutions. Since `scan` is the default verb, both `pumascan scan` and `pumascan` (with flags) work.

```bash
pumascan scan \
  --projects /path/to/MyApp.csproj \
  --format json,html \
  --output /path/to/output/myapp-scan \
  --settings /path/to/.pumafile
```

**Required flags:**

| Flag | Description |
|------|-------------|
| `-p, --projects` | Comma-separated list of `.csproj` files or `.sln` files to scan |
| `-f, --format` | Output format(s): `json`, `html`, `msbuild`, `vso`, `trx`, `csv`, `sonarcloud`, `sarif` |
| `-o, --output` | Output path without extension (extension added automatically per format) |

**Optional flags:**

| Flag | Description |
|------|-------------|
| `-s, --settings` | Path to settings file (`.pumafile` or `settings.json`) |
| `-v, --verbose` | Verbose console output (instance details, errors) |
| `-l, --license` | Directory containing offline license file |
| `-a, --auth-token` | Pipeline JWT for org/repo metadata |
| `--threshold-high` | Max high-risk findings before failing exit code |
| `--threshold-medium` | Max medium-risk findings before failing exit code |
| `--threshold-low` | Max low-risk findings before failing exit code |
| `--report-options` | Report options (e.g. `findings-only`) |
| `--system-information` | Display machine activation data and license info (see below) |

**Examples:**

```bash
# Scan a project, output JSON and HTML, with custom settings
pumascan scan \
  --projects /path/to/src/MyApp.csproj \
  --format json,html \
  --output /path/to/output/myapp-scan \
  --settings /path/to/.pumafile

# Scan multiple projects, SARIF output
pumascan scan \
  -p /path/to/Web.csproj,/path/to/Api.csproj \
  -f sarif \
  -o /path/to/output/myapp-scan

# Scan a solution, SARIF output
pumascan scan \
  -p /path/to/MySolution.sln \
  -f sarif \
  -o /path/to/output/myapp-scan

# CI/CD gate — fail the pipeline if thresholds are exceeded
# Thresholds are evaluated in order (high, medium, low) and stop at first failure.
# Exit code 0 = all passed, non-zero = a threshold was exceeded (e.g. 7 = ThresholdMedium).
pumascan scan \
  --projects /path/to/src/MyApp.csproj \
  --format json,html \
  --output /path/to/output/myapp-scan \
  --settings /path/to/.pumafile \
  --threshold-high 5 \
  --threshold-medium 10 \
  --threshold-low 10

# Verbose scan for debugging
pumascan scan \
  -p /path/to/src/MyApp.csproj \
  -f json \
  -o /path/to/output/myapp-scan \
  -v
```

**Example output:**

```
Puma Scan License:
        Product:        Puma Scan Pro: Server License
        Expiration:     12/31/2030

Enabled Integrations:
        Sonatype OSS Index

Loading...
Project Summary:
        Loaded project(s) ./MyApp.csproj in 0 min 01.98 sec
        C# Project Count:               2
        C# Document Count:              64
Scanning...
Scan Summary:
        Settings File: ./.pumafile
        Engine Version: 1.7.1.62
        Found 27 diagnostics in 00:00:01.3981553
        SEC0017: 1 instance(s)
        SEC0019: 11 instance(s)
        SEC0120: 10 instance(s)
        Total High:     2
        Total Medium:   24
        Total Low:      1

Output Summary:
        json results written to /path/to/output/myapp-scan.json
        html results written to /path/to/output/myapp-scan.html

Validating thresholds requirements:
        Error severity threshold requirement: 0 findings.
        Error severity threshold requirement passed: 0 findings.

Exit code: 0 - Success
```

**Exit codes:**

| Code | Name | Meaning |
|------|------|---------|
| `0` | Success | Scan completed and all thresholds passed |
| `1` | InvalidArguments | Bad or missing CLI arguments |
| `2` | InvalidSolutionFile | Solution/project file could not be loaded |
| `3` | InvalidSettingsFile | Settings file (`.pumafile`) is invalid |
| `4` | ScanError | Error during scan execution |
| `5` | InvalidLicense | License file missing, expired, or invalid |
| `6` | ThresholdHigh | High-risk findings exceeded `--threshold-high` |
| `7` | ThresholdMedium | Medium-risk findings exceeded `--threshold-medium` |
| `8` | ThresholdLow | Low-risk findings exceeded `--threshold-low` |
| `9` | InvalidOutputFile | Output file/path is invalid |
| `98` | CommandNotSupported | Unrecognized command |
| `99` | MSBuildNotInstalled | MSBuild not found on the system |
| `100` | UnknownError | Unexpected error |

In CI/CD, any non-zero exit code fails the pipeline step.

**Threshold behavior:**

- Thresholds are evaluated in order: high, then medium, then low
- Evaluation stops at the first failure — e.g. if medium fails, low is not checked
- When no thresholds are specified, only an "Error severity" threshold of 0 is applied
- Each threshold line in output shows the limit and actual count, with pass/fail status

### scan --system-information

Display machine activation data and license details for the current machine. Used for offline activation via the Puma Scan Portal license management screen.

```bash
pumascan scan --system-information
```

**Output contains two sections:**

1. **System Information** — JSON with `MachineData` and `MachineHash` fields. These values are submitted to the Puma Scan Portal to perform offline machine activation.
2. **License Information** — JSON with `Status`, `LicenseKey`, `Product`, `StartDateUtc`, `EndDateUtc`, and `LastVerifiedInUtc`. Shows the license file found on the machine.

**Example output:**

```
System Information:

{
  "MachineData": "L-HB-K9UmB7ai...(base64)",
  "MachineHash": "ExVcfTuwNeXgA...(base64)"
}

License Information:

{
  "Status": "Valid",
  "LicenseKey": "4edd57a0-d515-4b77-afb9-df1c3aac5f59",
  "Product": "Puma Scan Pro: Server License",
  "StartDateUtc": "2019-11-14T03:27:06.937472",
  "EndDateUtc": "2030-12-31T23:59:59",
  "LastVerifiedInUtc": "2026-04-13T22:16:06.1941446Z"
}
```

This is a standalone command — no `-p`, `-f`, or `-o` flags are needed.

**Behavior notes:**

- A single `.csproj` may load multiple C# projects (transitive references)
- Output files are written as `<output-path>.<format>` (e.g. `/path/to/output/myapp-scan.json`)
- The scan validates license on startup, then loads projects, scans, writes output, and checks thresholds

### add-exception

Add a suppression entry to the `Exceptions` array in the project's `.pumafile`. Used to suppress false positives or accepted risks.

```bash
pumascan add-exception \
  --projects /path/to/src/MyApp.csproj \
  --diagnostics SEC0017 \
  --file-path src/MyApp/Constants/PasswordOptions.cs \
  --begin-line 10 \
  --end-line 20 \
  --reason "approved by security as an exception"
```

The command finds the `.pumafile` associated with the project and appends an exception entry with the diagnostic IDs, path, line range, approver (from current user), reason, and timestamp.

**Required flags:**

| Flag | Description |
|------|-------------|
| `-p, --projects` | Comma-separated list of `.csproj` files |

**Optional flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `-d, --diagnostics` | | Diagnostic IDs to suppress (e.g. `SEC0001,SEC0025`) |
| `-f, --file-path` | `*` | File path to suppress (supports wildcards: `*.cs`) |
| `-b, --begin-line` | `*` | Beginning line number to suppress |
| `-e, --end-line` | `*` | End line number to suppress |
| `-r, --reason` | | Reason for the suppression |
| `-c, --custom-pattern` | | Custom regex pattern to bulk suppress findings matching line content (instead of targeting specific file/line) |

**Example output:**

```
Adding Exception to /path/to/.pumafile
        Diagnostic IDs: SEC0017
        Path: /src/MyApp/Constants/PasswordOptions.cs
        Start Line: 10
        End Line: 20
        Custom Pattern:
        Approved By: user
        Reason: approved by security as an exception

Exception successfully added.
```

**Exception entry added to `.pumafile`:**

```json
{
  "RuleIds": ["SEC0017"],
  "Path": "/src/MyApp/Constants/PasswordOptions.cs",
  "StartLine": "10",
  "EndLine": "20",
  "Pattern": null,
  "Expires": null,
  "Checksum": "",
  "ApprovedBy": "user",
  "Reason": "approved by security as an exception",
  "Timestamp": "2026-04-13T18:05:48.018483-05:00"
}
```

### version

Display the installed pumascan version.

```bash
pumascan version
```

## .pumafile schema

The `.pumafile` is a JSON configuration file with these top-level sections:

```json
{
  "Version": "1.6.0",
  "LogLevel": "Info",
  "GeneralSettings": { ... },
  "RuleOptions": [ ... ],
  "Exceptions": [ ... ],
  "CustomTaintedSources": [ ... ],
  "CustomSinks": [ ... ],
  "CustomCleanseMethods": [ ... ]
}
```

**GeneralSettings:**

| Field | Default | Description |
|-------|---------|-------------|
| `DataflowAnalysisEnabled` | `true` | Enable taint/dataflow analysis |
| `DataflowAnalysisEngineVersion` | `"2.0"` | Dataflow engine version |
| `DataflowAnalysisReportIndeterminates` | `false` | Report indeterminate dataflow results |
| `DataflowAnalysisNodeMaxDepth` | `9` | Max depth for dataflow traversal |
| `ProductionConfigurationTransform` | `"Release"` | Config transform name for production |

**RuleOptions** — array of rule configurations:

```json
{
  "Id": "SEC0017",
  "Name": "Identity Weak Password Complexity",
  "RiskRating": "Medium",        // High, Medium, Low
  "Severity": "Warning",
  "Enabled": true,
  // Some rules have additional fields:
  "Length": 10,                   // SEC0017: min password length
  "RequireNumber": true,          // SEC0017: require digit
  "TimeoutMax": 30,               // SEC0007/SEC0020: max timeout minutes
  "Patterns": ["^[Kk][Ee][Yy]$"] // SEC0131: custom secret patterns
}
```

**Exceptions** — array of suppression entries (managed by `add-exception` command):

```json
{
  "RuleIds": ["SEC0017"],
  "Path": "/src/MyApp/Constants/PasswordOptions.cs",
  "StartLine": "10",
  "EndLine": "20",
  "Pattern": "",
  "Expires": null,
  "Checksum": "",
  "ApprovedBy": "user",
  "Reason": "approved by security as an exception",
  "Timestamp": "2026-04-13T18:05:48.018483-05:00"
}
```

**CustomTaintedSources** — define custom taint sources for dataflow analysis:

```json
{
  "RuleIds": [],
  "Flag": "Web",
  "Syntax": "InvocationExpressionSyntax",
  "Namespace": "unirest_net.http",
  "Type": "Unirest",
  "Property": "*",
  "Method": "*"
}
```

**CustomSinks** — define custom sinks (same structure as tainted sources).

**CustomCleanseMethods** — define methods that cleanse tainted data:

```json
{
  "RuleIds": ["SEC0111"],
  "Flag": "Web",
  "Syntax": "InvocationExpressionSyntax",
  "Namespace": "MyApp.Validation",
  "Type": "Validator",
  "Method": "IsValidFileName"
}
```

## Notes

- Diagnostic IDs follow the pattern `SEC####` (e.g. `SEC0001` through `SEC0131`)
- The `-o` output path should omit the file extension — pumascan appends it based on `-f`
- Multiple formats can be comma-separated in `-f` to generate multiple output files
- The `.pumafile` is a JSON config controlling rule severity, risk ratings, and enable/disable per rule
- License validation requires either `PUMA_LICENSE`/`PUMA_AUTH_TOKEN` env vars or the `-l`/`-a` flags
- **Only load when the user requests more information about advanced configuration or the full ruleset**: see [Puma Scan Online Documentation](./references/documentation.md).
