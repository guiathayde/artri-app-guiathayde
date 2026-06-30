# Relatório de Análise e Testes — ArtriApp

---

## 📋 Features Mapeadas

| # | Feature | Tela(s) / Arquivo | Status do Teste |
|---|---------|-------------------|-----------------|
| 1 | Login | `login_page.dart` | ⚠️ Parcial — UI ok; login só funciona após corrigir API/cleartext; sem feedback de erro |
| 2 | Cadastro de usuário | `sign_up_page.dart` | ❌ Falhou — botão "ENVIAR" não faz nada; campo de e‑mail não conectado |
| 3 | Recuperação de senha | login + `send_password.dart` | ❌ Falhou — "ESQUECI MINHA SENHA" é botão morto |
| 4 | Diário — tela inicial | `user_diary_initial_selection.dart` | ✅ OK (renderização); card "Mito/Verdade" é aleatório a cada rebuild |
| 5 | Diário — Dor (com locais) | `user_level_selection_with_options.dart` | ⚠️ Parcial — UI ok, **"Salvar" não persiste nada** |
| 6 | Diário — Inchaço | `user_level_selection_with_options.dart` | ⚠️ Parcial — idem; "Salvar" não persiste |
| 7 | Diário — Fadiga | `user_level_selection.dart` | ⚠️ Parcial — "Salvar" não persiste |
| 8 | Diário — Sono | `user_level_selection.dart` | ⚠️ Parcial — "Salvar" não persiste |
| 9 | Medicamentos | `remedy_page.dart` | ⚠️ Parcial — lista é **mock**; checklist funciona (sessão); **adicionar não salva** |
| 10 | Exercícios — menu | `exercise_page.dart` | ✅ OK |
| 11 | Exercícios físicos (rotina + vídeo) | `physical_exercise/*` | ✅ OK — **Mãos e Pés** (todos os níveis) confirmados; dados reais do backend, vídeos reproduzidos **até o fim**, navegação FEITO/anterior/próximo, tela "Parabéns!" **(v2)** |
| 12 | Relaxamento (guiado / respiração) | `relaxation/*` | ⚠️ Parcial — **guiado**: player de áudio toca até o fim (OK); **respiração**: **exibe conteúdo errado** (exercícios de mão) |
| 13 | Informações sobre atividade física | `info_atividade_fisica.dart` | ✅ OK (não testado a fundo) |
| 14 | Informações — 9 páginas de conteúdo | `info/*` | ✅ OK (testado: Mitos e Verdades) |
| 15 | Evolução (gráfico) | `evolution_page.dart` | ❌ Falhou — **gráfico mostra dados fixos falsos**, não os do usuário |
| 16 | Configurações | `logged_settings_page.dart` | ❌ Falhou — Alterar Email/Senha levam a "Page Not Found"; "Permissões" é morto |
| 17 | Navegação (bottom nav + GoRouter) | `routes/*` | ✅ OK (bottom nav); ⚠️ rotas de settings não registradas |

**Resumo:** 17 grupos de features mapeados; **todos foram alcançados em runtime** após as correções de ambiente. As 4 telas do Diário, a Evolução, o Cadastro, a Recuperação de senha e as Configurações estão **não‑funcionais** no fluxo principal.

---

## 🐛 Bugs Identificados

### BUG-001 — App não consegue falar com o backend: tráfego HTTP cleartext bloqueado
- **Severidade:** 🔴 Crítico
- **Tela:** Todas (login, exercícios, etc.)
- **Passos para reproduzir:** Instalar o app como está e tentar logar em qualquer Android 9+ (targetSdk ≥ 28).
- **Esperado:** Requisições HTTP à API funcionam.
- **Atual:** O `AndroidManifest.xml` não declara `android:usesCleartextTraffic="true"` nem `networkSecurityConfig`. Como a API é `http://` (cleartext), o Android **bloqueia todas as requisições** (`Cleartext HTTP traffic not permitted`). A exceção é engolida pelo `try/catch` do login → falha silenciosa. Só foi possível logar após habilitar cleartext em build de debug.
- **Correção sugerida:** Migrar a API para HTTPS; ou (dev) adicionar `android:usesCleartextTraffic="true"` / um `network_security_config.xml` permitindo o host de desenvolvimento.
- **Evidência:** logcat (`Cleartext HTTP traffic ... not permitted`); login só passou após o ajuste (`13_after_login.png`).

