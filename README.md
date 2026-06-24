# eCITES / Border Force – Power Platform Solution Packages

This repository contains the exported Power Platform solution packages for the eCITES / Border Force digital permit work in Pegasus.

The purpose of this repository is to provide a clear handover point for the Dynamics / Dataverse components built during discovery and early development, including:

- Dataverse Custom APIs
- Plugin-backed integration points
- Permit field changes
- Document generation updates
- QR generation proof of concept
- Barcode-related work

---

## Repository Contents

    /solutions
      /07.Defra.eCites.WebAPI
      /08.Defra.eCites.PermitChanges
      /09.Defra.eCites.PermitBarcodeSolution
      /10.Defra.eCites.QRSolution

    /docs
      handover.md
      api-search-permit.md
      api-endorse-permit.md
      deployment-notes.md

> Folder structure may be adjusted as the project progresses, but solution packages should remain clearly separated by solution number and purpose.

---

## Solution Summary

| Solution | Name | Purpose |
|---|---|---|
| 07 | `07.Defra.eCites.WebAPI` | Dataverse Custom APIs, request/response parameters, plugin references |
| 08 | `08.Defra.eCites.PermitChanges` | Permit table changes, endorsement fields, permit document generation updates |
| 09 | `09.Defra.eCites.PermitBarcodeSolution` | Barcode/custom control work for permit number handling |
| 10 | `10.Defra.eCites.QRSolution` | QR generation proof of concept and related flow work |

---

## 07.Defra.eCites.WebAPI

This solution contains the API layer for the eCITES / Border Force integration.

Current APIs:

| Custom API | Purpose |
|---|---|
| `cites_SearchPermitByNumber` | Searches Pegasus for a permit using the permit number and returns permit details |
| `cites_EndorsePermit` | Writes Border Force endorsement/return data back to the permit record and updates permit status |

The APIs are implemented as Dataverse Custom APIs and are intended to be called by the Border Force Node.js frontend/backend using OAuth-secured Dataverse Web API calls.

### Plugin Dependency

These APIs are backed by C# plugin handlers.

Plugin deployment/package references must be checked before importing this solution downstream.

During development, a missing `pluginassembly` dependency was encountered when exporting Solution 07. The next developer should verify the approved plugin deployment approach before relying on solution import alone.

---

## 08.Defra.eCites.PermitChanges

This solution contains Dataverse and document generation changes for digital permits.

Includes:

- New fields on `cites_permit` for Border Force return/endorsement data
- Updates to the Generate Permit Power Automate flow
- PDF generation added alongside existing Word document generation
- Environment variables for permit template/document library identifiers
- Connection references for Dataverse, SharePoint, and Word Online

Example endorsement fields:

- Actual quantity / quantity returned
- Actual mass / net mass returned
- Unit returned
- Number of animals DOA
- Movement Reference Number (MRN)
- Trade date
- Customs officer / epaulette number
- Port

---

## 09.Defra.eCites.PermitBarcodeSolution

This solution contains the barcode-related proof of concept.

Current purpose:

- Support barcode generation/handling for permit documents
- Barcode currently represents/reads the permit number

Further work is required to confirm the final barcode placement, template usage, and production behaviour.

---

## 10.Defra.eCites.QRSolution

This solution contains QR generation proof-of-concept work.

Current purpose:

- Generate a QR code for inclusion on the eCITES permit
- Support future Border Force validation/check journey

The final QR destination URL is not yet agreed. The frontend route needs to be confirmed before this can be finalised.

Potential future pattern:

    https://<econtrol-frontend-url>/check-permit?permit=<permit-number>

This is a placeholder only and must be validated with the frontend and security teams.

---

## API Documentation

API documentation should be kept in `/docs` or linked from Confluence.

Current API contracts:

- Search Permit API: `cites_SearchPermitByNumber`
- Endorse Permit API: `cites_EndorsePermit`

