# Especificação Técnica e Documento de Requisitos: Note Run

**Versão:** 1.1.0  
**Metodologia:** Spec-Driven Development (SDD)  
**Status:** Decisões da Seção 5 fechadas / Pronto para gerar specs do MVP 1  

---

## 1. Visão Geral do Produto (Product Overview)
O **Note Run** é um aplicativo mobile voltado para o auxílio pessoal em treinos de corrida. Inspirado em plataformas como o Strava, o objetivo central da aplicação é guiar desde corredores iniciantes até intermediários, oferecendo acompanhamento de evolução baseado em metas personalizadas e características físicas.

### 1.1. Proposta de Valor e Diferencial Competitivo
O grande diferencial técnico e de produto do **Note Run** é o seu **Motor Inteligente de Geração de Rotas (Smart Route Engine)**. Ao contrário de rastreadores passivos que apenas gravam o caminho percorrido, o Note Run atua de forma prescritiva, gerando rotas dinâmicas antes do treino com base na distância desejada e na topologia escolhida pelo usuário.

---

## 2. Requisitos Funcionais (Core Features)

### 2.1. Gestão de Identidade e Perfil (Onboarding)
* **Autenticação:** Fluxo de cadastro e login seguro.
* **Perfil Físico:** Coleta de dados antropométricos básicos (peso, altura, idade) para cálculo de métricas de esforço.
* **Definição de Objetivos:** Seleção de metas primárias:
  * *Iniciante:* Começar a correr (transição caminhada/corrida).
  * *Performance:* Melhorar o *pace* (ritmo médio por km).
  * *Endurance:* Aumentar a distância máxima percorrida.

### 2.2. Motor de Geração de Rotas (Feature Diferencial)
O usuário poderá solicitar a criação de uma rota configurando os seguintes parâmetros:
* **Distância Alvo:** Seleção em um intervalo (*range*) de $500\text{ m}$ até $21\text{ km}$ (Meia Maratona).
* **Modalidades de Topologia de Rota:**
  1. **Com Retorno (Loop / Out-and-Back):** O ponto de partida ($A$) e o ponto de chegada ($B$) são idênticos ($A = B$). O algoritmo calcula um circuito ou um trajeto de ida e volta que totalize a distância escolhida.
  2. **Sem Retorno (Point-to-Point):** A corrida inicia no ponto $A$ (localização atual) e o algoritmo define um ponto $B$ distinto ($A \neq B$) cujo trajeto viário/pedonal resulte na distância configurada.
  3. **Com Voltas (Lap-based Circuit):** O sistema identifica uma área segura ou circuito menor próximo ao usuário (ex: um parque ou quarteirão amplo) e calcula o número de voltas ($N$) necessárias para atingir a distância total desejada.

### 2.3. Rastreamento e Evolução (Tracking & Analytics)
* Gravação de atividades via GPS em tempo real.
* Histórico de corridas realizadas (tempo, distância, *pace* médio, traçado no mapa).
* Painel de acompanhamento de evolução frente ao objetivo definido no perfil.

---

## 3. Arquitetura e Stack Tecnológica

A arquitetura será construída priorizando performance, tipagem forte e escalabilidade, aproveitando o conhecimento prévio em ecossistema Angular no front-end e a alta performance concorrente do Go no back-end.

