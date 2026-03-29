# 🌿 Havendor — Git Commit & Branch Instructions

Step-by-step git workflow for all Havendor repositories. Follow this every time you start work.

---

## 📦 Applies To

| Repo | Notes |
|---|---|
| `havendor-types` | Type definitions |
| `havendor-ui` | Shared UI components |
| `havendor-web` | Public landing site |
| `havendor-admin-dashboard` | Platform owner UI |
| `havendor-admin-api` | Control plane API |
| `havendor-tenant-dashboard` | Tenant management UI |
| `havendor-tenant-storefront` | Public tenant storefront |
| `havendor-tenant-api` | Data plane API |

---

## 🔁 Step-by-Step: Starting New Work

### 1. Always start from an up-to-date `develop`

```bash
git checkout develop
git pull origin develop
```

### 2. Create your branch using the correct prefix

```bash
# New feature
git checkout -b feature/tenant-theme-provider

# Bug fix
git checkout -b fix/apply-theme-on-mount

# Maintenance / config
git checkout -b chore/update-shadcn-deps

# Docs update
git checkout -b docs/usage-guide
```

### 3. Make your changes, then commit

```bash
git add .
git commit -m "feat: add TenantThemeProvider component"
```

### 4. Push your branch to GitHub

```bash
git push origin feature/tenant-theme-provider
```

### 5. Open a Pull Request on GitHub

- Base branch: `develop`
- Compare branch: your feature branch
- Add a clear title and description

### 6. After PR is merged — clean up

```bash
git checkout develop
git pull origin develop
git branch -d feature/tenant-theme-provider
git push origin --delete feature/tenant-theme-provider
```

---

## ✍️ Commit Message Format

```
<type>: <short description in lowercase>
```

| Type | Use When |
|---|---|
| `feat` | Adding a new feature or component |
| `fix` | Fixing a bug |
| `chore` | Deps, config, tooling, setup |
| `refactor` | Restructuring code without behavior change |
| `docs` | README, guides, comments |
| `style` | Formatting only, no logic change |
| `test` | Adding or updating tests |

---

## 📝 Real Commit Examples

### havendor-types

```bash
git commit -m "chore: initial project setup for havendor-types"
git commit -m "feat: add TenantConfig and TenantTheme interfaces"
git commit -m "feat: add User and UserRole types"
git commit -m "feat: add Product type definition"
git commit -m "feat: add Order and OrderStatus types"
git commit -m "chore: add barrel export in index.ts"
git commit -m "docs: add README with usage and type overview"
git commit -m "chore: add .gitignore for node_modules and dist"
git commit -m "fix: correct optional field on TenantConfig.customDomain"
git commit -m "refactor: split tenant types into separate files"
```

### havendor-ui

```bash
git commit -m "chore: initial project setup for havendor-ui"
git commit -m "chore: initialize shadcn/ui with CSS variables enabled"
git commit -m "chore: add button, card, input, badge, table via shadcn"
git commit -m "feat: add TenantThemeProvider for runtime CSS variable injection"
git commit -m "feat: add applyTheme utility to update :root CSS variables"
git commit -m "feat: add defaultTheme fallback for missing tenant config"
git commit -m "chore: add barrel export for havendor-ui public API"
git commit -m "docs: add README with setup, theming, and component usage"
git commit -m "fix: apply theme on theme prop change using useEffect deps"
git commit -m "feat: add ProductCard component built on shadcn Card"
```

---

## 🌲 Branch Flow Diagram

```
main
 └── develop
      ├── feature/tenant-theme-provider   ← open PR → develop
      ├── feature/add-product-card
      ├── fix/theme-not-applying
      └── chore/update-deps
```

```
# When ready to release
develop → release/v1.0.0 → main
                           └── tag: v1.0.0
```

---

## 🚑 Hotfix Flow (Production Bug)

```bash
# Branch from main — NOT develop
git checkout main
git pull origin main
git checkout -b hotfix/theme-crash-on-load

# Fix, commit, then merge to both main and develop
git checkout main && git merge hotfix/theme-crash-on-load
git tag -a v1.0.1 -m "Hotfix v1.0.1"
git push origin main --tags

git checkout develop && git merge hotfix/theme-crash-on-load
git push origin develop
```

---

## ✅ Golden Rules

1. **Never commit directly to `main`**
2. **Never commit directly to `develop`**
3. **Always pull `develop` before branching**
4. **One purpose per branch** — don't mix a feature and a bug fix
5. **Delete branches after merging**
6. **Keep commit messages lowercase and descriptive**
7. **Each commit should do one thing**

---

*Havendor — Multi-Tenant SaaS Platform*
