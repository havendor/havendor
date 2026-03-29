# 📦 Havendor — Using `havendor-types` & `havendor-ui`

A practical usage guide for consuming the shared type definitions and UI component library across all Havendor services.

---

## 1. `havendor-types`

### What's Inside

| Export | Description |
|---|---|
| `TenantConfig` | Full tenant record including theme, slug, domain |
| `TenantTheme` | Colors, font, border radius, logo |
| `User` | Platform and tenant user shape |
| `UserRole` | `platform_admin` \| `tenant_owner` \| `tenant_staff` \| `customer` |
| `Product` | Tenant product catalog item |
| `Order` | Order record |
| `OrderStatus` | `pending` \| `confirmed` \| `shipped` \| `delivered` \| `cancelled` |

---

### Install

```bash
npm install @havendor/types
```

> Until published to npm, install directly from GitHub:
> ```bash
> npm install github:havendor/havendor-types
> ```

---

### Usage by Repo

#### `havendor-admin-api` — Typing API responses

```ts
import { TenantConfig, User } from '@havendor/types'

// Typed route handler response
async function getTenant(id: string): Promise<TenantConfig> {
  const tenant = await db.tenant.findUnique({ where: { id } })
  return tenant as TenantConfig
}
```

#### `havendor-tenant-api` — Typing orders and products

```ts
import { Order, OrderStatus, Product } from '@havendor/types'

async function updateOrderStatus(
  orderId: string,
  status: OrderStatus
): Promise<Order> {
  return await db.order.update({
    where: { id: orderId },
    data: { status }
  })
}
```

#### `havendor-admin-dashboard` — Typing API response in frontend

```ts
import { TenantConfig } from '@havendor/types'

// In a Next.js server component
async function getTenants(): Promise<TenantConfig[]> {
  const res = await fetch('/api/tenants')
  return res.json()
}
```

#### `havendor-tenant-storefront` — Typing tenant config from host resolution

```ts
import { TenantConfig } from '@havendor/types'

// app/layout.tsx
async function getTenantByHost(host: string): Promise<TenantConfig> {
  const res = await fetch(`${process.env.TENANT_API_URL}/resolve`, {
    headers: { host }
  })
  return res.json()
}
```

---

### Adding New Types

1. Branch from `develop` in `havendor-types`

```bash
git checkout develop && git pull
git checkout -b feature/add-invoice-type
```

2. Add your type file

```ts
// src/invoice.ts
export type InvoiceStatus = 'draft' | 'sent' | 'paid' | 'overdue'

export interface Invoice {
  id: string
  tenantId: string
  orderId: string
  status: InvoiceStatus
  amount: number
  dueDate: Date
}
```

3. Export from `src/index.ts`

```ts
export * from './invoice'
```

4. Commit and push

```bash
git commit -m "feat: add Invoice and InvoiceStatus types"
git push origin feature/add-invoice-type
# Open PR → develop
```

5. Bump the version in `package.json` before merging to `main`

```bash
# patch = bug fix, minor = new type added, major = breaking change
npm version minor
git commit -m "chore: bump version to v1.1.0"
```

---

---

## 2. `havendor-ui`

### What's Inside

| Export | Description |
|---|---|
| `TenantThemeProvider` | Wraps your app — injects tenant CSS variables at runtime |
| `applyTheme(theme)` | Utility function to manually apply a `TenantTheme` |
| `defaultTheme` | Fallback theme when no tenant config is available |
| shadcn components | `Button`, `Card`, `Input`, `Badge`, `Table`, `Dialog`, etc. |

---

### Install

```bash
npm install @havendor/ui
```

> Until published to npm, install directly from GitHub:
> ```bash
> npm install github:havendor/havendor-ui
> ```

---

### Setup: Add to `tailwind.config.ts`

Each consuming app must include `havendor-ui` in its Tailwind content path so classes aren't purged:

```ts
// tailwind.config.ts
export default {
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './node_modules/@havendor/ui/src/**/*.{ts,tsx}', // ← add this
  ],
  // ...
}
```

---

### Usage by Repo

#### `havendor-tenant-storefront` — Apply tenant theme at layout level

