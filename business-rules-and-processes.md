# Business Rules And Processes

Энэ документ нь Kass POS системийн гол бизнес дүрэм, системийн behavior, процессын диаграм, холбогдох code reference-үүдийг нэг дор тайлбарлана.

## 1. Authentication And Organization Access

### Business Rules

- Хэрэглэгч нэг эрхээр олон байгууллага буюу `Org` рүү хандах боломжтой.
- Хэрэглэгч Admin систем рүү нэвтрэхэд тухайн хэрэглэгчид хамаарах бүх `Org` жагсаалт харагдана.
- Хэрэглэгч жагсаалтаас нэг `Org` сонгож тухайн байгууллагын context дотор ажиллана.
- Хэрэглэгч өөрийн шинэ `Org` үүсгэх боломжтой.
- Admin веб систем рүү хэрэглэгч нууц үгээр нэвтэрнэ.
- POS систем рүү хэрэглэгч PIN кодоор нэвтэрнэ.

### System Behavior

- Admin нэвтрэлт нь хэрэглэгчийн үндсэн credential буюу username/password дээр суурилна.
- POS нэвтрэлт нь username/PIN дээр суурилна.
- Нэвтэрсний дараа хэрэглэгчийн `orgId`, `branchId`, role, permission нь тухайн session-ийн context болно.
- Хэрэглэгч олон `Org`-той бол систем эхлээд `Org` сонголт харуулна.
- POS desktop нь branch сонголт болон device/license activation хийсний дараа үндсэн POS screen рүү орно.

### Process Diagram

![POS login and license activity](diagrams/pos-login-license-activity.svg)

### Related Code

- `apps/backend/pos/src/auth`
- `apps/backend/pos/src/org`
- `apps/backend/pos/src/license`
- `apps/frontend/desktop/src/main/services/auth`
- `apps/frontend/desktop/src/main/services/desktop-pos-service.ts`

## 2. POS License And Signed Requests

### Business Rules

- POS desktop төхөөрөмж бүр backend дээр license activation хийсэн байх ёстой.
- POS device нь `deviceId`, public key, private key ашиглан trusted device болж бүртгэгдэнэ.
- Backend зөвхөн active license бүхий байгууллагын POS activation-г зөвшөөрнө.
- Protected POS data request бүр signed header-тэй байна.
- Давтагдсан nonce, буруу signature, хугацаа хэтэрсэн timestamp бүхий request-г зөвшөөрөхгүй.

### System Behavior

- Desktop private key-г зөвхөн local settings дотор хадгална.
- Backend public key болон activation/session мэдээллийг хадгална.
- Protected POS request хийхдээ desktop request body-г hash хийж, nonce/timestamp үүсгэнэ.
- Backend signature-г public key ашиглан шалгана.
- Signature valid бол request цааш API handler руу орно.

### Process Diagram

![Signed POS request sequence](diagrams/signed-pos-request-sequence.svg)

### Related Code

- `apps/frontend/desktop/src/main/services/shared/pos-signed-client.ts`
- `apps/frontend/desktop/src/main/services/auth/pos-license-service.ts`
- `apps/backend/pos/src/signing/pos-signed-request.ts`
- `apps/backend/pos/src/license/license.service.ts`
- `packages/pos-signing/src/shared.ts`

## 3. Role And Permission Access

### Business Rules

- API route бүр permission шаардаж болно.
- Хэрэглэгч зөвхөн өөрийн role-д зөвшөөрөгдсөн permission-тэй route руу хандана.
- Permission шалгалт нь сонгосон `Org` context дотор хийгдэнэ.
- Permission bootstrap буюу `permission-sync` нь intentional бөгөөд RBAC permission seed хийх үүрэгтэй.

### System Behavior

- Backend bearer token-г шалгаж user, org, role context гаргана.
- Route permission шаардаж байвал backend role permission жагсаалтыг database-аас уншина.
- Required permission role дотор байвал request-г зөвшөөрнө.
- Permission байхгүй бол forbidden response буцаана.

### Process Diagram

![RBAC permission check activity](diagrams/rbac-permission-check-activity.svg)

### Related Code

- `apps/backend/src/common/protected-app.ts`
- `apps/backend/src/common/permission-sync.ts`
- `packages/permission-middleware`
- `packages/db-schema/src/core/org.ts`

## 4. POS Checkout And Payment

### Business Rules

- Checkout зөвхөн cart дотор item байгаа үед дуусах боломжтой.
- Product, variant, price, stock мэдээлэл checkout хийх үед valid байх ёстой.
- Payment амжилттай болсон тохиолдолд order үүснэ.
- Order үүссэний дараа stock quantity буурч product log бичигдэнэ.
- Payment failed бол cashier retry хийх эсвэл checkout-г cancel хийх боломжтой.

