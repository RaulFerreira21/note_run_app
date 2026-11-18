# Requisitos — MVP 1: Setup do Ambiente

**Spec:** `mvp1-setup-ambiente`
**Origem:** `especificao_tecnica_note_run.md` (v1.1), Seção 4 e Seção 7 (spec 1 de 3 do MVP 1)
**Objetivo:** Preparar o projeto front-end Angular Native para rodar como *development build* **no Android**, com a estrutura base de pastas, navegação nativa e esqueletos de tela (Login e Home) prontos para as specs seguintes (`mvp1-autenticacao-login` e `mvp1-gps-mapa-home`) preencherem.

> **Plataforma-alvo (decisão):** O MVP 1 foca **exclusivamente em Android** nesta fase. O suporte a iOS será avaliado depois; a estrutura e a navegação são agnósticas de plataforma, então essa escolha afeta apenas o build e a configuração nativa (requisitos 1 e 2).

## Escopo

Esta spec cobre **apenas a fundação**: ambiente, configuração nativa, estrutura e navegação. Ela **não** implementa login real, permissão de GPS nem o mapa — esses ficam nas specs 2 e 3. As telas aqui são esqueletos navegáveis (placeholders).

### Fora de escopo
- **Suporte a iOS** (build, Apple Maps, simulador) — adiado para fase posterior.
- Lógica de autenticação, formulários e chamadas HTTP (spec 2).
- Permissão de GPS, serviço de localização e `<expo-map>` (spec 3).
- Qualquer back-end Go / banco de dados.

---

## Requisitos

### Requisito 1 — Development build operacional (Android)
**História:** Como desenvolvedor, quero o projeto configurado para rodar como *development build* no Android, para que recursos nativos (mapa, GPS) funcionem fora do Expo Go nas specs seguintes.

#### Critérios de Aceitação
1. QUANDO o desenvolvedor executar `npx expo run:android` em um ambiente Android configurado, ENTÃO o app DEVE compilar e abrir em emulador/dispositivo.
2. ONDE o projeto depender de módulos nativos (`expo-maps`, `expo-location`), o `app.json` e os plugins DEVEM estar declarados de modo que um *prebuild* gere o projeto nativo Android sem erro.
3. SE o desenvolvedor tentar usar apenas o Expo Go, ENTÃO a documentação do projeto (README/AGENTS) DEVE deixar explícito que o mapa exige *development build*.
4. ONDE o iOS for retomado no futuro, a configuração NÃO DEVE conter bloqueios Android-only que impeçam adicionar `expo run:ios` depois (preferir configuração por plataforma a remover o iOS do projeto).

### Requisito 2 — Configuração da chave Google Maps (Android)
**História:** Como desenvolvedor, quero a configuração de mapas do Android preparada, para que a spec 3 apenas renderize o mapa sem reconfigurar o ambiente.

#### Critérios de Aceitação
1. O `app.json` DEVE conter o campo `expo.android.config.googleMaps.apiKey` lendo a chave de uma fonte não versionada (ex.: variável de ambiente / arquivo ignorado pelo git).
2. O projeto NÃO DEVE versionar a chave real da Google Maps API no repositório.
3. ONDE `expo-maps` for adicionado, o plugin DEVE declarar `requestLocationPermission` e a string de uso de localização no `app.json`.
4. QUANDO a chave da Google Maps estiver ausente ou inválida, ENTÃO a documentação do projeto DEVE orientar como obtê-la e configurá-la (Maps SDK for Android habilitada).

### Requisito 3 — Estrutura base de pastas e componentes
**História:** Como desenvolvedor, quero uma organização de pastas previsível sob `src/app`, para que login, mapa e futuras features tenham onde morar de forma consistente.

#### Critérios de Aceitação
1. QUANDO a estrutura for criada, ENTÃO DEVE existir uma organização por feature/tela sob `src/app` (ex.: `src/app/login`, `src/app/home`), cada uma com seu componente e espaço para teste colocado ao lado.
2. ONDE houver código compartilhado (serviços, tipos, tokens), ENTÃO DEVE existir um local comum (ex.: `src/app/core` ou `src/app/shared`).
3. A estrutura DEVE seguir as convenções descritas na steering do projeto (`structure.md`): um componente por pasta, template/estilos inline, teste `*.test.ts` ao lado.
4. O componente raiz existente (`src/app/app.ts`) DEVE ser adaptado para hospedar a navegação, deixando de ser o contador de exemplo.

### Requisito 4 — Navegação nativa com Router
**História:** Como usuário, quero que o app me leve do login para a home, para que o fluxo básico de telas exista mesmo antes das features.

#### Critérios de Aceitação
1. ONDE a navegação for configurada, o app DEVE usar o Router nativo (`provideNativeRouter` de `@ng-native/router`) com um `<native-stack-outlet />` no template raiz.
2. QUANDO o app iniciar, ENTÃO a rota inicial DEVE ser a tela de Login (esqueleto).
3. QUANDO uma ação de "entrar" (placeholder) for acionada na tela de Login, ENTÃO o app DEVE navegar para a tela de Home (esqueleto).
4. ONDE rotas receberem parâmetros no futuro, a configuração DEVE incluir `withComponentInputBinding()` para que cheguem como inputs.

### Requisito 5 — Esqueletos de tela navegáveis (Login e Home)
**História:** Como desenvolvedor, quero telas de Login e Home mínimas e navegáveis, para que as specs 2 e 3 preencham o conteúdo sem criar andaimes de navegação.

#### Critérios de Aceitação
1. QUANDO a tela de Login (esqueleto) for exibida, ENTÃO DEVE conter ao menos um título e um controle de ação placeholder que dispara a navegação para a Home.
2. QUANDO a tela de Home (esqueleto) for exibida, ENTÃO DEVE conter uma área reservada (placeholder) onde o mapa entrará na spec 3.
3. ONDE elementos nativos forem usados, ENTÃO DEVEM seguir as regras da steering (`tech.md`): elementos em minúsculo importados de `@ng-native/components`, eventos nativos, acessibilidade por props.
4. ONDE houver estado de UI, ENTÃO DEVE usar signals (sem zone.js).

### Requisito 6 — Verificação automatizada do esqueleto
**História:** Como desenvolvedor, quero que o build e os testes passem após o setup, para ter uma base verde antes de construir as features.

#### Critérios de Aceitação
1. QUANDO `npm run typecheck` for executado após o setup, ENTÃO DEVE concluir sem erros.
2. QUANDO `npm test` for executado, ENTÃO DEVE existir ao menos um teste (em Node, via `@ng-native/testing`) que verifica a navegação Login → Home do esqueleto, e ele DEVE passar.
3. ONDE o teste de exemplo do contador (`src/app/app.test.ts`) tratar de comportamento removido, ENTÃO DEVE ser atualizado ou substituído para refletir a nova raiz.