### BUG-002 — Diário não salva nada ("Salvar" é no-op em 4 das 5 métricas)
- **Severidade:** 🔴 Crítico (é a função central do app)
- **Telas:** Dor, Inchaço (`user_level_selection_with_options.dart:116`), Fadiga, Sono (`user_level_selection.dart:76`)
- **Passos para reproduzir:** Diário → Dor → marcar "Mãos" → escolher nível 8 → **Salvar**.
- **Esperado:** Salvar o registro (local/backend) e dar feedback.
- **Atual:** No `ConfirmationButtons`, o ramo `confirmed` é `: null`. Nada é salvo, nenhuma chamada de API, nenhum feedback; a tela apenas permanece. O valor coletado é descartado.
- **Evidência:** `15_dor_maos_scale.png` (nível 8 selecionado) → `16_dor_after_salvar.png` (após "Salvar": tela inalterada).

### BUG-003 — Evolução exibe dados falsos (hardcoded), não os do usuário
- **Severidade:** 🔴 Crítico
- **Tela:** `evolution_page.dart:120-162`
- **Passos para reproduzir:** Abrir aba "Evolução".
- **Esperado:** Gráfico dos sintomas reais dos últimos 7 dias.
- **Atual:** As séries "Dor" e "Fadiga" são `FlSpot` **fixos no código** (`8,6,7,5,4,6,3` e `4,5,5,3,6,8,4`). O gráfico ignora qualquer dado do usuário (que aliás nem é salvo — BUG‑003). Os rótulos do eixo X (Seg–Dom) também são fixos. Sem estado vazio: com os dois chips desmarcados o gráfico fica **em branco, sem mensagem**.
- **Evidência:** `27_evolution.png` (curva = valores do código), `28_evolution_empty.png` (gráfico vazio).

### BUG-004 — Configurações → "Alterar Email"/"Alterar Senha" levam a "Page Not Found"
- **Severidade:** 🔴 Crítico
- **Telas:** `logged_settings_page.dart:25,31`, `routes/settings.routes.dart`
- **Passos para reproduzir:** Diário → engrenagem → "Alterar Email".
- **Esperado:** Abrir tela de alterar e-mail.
- **Atual:** `SettingsRoutes.getGoRoutes()` **nunca é registrado** na árvore de rotas, e as constantes são caminhos relativos (`'configuration/change-email'`) passados a `context.go`. Resultado: **`GoException: no routes for location: configuration/change-email`**. Mesma falha em "Alterar Senha".
- **Evidência:** `17_settings.png` → `18_alterar_email.png` (tela "Page Not Found").

### BUG-005 — Login não persiste entre execuções (precisa logar toda vez)
- **Severidade:** 🔴 Crítico
- **Arquivos:** `services/security_token_service.dart:9-11,29-36`, `views/app.dart:13`
- **Passos para reproduzir:** Logar → fechar o app → reabrir.
- **Esperado:** Continuar logado.
- **Atual:** `App` decide a rota inicial com `SecurityTokenService().userLoggedIn()`, mas `_initTokens()` é `async` disparado sem `await` no construtor (cache em memória vazio no momento da checagem). Além disso, `saveToken()` **nunca atualiza** `_accessToken`/`_refreshToken` em memória. Logo `userLoggedIn()` é sempre `false` na inicialização → o app **sempre** abre no login.
- **Evidência:** ao sair para a home do Android e reabrir, o app voltou ao login (`20_relogin.png`).

