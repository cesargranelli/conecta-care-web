# Inventário funcional de telas e rotas — web-conecta-care

> Wayfinder #3 (`cesargranelli/web-conecta-care#3`). Levantamento estático do código em
> `feature/refatoracao-estrutura-projeto` (branch `research/inventario-telas`). Sem testes automatizados no repo:
> este documento é a referência de paridade para a migração React.

## Resumo executivo

| Métrica | Valor |
|---|---|
| Componentes standalone | 135 |
| Arquivos de rotas | 10 (`app.routes.ts` + 9 feature; **2 são código morto**: `auth.routes.ts`, `registration.routes.ts`) |
| Telas mapeáveis (rotas-folha) | **86** |
| Redirects legados PT→EN | 11 (+ `''`→`login` e `**`→`login`) |
| Telas públicas (sem `authGuard`) | 11 |
| Componentes com reactive forms | **113** de 135 (84%) |
| sweetalert2 | 79 componentes |
| ngx-mask | 106 componentes + provider global em `app.config.ts` |
| jQuery / material-dashboard / dataTables | scripts globais carregados em TODAS as páginas (`angular.json`: jquery, bootstrap-material-design, moment, sweetalert2.js, jquery.dataTables, jquery.bootstrap-wizard, app.js) + CSS `material-dashboard.min.css`; **61 componentes** com `declare var jQuery`/dataTables |
| @fullcalendar/* | 1 componente (`homecares.component` — agenda) |
| ng-recaptcha-2 | cadeia morta (ver seção final) |
| PrimeNG | navbar, subnav, register.component (menubar/tooltip/button/card/inputtext/password/config/themes) |
| Serviços HTTP | 67 arquivos `.service.ts`; todas as chamadas apontam para `environment.apiConnecta` ou `environment.apiCep` (ViaCEP) |
| Services de evento/estado compartilhados | `SharedLoadingService` injetado em **91 componentes**; `SharedValidService` em **78**; `SharedTokenService` em login/confirmações; `TratamentoStorageService` (estado em memória do fluxo de tratamento) |

Bases de API (`src/environments`): `apiConnecta` = herokuapp (dev), elasticbeanstalk (staging), `api-connecta.connectacare.com.br` (prod), localhost:5000 (local). `apiCep` = `https://viacep.com.br`.

---

## Auth / Login (`features/auth`)

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/login` | `LoginComponent` | sim | jQuery/material-dashboard | SharedLoading, SharedToken, SharedValid | POST `/login` (auth.service); GET `/usuarios` (validação via usuario.service) |
| `/login/manutencao-senha` | `ManutencaoSenhaComponent` | sim | — | SharedLoading | POST `/login/validacao`, POST `/login/valid` (login.service) |
| `/login/esqueci-minha-senha` | `EsqueciMinhaSenhaComponent` | sim | — | SharedLoading | POST `/login/esqueci-minha-senha` |
| `/login/nova-senha/:id` | `NovaSenhaComponent` | sim | sweetalert2 | SharedLoading | PATCH `/login/nova-senha` |
| `/admin/login` | `LoginAdminComponent` | sim | — | SharedLoading | POST `/admin/login` (auth-admin.service) |
| `/dashboard` | `DashboardComponent` (layout/home/dashboard) | não | PrimeNG | SharedValid | — (hub pós-login; usa menu/subnav) |

Observações: `auth.routes.ts` (AUTH_ROUTES) duplica estas rotas mas **não é importado por ninguém**. Token JWT persistido em `localStorage['token']` (JSON-wrapped) via `StorageService`; `authGuard` só checa existência do token (sem expiração/perfil).

## Cadastro público e confirmação (`features/registration`, `pages/`)

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/register` | `RegisterComponent` | sim | PrimeNG, ngx-mask | — | POST `/documentos` (consulta CNPJ/CPF, registration.service) |
| `/confirm-registration/:token` | `ConfirmacaoCadastroComponent` (pages) | não | sweetalert2 | SharedToken, SharedValid | PUT `/usuarios/validacao` (cadastro.service) |
| `/confirm-password/:token` | `ConfirmacaoNovaSenhaComponent` (pages) | sim | sweetalert2 | SharedLoading, SharedToken | POST `/usuarios` (credenciais, registration.service) |
| `/waiting-email-confirmation` | `EsperaConfirmacaoEmailComponent` (pages) | não | — | — | — |
| `/terms-of-use` | `TermoUsoComponent` | não | — | — | — |
| `/privacy-policy` | `TermoPrivacidadeComponent` | não | — | — | — |
| `/email` (service isolada) | — | — | — | — | POST `/emails` (email.service, reenvio de confirmação) |

## Cadastro de profissional pós-convite (`features/registration/professional`)

Rota-mãe `/register/professionals/:id` (**com guard**, loadChildren) — filhos ainda em **PT-BR**:

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/informacoes-gerais` | `CadastroInformacoesGeraisComponent` | sim | sweetalert2, ngx-mask | SharedLoading, SharedValid | GET/PUT `/profissionais/{id}`; GET `/dominio`; ViaCEP |
| `/endereco` | `EnderecoComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | POST/PUT `/enderecos`; ViaCEP; GET `/dominio` |
| `/contato` | `ContatoComponent` | sim | ngx-mask | SharedLoading, SharedValid | POST/PUT `/contatos/telefones` (core ContatoService) |
| `/carreira` | `CarreiraComponent` | sim | sweetalert2, jQuery | SharedLoading, SharedValid | GET/POST/PUT `/carreiras`; GET `/dominio` |
| `/experiencia` | `ExperienciaComponent` | sim | jQuery | SharedLoading, SharedValid | GET/POST/PUT `/experiencias`; GET `/dominio` |
| `/escolaridade` | `EscolaridadeComponent` | sim | jQuery | SharedLoading, SharedValid | GET/POST/PUT `/escolaridade`; GET `/dominio` |
| `/complemento` | `CadastroComplementoComponent` | sim | sweetalert2 | SharedLoading, SharedValid | POST/PUT `/complementos/profissional` |
| `/conta` | `CadastroContaComponent` | sim | ngx-mask | SharedLoading | POST/PUT `/contas`; GET `/dominio` |

`registration.routes.ts` (CADASTRO_ROUTES) **não é referenciado** — rota morta.

## Pacientes (`features/patients`)

Mãe `/patients/:patient_id` (guard). Endpoints desta área usam o **novo padrão `/api/v1/*`** (mistura PT/EN nos paths):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/patients/register-dependent` | `CadastroDependenteCpfComponent` | sim | jQuery | SharedLoading | POST `/documentos`; POST `/api/v1/paciente` |
| `/patients/:id` | `PacientesComponent` (lista) | não | ngx-mask, jQuery | SharedLoading, SharedValid | GET `/api/v1/patient/{id}`; modal QRCode (`app-qrcode`) com protocolo de atendimento |
| `/patients/:id/register/general-info` | `CadastroInformacoesGeraisComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | GET `/api/v1/patient/{id}`; GET `/api/v1/estado-civil`, `/genero`, `/tipo-sanguineo`, `/estado` |
| `/patients/:id/register/address` | `CadastroEnderecoComponent` | sim | ngx-mask | SharedLoading, SharedValid | POST `/api/v1/endereco`; ViaCEP |
| `/patients/:id/register/contact` | `CadastroContatoComponent` | sim | ngx-mask | SharedLoading, SharedValid | POST `/api/v1/contato` |
| `/patients/:id/register/supplemental` | `CadastroComplementoComponent` | sim | ngx-mask | SharedLoading, SharedValid | POST/PUT `/complementos/paciente`; POST `/contatos/telefones` (core) |
| `/patients/:id/register/medical-history` | `CadastroHistoricoMedicoComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | POST `/api/v1/historico-medico` |
| `/patients/:id/details` (7 filhas: `` , `login`, `general-info`, `address`, `contact`, `supplemental`, `medical-history`) | `DadosComponent` + tabs de detalhe (formulários compartilhados em `shared/components/forms/*`) | sim (todas menos índice) | ngx-mask, sweetalert2, jQuery | SharedLoading, SharedValid | GET/PUT `/api/v1/paciente`, `/api/v1/contato`, `/api/v1/endereco`, `/api/v1/historico-medico`; GET `/atendimentos` (atendimento.service local) |

Formulários compartilhados da área: `form-informacoes-gerais`, `form-endereco`, `form-contato`, `form-complemento`, `form-historico-medico` + modal `qrcode`.

## Profissionais (`features/professionals`)

Mãe `/professionals/:id` (guard):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/professionals/:id` | `ProfissionaisComponent` (lista) | não | ngx-mask | — | GET `/profissionais` |
| `/professionals/:id/professional-data` (10 filhas: ``, `login`, `general-info`, `address`, `contact`, `career`, `experience`, `education`, `complement`, `account`) | `DadosProfissionaisComponent` + tabs (reutiliza padrão do cadastro público) | sim | ngx-mask, sweetalert2, jQuery | SharedLoading, SharedValid | `/profissionais/{id}`, `/carreiras`, `/experiencias`, `/escolaridade`, `/complementos/profissional`, `/contas`, `/contatos/telefones`, `/enderecos`, `/dominio`, ViaCEP |
| `/professionals/:id/events` | `EventosComponent` (lista) | não | ngx-mask, jQuery | SharedLoading, SharedValid | GET `/profissionais/{id}/eventos` (via profissional.service); GET `/eventos` |
| `/professionals/:id/events/:eventId` | `EventoDetalheComponent` | não | ngx-mask | SharedLoading | GET `/eventos/{id}` |

Área mais acoplada ao legado core (9 services distintos + DominioService transversal).

## Homecares / Agenda-Calendário / Tratamentos (`features/homecares`)

Mãe `/homecares/:homecare_id` (guard). Área maior e mais complexa (33 componentes com reactive forms):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/homecares/:id` | `HomeCaresComponent` — **agenda FullCalendar** | sim | **@fullcalendar (daygrid/timegrid/list/interaction, locale pt-br)**, ngx-mask, jQuery, modais `modal-criar-tratamento` e `modal-detalhe-atendimento`, `card-atendimentos` | SharedLoading, SharedValid | GET `/homecares/{id}/atendimentos` (atendimento.service → `/atendimentos`); POST/PUT/DELETE `/tratamentos` |
| `/homecares/:id/register/homecare` | `CadastroHomeCareComponent` | sim | sweetalert2, ngx-mask, jQuery | SharedLoading, SharedValid | POST `/homecares`; POST `/documentos` (CNPJ) |
| `/homecares/:id/register/address` · `/contact` | `CadastroEnderecoComponent`, `CadastroContatoComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | POST/PUT `/homecares/{id}/enderecos`, `/homecares/{id}/contatos`; ViaCEP |
| `/homecares/:id/details` (5 filhas) | `DadosHomecaresComponent` + tabs (`login`, `homecare`, `address`, `contact`) | sim | ngx-mask, sweetalert2 | SharedLoading, SharedValid | GET/PUT `/homecares/{id}` e sub-recursos; ViaCEP |
| `/homecares/:id/medical-record/:record_id` | `ProntuarioComponent` (prontuário eletrônrico) | não | ngx-mask | SharedLoading, SharedValid | GET/POST `/atendimentos/{id}` (dados clínicos) |
| `/homecares/:id/treatment/request` | `SolicitacaoTratamentoComponent` + subcomponentes `tratamento-solicitacao-{paciente,endereco,acompanhante,profissional}` | sim | sweetalert2, ngx-mask, jQuery | SharedLoading, SharedValid | POST `/tratamentos`; GET `/profissionais`; GET `/api/v1/paciente` (busca de paciente!) |
| `/homecares/:id/treatment/preview` | `TratamentoPreviewComponent` | sim | ngx-mask | SharedLoading, SharedValid | POST `/tratamentos` (confirmação) |
| `/homecares/:id/treatment/in-progress` | `TratamentoListaEmAbertoComponent` | não | ngx-mask, jQuery | SharedLoading, SharedValid | GET `/tratamentos?status=aberto` |
| `/homecares/:id/treatment/in-progress/:tid` | `TratamentoComponent` + subcomponentes `{paciente,endereco,profissional,acompanhante}` | sim | sweetalert2, ngx-mask, jQuery | SharedLoading, SharedValid, **TratamentoStorageService** (estado em memória entre telas) | PUT `/tratamentos/{id}` (encerrar etc.) |
| `/homecares/:id/treatment/in-progress/:tid/new-attendance` | `NovoAtendimentoComponent` (+ `tratamento-lista-atendimentos`) | sim | ngx-mask, jQuery | SharedLoading, SharedValid, TratamentoStorageService | POST `/atendimentos`; GET `/dominio`; ViaCEP |
| `/homecares/:id/professional` | `HomecareProfissionalComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | GET `/profissionais/{cpf}` (vínculo profissional↔homecare) |
| `/homecares/:id/patient` | `HomecarePacienteComponent` | sim | sweetalert2, ngx-mask, jQuery | SharedLoading, SharedValid | GET `/api/v1/paciente` (busca por CPF/nome) |

## Planos de saúde (`features/health-plans`)

Mãe `/health-plans/:id` (guard):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/health-plans/:id` | `PlanosSaudeComponent` (lista) | não | ngx-mask | — | GET `/planos-saude` |
| `/health-plans/:id/register/health-plan` | `CadastroPlanoSaudeComponent` | sim | sweetalert2, ngx-mask | SharedLoading, SharedValid | POST `/planos-saude`; POST `/documentos` (CNPJ) |
| `/health-plans/:id/register/address` · `/contact` | `CadastroEnderecoComponent`, `CadastroContatoComponent` (+ forms compartilhados `shared/components/forms/*`) | sim | ngx-mask | SharedLoading, SharedValid | POST/PUT `/planos-saude/{id}/enderecos`, `/planos-saude/{id}/contatos`; ViaCEP; GET `/dominio` |
| `/health-plans/:id/details` (5 filhas) | `DadosPlanosSaudeComponent` + tabs | sim | ngx-mask, sweetalert2 | SharedLoading, SharedValid | GET/PUT `/planos-saude/{id}` e sub-recursos |

## Filiais de plano de saúde (`features/health-plan-branches`)

Mãe `/health-plan-branches/:id` (guard). Espelho quase idêntico da área anterior (código duplicado):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/health-plan-branches/:id` | `PlanosSaudeFilialComponent` (lista) | não | ngx-mask | — | GET `/planos-saude` (mesmo endpoint de planos!) |
| `/health-plan-branches/:id/register/branch` | `CadastroPlanoSaudeFilialComponent` | sim | sweetalert2, ngx-mask | SharedLoading, SharedValid | POST `/planos-saude` (filial = recurso do mesmo agregado); POST `/documentos` |
| `/health-plan-branches/:id/register/address` · `/contact` · `/login` | `CadastroEnderecoComponent`, `CadastroContatoComponent`, `CadastroLoginComponent` | sim | ngx-mask, jQuery | SharedLoading, SharedValid | POST/PUT `/planos-saude/{id}/enderecos`, `/planos-saude/{id}/contatos`; POST `/usuarios` (criação de credenciais); ViaCEP |
| `/health-plan-branches/:id/details` (5 filhas) | `DadosPlanosSaudeFilialComponent` + tabs | sim | ngx-mask, sweetalert2 | SharedLoading, SharedValid | GET/PUT `/planos-saude/{id}` e sub-recursos |

## Admin / Eventos (`features/admin`)

Mãe `/admin` (guard, redirect → `events`). Única área 100% lazy (`loadComponent`):

| Rota | Tela(s)/Componente(s) | Formulários | Libs especiais | Services de evento injetados | Endpoints consumidos |
|---|---|---|---|---|---|
| `/admin/events` | `EventosComponent` (lista admin) | não | sweetalert2, ngx-mask, jQuery/dataTables | SharedLoading | GET/DELETE `/eventos` |
| `/admin/events/new` | `EventoCadastroComponent` | sim | sweetalert2, ngx-mask | SharedLoading | POST `/eventos`; GET `/dominio` (tipos) |
| `/admin/events/:id` | `EventoDetalheComponent` | sim (edição) | sweetalert2, ngx-mask | SharedLoading | GET/PUT `/eventos/{id}` |

---

## Apêndices

### 1. Redirects legados PT→EN (`app.routes.ts`)

| Legado (PT-BR) | Novo (EN) |
|---|---|
| `/cadastro` | `/register` |
| `/cadastro/profissionais/:id` | `/register/professionals/:id` |
| `/confirmacao-cadastro/:token` | `/confirm-registration/:token` |
| `/confirmacao-nova-senha/:token` | `/confirm-password/:token` |
| `/espera-confirmacao-email` | `/waiting-email-confirmation` |
| `/termo-e-condicoes-de-uso` | `/terms-of-use` |
| `/politica-de-privacidade` | `/privacy-policy` |
| `/pacientes` | `/patients` |
| `/profissionais` | `/professionals` |
| `/planos-saude` | `/health-plans` |
| `/planos-saude-filial` | `/health-plan-branches` |

Regra global: `''` → `/login`; `**` → `/login`. **Atenção**: os filhos de `/register/professionals/:id` continuam PT-BR (`informacoes-gerais`, `endereco`, `contato`, `carreira`, `experiencia`, `escolaridade`, `complemento`, `conta`).

### 2. Rotas sem guard (públicas)

`/login`, `/login/manutencao-senha`, `/login/esqueci-minha-senha`, `/login/nova-senha/:id`, **`/admin/login`**, `/register`, `/confirm-registration/:token`, `/confirm-password/:token`, `/waiting-email-confirmation`, `/terms-of-use`, `/privacy-policy`.

Riscos: `/admin/login` pública no nível raiz enquanto `/admin/*` exige apenas "logado" (sem checagem de role — o papel admin é decidido só pelo payload do token na tela). Guard não valida expiração.

### 3. Endpoints órfãos / serviços sem tela

| Service | Endpoint | Situação |
|---|---|---|
| `core/services/convenio.service.ts` | POST `/convenios/cnpj` | **Sem nenhum componente consumidor** — candidato a remoção ou era de outra tela |
| `shared/services/shared-status-page.service.ts` | — | **Sem uso em todo o app** |
| `shared-event-token.service` / `shared-event-valid.service` | — | Só usados indiretamente pelos wrappers `SharedTokenService`/`SharedValidService`; camada de indirecção sem assinante visível do evento |
| `core/services/storage.service.ts` | — | Usado apenas pelos wrappers de token/valid (localStorage chave `token`) |

Duplicação relevante: cada feature tem seu próprio `contato.service`/`endereco.service` com código quase idêntico (health-plans vs health-plan-branches vs homecares), todos batendo em `/planos-saude/{id}/...` ou `/homecares/{id}/...`. O `core/services/contato.service.ts` (`/contatos/telefones`) é usado por pacientes/complemento, profissionais e cadastro-profissional — três gerações de API coexistindo.

Gerações de API misturadas: pacientes/homecares usam `/api/v1/*` com paths **misturando PT e EN** (`/api/v1/paciente` × `/api/v1/patient/{id}`; `/api/v1/contact/{id}` × `/api/v1/contato`); o restante usa paths PT-BR sem versionamento.

### 4. Componentes sem rota (modais, formulários compartilhados, layout)

Pertencem às telas indicadas:

- **Modais**: `QrcodeComponent` (protocolo de atendimento — exibido em `pacientes.component.html`); `ModalCriarTratamentoComponent` e `ModalDetalheAtendimentoComponent` (agenda homecares); `CardAtendimentosComponent`.
- **Sub-componentes de fluxo**: `tratamento-solicitacao-{patient,address,companion,professional}` (dentro de Solicitacao), `tratamento-{patient,address,professional,companion}` (dentro de Tratamento), `tratamento-lista-atendimentos` (NovoAtendimento).
- **Forms compartilhados**: `features/*/shared/components/forms/*` (por feature) e `shared/components/forms/{endereco,form-contato,password-validation}` — embutidos nas telas de detalhe/cadastro.
- **Layout**: `NavbarComponent`, `FooterComponent` (condicionais em `app.component.html` via `isPublicPage()`), `SubnavComponent`, `DashboardComponent`.
- **CÓDIGO MORTO (sem rota, sem template consumer)**: `ConnectaComponent`; `MenuComponent`, `MenuLogadoComponent` e os 6 menus específicos (`menu-admin`, `menu-homecares`, `menu-pacientes`, `menu-planos-saude`, `menu-planos-saude-filial`, `menu-profissionais`) — a navegação real hoje é Navbar+Subnav (PrimeNG); `TratamentoAtendimentoComponent`; cadeia recaptcha morta: `BasicRecaptchaComponent` + `FormInformacoesLoginComponent` + `SelectPickerComponent` + `shared-component.module.ts`; arquivos de rotas `auth.routes.ts` e `registration.routes.ts`.
- **Dependências sem uso vivo**: `@angular/google-maps` (nenhum componente importa), `ng-recaptcha-2` e `siteKey` nos 4 environments (só a cadeia morta referencia).
