# Inventory Stock Movement

```plantuml

@startuml
title Inventory Stock Movement

|Inventory User|
start
:Choose stock operation;

|Admin/POS Frontend|
if (Receive or adjust stock?) then (Yes)
  :Enter product, variant, branch, and quantity;
else (No)
  if (Sale stock movement?) then (Yes)
    :Complete POS checkout;
  else (No)
    :Select completed order item for refund;
  endif
endif

|Backend or Desktop Service|
:Validate organization and branch context;
:Load current stock record;

if (Stock record exists?) then (Yes)
else (No)
  :Create stock record for product, variant, and branch;
endif

if (Operation reduces stock?) then (Yes)
  if (Enough stock?) then (Yes)
    :Decrease quantity;
  else (No)
    :Reject stock movement;
    |Inventory User|
    :Resolve stock issue;
    stop
  endif
else (No)
  :Increase quantity;
endif

:Write product log with movement type and value;

if (Safety stock config active?) then (Yes)
  if (Quantity below minimum?) then (Yes)
    :Mark low stock condition;
    :Prepare reminder or notification context;
  else (No)
  endif
else (No)
endif

|Database|
:Persist stock and product log changes;

|Admin/POS Frontend|
:Show updated stock result;

|Inventory User|
:Finish inventory operation;
stop
@enduml
