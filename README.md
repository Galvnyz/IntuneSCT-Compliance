# IntuneSCT-Compliance

Ready-to-deploy Intune custom compliance packages generated from [Microsoft Security Compliance Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=55319) baselines.

## What This Is

Microsoft provides security baselines as GPO exports. Intune has built-in security baselines for *configuration*, but no equivalent for *compliance reporting*. These packages close that gap by providing:

- **Detection scripts** (PowerShell) that check security settings on endpoints and output JSON
- **Rules JSON** files that define expected values, operators, and remediation guidance

Together, they form [Intune custom compliance policies](https://learn.microsoft.com/en-us/mem/intune/protect/compliance-custom-script) that report which baseline settings are correctly applied on each device.

## Available Baselines

### Windows 11 v25H2 Security Baseline

425 settings across 15 categories. Each category is a separate compliance package to give you granular control over what to monitor.

| Category | Settings | Severity Mix | Script Size |
|----------|----------|-------------|-------------|
| Application Control | 13 | Critical: 1, High: 1, Medium: 10, Low: 1 | 4.5 KB |
| Audit Logging | 26 | Critical: 2, High: 11, Medium: 13 | 6.8 KB |
| Browser Security | 136 | Medium: 96, Low: 40 | 38.6 KB |
| Credential Protection | 20 | Critical: 17, High: 2, Medium: 1 | 6.3 KB |
| Device Security | 5 | Critical: 1, Medium: 4 | 2.0 KB |
| Encryption | 4 | High: 4 | 1.4 KB |
| Endpoint Protection | 45 | High: 44, Medium: 1 | 16.2 KB |
| Identity & Access | 63 | Critical: 7, High: 34, Medium: 22 | 19.0 KB |
| Network Security | 71 | High: 55, Medium: 16 | 22.1 KB |
| OS Hardening | 8 | Medium: 8 | 2.6 KB |
| Power Management | 4 | Low: 4 | 1.8 KB |
| Print Security | 10 | High: 1, Medium: 9 | 3.3 KB |
| Privacy & Telemetry | 6 | Medium: 2, Low: 4 | 2.4 KB |
| Remote Access | 10 | High: 10 | 3.6 KB |
| Service Hardening | 4 | Low: 4 | 1.0 KB |

## Quick Start

### 1. Choose your categories

Start with the highest-impact categories:

- **Credential Protection** (20 settings) - LSASS protection, Device Guard, NTLM controls
- **Network Security** (71 settings) - Firewall, SMB signing, LDAP, WinRM
- **Identity & Access** (63 settings) - UAC, privilege rights, password policies

### 2. Upload to Intune

1. Go to **Intune > Devices > Compliance > Scripts** (or **Compliance policies > Scripts**)
2. Click **Add** and upload the `Detection.ps1` from your chosen category
3. Create a new **Compliance policy > Custom compliance**
4. Select your uploaded detection script
5. Upload the corresponding `Rules.json`
6. Assign the policy to your device groups

### 3. Review results

After devices check in (may take several hours for the first evaluation):
- **Compliant** - Setting matches the baseline expected value
- **Not compliant** - Setting exists but has a different value, or the setting is absent

## How Detection Works

Each detection script:
1. Reads registry values, audit policies, privilege rights, system access policies, or service configurations
2. Outputs a JSON object with one key per setting (e.g., `"REG-Control-Lsa-RunAsPPL": 1`)
3. Uses `-1` as a sentinel value for numeric settings that don't exist on the endpoint (ensures "Not compliant" rather than "Error")

The rules JSON defines the expected value and comparison operator for each setting. Intune's compliance engine compares the detection output against the rules and reports per-setting compliance.

## Setting ID Format

Each setting has a deterministic ID:

| Prefix | Type | Example |
|--------|------|---------|
| `REG-` | Registry | `REG-Control-Lsa-RunAsPPL` |
| `AUD-` | Audit Policy | `AUD-CredentialValidation` |
| `PRV-` | Privilege Right | `PRV-SeDebugPrivilege` |
| `ACC-` | System Access | `ACC-MinimumPasswordLength` |
| `SVC-` | Service Config | `SVC-XboxGipSvc` |

## Severity Tiers

Settings are classified by security impact:

- **Critical** - Credential dumping prevention, virtualization-based security, audit integrity
- **High** - Firewall, SMB signing, remote access controls, encryption, endpoint protection
- **Medium** - Event log sizing, device restrictions, printer settings, audit policies
- **Low** - Cloud content, legacy browser settings, power management, gaming features

## License

MIT - See [LICENSE](LICENSE).

## Disclaimer

These packages are generated from Microsoft's publicly available security baselines. The expected values reflect Microsoft's recommended security configuration. Always test compliance policies in a pilot group before broad deployment.

Settings that show "Not compliant" indicate the endpoint's configuration differs from the baseline recommendation. This may be intentional (e.g., a different security posture) or unintentional (e.g., a missing hardening configuration).