Current DEV Dataverse base URL:

    https://org99791a21.api.crm11.dynamics.com

Web API base path:

    /api/data/v9.2/

Required headers:

    Authorization: Bearer <token>
    Accept: application/json
    Content-Type: application/json
    OData-MaxVersion: 4.0
    OData-Version: 4.0

Authentication is currently developer/test setup only. The final Border Force authentication model is still to be agreed.

---

## Search Permit API

### Custom API

    cites_SearchPermitByNumber

### Endpoint

    POST /api/data/v9.2/cites_SearchPermitByNumber

### Purpose

Searches Pegasus for a permit using the permit number and returns permit details needed by the Border Force frontend.

### Example Request

    {
      "permitNumber": "24GBEXP1CLWMD"
    }

### Current Response Shape

    {
      "country": null,
      "countryOfImport": null,
      "gbAnnex": "11",
      "countryOfExport": null,
      "permitType": "Article 10",
      "citesAppendix": "I",
      "importerName": "Manju Chaudhary",
      "validityDate": null,
      "permitId": "f0cf6dcd-bda0-ee11-be37-0022481abcc2",
      "importerAddress": "WELL FOUND, 309, GREENFORD ROAD, GREENFORD, EALING, UB6 8RE, United Kingdom",
      "sourceCode": null,
      "statusLabel": "Returned - Used",
      "purposeCode": null,
      "countryOfReExport": null,
      "exporterName": null,
      "specialConditions": "Permit valid only for artificially propagated plants as defined by Conference Resolution 11.11 (Rev CoP18). Valid only for the following taxa….",
      "quantity": "20",
      "countryOfLastReExport": null,
      "originPermitNumber": null,
      "netMass": null,
      "statuscode": 149900002,
      "describeSpecimen": "Lorem ipsum dolor sit amet...",
      "commonNameOfSpecies": null,
      "exporterAddress": "address line al01, al02, alo03, al04, al05, 123456, Algeria",
      "scientificName": "Panthera tigris-23qwewx",
      "permitNumber": "23GBA10AUOAJF"
    }

### Search Response Schema

| Field | Type | Nullable | Notes |
|---|---|---|---|
| `permitId` | string GUID | No | Permit record id |
| `permitNumber` | string | No | Permit number |
| `statuscode` | integer | No | Permit status option set value |
| `statusLabel` | string | Yes | Human-friendly status label |
| `permitType` | string | Yes | Permit type |
| `validityDate` | string date-only | Yes | Date-only string, expected format `YYYY-MM-DD` |
| `country` | string | Yes | Confirm source/meaning |
| `countryOfImport` | string | Yes | Country of import |
| `countryOfExport` | string | Yes | Country of export |
| `countryOfReExport` | string | Yes | Country of re-export |
| `countryOfLastReExport` | string | Yes | Last re-export country |
| `gbAnnex` | string | Yes | GB Annex |
| `citesAppendix` | string | Yes | CITES appendix |
| `sourceCode` | string | Yes | Source code |
| `purposeCode` | string | Yes | Purpose code |
| `importerName` | string | Yes | Importer name |
| `importerAddress` | string | Yes | Concatenated importer address |
| `exporterName` | string | Yes | Exporter/re-exporter name |
| `exporterAddress` | string | Yes | Concatenated exporter address |
| `scientificName` | string | Yes | Scientific name |
| `commonNameOfSpecies` | string | Yes | Common name |
| `describeSpecimen` | string | Yes | Specimen description |
| `quantity` | string | Yes | Whole number represented as string |
| `netMass` | number/string | Yes | Confirm final contract |
| `originPermitNumber` | string | Yes | Origin history permit number |
| `specialConditions` | string | Yes | Special conditions text |

### Search API Notes

Most of these fields are not on the Permit table.

The plugin needs to fetch related records, primarily:

- Permit: `cites_permit`
- Application / Case: related 1:1 with Permit
- Submission: `cites_submission`
- Specimen: `cites_specimen`
- Species: `cites_species`

