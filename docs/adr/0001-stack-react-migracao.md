# Stack React da migração

## Status

accepted — 2026-08-25 (decisão HITL do dono, registrada no [ticket #5](https://github.com/cesargranelli/conecta-care-web/issues/5) do mapa de modernização)

## Decisão

O app React que substituirá o legado Angular usa **Vite + TypeScript strict + React 19 como SPA pura**, com **React Router v7** em modo biblioteca, **TanStack Query** para estado servidor, **Zustand** para estado cliente transversal, **React Hook Form + zod + Imask** para formulários/máscaras e **PrimeReact** como design system (DataTable, Toast, ConfirmDialog nativos), com tema ajustado à paridade visual com o Material Dashboard atual. Calendário via `@fullcalendar/react`.

Motivações centrais: o time conhece React melhor que Angular (motivo da migração); agentes executam o código, então escolhas mainstream e bem documentadas reduzem atrito; TanStack Query absorve mudanças de contrato ainda por vir (backend em redesenho); PrimeReact minimiza a distância cultural do PrimeNG já presente no legado.

## Considered Options

- **Meta-framework (Next.js etc.)**: rejeitado — app admin atrás de autenticação servido em Apache; SSR/RSC não trazem valor aqui.
- **MUI**: rejeitado apesar da estética Material herdar do dashboard atual — suíte admin menos completa que a do PrimeReact para tabelas/diálogos.
- **Manter sweetalert2** (79 usos no legado): rejeitado em favor de consolidar feedback no Toast/ConfirmDialog do próprio design system — um kit a menos para manter.
- **Redirects legados PT→EN**: rejeitados — sem produção nem usuários, os paths nascem normalizados em inglês.
- **Portar recaptcha/google-maps/crypto-js**: rejeitados — cadeias mortas confirmadas pelo inventário do legado.

## Consequences

- Fase 0 do scaffold deve incluir tema PrimeReact calibrado contra o Material Dashboard (paridade é requisito de aceite).
- Máscaras e validação compartilham tipos com os contratos REST pós-backend (ticket #15 do mapa).
- Testes não unitarizam: qualidade via e2e no repo `conecta-care-e2e`.
