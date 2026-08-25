# conecta-care-web

Frontend web da plataforma **Conecta Care** (Home Care — atendimento de saúde domiciliar).

## Estrutura

| Caminho            | O que é                                                                 |
| ------------------ | ----------------------------------------------------------------------- |
| `legacy/angular/`  | App Angular 21 atual (v1.14.0), **congelado** — somente referência de paridade durante a migração. Não recebe features. |
| raiz               | App **React** que substituirá o legado (a nascer; ver mapa de migração). |

## Migração para React

A migração está sendo conduzida pelo wayfinder no issue
[Mapa: modernizar a plataforma Home Care](https://github.com/cesargranelli/conecta-care-web/issues/1),
com decisões registradas nos tickets filhos e a spec final prevista em
`docs/migration/react-migration-spec.md`.

Inventário funcional do legado: `legacy/angular/docs/migration/inventory.md`.

## Histórico

O código foi importado de [`web-conecta-care`](https://github.com/cesargranelli/web-conecta-care)
(arquivado/read-only após conferência); o histórico git completo permanece lá.
