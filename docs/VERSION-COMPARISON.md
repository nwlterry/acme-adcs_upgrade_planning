# Version Comparison: v1.0.3 vs v3.0.7

| Category                      | v1.0.3 (Current)                          | v3.0.7 (Target)                                      | Impact / Action |
|-------------------------------|-------------------------------------------|------------------------------------------------------|-----------------|
| **.NET Version**              | .NET 6.0                                  | .NET 10.0                                            | Must upgrade Hosting Bundle |
| **Configuration Style**       | Flat `ADCSIssuer` section                 | `Profiles` + `ADCSOptions` inside profile            | Major rewrite required |
| **ADCS Settings**             | Top-level `ADCSIssuer`                    | Inside `Profiles.Default-Internal.ADCSOptions`       | Moved |
| **Identifiers**               | DNS only (implicit)                       | Explicit `dns` + `ip`                                | Added IP support |
| **Challenge Types**           | Implicit (all)                            | Per-profile `AllowedChallengeTypes` (HTTP-01 only)   | Restricted for security |
| **HTTP-01 Validation**        | Basic                                     | `ChallengeValidation.Http01.IgnoreServerCertificate` | Useful for internal/OpenShift |
| **Identifier Validation**     | Basic                                     | `IdentifierValidation.DNS` + `IP` (SkipCAA, etc.)    | Enhanced |
| **AllowCNSuffix / AllowEmptyCN** | In `ADCSIssuer`                        | In `IdentifierValidation.DNS`                        | Migrated |
| **Logging**                   | Old `EnableFileLog` + `PathFormat`        | New `Logging.File` structure                         | Updated |
| **TOS Agreement**             | Configurable                              | `RequireTOSAgreement: false` (internal)             | Disabled |
| **BaseUrl & Hosting**         | Basic                                     | `Kestrel` + `ForwardedHeaders` + `CanonicalHostName`| Recommended updates |
| **Profiles**                  | None (single config)                      | Required                                             | New mandatory structure |
| **Revocation**                | Always enabled                            | Configurable per profile                             | More control |

**Key Breaking Changes**
- Configuration completely restructured around **Profiles**
- `ADCSIssuer` section removed
- Must define at least one profile
- .NET 6 → 10 requires full redeployment

**Migration Status**: Completed (see `appsettings.Production.json`)