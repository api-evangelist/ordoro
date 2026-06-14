# Ordoro GraphQL Schema

## Overview

This document describes a conceptual GraphQL schema for the Ordoro multi-channel order management and ecommerce logistics platform. Ordoro provides a REST API for syncing orders across sales channels, managing inventory in multiple warehouses, creating shipping labels from major carriers, and handling dropshipping workflows with suppliers.

The GraphQL schema covers all major resource domains in the Ordoro platform:

- Orders and order line items across multiple sales channels
- Products, variants, SKUs, barcodes, and dimensions
- Inventory levels, adjustments, and warehouse locations
- Warehouses, bins, and physical locations
- Shipments, carriers, rates, labels, and tracking
- Suppliers and purchase orders
- Returns and return items
- Tags, API keys, tokens, and webhooks

## Schema Source

This schema is derived from the Ordoro REST API documentation at https://docs.ordoro.com/ and the Ordoro integrations API page at https://ordoro.com/integrations/api/. It represents a conceptual GraphQL translation of the underlying REST resource model.

## Types

### Order Domain

- `Order` — Top-level order record linking a customer purchase to a sales channel
- `OrderDetails` — Extended metadata for an order including dates, notes, and status history
- `OrderStatus` — Enum and type representing the lifecycle state of an order
- `OrderChannel` — The originating channel (e.g., Shopify, Amazon, manual) for an order
- `SalesChannel` — A configured sales channel integration in Ordoro
- `ChannelDetails` — Configuration and credentials details for a sales channel

### Line Items

- `OrderLine` — A single line entry on an order referencing product and quantity
- `LineItem` — Detailed line item with pricing, discounts, and fulfillment state
- `LineItemDetails` — Extended attributes on a line item
- `LineItemQuantity` — Quantity breakdowns (ordered, shipped, returned) for a line item

### Products

- `Product` — A sellable product record with name, description, and associated SKUs
- `ProductDetails` — Extended product metadata including images, categories, and custom fields
- `ProductSKU` — A specific stock-keeping unit identifier tied to a product
- `ProductBarcode` — Barcode value (UPC, EAN, etc.) associated with a product or variant
- `ProductWeight` — Weight value and unit for a product used in shipping calculations
- `ProductDimensions` — Length, width, height, and unit for a product

### Variants

- `Variant` — A specific combination of product options (size, color, etc.)
- `VariantDetails` — Extended metadata for a variant
- `VariantOption` — A single option dimension (e.g., Size: Large) on a variant

### Inventory

- `Inventory` — Aggregate inventory record for a product or variant
- `InventoryDetails` — Detailed inventory state including reserved and available quantities
- `InventoryLevel` — Inventory quantity at a specific warehouse location
- `InventoryAdjustment` — A recorded change to inventory quantity with reason and timestamp

### Warehouses

- `Warehouse` — A physical or virtual storage facility configured in Ordoro
- `WarehouseDetails` — Address, contact, and configuration details for a warehouse
- `WarehouseLocation` — A named location zone within a warehouse
- `Bin` — A specific bin or shelf slot within a warehouse location
- `BinDetails` — Position, capacity, and contents metadata for a bin

### Shipments

- `Shipment` — A shipment record grouping one or more packages sent to fulfill order lines
- `ShipmentDetails` — Carrier, service, dimensions, weight, and cost for a shipment
- `ShipmentStatus` — Lifecycle state of a shipment (pending, in transit, delivered, etc.)
- `ShipmentCarrier` — The carrier assigned to a shipment
- `TrackingNumber` — A carrier-issued tracking identifier for a shipment
- `ShipmentLabel` — The label generated for a shipment package

### Labels

- `Label` — A shipping label record with image URL and associated metadata
- `LabelDetails` — Cost, format, dimensions, and carrier details for a label

### Carriers and Rates

- `Carrier` — A shipping carrier (UPS, FedEx, USPS, DHL, etc.) configured in Ordoro
- `CarrierDetails` — Account credentials and service options for a carrier
- `CarrierRate` — A quoted rate from a carrier for a given package and destination
- `ShippingRate` — A normalized rate result including transit time and cost
- `Package` — A physical package definition with weight and dimensions
- `PackageDetails` — Packaging type, material, and dimensional details