### BUG-006 — Cabeçalho de autenticação nunca é enviado (interceptor quebrado e não conectado)
- **Severidade:** 🔴 Crítico
- **Arquivos:** `utils/interceptors/auth_interceptor.dart:14-16`
- **Detalhe:** `interceptRequest` faz `request.headers['Authorization'] = 'Bearer $accessToken'`, mas `getToken(...)` retorna um `Future` **não‑aguardado** → o header vira literalmente `Bearer Instance of 'Future<String?>'`. Pior: nem o `AuthInterceptor` nem o `RefreshTokenPolicy` são instanciados/conectados a qualquer cliente HTTP (verificado por busca global). **Nenhuma requisição do app envia token.** Endpoints protegidos retornariam 401 (no teste, `/trainings` e `/exercises` são públicos, por isso funcionaram).
- **Correção:** tornar `interceptRequest` `async` e `await` o token; e de fato usar um `InterceptedClient` nos serviços.

### BUG-007 — Endpoints de submissão do diário não existem no backend + concatenação de URL quebrada
- **Severidade:** 🔴 Crítico
- **Arquivo:** `view_models/diary_view_model.dart:42-69`
- **Detalhe:** O app posta para `daily-sleep-report/`, `daily-fatigue-report/`, `daily-pain-report/`, `daily-swelling-report/`. O backend (`authentication/urls.py`) só expõe `daily-pain-reports/` (plural) e `training-reports/` — **não existem** endpoints de sono/fadiga/inchaço, e nem o de dor bate (singular × plural). Além disso a concatenação `'${apiUrl}$endpoint'` sem separador gera `.../apidaily-sleep-report/`. Os métodos de Fadiga/Dor/Inchaço sequer são chamados; só `enviarRelatorioSono` é usado. Resultado: submissão do diário **nunca** funcionaria, mesmo com BUG‑003 corrigido.

### BUG-008 — Botões mortos (sem ação)
- **Severidade:** 🟠 Alto
- **Ocorrências confirmadas:**
  - Login → "ESQUECI MINHA SENHA": `onPressed: () {}` (`login_page.dart:82`).
  - Cadastro → "ENVIAR": `onPressed: () {}` (`sign_up_page.dart:75`); campo de e‑mail é `const InputText` sem `onValueChanged`.
  - `send_password.dart` → "ENVIAR" morto; seta de voltar não é botão.
  - Alterar Email / Alterar Senha → "Enviar" no-op (`change_email_page.dart:34`, `change_password.dart:60`).
  - Configurações → "Permissões": corpo `// Do something` (`logged_settings_page.dart:37`).
  - Diário inicial → "EXERCÍCIOS" tem `onPressed` vazio (`user_diary_initial_selection.dart:74`).
  - **Medicamentos → "Novo Medicamento" → "Salvar"** só faz `Navigator.pop` (TODO); confirmado em runtime: "TesteMed" **não** foi adicionado.
- **Evidência:** `38_med_add_dialog.png` → `39_med_after_save.png` (lista inalterada).

### BUG-009 — Cadastro inviável + payload de data incompatível
- **Severidade:** 🟠 Alto
- **Detalhe:** A tela de "Cadastro" na verdade é uma tela de "receber senha por e‑mail" (duplicata de `send_password.dart`), com botão morto — **não há como criar conta pela UI**. Adicionalmente, `UserRegistration.toMap()` envia `birth_date` como `toIso8601String()` (datetime), mas o backend exige `YYYY-MM-DD` → **400** (confirmado via API: a primeira tentativa de registro falhou com "Date has wrong format"). A conta de teste só foi criada enviando a data no formato correto.

### BUG-010 — Login sem feedback de erro/carregamento (exceção engolida)
- **Severidade:** 🟠 Alto
- **Arquivo:** `view_models/login.view_model.dart:35-37`
- **Detalhe:** Falhas de login só fazem `log(...)`; não há spinner nem mensagem. Com credenciais erradas/sem rede, o usuário não vê nada acontecer. Também não há validação de campos vazios (tocar ENTRAR vazio dispara requisição e falha em silêncio — `02_login_empty_submit.png`).

### BUG-011 — `InputText` recria o controller a cada build e perde caracteres
- **Severidade:** 🟠 Alto
- **Arquivo:** `views/widgets/input_text.dart:24`
- **Detalhe:** O `TextEditingController` é criado dentro do `build()`. Durante a animação do teclado (que dispara rebuilds), os caracteres digitados se perdem. **Confirmado em runtime:** ao digitar "qa_tester" com o campo recém‑focado, **só "q" permanecia** (`10`,`11_username_retry.png`); só foi possível preencher digitando em pedaços com o teclado já aberto (`12_username_ok.png`). É um `StatefulWidget` ausente (controller deveria persistir e ser `dispose`d).

