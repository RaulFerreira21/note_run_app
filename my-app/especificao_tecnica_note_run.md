# Especificação Técnica e Documento de Requisitos: Note Run

**Versão:** 1.0.0 (Draft)  
**Metodologia:** Spec-Driven Development (SDD)  
**Status:** Em Discussão / Planejamento  

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
* **Justificativa:** Aproveitamento da expertise em Angular para construção de interfaces mobile verdadeiramente nativas, utilizando como ponto de partida a arquitetura do aplicativo de corrida de exemplo presente na documentação oficial da ferramenta.

### 3.2. Back-end (API & Serviços)
* **Linguagem:** Go (Golang)
* **Responsabilidades:**
  * Gerenciamento de autenticação e sessões (JWT / OAuth2).
  * CRUD de usuários, perfis e histórico de treinos.
  * Orquestração e integração com APIs de mapas e roteamento (ex: OpenStreetMap/OSRM, Mapbox ou Google Maps API) para o cálculo das rotas inteligentes.

### 3.3. Banco de Dados
* **SGBD Principal:** PostgreSQL
* **Extensões Recomendadas:** **PostGIS** (essencial para armazenamento, indexação e consultas espaciais/geográficas de coordenadas, polígonos de rotas e cálculo de distâncias diretamente no banco).

---

## 4. Escopo do MVP 1 (Minimum Viable Product)

Seguindo a abordagem iterativa, o **MVP 1** tem como objetivo validar a stack tecnológica (especialmente a integração do Angular Native com recursos nativos do dispositivo e comunicação com a API em Go).

### 4.1. Entregáveis do MVP 1
1. **Setup do Ambiente:** Estruturação dos repositórios (Front-end Angular Native + Back-end Go + Container PostgreSQL).
2. **Fluxo de Autenticação:** Tela de Login funcional integrada ao back-end.
3. **Permissões Nativas:** Solicitação e validação de permissão de uso do GPS do dispositivo (Foreground Location).
4. **Visualização de Mapa (Home):** Renderização de uma tela principal contendo um mapa interativo centralizado na coordenada geográfica atual (latitude/longitude) do usuário em tempo real.

*(Nota: A geração de rotas, criação de perfil detalhado e gravação de treinos ficam explicitamente fora do escopo do MVP 1, entrando nas fases subsequentes).*

---

## 5. Pontos Abertos para Discussão (Spec-Driven Development)

Para garantirmos que a especificação esteja fechada antes de codificarmos, precisamos debater e tomar decisões arquiteturais sobre os seguintes tópicos:

1. **Banco de Dados e Dados Geoespaciais:**
   * *Discussão:* O PostgreSQL com a extensão **PostGIS** é o padrão ouro do mercado para dados geográficos. Estamos de acordo em utilizar o PostGIS para salvar os traçados das corridas, ou preferimos salvar as rotas apenas como arrays JSON/GeoJSON simples no início (especialmente no MVP 1)?
2. **Provedor de Mapas e Roteamento (Routing Engine):**
   * *Discussão:* Qual serviço usaremos para renderizar o mapa no Angular Native e, futuramente, calcular as rotas de $500\text{ m}$ a $21\text{ km}$? (Opções comuns: *Mapbox*, *Google Maps Platform*, ou soluções open-source baseadas em *OpenStreetMap* + *GraphHopper/OSRM*).
3. **Estratégia de Autenticação no MVP 1:**
   * *Discussão:* Para o login do MVP 1, faremos autenticação tradicional (E-mail/Senha gerenciados pelo nosso back-end em Go com JWT) ou utilizaremos um provedor externo (Firebase Auth, Supabase, OAuth com Google/Apple)?
4. **Padrão de Comunicação da API:**
   * *Discussão:* A comunicação entre o app Angular Native e o servidor Go será feita via REST padrão (JSON) ou consideramos gRPC/GraphQL? (Para o MVP 1, REST costuma ser o caminho mais ágil).

---

## 6. Próximos Passos do Planejamento
- [ ] Responder e fechar as decisões da **Seção 5 (Pontos Abertos)**.
- [ ] Especificar o contrato da API (Endpoints REST/Swagger) para o Login do MVP 1.
- [ ] Desenhar a modelagem inicial do banco de dados (Tabela `users`).
- [ ] Analisar o exemplo de *Running App* da documentação do `ng-native` para extrair a estrutura base de componentes e mapa.