### Suppliers and Purchase Orders

- `Supplier` — A supplier or vendor record for dropshipping or restocking
- `SupplierDetails` — Contact, lead time, and account information for a supplier
- `PurchaseOrder` — A purchase order sent to a supplier to restock inventory
- `PODetails` — Line items, costs, and expected receipt dates for a purchase order
- `POLineItem` — A single product line on a purchase order with quantity and cost
- `POStatus` — Lifecycle state of a purchase order (draft, sent, received, cancelled)

### Returns

- `Return` — A return record associated with an original order
- `ReturnDetails` — Reason, condition, and resolution details for a return
- `ReturnItem` — A specific product or variant being returned
- `ReturnStatus` — State of a return (requested, received, refunded, restocked)

### Platform

- `Tag` — A user-defined label applied to orders, products, or other resources
- `TagDetails` — Color, description, and usage metadata for a tag
- `APIKey` — An API credential key for authenticating to the Ordoro API
- `Token` — A session or access token issued for API access
- `Webhook` — A configured webhook endpoint for receiving event notifications
- `WebhookEvent` — A specific event payload delivered to a webhook endpoint

## Queries

The schema exposes the following top-level queries:

- `order(id: ID!)` — Fetch a single order by ID
- `orders(status: OrderStatus, channel: String, limit: Int, offset: Int)` — List orders with optional filters
- `product(id: ID!)` — Fetch a single product
- `products(sku: String, tag: String, limit: Int, offset: Int)` — List products
- `inventory(productId: ID!, warehouseId: ID)` — Get inventory levels for a product
- `warehouse(id: ID!)` — Fetch a single warehouse
- `warehouses` — List all warehouses
- `shipment(id: ID!)` — Fetch a single shipment
- `shipments(orderId: ID, status: ShipmentStatus, limit: Int, offset: Int)` — List shipments
- `carrier(id: ID!)` — Fetch a carrier configuration
- `carriers` — List all configured carriers
- `supplier(id: ID!)` — Fetch a supplier
- `suppliers` — List all suppliers
- `purchaseOrder(id: ID!)` — Fetch a purchase order
- `purchaseOrders(supplierId: ID, status: POStatus, limit: Int, offset: Int)` — List POs
- `return(id: ID!)` — Fetch a return record
- `returns(orderId: ID, status: ReturnStatus, limit: Int, offset: Int)` — List returns
- `tags` — List all tags
- `webhooks` — List configured webhooks

## Mutations

- `createOrder(input: OrderInput!)` — Create a new manual order
- `updateOrderStatus(id: ID!, status: OrderStatus!)` — Update an order's status
- `addTagToOrder(orderId: ID!, tagId: ID!)` — Apply a tag to an order
- `allocateInventory(productId: ID!, warehouseId: ID!, quantity: Int!)` — Reserve inventory
- `adjustInventory(input: InventoryAdjustmentInput!)` — Record an inventory adjustment
- `createShipment(input: ShipmentInput!)` — Create a new shipment
- `purchaseLabel(shipmentId: ID!, carrierId: ID!, serviceCode: String!)` — Buy a shipping label
- `voidLabel(labelId: ID!)` — Void a previously purchased label
- `createPurchaseOrder(input: POInput!)` — Create a purchase order
- `receivePurchaseOrder(id: ID!, lines: [POReceiptLine!]!)` — Mark PO lines as received
- `createReturn(input: ReturnInput!)` — Initiate a return for an order
- `createWebhook(input: WebhookInput!)` — Register a webhook endpoint
- `deleteWebhook(id: ID!)` — Remove a webhook

## Authentication

The Ordoro API uses Basic HTTP Authentication with API keys. In a GraphQL context, the API key would be passed as a Bearer token or Basic Auth header on every request. API access requires a Premium-level Ordoro plan or higher.

## Rate Limiting

The Ordoro API enforces a rate limit of 500 requests per minute. GraphQL queries that resolve multiple nested resources may internally make multiple REST calls and should be designed with batching and caching in mind to stay within this limit.

## References

- Ordoro API Documentation: https://docs.ordoro.com/
- Ordoro Integrations API: https://ordoro.com/integrations/api/
- Ordoro Developer: https://www.ordoro.com/developer
