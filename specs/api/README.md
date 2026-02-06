# API Specifications

This directory contains OpenAPI / Swagger contract definitions for the OctoCAT Supply Chain API.

## Endpoints

| Endpoint | Entity | Methods |
|----------|--------|---------|
| `/api/suppliers` | Supplier | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/products` | Product | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/orders` | Order | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/order-details` | OrderDetail | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/deliveries` | Delivery | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/order-detail-deliveries` | OrderDetailDelivery | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/headquarters` | Headquarters | GET, POST, GET/:id, PUT/:id, DELETE/:id |
| `/api/branches` | Branch | GET, POST, GET/:id, PUT/:id, DELETE/:id |

## Conventions

- All responses use JSON
- List endpoints return arrays
- Create (`POST`) returns `201` with the created entity
- Update (`PUT`) returns `200` with the updated entity
- Delete (`DELETE`) returns `204` with no body
- Not found returns `404`
- Validation errors return `400`
- Conflict errors return `409`

Refer to the Swagger UI served by the API at `/api-docs` for interactive documentation.