The Application table contains much of the permit data displayed on the frontend wireframe.

A full mapping table still needs to be completed for every frontend field.

---

## Endorse Permit API

### Custom API

    cites_EndorsePermit

### Endpoint

    POST /api/data/v9.2/cites_EndorsePermit

### Purpose

Writes Border Force endorsement/return data back to Pegasus and updates the permit status.

### Example Request

    {
      "permitId": "a18de960-1eab-ee11-be37-0022480174fd",
      "cites_quantityreturned": 10.5,
      "cites_netmassreturned": 25.2,
      "cites_unitreturned": 149900001,
      "cites_NumberofanimalsDOA": 2,
      "cites_MovementReferenceNumberMRN": "MRN12345",
      "cites_tradedate": "2026-04-15",
      "cites_CustomsOfficerEpauletteNumber": "OFF-7788",
      "cites_Port": "Dover"
    }

### Request Schema

| Field | Type | Required | Notes |
|---|---|---|---|
| `permitId` | string GUID | Yes | Permit record id |
| `cites_quantityreturned` | number | No | Decimal |
| `cites_netmassreturned` | number | No | Decimal |
| `cites_unitreturned` | integer | No | Choice/option set value |
| `cites_NumberofanimalsDOA` | integer | No | Whole number |
| `cites_MovementReferenceNumberMRN` | string | No | Text |
| `cites_tradedate` | string date-only | No | Recommended format `YYYY-MM-DD` |
| `cites_CustomsOfficerEpauletteNumber` | string | No | Text |
| `cites_Port` | string | No | Text |

### Important Payload Rule for Frontend

For optional fields, do not include the field in the JSON payload unless it has a value.

Good:

    {
      "permitId": "a18de960-1eab-ee11-be37-0022480174fd",
      "cites_quantityreturned": 10,
      "cites_tradedate": "2026-04-15"
    }

Avoid:

    {
      "permitId": "a18de960-1eab-ee11-be37-0022480174fd",
      "cites_unitreturned": null
    }

Reason:

- Dataverse Custom APIs validate parameter types.
- `cites_unitreturned` is a Dataverse Choice field and expects an integer.
- Sending `null` for a typed integer/choice parameter can cause a 400.
- Omitting the field entirely is treated as “not supplied”.

### Current Business Rules

- Permit must be in Issued state to endorse.
- Issued status code: `149900001`
- On success, current implementation sets permit status to Returned Used.
- Returned Used status code: `149900002`

### Response Schema

    {
      "permitId": "b35dd13d-1eab-ee11-be37-0022480191c1",
      "permitNumber": "24GBEXP9KQWWV",
      "previousStatuscode": 149900001,
      "newStatuscode": 149900002
    }

| Field | Type | Notes |
|---|---|---|
| `permitId` | string GUID | Permit record id |
| `permitNumber` | string | Permit number |
| `previousStatuscode` | integer | Status before endorsement |
| `newStatuscode` | integer | Status after endorsement |

### Current Endorsement Status Code Values

| Label | Value |
|---|---:|
| Draft | 1 |
| Issued | 149900001 |
| Returned - Used | 149900002 |
| Returned - Unused | 149900003 |
| Seized | 149900004 |
| Expired | 149900005 |

### Unit Choice Values

| Label | Value |
|---|---:|
| g | 149900000 |
| kg | 149900001 |
| l | 149900002 |
| cm3 | 149900003 |
| ml | 149900004 |
| m | 149900005 |
| m2 | 149900006 |
| m3 | 149900007 |
| no. = number of specimens | 149900008 |
| NB. If no unit of measure is specified | 149900009 |
| tonne | 149900010 |

---

## Deployment Notes

The current recommendation is to continue build and testing in cloned environments before touching the main Pegasus DEV/TEST streams.

Recommended path:

    Cloned DEV → Cloned TEST → PRE → PROD

