# VendorHub — Vendor Product Catalog & Purchase Order Portal

A retail buying portal where vendors upload new item photos/details, Purchase Managers approve and select products, assign quantities and destination stores, and create **one PO per vendor**. The Purchase Manager can download a real `.xlsx` workbook directly from the Purchase Orders register. **The application does not send email.**

## Included

- `web/` — responsive React/Vite app with Vendor and Purchase Manager views. Without Supabase credentials it runs in demo preview mode; demo changes are local to the browser.
- `sql/001_schema.sql` — Supabase/Postgres tables, access policies, private item-photo storage, and an atomic PO creation function.
- `TEST_PLAN.md` — checks for vendor access, PO creation and Excel downloads.

## How the workflow works

1. Vendor signs in and uploads item photo, SKU, item name, rate, fabric, colour, size and other details.
2. Purchase Manager reviews and approves submitted products.
3. Manager selects approved products, enters required quantities, chooses store names and creates purchase orders.
4. The application groups selected lines by vendor and creates one PO per vendor in the database.
5. From **Purchase Orders**, the manager clicks the download icon for each PO. The browser creates and downloads a true Excel `.xlsx` workbook with PO/vendor details, store-wise item lines, quantities, rates, line totals and grand total.

No SMTP account, email credential, vendor email delivery or n8n workflow is required for PO Excel generation. The workbook is generated on demand when the user clicks Download Excel.

## Database setup

1. Create a Supabase project for the company.
2. Run `sql/001_schema.sql` in Supabase SQL Editor. Review the access policies with your DBA/security owner before production use.
3. Add vendor master data, for example:

```sql
insert into public.vendors (vendor_code, name, email, phone, address, gstin)
values ('V-001', 'Example Textiles', 'vendor@example.com', '9000000000', 'Pune, Maharashtra', null);
```

The vendor email field is retained as required vendor master/contact data only; this application does not send mail. Add store masters, for example:

```sql
insert into public.stores (store_code, store_name, address)
values ('PUN-01', 'Pune - Main Store', 'Pune');
```

4. In Supabase Authentication → Users, create accounts for vendor users and internal users. Public signup is intentionally not included.
5. Add a `profiles` row after creating the matching vendor/user. For a Purchase Manager:

```sql
insert into public.profiles (id, full_name, role, vendor_id)
values ('AUTH-USER-UUID', 'Purchase Manager', 'purchase_manager', null);
```

For a vendor, set `role='vendor'` and `vendor_id` to that vendor's UUID. Never allow end users to choose or change their own role.

## Web app setup

1. Install Node.js 20+.
2. Open a terminal in `web/` and run `npm install`.
3. Copy `.env.example` to `.env`; set `VITE_SUPABASE_URL` and the Supabase publishable/anon key.
4. Run `npm run dev` and open the Vite URL shown in the terminal.
5. Deploy the built Vite app to a company-approved private/internal host and configure those same two public environment values.

Only the publishable/anon key belongs in the browser. Never put the Supabase service-role key in the web app or any `VITE_` variable. Database RLS protects vendor-specific data and purchase-order access.

## Excel purchase order format

Each `.xlsx` file contains a `Purchase Order` worksheet with PO Number, PO Date, Vendor Name/Code/Phone/GSTIN/Address, notes and a table of Sr. No., Store Code, Store Name, Item SKU, Item Name, Quantity, Rate (INR), and Line Total (INR), plus GRAND TOTAL. A separate file is downloaded for each vendor PO. Workbook generation uses the SheetJS browser API; see the [official write-files documentation](https://docs.sheetjs.com/docs/api/write-options/).

## Business rules and limits

- Item details include SKU, name, category, vendor rate, fabric, colour, size range, extra details and product photo.
- Vendor accounts can access their own submissions; Purchase Managers/admins can review the buying catalogue.
- Only approved items can be ordered.
- Each selected item line has a required quantity and destination store.
- A multi-vendor selection creates one purchase order per vendor. Each vendor PO can include multiple destination stores.
- PO line name, SKU and rate are stored as snapshots so later edits to the catalogue do not rewrite historical order values.
- Photo storage is private and protected by separate storage policies.
- The initial PO value is quantity × vendor rate. GST, discounts, freight, MOQ, payment terms, delivery dates, multi-level approval, ERP sync, and audit reporting need to be defined with Finance/Purchase before production use.
- The starter project has not been deployed, connected to your company's Supabase project, or security-tested in production.
