# ACME-ADCS Server Upgrade Planning

## Overview
Upgrade from **v1.0.3** (.NET 6) to **v3.0.7** (.NET 10) on internal environment with **HTTP-01 challenge only**.

ACME requests handled by **cert-manager on OpenShift**.

## Upgrade Path

### 1. Preparation
- Backup current deployment (files, `appsettings-custom.json`, `F:\ACME-ADCS\` data, IIS config, logs).
- Record current `CAServer`, `TemplateName`, `BasePath`.
- **Upgrade .NET Hosting Bundle** to version **10.0** on the server.
- Test current cert-manager integration.

### 2. Infrastructure
- Deploy v3.0.7 to new folder (e.g. `F:\ACME-ADCS-v3\`).
- Keep old v1.0.3 for rollback.

### 3. Configuration
Use the migrated `appsettings.Production.json` below.

### 4. Testing
- Verify `/.well-known/acme-directory` endpoint.
- Test certificate issuance for DNS and IP identifiers.
- Monitor logs.
- Cutover by updating cert-manager Issuer.

## Major Changes Across Versions
- **.NET Runtime**: 6.0 → 10.0 (full recompile/redeploy required)
- **Configuration**: Flat `ADCSIssuer` → Structured `Profiles` with `ADCSOptions`
- **Identifiers**: Added full `ip` support
- **Validation**: New `ChallengeValidation`, `IdentifierValidation`, `SkipCAA`, etc.
- **HTTP-01**: `IgnoreServerCertificate` option for internal setups

## Full Converted Configuration

**File**: `appsettings.Production.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Warning"
    },
    "File": {
      "PathFormat": "F:\\ACME-ADCS\\Logs\\{Date}.json",
      "Json": true,
      "MaxFileSize": 10485760,
      "MaxRetainedFiles": 30
    }
  },

  "AcmeFileStore": {
    "BasePath": "F:\\ACME-ADCS\\"
  },

  "AcmeServer": {
    "BaseUrl": "https://acme.internal.yourcompany.local",
    "RequireTOSAgreement": false,
    "ExternalAccountBinding": {
      "Enabled": false
    }
  },

  "Profiles": {
    "Default-Internal": {
      "Description": "Default profile migrated from v1.0.3 (DNS + IP, HTTP-01 only)",
      
      "SupportedIdentifiers": [ "dns", "ip" ],

      "ADCSOptions": {
        "CAServer": "labdc01.devops.local\\devops-LABDC01-CA",
        "TemplateName": "WebServerACME"
      },

      "AllowedChallengeTypes": {
        "dns": [ "http-01" ],
        "ip":  [ "http-01" ]
      },

      "ChallengeValidation": {
        "Http01": {
          "IgnoreServerCertificate": true
        }
      },

      "IdentifierValidation": {
        "DNS": {
          "SkipCAA": true,
          "AllowCNSuffix": true,
          "AllowEmptyCN": true
        },
        "IP": {
          "AllowedIPRanges": []
        }
      },

      "Revocation": {
        "Enabled": true
      }
    }
  },

  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://0.0.0.0:80"
      },
      "HttpsInlineCertFile": {
        "Url": "https://0.0.0.0:443",
        "Certificate": {
          "Path": "F:\\ACME-ADCS\\cert.pfx",
          "Password": "YOUR-PFX-PASSWORD"
        }
      }
    }
  },

  "ForwardedHeaders": {
    "Enabled": true,
    "KnownNetworks": [ "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16", "fd00::/8" ]
  },

  "CanonicalHostName": "acme.internal.yourcompany.local"
}
```

## Next Steps
1. Replace placeholders (`BaseUrl`, certificate path/password, CA details).
2. Deploy and test.
3. Update OpenShift cert-manager `ClusterIssuer` with new ACME server URL.

## References
- Original Repo: https://github.com/glatzert/ACME-Server-ADCS
- Your Upgrade Repo: https://github.com/nwlterry/acme-adcs_upgrade_planning

*Document compiled from conversation on May 19, 2026*