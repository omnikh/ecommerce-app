
-- =========================================================
-- 1. USERS, ROLES AND PERMISSIONS
-- =========================================================

CREATE TABLE dbo.users (
    user_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_users PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    phone_no VARCHAR(30) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_users_is_active DEFAULT (1),
    is_deleted BIT NOT NULL
       CONSTRAINT DF_users_is_deleted DEFAULT (0),	
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_users_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT UQ_users_email UNIQUE (email),
    CONSTRAINT UQ_users_phone_no UNIQUE (phone_no)
);

CREATE TABLE dbo.roles (
    role_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_roles PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_roles_is_active DEFAULT (1),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_roles_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT UQ_roles_role_name UNIQUE (role_name)
);

CREATE TABLE dbo.permissions (
    permission_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_permissions PRIMARY KEY,
    permission_name VARCHAR(100) NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_permissions_is_active DEFAULT (1),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_permissions_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT UQ_permissions_permission_name UNIQUE (permission_name)
);

CREATE TABLE dbo.user_role_mapping (
    user_role_mapping_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_user_role_mapping PRIMARY KEY,
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_user_role_mapping_is_active DEFAULT (1),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_user_role_mapping_created_at DEFAULT (SYSUTCDATETIME()),
    CONSTRAINT FK_user_role_mapping_users
        FOREIGN KEY (user_id) REFERENCES dbo.users(user_id),
    CONSTRAINT FK_user_role_mapping_roles
        FOREIGN KEY (role_id) REFERENCES dbo.roles(role_id)
		ON DELETE CASCADE,
    CONSTRAINT UQ_user_role_mapping UNIQUE (user_id, role_id)
);

CREATE TABLE dbo.role_permission_mapping (
    role_permission_mapping_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_role_permission_mapping PRIMARY KEY,
    role_id BIGINT NOT NULL,
    permission_id BIGINT NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_role_permission_mapping_is_active DEFAULT (1),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_role_permission_mapping_created_at DEFAULT (SYSUTCDATETIME()),
    CONSTRAINT FK_role_permission_mapping_roles
        FOREIGN KEY (role_id) REFERENCES dbo.roles(role_id)
		ON DELETE CASCADE,
    CONSTRAINT FK_role_permission_mapping_permissions
        FOREIGN KEY (permission_id) REFERENCES dbo.permissions(permission_id)
		ON DELETE CASCADE,
    CONSTRAINT UQ_role_permission_mapping UNIQUE (role_id, permission_id)
);


-- =========================================================
-- 2. ADDRESS BOOK
-- =========================================================

CREATE TABLE dbo.addresses (
    address_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_addresses PRIMARY KEY,
    user_id BIGINT NOT NULL,
    address_label VARCHAR(50) NOT NULL
        CONSTRAINT DF_addresses_label DEFAULT ('Home'),
    full_name VARCHAR(150) NOT NULL,
    phone_number VARCHAR(30) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    address_line_2 VARCHAR(255) NULL,
    city VARCHAR(100) NOT NULL,
    state_province_region VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    country_code CHAR(2) NOT NULL
        CONSTRAINT DF_addresses_country DEFAULT ('IN'),
    is_default_shipping BIT NOT NULL
        CONSTRAINT DF_addresses_default_shipping DEFAULT (0),
    is_default_billing BIT NOT NULL
        CONSTRAINT DF_addresses_default_billing DEFAULT (0),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_addresses_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_addresses_users
        FOREIGN KEY (user_id) REFERENCES dbo.users(user_id)
);

-- At most one default shipping and billing address per user.
CREATE UNIQUE INDEX UX_addresses_default_shipping
ON dbo.addresses(user_id)
WHERE is_default_shipping = 1;

CREATE UNIQUE INDEX UX_addresses_default_billing
ON dbo.addresses(user_id)
WHERE is_default_billing = 1;


-- =========================================================
-- 3. CATALOGUE
-- =========================================================
--can be hard deleted, the application layer should handle and re-assign new category to relationships
CREATE TABLE dbo.categories (
    category_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_categories PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    category_description VARCHAR(500) NULL,
    parent_category_id BIGINT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_categories_is_active DEFAULT (1),		
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_categories_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_categories_parent
        FOREIGN KEY (parent_category_id)
        REFERENCES dbo.categories(category_id),
    CONSTRAINT CK_categories_not_own_parent
        CHECK (parent_category_id IS NULL OR parent_category_id <> category_id)
);

-- Root categories: names must be unique among root categories.
CREATE UNIQUE INDEX UX_categories_root_name
ON dbo.categories(category_name)
WHERE parent_category_id IS NULL;

-- Child categories: names must be unique within the same parent.
CREATE UNIQUE INDEX UX_categories_child_name
ON dbo.categories(parent_category_id, category_name)
WHERE parent_category_id IS NOT NULL;


