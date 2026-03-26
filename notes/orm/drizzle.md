# Drizzle ORM

## Important commands
> Install: npm install drizzle-orm
> Dev dep: npm install -D drizzle-kit
> Generate migration: npx drizzle-kit generate
> Run migration: npx drizzle-kit migrate
> Push (no migration file): npx drizzle-kit push
> Visual UI: npx drizzle-kit studio

## Data Types
### Numbers
- smallint       : whole number, small range (-32k to 32k)
- integer        : whole number, normal range (±2.1B) ← use 90% of time
- bigint         : whole number, huge range, needs mode:'number'|'bigint'
- real           : approximate decimal, 4 bytes, 6 digit precision (rarely use)
- doublePrecision: approximate decimal, 8 bytes, 15 digit precision ← use for lat/lng
- decimal/numeric: EXACT decimal, use for money/balance ← returns string in TS!
                   decimal("col", { precision: 10, scale: 2 })
                   precision = total digits, scale = digits after decimal

### Rules
- money/balance  → decimal
- coordinates    → doublePrecision  
- counts/ratings → integer
- forget real    → use doublePrecision instead


### Strings
- text      : unlimited length, no size concern ← use 90% of time
              eg: comment, description, bio
- varchar   : variable length with max limit, uses only space needed
              varchar("col", { length: 255 })
              eg: email, fullName, shopName, phoneNumber
- char      : fixed exact length, always uses full defined space (pads with spaces)
              char("col", { length: 6 })
              eg: OTP codes, country codes ('NP', 'US')
- citext    : case insensitive text, needs Postgres extension
              rarely used — just lowercase in app layer instead

### Rules
- no length concern     → text
- max length matters    → varchar
- always exact length   → char (OTP, country codes)
- forget citext         → handle case in app layer (.toLowerCase())

### Key difference — char vs varchar
- char(6)     → FIXED   → 'AB' stored as 'AB    ' (always 6 bytes)
- varchar(255)→ VARIABLE → 'AB' stored as 'AB'    (only 2 bytes used)
- text        → same as varchar but no max limit

### Min/Max on numbers
- no .min() .max() in drizzle — use check constraint
  check('rating_range', sql`${table.rating} >= 1 AND ${table.rating} <= 5`)
- for user-facing validation use Zod .min() .max() instead