```tsx
// app/layout.tsx
import { TenantThemeProvider } from '@havendor/ui'
import { TenantConfig } from '@havendor/types'
import { headers } from 'next/headers'

async function getTenantByHost(host: string): Promise<TenantConfig> {
  const res = await fetch(`${process.env.TENANT_API_URL}/resolve`, {
    headers: { host },
    cache: 'no-store'
  })
  return res.json()
}

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const host = headers().get('host') ?? ''
  const tenant = await getTenantByHost(host)

  return (
    <html lang="en">
      <body>
        <TenantThemeProvider theme={tenant.theme}>
          {children}
        </TenantThemeProvider>
      </body>
    </html>
  )
}
```

#### `havendor-tenant-dashboard` — Same pattern, tenant manages their store

```tsx
// app/layout.tsx
import { TenantThemeProvider } from '@havendor/ui'

export default async function DashboardLayout({ children, tenant }) {
  return (
    <TenantThemeProvider theme={tenant.theme}>
      <div className="flex min-h-screen">
        <Sidebar />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </TenantThemeProvider>
  )
}
```

#### `havendor-admin-dashboard` — Use defaultTheme (platform UI, no tenant)

```tsx
import { TenantThemeProvider, defaultTheme } from '@havendor/ui'

export default function AdminLayout({ children }) {
  return (
    <TenantThemeProvider theme={defaultTheme}>
      {children}
    </TenantThemeProvider>
  )
}
```

#### Using shadcn components from `havendor-ui`

```tsx
import { Button, Card, CardContent, CardHeader, Badge } from '@havendor/ui'

export function ProductCard({ product }) {
  return (
    <Card>
      <CardHeader>{product.name}</CardHeader>
      <CardContent>
        <p className="text-primary font-semibold">${product.price}</p>
        <Badge variant={product.stock > 0 ? 'default' : 'destructive'}>
          {product.stock > 0 ? 'In Stock' : 'Out of Stock'}
        </Badge>
        <Button className="mt-4 w-full">Add to Cart</Button>
      </CardContent>
    </Card>
  )
}
```

---

### Adding a New Component to `havendor-ui`

1. Branch from `develop`

```bash
git checkout develop && git pull
git checkout -b feature/add-product-card
```

2. Add shadcn base if needed

```bash
npx shadcn@latest add card badge
```

3. Create your component

```tsx
// src/components/ProductCard/ProductCard.tsx
import { Card, CardContent, CardHeader } from '../ui/card'
import { Badge } from '../ui/badge'
import type { Product } from '@havendor/types'

interface ProductCardProps {
  product: Product
}

export function ProductCard({ product }: ProductCardProps) {
  return (
    <Card>
      <CardHeader className="font-semibold">{product.name}</CardHeader>
      <CardContent>
        <p className="text-primary">${product.price}</p>
        <Badge>{product.stock > 0 ? 'In Stock' : 'Out of Stock'}</Badge>
      </CardContent>
    </Card>
  )
}
```

4. Export from `src/index.ts`

```ts
export { ProductCard } from './components/ProductCard/ProductCard'
```

5. Commit and push

```bash
git add .
git commit -m "feat: add ProductCard component built on shadcn Card and Badge"
git push origin feature/add-product-card
# Open PR → develop
```

---

## 3. How Both Work Together

```
havendor-types
  └── defines TenantTheme, TenantConfig, Product, Order ...
        ↓ imported by
havendor-ui
  └── TenantThemeProvider accepts TenantTheme
  └── ProductCard accepts Product
        ↓ imported by
havendor-tenant-storefront
havendor-tenant-dashboard
havendor-admin-dashboard
```

**The dependency chain is one-way:**

```
havendor-types  ←  havendor-ui  ←  all apps
```

- `havendor-types` has **no dependencies** on other Havendor packages
- `havendor-ui` depends on `havendor-types` only
- All apps depend on both

> ⚠️ Never import from an app repo into `havendor-types` or `havendor-ui`. The dependency must always flow downward.

---

## 4. Version Bump Cheatsheet

```bash
npm version patch   # 1.0.0 → 1.0.1  (bug fix)
npm version minor   # 1.0.0 → 1.1.0  (new type or component added)
npm version major   # 1.0.0 → 2.0.0  (breaking change — renamed or removed export)
```

After bumping, update the version in consuming apps:

```bash
npm install @havendor/types@latest
npm install @havendor/ui@latest
```

---

*Havendor — Multi-Tenant SaaS Platform*
