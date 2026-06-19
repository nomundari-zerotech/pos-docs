# Signed POS Request Verification

```plantuml
@startuml
title Signed POS Request Verification

actor Cashier
participant "POS Desktop" as Desktop
database "Local Settings" as Settings
participant "Signed POS Client" as SignedClient
participant "Backend /pos" as PosApi
participant "License Service" as LicenseService
database "License Tables" as LicenseDb

Cashier -> Desktop: Open protected POS data screen
Desktop -> SignedClient: Request protected POS data
SignedClient -> Settings: Read auth session and activation keys
Settings --> SignedClient: accessToken, deviceId, privateKey

alt Session expired or near expiry
  SignedClient -> PosApi: POST /pos/session
  PosApi --> SignedClient: Refreshed accessToken
  SignedClient -> Settings: Save refreshed session
end

SignedClient -> SignedClient: Hash request body
SignedClient -> SignedClient: Create nonce and timestamp
SignedClient -> SignedClient: Sign canonical request with private key
SignedClient -> PosApi: GET protected data with bearer token and signed headers

PosApi -> LicenseService: Verify device id, nonce, timestamp, hash, signature
LicenseService -> LicenseDb: Load activation and latest session
LicenseDb --> LicenseService: Public key and activation context
LicenseService -> LicenseService: Reject replayed nonce or stale timestamp
LicenseService -> LicenseService: Verify signature using public key

alt Signature valid
  LicenseService --> PosApi: Verification accepted
  PosApi --> SignedClient: Protected POS data
  SignedClient --> Desktop: Data result
else Signature invalid
  LicenseService --> PosApi: Verification rejected
  PosApi --> SignedClient: Unauthorized response
  SignedClient --> Desktop: Show access error
end
@enduml