CREATE TABLE dbo.products (
    product_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_products PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    product_description NVARCHAR(MAX) NOT NULL,
    category_id BIGINT NOT NULL,
    cod_eligible BIT NOT NULL
        CONSTRAINT DF_products_cod_eligible DEFAULT (1),
    is_active BIT NOT NULL
        CONSTRAINT DF_products_is_active DEFAULT (1),
	is_deleted BIT NOT NULL
       CONSTRAINT DF_products_is_deleted DEFAULT (0),		
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_products_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_products_categories
        FOREIGN KEY (category_id) REFERENCES dbo.categories(category_id)
);


CREATE TABLE dbo.product_variants (
    product_variant_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_product_variants PRIMARY KEY,
    product_id BIGINT NOT NULL,
    variant_specs NVARCHAR(1000) NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_product_variants_is_active DEFAULT (1),
		is_deleted BIT NOT NULL
       CONSTRAINT DF_product_variants_is_deleted DEFAULT (0),	
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_product_variants_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_product_variants_products
        FOREIGN KEY (product_id) REFERENCES dbo.products(product_id)
);


-- A seller creates a listing for a product variant.
CREATE TABLE dbo.listings (
    listing_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_listings PRIMARY KEY,
    product_variant_id BIGINT NOT NULL,
    seller_id BIGINT NOT NULL,
    price DECIMAL(19,4) NOT NULL,
    inventory INT NOT NULL,
    is_active BIT NOT NULL
        CONSTRAINT DF_listings_is_active DEFAULT (1),
	is_deleted BIT NOT NULL
    CONSTRAINT DF_listings_is_deleted DEFAULT (0),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_listings_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_listings_product_variants
        FOREIGN KEY (product_variant_id)
        REFERENCES dbo.product_variants(product_variant_id),
    CONSTRAINT FK_listings_sellers
        FOREIGN KEY (seller_id) REFERENCES dbo.users(user_id),
    CONSTRAINT CK_listings_price CHECK (price > 0),
    CONSTRAINT CK_listings_inventory CHECK (inventory >= 0)
);


-- =========================================================
-- 4. CART
-- =========================================================

CREATE TABLE dbo.carts (
    cart_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_carts PRIMARY KEY,
    user_id BIGINT NOT NULL,
    shipping_address_id BIGINT NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_carts_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_carts_users
        FOREIGN KEY (user_id) REFERENCES dbo.users(user_id),
    CONSTRAINT FK_carts_shipping_address
        FOREIGN KEY (shipping_address_id) REFERENCES dbo.addresses(address_id)
		ON DELETE SET NULL,
    CONSTRAINT UQ_carts_user UNIQUE (user_id)
);


CREATE TABLE dbo.cart_items (
    cart_item_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_cart_items PRIMARY KEY,
    cart_id BIGINT NOT NULL,
    listing_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    save_later BIT NOT NULL
        CONSTRAINT DF_cart_items_save_later DEFAULT (0),
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_cart_items_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
   CONSTRAINT FK_cart_items_carts
    FOREIGN KEY (cart_id)
    REFERENCES dbo.carts(cart_id)
    ON DELETE CASCADE,
    CONSTRAINT FK_cart_items_listings
        FOREIGN KEY (listing_id) REFERENCES dbo.listings(listing_id),
    CONSTRAINT CK_cart_items_quantity CHECK (quantity > 0),
    CONSTRAINT UQ_cart_items_listing UNIQUE (cart_id, listing_id)
);


-- =========================================================
-- 5. ORDERS AND ORDER ITEMS
-- =========================================================

CREATE TABLE dbo.orders (
    order_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_orders PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    order_status VARCHAR(40) NOT NULL,
    payment_method VARCHAR(30) NOT NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_orders_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_orders_customers
        FOREIGN KEY (customer_id) REFERENCES dbo.users(user_id),
    CONSTRAINT CK_orders_payment_method
        CHECK (payment_method IN ('COD', 'PREPAID'))
);


-- Snapshot addresses used by the order at checkout.
-- Keep these even if the user's address book changes later.
CREATE TABLE dbo.order_addresses (
    order_address_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_order_addresses PRIMARY KEY,
    order_id BIGINT NOT NULL,
    address_type VARCHAR(10) NOT NULL,
    source_address_id BIGINT NULL,
    full_name VARCHAR(150) NOT NULL,
    phone_number VARCHAR(30) NOT NULL,
    address_line_1 VARCHAR(255) NOT NULL,
    address_line_2 VARCHAR(255) NULL,
    city VARCHAR(100) NOT NULL,
    state_province_region VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    country_code CHAR(2) NOT NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_order_addresses_created_at DEFAULT (SYSUTCDATETIME()),
    CONSTRAINT FK_order_addresses_orders
        FOREIGN KEY (order_id) REFERENCES dbo.orders(order_id),
    CONSTRAINT FK_order_addresses_source
        FOREIGN KEY (source_address_id) REFERENCES dbo.addresses(address_id)
        ON DELETE SET NULL,
    CONSTRAINT CK_order_addresses_type
        CHECK (address_type IN ('SHIPPING', 'BILLING')),
    CONSTRAINT UQ_order_addresses_type UNIQUE (order_id, address_type)
);


