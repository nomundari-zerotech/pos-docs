# POS Login And License Activation

```plantuml
@startuml
title POS Login And License Activation

|Cashier|
start
:Open POS desktop;
:Enter username and PIN;

|Desktop App|
:Read saved orgId and branchId;
:Send login request;

|POS Backend|
:Authenticate POS user;
if (Credentials valid?) then (Yes)
  :Return bearer access token;
else (No)
  :Return login error;
  |Desktop App|
  :Show login error;
  |Cashier|
  :Retry login;
  stop
endif

|Desktop App|
if (Saved orgId exists?) then (Yes)
  :Request scoped POS session;
else (No)
  :Load organization list;
  |Cashier|
  :Select organization;
  |Desktop App|
  :Request scoped POS session;
endif

|POS Backend|
:Issue organization-scoped session;

|Desktop App|
if (Saved branchId exists?) then (Yes)
  :Use saved branch;
else (No)
  :Load branch list;
  |Cashier|
  :Select branch;
endif

|Desktop App|
:Save local user, org, and branch settings;
:Generate or read device id and key pair;
:Request POS license activation;

|POS Backend|
:Find active organization license;
if (License active?) then (Yes)
else (No)
  :Reject activation;
  |Desktop App|
  :Show license error;
  stop
endif

|POS Backend|
if (Device limit available?) then (Yes)
  :Create or reuse POS activation;
  :Create POS activation session;
  :Return activation expiry;
else (No)
  :Reject activation;
  |Desktop App|
  :Show device limit error;
  stop
endif

|Desktop App|
:Persist activation state locally;
:Store auth session for POS app;

|Cashier|
:Enter POS main screen;
stop
@enduml
