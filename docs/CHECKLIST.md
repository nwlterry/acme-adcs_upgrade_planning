# ACME-ADCS Upgrade Checklist (v1.0.3 → v3.0.7)

**Environment**: Internal only | HTTP-01 | cert-manager on OpenShift | F:\ACME-ADCS\

## Phase 1: Preparation
- [ ] Backup entire `F:\ACME-ADCS\` folder (Data + Logs)
- [ ] Backup current IIS site configuration and application pool
- [ ] Backup `appsettings-custom.json` and any .pfx certificates
- [ ] Document current CA details (`labdc01.devops.local\devops-LABDC01-CA` + `WebServerACME`)
- [ ] Download latest v3.0.7 release ZIP
- [ ] Install **.NET 10.0 Hosting Bundle** on the server → Restart IIS
- [ ] Verify .NET 10.0 is listed in IIS Application Pools

## Phase 2: Deployment (Side-by-Side)
- [ ] Extract v3.0.7 to `F:\ACME-ADCS-v3\`
- [ ] Copy existing `Data` folder into `F:\ACME-ADCS-v3\`
- [ ] Create `appsettings.Production.json` (use the one from README.md)
- [ ] Update placeholders (`BaseUrl`, PFX path/password, CanonicalHostName)
- [ ] Create new IIS Application Pool (`No Managed Code`) with same identity
- [ ] Create new IIS website pointing to `F:\ACME-ADCS-v3` (or update existing)
- [ ] Bind internal hostname (`acme.internal.yourcompany.local`)

## Phase 3: Configuration & Validation
- [ ] Confirm `appsettings.Production.json` contains:
  - `Default-Internal` profile with DNS + IP
  - HTTP-01 only
  - `SkipCAA: true`, `AllowCNSuffix`, `AllowEmptyCN`
  - `IgnoreServerCertificate: true`
- [ ] Restart Application Pool
- [ ] Access `https://acme.internal.yourcompany.local/.well-known/acme-directory` (should return JSON)

## Phase 4: Testing
- [ ] Test DNS identifier certificate request via cert-manager
- [ ] Test IP identifier certificate request via cert-manager
- [ ] Verify HTTP-01 challenge works from OpenShift pods
- [ ] Check logs in `F:\ACME-ADCS\Logs\`
- [ ] Test revocation (if enabled)
- [ ] Validate issued certificates in ADCS console

## Phase 5: Cutover & Monitoring
- [ ] Update cert-manager `ClusterIssuer` with new ACME server URL
- [ ] Monitor certificate renewals for 48 hours
- [ ] Decommission old v1.0.3 site (after successful cutover)

## Rollback
- [ ] Switch IIS site back to old folder
- [ ] Restart old Application Pool

**Last Updated**: 2026-05-19