### BUG-012 — Relaxamento exibe conteúdo errado (exercícios de mão como "respiração")
- **Severidade:** 🟠 Alto
- **Arquivos:** `relaxation/breathing_page.dart:51-62`, `guided_relaxation_page.dart`
- **Detalhe:** Quando nenhum treino casa com "respiração"/"relaxamento", o código cai em `trainings.first`. Em runtime, "Técnicas de respiração" listou **exercícios de mão** ("Dobrar a mão com dedos esticados") com vídeos de mão. O player funciona, mas o conteúdo está incorreto.
- **Evidência:** `23_breathing_list.png`, `24_relaxation_video.png`.

### BUG-013 — `firstWhere` sem `orElse` pode derrubar o fluxo de exercícios
- **Severidade:** 🟠 Alto
- **Arquivo:** `services/physical_exercises_service.dart:31`
- **Detalhe:** `getExercisesFromTraining` usa `trainings.firstWhere(name.startsWith(type.toString()) && difficulty==)` sem `orElse` → `StateError` se não houver treino correspondente. O `ViewModel` não tem `try/catch`, então a navegação trava sem feedback. O casamento ainda depende do label de UI localizado (`'MÃOS'`/`'PÉS'`), o que é frágil. (No teste com dados reais houve correspondência, então funcionou.)

### BUG-014 — Players de vídeo/áudio do YouTube criados no `build()` e nunca liberados (`dispose`)
- **Severidade:** 🟠 Alto
- **Arquivos:** `widgets/video_player.dart:24-71`, `physical_exercise/widgets/exercise_routine_step_view.dart:16-25`, `level_selector_dialog.dart:13-20`
- **Detalhe:** Controllers do YouTube são instanciados a cada build (alguns em `StatelessWidget`) e nunca `dispose`d → vazamento de recursos/áudio que pode continuar tocando ao navegar. Funciona, mas com leak.

### BUG-015 — `DailyPainReport.fromMap` lê chave errada (`symptoms` × `pain_location`)
- **Severidade:** 🟡 Médio
- **Arquivo:** `models/api_responses/daily_pain_report.dart:30`
- **Detalhe:** `toMap` grava `pain_location`, mas `fromMap` lê `map['symptoms']` → `painLocation` viria `null` e estouraria no campo não‑nulo. Casts diretos sem validação em vários modelos (`exercise.dart`, `training_report.dart`) podem lançar `TypeError` com payload malformado.

### BUG-016 — `RemedyViewModel` usa dados mock fixos (sem integração real) e endpoint divergente
- **Severidade:** 🟡 Médio
- **Arquivo:** `view_models/remedy_view_model.dart:33-39`
- **Detalhe:** A lista de medicamentos é **hardcoded** (Metotrexato, Ácido Fólico, Prednisona); a chamada real está comentada e aponta para `/remedy/`, enquanto o backend expõe `/remedies/`. A UI mostra "Sem detalhes de dose/horário" mesmo havendo `hour` no mock.
- **Evidência:** `37_medicamentos.png`; reconfirmado na 2ª passada (`v2_74_medicamentos.png`). Marcar/desmarcar o checkbox funciona nos dois sentidos dentro da sessão (`v2_77_med_checked.png`/`v2_78_med_unchecked.png`).

### BUG-017 — Erros de ortografia em conteúdo de texto **(v2)**
- **Severidade:** 🟢 Baixo
- **Telas:** diálogo "Qual devo escolher?" (vídeo), tela "Parabéns!", lista de exercícios de Pés.
- **Detalhe:** Foram encontrados erros de digitação visíveis ao usuário:
  - **"INTRUÇÕES EXERCÍCIOS"** no título do vídeo do diálogo "Qual devo escolher?" → correto: **"INSTRUÇÕES"** (`v2_94_qual_escolher.png`).
  - **"Você concluiu mais uma série de execícios..."** na tela "Parabéns!" → correto: **"exercícios"** (`v2_112_end.png`).
  - **"Movimentos circulares com os dedo"** → deveria ser **"dedos"**; **"Movimentos circulares o tornozelo"** → falta preposição (**"do tornozelo"**) (`v2_97_pes_ini_list.png`).
