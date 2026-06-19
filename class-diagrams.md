# Class Diagrams

Энэ документ нь Kass POS системийн үндсэн domain object-уудын class diagram-уудыг харуулна. Joplin дээр Mermaid plugin enabled үед эдгээр diagram шууд render хийгдэнэ.

## 1. Organization And Access

```mermaid
classDiagram
  class Organization {
    +id
    +name
    +regnum
    +email
    +industry
  }

  class Branch {
    +id
    +orgId
    +name
    +isDefault
    +location
    +isActive
  }

  class User {
    +id
    +email
    +name
    +lastName
    +phone
    +userName
  }

  class Role {
    +id
    +orgId
    +name
    +permissions
  }

  class UserRole {
    +id
    +userId
    +roleId
    +orgId
    +branchIds
  }

  class Permission {
    +id
    +name
    +description
    +permission
    +type
  }

  class BranchTaxConfig {
    +id
    +branchId
    +hasCityTax
    +cityTaxRate
    +hasVat
    +vatRate
    +hasServiceFee
    +serviceFeeRate
  }

  Organization "1" --> "many" Branch
  Organization "1" --> "many" Role
  Organization "1" --> "many" UserRole
  User "1" --> "many" UserRole
  Role "1" --> "many" UserRole
  Branch "1" --> "1" BranchTaxConfig
  Role ..> Permission : permissions JSON
```

## 2. Product Catalog And Inventory

```mermaid
classDiagram
  class Organization {
    +id
    +name
  }

  class Branch {
    +id
    +orgId
    +name
  }

  class Category {
    +id
    +orgId
    +referenceId
    +name
    +parentId
    +icon
  }

  class Product {
    +id
    +orgId
    +referenceId
    +categoryId
    +name
    +brand
    +vendorId
    +hasTax
    +hasCityTax
    +isBundle
    +status
  }

  class Variant {
    +id
    +orgId
    +referenceId
    +productId
    +sku
    +barCode
    +name
    +balance
    +options
  }

  class Price {
    +id
    +orgId
    +referenceId
    +branchId
    +productId
    +variantId
    +costPrice
    +sellPrice
    +bulkPrice
    +isRefundable
    +isActive
  }

  class Vendor {
    +id
    +orgId
    +name
  }

  class Stock {
    +id
    +orgId
    +branchId
    +productId
    +variantId
    +quantity
    +initialBalance
    +closingBalance
  }

  class ProductLog {
    +id
    +orgId
    +productId
    +variantId
    +priceId
    +inventoryItemId
    +orderItemId
    +type
    +value
    +unit
  }

  class SafetyStockConfig {
    +id
    +orgId
    +productId
    +variantId
    +minStock
    +isActive
  }

  class Bundle {
    +id
    +orgId
    +bundleId
    +itemId
    +count
  }

  Organization "1" --> "many" Category
  Organization "1" --> "many" Product
  Organization "1" --> "many" Vendor
  Category "1" --> "many" Product
  Product "1" --> "many" Variant
  Product "1" --> "many" Price
  Variant "1" --> "many" Price
  Branch "1" --> "many" Price
  Branch "1" --> "many" Stock
  Product "1" --> "many" Stock
  Variant "1" --> "many" Stock
  Product "1" --> "many" ProductLog
  Variant "1" --> "many" ProductLog
  Price "1" --> "many" ProductLog
  Stock "1" --> "many" ProductLog
  Product "1" --> "many" SafetyStockConfig
  Variant "1" --> "many" SafetyStockConfig
  Product "1" --> "many" Bundle : bundleId
  Variant "1" --> "many" Bundle : itemId
```

## 3. Checkout, Order, And Payment

```mermaid
classDiagram
  class Organization {
    +id
    +name
  }

  class Branch {
    +id
    +orgId
    +name
  }

  class Payment {
    +id
    +orgId
    +branchId
    +localId
    +status
    +paymentDate
    +paidAmount
    +method
  }

  class Receipt {
    +id
    +orgId
    +branchId
    +localId
    +status
  }

  class Order {
    +id
    +orgId
    +localId
    +receiptId
    +paymentId
    +totalAmount
    +taxAmount
    +discountAmount
    +finalAmount
  }

  class OrderItem {
    +id
    +orgId
    +localId
    +orderId
    +productId
    +variantId
    +priceId
    +quantity
    +unitPrice
    +totalAmount
  }

  class Product {
    +id
    +name
  }

  class Variant {
    +id
    +productId
    +name
    +sku
    +barCode
  }

  class Price {
    +id
    +productId
    +variantId
    +sellPrice
  }

  class Stock {
    +id
    +branchId
    +productId
    +variantId
    +quantity
  }

  class ProductLog {
    +id
    +productId
    +variantId
    +orderItemId
    +inventoryItemId
    +type
    +value
  }

  Organization "1" --> "many" Payment
  Organization "1" --> "many" Receipt
  Organization "1" --> "many" Order
  Branch "1" --> "many" Payment
  Branch "1" --> "many" Receipt
  Payment "1" --> "many" Order
  Receipt "1" --> "many" Order
  Order "1" --> "many" OrderItem
  Product "1" --> "many" OrderItem
  Variant "1" --> "many" OrderItem
  Price "1" --> "many" OrderItem
  OrderItem "1" --> "many" ProductLog
  Stock "1" --> "many" ProductLog
```

## 4. POS License And Device

```mermaid
classDiagram
  class Organization {
    +id
    +name
  }

  class LicenseType {
    +id
    +name
    +deviceLimit
    +userLimit
    +durationDays
    +offlineDayConfig
  }

  class License {
    +id
    +licenseTypeId
    +orgId
    +startDate
    +endDate
    +licenseStatus
  }

  class PosActivation {
    +id
    +orgId
    +branchId
    +publicKey
    +hwId
    +isActive
  }

  class PosSession {
    +id
    +installationId
    +issuedAt
    +expiresAt
  }

  class DesktopLocalSettings {
    +deviceId
    +publicKey
    +privateKey
    +orgId
    +branchId
    +activated
    +expiresAt
  }

  Organization "1" --> "many" License
  LicenseType "1" --> "many" License
  Organization "1" --> "many" PosActivation
  PosActivation "1" --> "many" PosSession
  DesktopLocalSettings ..> PosActivation : activates with publicKey
  DesktopLocalSettings ..> PosSession : stores activation expiry
```
