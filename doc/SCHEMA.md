# EXPAND — Database Schema (Supabase Postgres)

> Starting point, not final. Extend as phases progress, but keep this file updated as the source of truth — don't let it drift from the actual migrations.

```sql
-- USERS
create type user_role as enum ('customer', 'store_owner', 'sponsor');

create table users (
  id uuid primary key references auth.users(id),
  role user_role not null,
  full_name text not null,
  phone text,
  avatar_url text,
  created_at timestamptz default now()
);

-- STORES
create table stores (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid references users(id) not null,
  name text not null,
  description text,
  logo_url text,
  qris_image_url text,
  created_at timestamptz default now()
);

-- PRODUCTS
create table products (
  id uuid primary key default gen_random_uuid(),
  store_id uuid references stores(id) not null,
  name text not null,
  description text,
  price numeric not null,
  stock int not null default 0,
  category text,
  images text[],
  created_at timestamptz default now()
);

-- CARTS / CART ITEMS
create table cart_items (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) not null,
  product_id uuid references products(id) not null,
  quantity int not null default 1,
  created_at timestamptz default now()
);

-- ORDERS
create type order_status as enum (
  'menunggu_pembayaran',
  'menunggu_verifikasi',
  'diproses',
  'dikirim',
  'selesai',
  'dibatalkan'
);

create table orders (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references users(id) not null,
  store_id uuid references stores(id) not null,
  status order_status not null default 'menunggu_pembayaran',
  total_amount numeric not null,
  shipping_address jsonb not null,
  shipping_method text,
  payment_proof_note text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table order_items (
  id uuid primary key default gen_random_uuid(),
  order_id uuid references orders(id) not null,
  product_id uuid references products(id) not null,
  quantity int not null,
  price_at_purchase numeric not null
);

-- TOURNAMENTS
create table tournaments (
  id uuid primary key default gen_random_uuid(),
  sponsor_id uuid references users(id) not null,
  title text not null,
  description text,
  banner_url text,
  location text,
  maps_url text,
  registration_fee numeric default 0,
  quota int,
  schedule_start timestamptz,
  schedule_end timestamptz,
  rules text,
  prizes jsonb,
  gallery text[],
  created_at timestamptz default now()
);

create table tournament_registrations (
  id uuid primary key default gen_random_uuid(),
  tournament_id uuid references tournaments(id) not null,
  customer_id uuid references users(id) not null,
  qr_code text not null,
  checked_in boolean default false,
  certificate_url text,
  created_at timestamptz default now()
);

-- REVIEWS
create table reviews (
  id uuid primary key default gen_random_uuid(),
  product_id uuid references products(id) not null,
  customer_id uuid references users(id) not null,
  rating int check (rating between 1 and 5),
  comment text,
  created_at timestamptz default now()
);

-- WISHLIST
create table wishlist_items (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) not null,
  product_id uuid references products(id) not null,
  created_at timestamptz default now()
);
```

## RLS Notes
- `users`: a user can read/update only their own row.
- `products`/`stores`: public read; write restricted to owning `store_owner`.
- `orders`/`order_items`: readable by the owning `customer_id` and the owning store's `owner_id`; writable by the same, with status transitions restricted by role (customer can't set `selesai`, store owner can't set `dibatalkan` after shipped, etc. — refine during Phase 4/5).
- `tournament_registrations`: readable by owning customer and the tournament's sponsor.