- **Correção sugerida:** revisar os textos/labels (provavelmente vindos do backend para os nomes de exercícios, e de constantes de UI para os diálogos).

---

## ⚠️ Problemas Potenciais

### PP-001 — Segurança: credenciais/tokens trafegam em HTTP puro
- **Categoria:** Segurança
- **Descrição:** `Environment.apiUrl` tem fallback `http://200.136.215.174/api` (IP fixo, cleartext). Login/refresh enviam usuário, senha e JWT sem TLS.
- **Impacto:** Interceptação de credenciais e tokens em rede não confiável. Migrar para HTTPS.

### PP-002 — Ausência total de testes automatizados
- **Categoria:** Manutenção / Qualidade
- **Descrição:** `flutter test` retornou *"Test directory 'test' not found"* — **0 testes**, apesar de `mockito` estar nas dev_dependencies. Nenhuma rede de segurança contra regressões.

### PP-003 — Duas camadas de serviço duplicadas e inconsistentes
- **Categoria:** Manutenção
- **Descrição:** `TrainingService` e `PhysicalExercisesService` fazem o mesmo (`/trainings`, `/exercises`), com diferenças (checagem de status, barra final). `TrainingService` é instanciado direto em telas (não injetável). Risco de divergência.

### PP-004 — `getProviders()` instancia `GlobalProviders` duas vezes
- **Categoria:** Performance / Código
- **Descrição:** `global_providers.dart:33-37` cria dois objetos `GlobalProviders`, construindo listas de providers em duplicidade.

### PP-005 — Sobrecarga de `toString()` como contrato de serialização
- **Categoria:** Manutenção
- **Descrição:** `ExerciseDifficulty.toString()` retorna `easy/medium/hard` (usado como segmento de URL) e `TrainingType.toString()` retorna rótulos de UI (`'MÃOS'`) usados para casar dados do backend. Usar `toString()` como dado é frágil; preferir `.value`/`.label`.

### PP-006 — Acessibilidade / escala de fonte
- **Categoria:** Acessibilidade
- **Descrição:** Para um app voltado a pacientes com artrite, há botões com `fixedSize`/`fontSize` fixos (texto corta — ex.: título "EXERCÍCIOS FÍSI…", botão "Informações sobre atividades físicas" com texto vazando). `InputText` não define `keyboardType`/`autofillHints`. Cores como única pista (vermelho/verde) nos cards de mito/verdade (mitigado por ícones/labels).

### PP-007 — `ListView` dentro de `SingleChildScrollView` (Dor/Inchaço)
- **Categoria:** Performance / Layout
- **Descrição:** `user_level_selection_with_options.dart:109` aninha `ListView` (shrinkWrap) vertical dentro de scroll vertical — anula a virtualização e pode gerar conflito de scroll.

### PP-008 — Subtítulo de dificuldade sobreposto pela lista
- **Categoria:** UX / Layout
- **Descrição:** Nas telas de rotina/passo, o subtítulo "INICIANTE" aparece parcialmente coberto pelo primeiro item da lista/vídeo (`31_routine_overview.png`, `34_exercise_step.png`).

### PP-009 — Botão "voltar" do Android encerra o app em telas que deveriam apenas voltar **(v2 amplia)**
- **Categoria:** UX / Navegação
- **Descrição:** Ao apertar "voltar" na tela "Page Not Found" (BUG‑005), o app foi para a home do Android (saiu do app) em vez de retornar às Configurações. Na 2ª passada, o mesmo ocorreu na tela **Medicamentos**: o botão voltar do sistema **fechou o app** em vez de retornar à Home (`v2_82_nav.png` → `v2_83_back.png`). Indica que várias telas empilhadas não têm rota de retorno válida (uso de `context.go` que substitui a stack em vez de `push`). Combinado com a sessão não persistente (BUG‑006), o usuário é forçado a relogar.

