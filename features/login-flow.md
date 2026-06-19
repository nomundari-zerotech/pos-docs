# Desktop Login Flow

```mermaid
sequenceDiagram
  autonumber
  actor Cashier
  participant Desktop as Desktop App
  participant LocalDB as Local DB / Settings
  participant PosAuth as POS Auth API
  participant PosOrg as POS Org API
  participant PosLicense as POS License API
  participant PosData as POS Data APIs

  Cashier->>Desktop: Enter username + PIN
  Desktop->>LocalDB: Read saved orgId / branchId
  Desktop->>PosAuth: POST /pos/login
  PosAuth-->>Desktop: accessToken, expiresAt, user orgId

  alt Saved orgId exists
    Desktop->>PosAuth: POST /pos/session with saved orgId
    PosAuth-->>Desktop: scoped accessToken, expiresAt
  else User must choose organization
    Desktop->>PosOrg: GET /pos/org
    PosOrg-->>Desktop: organizations
    Cashier->>Desktop: Select organization
    Desktop->>PosAuth: POST /pos/session with selected orgId
    PosAuth-->>Desktop: scoped accessToken, expiresAt
    Desktop->>PosOrg: GET /pos/org/branches
    PosOrg-->>Desktop: branches
    Cashier->>Desktop: Select branch
  end

  Desktop->>LocalDB: Save local user login
  Desktop->>LocalDB: Save orgId and branchId in settings
  Desktop->>LocalDB: Generate/read deviceId + public/private key
  Desktop->>PosLicense: POST /pos/license/activate with branchId, deviceId, publicKey
  PosLicense-->>Desktop: POS activation session expiresAt
  Desktop->>LocalDB: Save activation state and activation expiresAt
  Desktop->>Desktop: Store auth session in sessionStorage
  Desktop-->>Cashier: Open POS main screen

  Cashier->>Desktop: Fetch/sync POS data
  Desktop->>Desktop: Check auth session expiresAt
  alt Auth session expired or near expiry
    Desktop->>PosAuth: POST /pos/session with saved orgId
    PosAuth-->>Desktop: refreshed accessToken, expiresAt
    Desktop->>Desktop: Update sessionStorage
  end

  Desktop->>LocalDB: Read activation keys and deviceId
  Desktop->>Desktop: Hash request body
  Desktop->>Desktop: Create nonce + timestamp
  Desktop->>Desktop: Sign canonical request with privateKey
  Desktop->>PosData: GET protected data with signed headers
  PosData->>PosLicense: Verify deviceId, bodyHash, nonce, timestamp, signature
  PosLicense-->>PosData: Signature valid
  PosData-->>Desktop: Protected POS data
```

## Signed Headers

- `x-pos-device-id`
- `x-pos-body-hash`
- `x-pos-nonce`
- `x-pos-signature`
- `x-pos-timestamp`

Org selection, branch selection, session creation, and activation are bearer-authenticated but not signed because they run before the desktop has an activated trusted device key.
