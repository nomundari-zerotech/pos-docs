# Master Data Product Import Sequence

This sequence follows the current backend implementation in `apps/backend/master-data/src/product`.

```mermaid
sequenceDiagram
  autonumber
  actor MasterUser as Master User
  participant API as Import endpoint
  participant Controller as product.controller.importProducts
  participant Category as CategoryService
  participant Vendor as VendorService
  participant Product as product.service.createProduct
  participant Variant as VariantService.createVariants
  participant Bundle as BundleService.createBundle
  participant DB as masterDb

  MasterUser->>API: POST /master-data/products/import with JSON file
  API->>Controller: Validate body with fileInputSchema
  Controller->>Controller: requireTextFile(file)
  Controller->>Controller: JSON.parse(await file.text())

  loop For each product in uploaded JSON
    alt product.categoryName exists
      Controller->>Category: findOrCreateCategoryByName(categoryName, "supermarket")
      Category->>DB: Select category by name and industry
      alt Category exists
        DB-->>Category: Existing category
      else Category missing
        Category->>DB: Insert masterCategories row
        DB-->>Category: Created category
      end
      Category-->>Controller: category id
    end

    alt product.vendor exists
      Controller->>Vendor: findOrCreateVendorByName(vendor)
      Vendor->>DB: Select vendor by name match
      alt Vendor exists
        DB-->>Vendor: Existing vendor
      else Vendor missing
        Vendor->>DB: Insert masterVendors row
        DB-->>Vendor: Created vendor
      end
      Vendor-->>Controller: vendor id
    end

    opt product.variants is an array
      Controller->>Controller: Normalize each variant with price = 0
    end

    Controller->>Product: createProduct(product + categoryId + vendorId)
    Product->>DB: Start transaction
    Product->>DB: Insert masterProducts row

    alt Product has variants
      Product->>Variant: createVariants(variants with productId and default prices)
      Variant->>DB: Insert masterVariants rows
      Variant->>DB: Insert masterPrices rows
      Variant-->>Product: Variants with price rows
    else No variants
      Product-->>Controller: Product with empty variants
    end

    opt product.isBundle and bundleItems exist
      Product->>Bundle: createBundle(bundleItems, product.id)
      Bundle->>DB: Insert bundle rows
    end

    Product->>DB: Commit transaction
    Product-->>Controller: Created product result
    Controller->>Controller: Push result into results
  end

  Controller-->>API: Imported product results
  API-->>MasterUser: Products imported successfully
```

## Code References

- `apps/backend/master-data/src/product/product.routes.ts`
- `apps/backend/master-data/src/product/product.controller.ts`
- `apps/backend/master-data/src/product/product.service.ts`
- `apps/backend/master-data/src/variant/variant.service.ts`
- `apps/backend/master-data/src/category/master.category.service.ts`
- `apps/backend/master-data/src/vendor/vendor.service.ts`
- `apps/backend/master-data/src/bundles/master-bundle.service.ts`
