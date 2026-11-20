# AlRasedah Wazuh Detection

Sysmon configuration and Wazuh rules for Windows endpoint detection, maintained by Al-Rasedah Technology.

## Contents

| Path | What it is |
|------|------------|
| `sysmon/sysmon_config.xml` | Sysmon configuration (schema 4.90, Sysmon 15+) tuned for Wazuh |
| `rules/0910-sysmon_enhanced_rules.xml` | Wazuh rules for the detections the Sysmon config enables (IDs 102300–102349) |
| `rules/0415-sophos_central_rules.xml` | Sophos Central (API/JSON) rules, replacing the stock on-prem Sophos ruleset |
| `decoders/0295-sonicwall_decoders_override.xml` | Full replacement for the stock SonicWall decoder, extracting 37 `sonicwall.*` fields the compiled plugin drops |

## Sysmon configuration

Based on [paolokappa/Sysmon_Config_for_Wazuh](https://github.com/paolokappa/Sysmon_Config_for_Wazuh) v2025.1.0 (MIT). Main changes:

- Fixed conflicting rule groups that silently reduced coverage for CreateRemoteThread (8), WMI (19–21) and FileExecutableDetected (29)
- Fixed 85 filters that compared full paths against bare filenames and never matched
- Closed exclusion gaps: masquerading `svchost.exe`, Task Manager LSASS dumps, PowerShell under ManageEngine, WHCP-signed drivers, name-only vendor exclusions
- Removed environment-specific account exclusions from the upstream file
- Added coverage: LSASS access, comsvcs MiniDump, hive dumping, tunneling and exfiltration tools, RMM tools, AD recon, Kerberos abuse, registry tampering (AMSI, LSA, UAC, COR_PROFILER), raw disk reads, event log deletion
- Wazuh agent traffic excluded; hashes set to `MD5,SHA256,IMPHASH`

Install or update:

```
sysmon64.exe -accepteula -i sysmon_config.xml
sysmon64.exe -c sysmon_config.xml
```

## Wazuh agent

The Sysmon channel must not be filtered down to a few event IDs, or most of these detections never reach the manager:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

## Wazuh rules

Copy rule files to `/var/ossec/etc/rules/` on the manager, then:

```
/var/ossec/bin/wazuh-logtest -t
systemctl restart wazuh-manager
```

Rules chain on the built-in Sysmon base rules (`0595-win-sysmon_rules.xml`, Wazuh 4.x). Check the ID range against your own custom rules before deploying.

The Sophos Central rules and SonicWall decoder replace stock files, so exclude the originals in `ossec.conf`:

```xml
<ruleset>
  <rule_exclude>0415-sophos_rules.xml</rule_exclude>
  <decoder_exclude>ruleset/decoders/0295-sonicwall_decoders.xml</decoder_exclude>
</ruleset>
```

Copy decoders to `/var/ossec/etc/decoders/`.

## License

MIT, see [LICENSE](LICENSE). `sysmon/sysmon_config.xml` is a derivative of work by Paolo Kappa (Paolo Caparrelli), MIT, see [sysmon/LICENSE-upstream](sysmon/LICENSE-upstream).
