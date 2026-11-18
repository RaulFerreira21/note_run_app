# Requisitos — MVP 1: GPS e Mapa na Home

**Spec:** `mvp1-gps-mapa-home`
**Origem:** `especificao_tecnica_note_run.md` (v1.1), Seção 4 e Seção 7 (spec 3 de 3 do MVP 1)
**Depende de:** `mvp1-setup-ambiente` (dev build Android, `expo-maps`/`expo-location` instalados, chave Google Maps, tela de Home esqueleto) e `mvp1-autenticacao-login` (sessão válida para acessar a Home).
**Objetivo:** Entregar a Home com um mapa interativo centralizado na posição atual do usuário em tempo real, após solicitar e validar a permissão de localização (foreground) no Android.

## Escopo

Cobre **permissão de GPS (foreground), leitura da posição e renderização do mapa centralizado** no Android. Fecha o MVP 1.

### Fora de escopo
- Geração de rotas (Smart Route Engine) — fase posterior.
- Gravação/rastreamento de treino e histórico — fase posterior.
- Localização em *background*.
- iOS (adiado; `<expo-map>` e `Location` são agnósticos, mas build/validação são Android nesta fase).

## Pressupostos

- `expo-maps` e `expo-location` já instalados e declarados no `app.config.js` (feito na spec 1), com `GOOGLE_MAPS_API_KEY` no `.env`.
- O app roda como *development build* (mapa não funciona em Expo Go).

---

## Requisitos

### Requisito 1 — Solicitação e validação da permissão de localização
**História:** Como usuário, quero que o app peça permissão de localização de forma clara, para controlar o acesso ao meu GPS.

#### Critérios de Aceitação
1. QUANDO a Home for aberta sem permissão de localização concedida, ENTÃO o app DEVE solicitar a permissão de *foreground* ao usuário.
2. QUANDO o usuário conceder a permissão, ENTÃO o app DEVE prosseguir para ler a posição e centralizar o mapa.
3. QUANDO o usuário negar a permissão, ENTÃO o app DEVE exibir um estado explicativo (sem travar), informando que o mapa precisa da localização.
4. ONDE a permissão estiver bloqueada definitivamente ("não perguntar novamente"), ENTÃO o app DEVE oferecer um caminho para abrir as Configurações do sistema.
5. QUANDO o usuário retornar das Configurações com a permissão concedida, ENTÃO o app DEVE refletir o novo estado sem exigir reinício manual.

### Requisito 2 — Leitura da posição atual (GPS)
**História:** Como usuário, quero que o app saiba onde estou, para centralizar o mapa na minha localização.

#### Critérios de Aceitação
1. QUANDO a permissão estiver concedida, ENTÃO o app DEVE obter a posição atual (latitude/longitude) via serviço de localização.
2. QUANDO a posição mudar durante o uso da Home, ENTÃO o app DEVE acompanhar a atualização em tempo real (atualização contínua).
3. ENQUANTO a primeira posição ainda não chegou, o app DEVE exibir um estado de carregamento.
4. QUANDO a Home deixar de estar em primeiro plano/visível, ENTÃO o app DEVE parar o acompanhamento contínuo para poupar bateria.
5. ONDE a precisão for configurável, o app DEVE escolher uma precisão adequada à exibição do mapa (sem manter o GPS em modo navegação desnecessariamente).

### Requisito 3 — Mapa interativo centralizado na posição
**História:** Como usuário, quero ver um mapa centrado em mim, para me situar antes de um treino.

#### Critérios de Aceitação
1. QUANDO houver posição e permissão, ENTÃO a Home DEVE renderizar `<expo-map>` ocupando a área principal da tela (com tamanho definido, ex.: `flex: 1`).
2. QUANDO a primeira posição chegar, ENTÃO a câmera do mapa DEVE centralizar nas coordenadas atuais com um zoom adequado.
3. O mapa DEVE indicar a posição do usuário (ponto de localização nativo, ex.: `isMyLocationEnabled`).
4. QUANDO o usuário interagir com o mapa (arrastar/zoom), ENTÃO o mapa DEVE responder de forma nativa.
5. ONDE a chave da Google Maps estiver ausente/inválida, ENTÃO o app DEVE lidar com o mapa em branco de forma controlada (mensagem/estado), sem crashar.

### Requisito 4 — Estados da Home
**História:** Como usuário, quero entender o que está acontecendo em cada situação, para não ficar diante de uma tela confusa.

#### Critérios de Aceitação
1. A Home DEVE representar claramente os estados: permissão pendente, permissão negada/bloqueada, carregando posição, e mapa pronto.
2. ONDE ocorrer erro de localização, ENTÃO o app DEVE exibir mensagem e permitir nova tentativa.
3. O estado de UI DEVE usar signals (sem zone.js), conforme a steering.

### Requisito 5 — Verificação automatizada
**História:** Como desenvolvedor, quero testes cobrindo a lógica da Home, para garantir comportamento sem simulador.

#### Critérios de Aceitação
1. QUANDO `npm test` for executado, ENTÃO DEVEM existir testes (Node, `@ng-native/testing`, com fakes de permissão/localização) cobrindo: permissão concedida → carregando → centraliza; permissão negada → estado explicativo.
2. QUANDO `npm run typecheck` for executado, ENTÃO DEVE concluir sem erros.
3. ONDE a validação do mapa em si exigir dispositivo (render nativo do `<expo-map>`), ENTÃO isso DEVE ser documentado como verificação manual no *development build* Android, fora dos testes em Node.