Before importing downstream, confirm:

- Target environment is correct
- Required SharePoint site integration is configured
- Connection references are rebound
- Environment variables are populated
- Plugin assemblies/handlers are deployed correctly
- Custom API Plugin Type references are valid
- Required app registrations/application users exist
- Security roles are assigned

---

## SharePoint Dependencies

Pegasus uses SharePoint for permit templates and generated documents.

SharePoint is used for:

- Word permit templates
- Generated Word documents
- Generated PDF documents
- Document surfacing through Dynamics/Dataverse SharePoint integration

Known DEV/TEST SharePoint sites:

| Environment | SharePoint site |
|---|---|
| DEV | `https://defradev.sharepoint.com/sites/CITES01-DEV-SPO` |
| TEST | `https://defradev.sharepoint.com/sites/CITES01-TEST-SPO` |

Access to these sites is required for testing template/document generation changes.

---

## App Registration / Authentication Notes

Known app registrations used during development/testing:

| Environment | App registration |
|---|---|
| DEV | `ecites_pp_permit_management` |
| TEST | `ecites_pp_permit_management_test` |

Required Dataverse permission:

    Dynamics CRM → Delegated → user_impersonation

Redirect URIs identified for frontend testing:

    https://econtrol-frontend.dev.cdp-int.defra.cloud/auth/callback
    https://econtrol-frontend.test.cdp-int.defra.cloud/auth/callback

Do not commit or share client secrets in this repository.

Secrets must be stored in the approved Key Vault / secret management process.

---

## Known Issues / Follow-up

The following items were still open when the project paused:

- Final Border Force authentication model not agreed
- APIM not currently in place
- No automatic OpenAPI export currently available from APIM
- Manual OpenAPI file may be required if frontend needs schema import/codegen
- Plugin packaging/deployment approach needs to be confirmed
- Expanded Search API mapping still needs final validation
- QR URL/route not confirmed
- Internal APHA eCITES process still needs agreement
- Transition process required while current paper/manual process and new digital process coexist

---

## Important Implementation Notes

### Optional Endorsement Fields

For the Endorse API, optional fields should be omitted from the JSON request if they do not have a value.

Do not send `null` for typed fields such as Dataverse Choice/OptionSet values.

Good:

    {
      "permitId": "a18de960-1eab-ee11-be37-0022480174fd",
      "cites_quantityreturned": 10,
      "cites_tradedate": "2026-04-15"
    }

Avoid:

    {
      "permitId": "a18de960-1eab-ee11-be37-0022480174fd",
      "cites_unitreturned": null
    }

Reason: Dataverse Custom APIs validate parameter types and a Choice field expects an integer value.

---

## Next Steps for the Next Developer

1. Confirm cloned DEV and TEST environments.
2. Confirm solution import/export route.
3. Confirm plugin deployment approach.
4. Confirm SharePoint access and document library/template configuration.
5. Validate Solution 07 API/plugin dependencies.
6. Validate Solution 08 permit fields and Generate Permit flow.
7. Complete mapping for expanded Search API response.
8. Complete and test Endorse Permit plugin.
9. Confirm QR/barcode final design with frontend/security.
10. Agree internal APHA digital permit process and transition journey.

---

## Contact / Ownership

| Area | Contact / Team |
|---|---|
| Frontend / Node.js | Olivia Mackintosh |
| Pegasus / Dynamics | Accenture AMS / Pegasus technical lead |
| SharePoint access | Current CITES SharePoint owners |
| Entra ID / App registrations | CCoE / Entra support |
| Security / Cyber | DEFRA Security / Cyber representative |
| Product / process | APHA / Border Force product owners |

---

## Status

This repository represents an in-progress handover point.

The solutions are not yet production-ready and should not be deployed downstream without validating:

- Plugin dependencies
- Environment variables
- Connection references
- SharePoint configuration
- App registration/application user setup
- Process and security approvals