### PP-010 — SVG do logo com elemento não suportado
- **Categoria:** UX
- **Descrição:** logcat repetia `unhandled element <filter/>; Svg loader` ao renderizar `logo-ArtriApp-v2.svg` — `flutter_svg` ignora o filtro (logo ainda aparece, mas o asset deveria ser limpo).

---

## 💡 Sugestões de Melhoria

### SM-001 — Centralizar e endurecer a camada HTTP
- **Categoria:** Arquitetura
- **Descrição:** Unificar `TrainingService`/`PhysicalExercisesService` em um cliente único com `InterceptedClient` (auth + refresh já existentes), checagem de `statusCode`, timeouts e tratamento de erro padronizado. Migrar `DiaryViewModel` (que usa `dart:io HttpClient` cru) para essa camada.
- **Benefício:** Auth consistente, menos duplicação, erros tratáveis. **Esforço:** Médio.

### SM-002 — Implementar de fato a persistência do Diário e a Evolução
- **Categoria:** Funcionalidade
- **Descrição:** Conectar "Salvar" do diário ao backend (endpoints corretos/plurais) e alimentar o gráfico de Evolução com dados reais (incluindo Sono/Inchaço, hoje ausentes). Adicionar estados de carregando/vazio/erro.
- **Benefício:** Torna funcional o propósito central do app. **Esforço:** Alto.

### SM-003 — Corrigir/normalizar formulários e o widget `InputText`
- **Categoria:** UX / Código
- **Descrição:** Transformar `InputText` em `StatefulWidget` com controller persistente + `dispose`; adicionar validação, `errorText`, `keyboardType`, obscurecimento de senha; ligar todos os campos a controllers e implementar os botões hoje mortos (cadastro, recuperação, alterar e‑mail/senha).
- **Benefício:** Formulários utilizáveis e sem perda de digitação. **Esforço:** Médio.

### SM-004 — Registrar as rotas de Configurações e revisar `go` × `push`
- **Categoria:** Código
- **Descrição:** Inserir `SettingsRoutes.getGoRoutes()` na árvore e usar caminhos absolutos; padronizar `context.go`/`context.push` (hoje misturados em `info_page.dart` e relaxamento), evitando back-stack inconsistente.
- **Benefício:** Navegação previsível, fim do "Page Not Found". **Esforço:** Baixo.

### SM-005 — Migrar para HTTPS e remover IP/segredos hardcoded
- **Categoria:** Segurança
- **Descrição:** Servir a API por HTTPS, remover o fallback `http://200.136.215.174/api`, e configurar `network_security_config` apenas para hosts de dev.
- **Benefício:** Proteção de credenciais/tokens. **Esforço:** Médio.

### SM-006 — Introduzir testes automatizados
- **Categoria:** Testes
- **Descrição:** Criar `test/` com testes de unidade (view models, parsers de modelos, `ExerciseDifficulty`) e de widget (login, scale selector). Aproveitar o `mockito` já incluso.
- **Benefício:** Previne regressões nas correções acima. **Esforço:** Médio.

### SM-007 — Liberar recursos de mídia e reduzir duplicação de UI
- **Categoria:** Performance / Código
- **Descrição:** Mover controllers do YouTube para `StatefulWidget` com `dispose`; extrair botões repetidos (9 botões idênticos em `info_page.dart`, `CustomSolidButton`/`CustomOutlinedButton`) e páginas de conteúdo para o `InfoTemplatePage` existente.
- **Benefício:** Sem leaks, menos código. **Esforço:** Médio.

### SM-008 — Limpeza geral
- **Categoria:** Código
- **Descrição:** Remover `print()` (`swelling_page.dart:123`), código morto comentado (`exercise_page.dart:77-180`), corrigir `_FatgiuePageState`/título na tela de Sono, atualizar `withOpacity → withValues` (lints), trailing commas, e os tipos privados em API pública.
- **Benefício:** Código mais limpo; `flutter analyze` zerado. **Esforço:** Baixo.