### 3.1. Front-end Mobile
* **Framework:** [Angular Native (ng-native)](https://ng-native.com/)
* **Justificativa:** Aproveitamento da expertise em Angular para construção de interfaces mobile verdadeiramente nativas.
* **Correção (v1.1):** Não existe um "aplicativo de corrida de exemplo" completo na documentação oficial para usar como base. A documentação traz componentes individuais (incluindo um trecho de mapa com `polyline` de rota em [Maps](https://ng-native.com/packages/expo/maps.md)), mas a estrutura base de componentes e telas será construída pela equipe.
* **Recursos nativos confirmados na documentação:**
  * **Mapa:** `<expo-map>` via `@ng-native/expo/map-view` — Apple Maps no iOS, Google Maps no Android; suporta marcadores, `polylines` (traçado da rota), câmera e `isMyLocationEnabled`.
  * **GPS (Foreground):** serviço `Location` via `@ng-native/expo/location` — `current()` (leitura única) e `start()` (contínuo) expõem a posição como *signal*; a permissão de *foreground* é solicitada pelo próprio serviço.
  * **HTTP:** `provideNativeHttpClient()` (de `@ng-native/platform/http`), não o `provideHttpClient()` do Angular web.

### 3.2. Back-end (API & Serviços)
* **Linguagem:** Go (Golang)
* **Responsabilidades:**
  * Gerenciamento de autenticação e sessões (JWT / OAuth2).
  * CRUD de usuários, perfis e histórico de treinos.
  * Orquestração e integração com APIs de mapas e roteamento (ex: OpenStreetMap/OSRM, Mapbox ou Google Maps API) para o cálculo das rotas inteligentes.
  * **Nota de risco (v1.1):** A geração de rota com distância-alvo exata nas topologias *Loop* e *Point-to-Point* não é trivial. O OSRM não oferece roteamento *round-trip* nativamente; o **GraphHopper** oferece (`round_trip`) e o Mapbox tem otimização. A escolha do *routing engine* fica condicionada a suportar *round-trip*, caso contrário será necessário um algoritmo próprio sobre o grafo. Fora do escopo do MVP 1.

### 3.3. Banco de Dados
* **SGBD Principal:** PostgreSQL
* **Extensões Recomendadas:** **PostGIS** (essencial para armazenamento, indexação e consultas espaciais/geográficas de coordenadas, polígonos de rotas e cálculo de distâncias diretamente no banco).

---

## 4. Escopo do MVP 1 (Minimum Viable Product)

Seguindo a abordagem iterativa, o **MVP 1** tem como objetivo validar a stack tecnológica (especialmente a integração do Angular Native com recursos nativos do dispositivo e comunicação com a API em Go).

### 4.1. Entregáveis do MVP 1
1. **Setup do Ambiente:** Estruturação do repositório front-end Angular Native e preparação do *development build*. O back-end Go + PostgreSQL será construído em fase posterior, com spec própria.
2. **Fluxo de Autenticação:** Tela de Login funcional. **Como o back-end ainda não existe (v1.1), o front-end do MVP 1 roda contra um *mock/stub* que respeita um contrato REST definido**, a ser substituído pela API Go quando pronta.
3. **Permissões Nativas:** Solicitação e validação de permissão de uso do GPS do dispositivo (Foreground Location).
4. **Visualização de Mapa (Home):** Renderização de uma tela principal contendo um mapa interativo centralizado na coordenada geográfica atual (latitude/longitude) do usuário em tempo real.

### 4.2. Pré-requisitos técnicos do MVP 1 (v1.1)
* **Plataforma-alvo: Android-first.** O MVP 1 foca **exclusivamente em Android** nesta fase; o suporte a iOS será avaliado depois. A estrutura de código e a navegação são agnósticas de plataforma, então a decisão afeta apenas build e configuração nativa.
* **Development build obrigatório:** `expo-maps` inclui código nativo e **não roda no Expo Go**. É necessário um *development build* (`npx expo run:android` ou EAS Build) desde o item 4.
* **Chave Google Maps (Android):** o mapa no Android exige uma chave da *Maps SDK for Android* configurada no `app.json`, lida de fonte não versionada.
* **iOS (adiado):** quando retomado, o Apple Maps é parte do sistema (sem chave), renderiza no iOS 17+ e toque em marcador exige iOS 18. A configuração deve evitar bloqueios Android-only que impeçam retomar o iOS.
* **Status do `expo-maps`:** marcado como *alpha* pela Expo; pode haver mudanças entre versões do SDK.

*(Nota: A geração de rotas, criação de perfil detalhado e gravação de treinos ficam explicitamente fora do escopo do MVP 1, entrando nas fases subsequentes).*

---

## 5. Decisões Arquiteturais (Fechadas na v1.1)

As decisões abaixo foram debatidas e fechadas. Ficam registradas como *Architecture Decision Records* resumidos.

1. **Banco de Dados e Dados Geoespaciais — DECIDIDO:**
   * **PostGIS no PostgreSQL desde o início** no back-end (custo baixo de adicionar e evita migração dolorosa depois). Observação: o MVP 1 não grava corridas, então o PostGIS não é exercitado nesta fase e não bloqueia o spec do front-end.
2. **Provedor de Mapas e Roteamento (Routing Engine) — DECIDIDO:**
   * **Mapa no app:** `<expo-map>` (Apple Maps no iOS, Google Maps no Android), já que é o componente nativo do ng-native.
   * **Routing engine (fase posterior):** avaliar **GraphHopper** por suportar roteamento `round_trip`. Decisão final condicionada a benchmark na fase do motor de rotas.
3. **Estratégia de Autenticação no MVP 1 — DECIDIDO:**
   * **E-mail/senha com JWT gerenciado pelo back-end Go próprio** (sem provedor externo), alinhado ao objetivo de validar a integração com o Go. No MVP 1, consumido via *mock/stub* enquanto o back-end não existe.
4. **Padrão de Comunicação da API — DECIDIDO:**
   * **REST/JSON** para o MVP 1.

---

## 6. Próximos Passos do Planejamento
- [x] Responder e fechar as decisões da **Seção 5**. *(Fechadas na v1.1.)*
- [x] Gerar as specs do MVP 1 (front-end), divididas por finalidade — ver Seção 7. *(Requisitos, design e tarefas redigidos.)*
- [x] Definir o contrato REST de Login (request/response, erros) — ver Seção 7 e a spec `mvp1-autenticacao-login`. *(Formato interno do JWT fica a cargo da spec do back-end.)*
- [ ] Executar as specs na ordem 1 → 2 → 3 (implementação).
- [ ] (Fase do back-end) Desenhar a modelagem inicial do banco de dados (Tabela `users`) e spec própria do serviço Go.

---

## 7. Plano de Specs do MVP 1 (Front-end)

O MVP 1 do front-end foi dividido em specs separadas por finalidade, para estruturar melhor o desenvolvimento. **As três specs estão com requisitos, design e tarefas redigidos** (em `.kiro/specs/`); a implementação ainda não foi executada.

1. **`mvp1-setup-ambiente`** *(especificada)* — Development build Android, chave Google Maps, estrutura base de pastas/componentes, navegação (Router nativo) e esqueleto das telas.
2. **`mvp1-autenticacao-login`** *(especificada)* — Tela de login, Signal Forms, serviço de auth consumindo contrato REST via *mock/stub*, armazenamento seguro do JWT (`SecureStorage`), guard de sessão e interceptor `Bearer`.
3. **`mvp1-gps-mapa-home`** *(especificada)* — Permissão de GPS (foreground), serviço de localização e Home com `<expo-map>` centralizado na posição atual em tempo real, com máquina de estados (permissão → posição → mapa).

**Ordem de execução:** 1 → 2 → 3 (cada spec depende da fundação da anterior).

**Contrato REST de login** (consumido pelo front via *mock/stub*, base para a futura spec do back-end):
`POST /auth/login` ← `{ email, password }` → `200 { token, user }` | `401` credenciais inválidas.

*(O back-end Go + PostgreSQL terá sua própria spec em fase posterior.)*