### System Behavior

- POS desktop local product, price, stock data ашиглан cart бүрдүүлнэ.
- Checkout үед subtotal, tax, discount, final amount тооцно.
- Payment method provider шаарддаг бол backend `/pos/payments` endpoint ашиглана.
- Cash/manual payment бол local payment record үүсгэнэ.
- Амжилттай payment-ийн дараа local DB-д payment, order, order items, stock update, product logs хадгалагдана.

### Process Diagrams

![POS checkout activity](diagrams/pos-checkout-activity.svg)

![POS checkout sequence](diagrams/pos-checkout-sequence.svg)

### Related Code

- `apps/frontend/desktop/src/main/db/repositories/order-repository.ts`
- `apps/frontend/desktop/src/main/services/payment-process`
- `apps/backend/pos/src/payment`
- `packages/db-schema/src/lite/sales.ts`
- `packages/db-schema/src/lite/payment.ts`
- `packages/db-schema/src/lite/inventory.ts`

## 5. Master Data Import

### Business Rules

- Master data нь category, product, variant, price, vendor, bundle-ийн эх үүсвэр болно.
- Organization-local product үүсэхдээ master record-ийн `referenceId`-г хадгална.
- Аль хэдийн import хийгдсэн product/category/vendor байвал duplicate үүсгэхгүй.
- Bundle product бол bundle item reference-үүд хамт үүснэ.

### System Behavior

- Admin эсвэл super-admin хэрэглэгч master data-аас product сонгоно.
- Backend master schema-аас selected master records уншина.
- Backend тухайн organization-д category/vendor/product/variant/price reference байгаа эсэхийг шалгана.
- Байхгүй reference-үүдийг үүсгээд organization-local catalog дотор хадгална.

### Process Diagram

![Master data import activity](diagrams/master-data-import-activity.svg)

### Related Code

- `apps/backend/master-data/src`
- `apps/backend/super-admin/src/product`
- `apps/backend/admin/src/category`
- `apps/backend/src/domain/product/product-create.ts`
- `packages/db-schema/src/core/master.ts`

## 6. Inventory Stock Movement

### Business Rules

- Stock movement бүр product, variant, branch context-тэй байна.
- Stock бууруулах operation хийхэд хангалттай үлдэгдэл байх ёстой.
- Sale checkout амжилттай бол stock буурна.
- Refund эсвэл receive operation stock нэмэгдүүлж болно.
- Stock movement бүр product log үүсгэнэ.
- Safety stock active бөгөөд үлдэгдэл minimum stock-оос бага бол low stock condition үүснэ.

### System Behavior

- Систем stock record байгаа эсэхийг шалгана.
- Байхгүй бол product/variant/branch-д зориулж stock record үүсгэнэ.
- Operation type-оос хамаарч quantity нэмэгдэнэ эсвэл буурна.
- Product log нь movement type, value, unit, product/variant reference хадгална.
- Safety stock тохиргоо active бол movement-ийн дараа үлдэгдлийг шалгана.

### Process Diagram

![Inventory stock movement activity](diagrams/inventory-stock-movement-activity.svg)

### Related Code

- `packages/db-schema/src/core/inventory.ts`
- `packages/db-schema/src/lite/inventory.ts`
- `apps/frontend/desktop/src/main/db/repositories/order-repository.ts`
- `apps/backend/pos/src`

## 7. System Component Overview

### Business Rules

- Backend нь нэг Elysia monolith хэвээр байна.
- Backend deployable service-г `/admin`, `/pos`, `/super-admin`, `/master-data` гэж тусад нь салгахгүй.
- Frontend app бүр өөрийн surface рүү хандана.
- Shared packages нь schema, database access, UI, permission middleware зэрэг cross-cutting logic-г хуваалцана.

### System Behavior

- `apps/backend` нь бүх backend surface-г нэг Elysia app дотор mount хийдэг.
- Admin, Super Admin, Internal Master Data, Desktop renderer нь тусдаа frontend app байна.
- PostgreSQL нь backend-ийн үндсэн persistent database.
- POS desktop нь PGlite local DB болон local settings ашиглана.

### Component Diagram

![System components](diagrams/system-components.svg)

### Related Code

- `apps/backend/src/app.ts`
- `apps/backend/src/index.ts`
- `apps/frontend/admin`
- `apps/frontend/super-admin`
- `apps/frontend/internal-master-data`
- `apps/frontend/desktop`
- `packages/db-core`
- `packages/db-lite`
- `packages/db-schema`