CREATE TABLE dbo.order_items (
    order_item_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_order_items PRIMARY KEY,
    order_id BIGINT NOT NULL,
    listing_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(19,4) NOT NULL,
    item_status VARCHAR(40) NOT NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_order_items_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_order_items_orders
        FOREIGN KEY (order_id) REFERENCES dbo.orders(order_id),
    CONSTRAINT FK_order_items_listings
        FOREIGN KEY (listing_id) REFERENCES dbo.listings(listing_id),
    CONSTRAINT CK_order_items_quantity CHECK (quantity > 0),
    CONSTRAINT CK_order_items_unit_price CHECK (unit_price > 0)
);


-- =========================================================
-- 6. SHIPMENTS
-- =========================================================

CREATE TABLE dbo.shipments (
    shipment_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_shipments PRIMARY KEY,
    order_id BIGINT NOT NULL,
    shipment_type VARCHAR(10) NOT NULL,
    shipment_status VARCHAR(40) NOT NULL,
    shipping_provider VARCHAR(50) NULL,
    provider_order_id VARCHAR(100) NULL,
    provider_shipment_id VARCHAR(100) NULL,
    awb_code VARCHAR(100) NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_shipments_created_at DEFAULT (SYSUTCDATETIME()),
    updated_at DATETIME2(3) NULL,
    CONSTRAINT FK_shipments_orders
        FOREIGN KEY (order_id) REFERENCES dbo.orders(order_id),
    CONSTRAINT CK_shipments_type
        CHECK (shipment_type IN ('OUTBOUND', 'RETURN'))
);


-- Maps each shipment to the order items physically included in it.
-- The same order item can be present in multiple shipments.
CREATE TABLE dbo.shipment_items (
    shipment_item_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_shipment_items PRIMARY KEY,
    shipment_id BIGINT NOT NULL,
    order_item_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    created_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_shipment_items_created_at DEFAULT (SYSUTCDATETIME()),
    CONSTRAINT FK_shipment_items_shipments
        FOREIGN KEY (shipment_id) REFERENCES dbo.shipments(shipment_id),
    CONSTRAINT FK_shipment_items_order_items
        FOREIGN KEY (order_item_id) REFERENCES dbo.order_items(order_item_id),
    CONSTRAINT CK_shipment_items_quantity CHECK (quantity > 0),
    CONSTRAINT UQ_shipment_items UNIQUE (shipment_id, order_item_id)
);


-- Append-only history of tracking events / webhook payloads.
CREATE TABLE dbo.shipment_tracking (
    shipment_tracking_id BIGINT IDENTITY(1,1) NOT NULL
        CONSTRAINT PK_shipment_tracking PRIMARY KEY,
    shipment_id BIGINT NOT NULL,
    provider_event_id VARCHAR(150) NULL,
    shipment_status VARCHAR(40) NOT NULL,
    provider_event_at DATETIME2(3) NULL,
    received_at DATETIME2(3) NOT NULL
        CONSTRAINT DF_shipment_tracking_received_at DEFAULT (SYSUTCDATETIME()),
    shipment_incoming_json NVARCHAR(MAX) NOT NULL,
    CONSTRAINT FK_shipment_tracking_shipments
        FOREIGN KEY (shipment_id) REFERENCES dbo.shipments(shipment_id),
    CONSTRAINT CK_shipment_tracking_json
        CHECK (ISJSON(shipment_incoming_json) = 1)
);

-- Deduplicate provider events when the provider supplies an event ID.
CREATE UNIQUE INDEX UX_shipment_tracking_provider_event
ON dbo.shipment_tracking(provider_event_id)
WHERE provider_event_id IS NOT NULL;


-- =========================================================
-- 7. USEFUL INDEXES FOR COMMON LOOKUPS
-- =========================================================

CREATE INDEX IX_addresses_user_id
ON dbo.addresses(user_id);

CREATE INDEX IX_products_category_id
ON dbo.products(category_id);

CREATE INDEX IX_product_variants_product_id
ON dbo.product_variants(product_id);

CREATE INDEX IX_listings_variant_id
ON dbo.listings(product_variant_id);

CREATE INDEX IX_listings_seller_id
ON dbo.listings(seller_id);

CREATE INDEX IX_cart_items_listing_id
ON dbo.cart_items(listing_id);

CREATE INDEX IX_orders_customer_created
ON dbo.orders(customer_id, created_at DESC);

CREATE INDEX IX_order_items_order_id
ON dbo.order_items(order_id);

CREATE INDEX IX_order_items_listing_id
ON dbo.order_items(listing_id);

CREATE INDEX IX_shipments_order_id
ON dbo.shipments(order_id);

CREATE INDEX IX_shipments_awb
ON dbo.shipments(awb_code)
WHERE awb_code IS NOT NULL;

CREATE INDEX IX_shipment_items_order_item_id
ON dbo.shipment_items(order_item_id);

CREATE INDEX IX_shipment_tracking_shipment_received
ON dbo.shipment_tracking(shipment_id, received_at DESC);
