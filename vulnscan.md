# Linux - Vulnerability Scanning

Never rely on 1 scanner


Should scan:

- after system patches
- after major system change
- when performing new install



Types:

- General
  - Nessus - $$$$ but covers everything from OS to SCADA. Can also validate compliance (PCI, STIGs, etc). Can be used with credentials
  - CoreImpact, QualysGuard
- OS
  - MS Security Baseline analyzer
- Service/application-specific
  - Web: W3AF, Arachni (apache and iis)
  - SQL: SQLMap, Havij
  - Misc: Nmap, nikto, cisco-torch