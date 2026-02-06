# OctoCAT Entity Model

This document describes the domain entities used throughout the OctoCAT Supply Chain application. These entities are shared across the API (`octocat-supply-api`) and frontend (`octocat-supply-web`) repositories.

## Entity Relationships

```
  Headquarters ──┐
                 ├──▶ Branch ──▶ Order ──▶ OrderDetail ──▶ OrderDetailDelivery
  Supplier ──────┤                              │                  │
                 └──▶ Delivery ────────────────────────────────────┘
                                          Product ◄────────┘
```

## Entities

### Supplier
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| supplierId | number | ✅ | Primary key |
| name | string | ✅ | Supplier name |
| description | string | | Additional details |
| contactPerson | string | | Primary contact |
| email | string | | Contact email |
| phone | string | | Contact phone |
| active | boolean | | Whether active |
| verified | boolean | | Whether verified |

### Product
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| productId | number | ✅ | Primary key |
| supplierId | number | ✅ | FK → Supplier |
| name | string | ✅ | Product name |
| description | string | | Product description |
| price | number | ✅ | Current price |
| sku | string | | Stock keeping unit |
| unit | string | | Unit of measure |
| imgName | string | | Product image filename |
| discount | number | | Discount percentage (0.25 = 25%) |

### Order
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| orderId | number | ✅ | Primary key |
| branchId | number | ✅ | FK → Branch |
| orderDate | string | ✅ | ISO 8601 datetime |
| name | string | | Order name |
| description | string | | Order description |
| status | string | | pending / processing / shipped / delivered / cancelled |

### OrderDetail
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| orderDetailId | number | ✅ | Primary key |
| orderId | number | ✅ | FK → Order |
| productId | number | ✅ | FK → Product |
| quantity | number | ✅ | Quantity ordered |
| unitPrice | number | ✅ | Price per unit |
| notes | string | | Additional notes |

### Delivery
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| deliveryId | number | ✅ | Primary key |
| supplierId | number | ✅ | FK → Supplier |
| deliveryDate | string | | ISO 8601 datetime |
| name | string | | Delivery name |
| description | string | | Notes |
| status | string | | pending / in-transit / delivered / failed |

### OrderDetailDelivery
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| orderDetailDeliveryId | number | ✅ | Primary key |
| orderDetailId | number | ✅ | FK → OrderDetail |
| deliveryId | number | ✅ | FK → Delivery |
| quantity | number | | Items in this delivery |
| notes | string | | Additional notes |

### Headquarters
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| headquartersId | number | ✅ | Primary key |
| name | string | ✅ | HQ name |
| description | string | | Additional details |
| address | string | | Main office address |
| contactPerson | string | | Primary contact |
| email | string | | Contact email |
| phone | string | | Contact phone |
| city | string | | City location |
| country | string | | Country location |
| floorCount | number | | Number of floors |
| capacity | number | | Total capacity |

### Branch
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| branchId | number | ✅ | Primary key |
| headquartersId | number | ✅ | FK → Headquarters |
| name | string | ✅ | Branch name |
| description | string | | Additional details |
| address | string | | Physical address |
| contactPerson | string | | Primary contact |
| email | string | | Contact email |
| phone | string | | Contact phone |

## Naming Conventions

| Context | Convention | Example |
|---------|------------|---------|
| TypeScript interface | PascalCase | `Supplier` |
| Interface property | camelCase | `supplierId` |
| Database table | snake_case, plural | `suppliers` |
| Database column | snake_case | `supplier_id` |
| Route path | kebab-case, plural | `/api/suppliers` |
| Repository class | PascalCase + Repository | `SuppliersRepository` |
