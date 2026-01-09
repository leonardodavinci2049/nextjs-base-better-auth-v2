# Migração de Stripe para Assinatura Manual

## Alterações Realizadas

### Arquivos Criados

#### [plans.ts](file:///home/leo/projects/nextjs/nextjs-base-better-auth-v2/src/lib/auth/plans.ts)

Novo arquivo de configuração com:
- Interface `SubscriptionPlan` para tipagem de planos
- Constante `SUBSCRIPTION_STATUS` com status possíveis
- Array `SUBSCRIPTION_PLANS` com 3 planos (basic, pro, enterprise)
- Mapeamento `PLAN_TO_PRICE` para preços
- Funções utilitárias `getPlanByName()` e `isSubscriptionActive()`

---

### Arquivos Modificados

#### [subscriptions-tab.tsx](file:///home/leo/projects/nextjs/nextjs-base-better-auth-v2/src/app/organizations/_components/subscriptions-tab.tsx)

**Principais mudanças:**

| Antes (Stripe) | Depois (Manual) |
|----------------|-----------------|
| `import type { Subscription } from "@better-auth/stripe"` | `import type { Subscription } from "@/services/db/auth/dto/auth.dto"` |
| `authClient.subscription.list()` | `getMySubscription()` |
| `authClient.subscription.cancel()` | `updateMySubscription({ status: "cancelled" })` |
| `authClient.subscription.upgrade()` | `createSubscription()` / `updateMySubscription()` |
| Billing Portal | Removido (não aplicável) |

---

## Configuração de Planos

Os planos são configurados no arquivo `src/lib/auth/plans.ts`:

```typescript
export const SUBSCRIPTION_PLANS: SubscriptionPlan[] = [
  {
    name: "basic",
    displayName: "Basic",
    price: 9.99,
    currency: "BRL",
    limits: { projects: 3 },
    features: ["3 projetos", "Suporte por email", "Atualizações básicas"],
  },
  {
    name: "pro",
    displayName: "Pro",
    price: 29.99,
    currency: "BRL",
    limits: { projects: 10 },
    features: ["10 projetos", "Suporte prioritário", "API access"],
  },
  {
    name: "enterprise",
    displayName: "Enterprise",
    price: 99.99,
    currency: "BRL",
    limits: { projects: -1 }, // ilimitado
    features: ["Projetos ilimitados", "Suporte 24/7", "Customização"],
  },
];
```

## Estrutura da Tabela Subscription

A tabela `subscription` possui os seguintes campos:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | string | ID único da assinatura |
| `userId` | string | ID do usuário |
| `plan` | string | Nome do plano (basic, pro, enterprise) |
| `status` | string | Status (pending, active, cancelled, expired) |
| `approvedAt` | Date | Data de aprovação |
| `createdAt` | Date | Data de criação |

## Server Actions Disponíveis

O CRUD de assinaturas está em `src/server/subscription.ts`:

- `createSubscription({ plan, status })` - Cria nova assinatura
- `getMySubscription()` - Busca assinatura do usuário logado
- `updateMySubscription({ plan?, status? })` - Atualiza assinatura
- `deleteMySubscription()` - Remove assinatura
