# IntuneSCT-Compliance

Ready-to-deploy Intune custom compliance packages generated from [Microsoft Security Compliance Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=55319) baselines.

## What This Is

Microsoft provides security baselines as GPO exports. Intune has built-in security baselines for *configuration*, but no equivalent for *compliance reporting*. These packages close that gap by providing:

- **Detection scripts** (PowerShell) that check security settings on endpoints and output JSON
- **Rules JSON** files that define expected values, operators, and remediation guidance

Together, they form [Intune custom compliance policies](https://learn.microsoft.com/en-us/mem/intune/protect/compliance-custom-script) that report which baseline settings are correctly applied on each device.

## Available Baselines

### Windows 11 v25H2 Security Baseline

425 settings across 39 packages. Categories are aligned to the Intune security baseline configuration profile sections, so you can match each compliance package to the exact section you configured.

#### Top-Level Sections

| Category | Settings | Directory |
|----------|----------|-----------|
| Auditing | 23 | `Auditing/` |
| Browser | 136 | `Browser/` |
| Defender | 40 | `Defender/` |
| Device Guard | 8 | `DeviceGuard/` |
| Device Lock | 9 | `DeviceLock/` |
| Dma Guard | 1 | `DmaGuard/` |
| Firewall | 24 | `Firewall/` |
| Kerberos | 10 | `Kerberos/` |
| LAPS | 3 | `LAPS/` |
| Lanman Server | 9 | `LanmanServer/` |
| Lanman Workstation | 10 | `LanmanWorkstation/` |
| Local Policies Security Options | 16 | `LocalPoliciesSecurityOptions/` |
| Local Security Authority | 16 | `LocalSecurityAuthority/` |
| Smart Screen | 4 | `SmartScreen/` |
| Sudo | 1 | `Sudo/` |
| System Services | 4 | `SystemServices/` |
| User Rights | 23 | `UserRights/` |
| Wi-Fi Settings | 2 | `Wi-FiSettings/` |
| Windows Hello For Business | 1 | `WindowsHelloForBusiness/` |
| Windows Ink Workspace | 1 | `WindowsInkWorkspace/` |

#### Administrative Templates Sub-Categories

These match the sub-headings within the "Administrative Templates" section of the Intune security baseline profile.

| Sub-Category | Settings | Directory |
|-------------|----------|-----------|
| BitLocker Drive Encryption | 4 | `AdministrativeTemplates-BitLockerDriveEncryption/` |
| Control Panel | 2 | `AdministrativeTemplates-ControlPanel/` |
| Credential Delegation | 3 | `AdministrativeTemplates-CredentialDelegation/` |
| Device Installation | 4 | `AdministrativeTemplates-DeviceInstallation/` |
| Event Log Service | 3 | `AdministrativeTemplates-EventLogService/` |
| Experience | 3 | `AdministrativeTemplates-Experience/` |
| File Explorer | 5 | `AdministrativeTemplates-FileExplorer/` |
| MS Security Guide | 4 | `AdministrativeTemplates-MSSecurityGuide/` |
| MSS (Legacy) | 5 | `AdministrativeTemplates-MSS(Legacy)/` |
| Network | 7 | `AdministrativeTemplates-Network/` |
| Power | 4 | `AdministrativeTemplates-Power/` |
| Printers | 10 | `AdministrativeTemplates-Printers/` |
| Privacy | 2 | `AdministrativeTemplates-Privacy/` |
| Remote Desktop Services | 10 | `AdministrativeTemplates-RemoteDesktopServices/` |
| Search | 1 | `AdministrativeTemplates-Search/` |
| System | 6 | `AdministrativeTemplates-System/` |
| Windows Installer | 3 | `AdministrativeTemplates-WindowsInstaller/` |
| Windows PowerShell | 2 | `AdministrativeTemplates-WindowsPowerShell/` |
| Windows Remote Management | 6 | `AdministrativeTemplates-WindowsRemoteManagement/` |

## Quick Start

### 1. Choose your categories

Start with the categories that match what you've configured in your Intune security baseline profile. If you configured "Device Guard" and "Local Security Authority" in the baseline, deploy those same compliance packages.

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
