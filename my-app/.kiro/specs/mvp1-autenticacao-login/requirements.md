# Requisitos — MVP 1: Autenticação / Login

**Spec:** `mvp1-autenticacao-login`
**Origem:** `especificao_tecnica_note_run.md` (v1.1), Seção 4 e Seção 7 (spec 2 de 3 do MVP 1)
**Depende de:** `mvp1-setup-ambiente` (navegação nativa, estrutura de pastas e esqueleto da tela de Login já existentes).
**Objetivo:** Transformar o esqueleto de Login em um fluxo de autenticação funcional — formulário com validação (Signal Forms), chamada a um contrato REST de login (consumido via *mock/stub* enquanto o back-end Go não existe), persistência segura do token (JWT) e navegação para a Home somente quando autenticado.

## Escopo

Cobre o fluxo de **login por e-mail/senha com JWT** no app Android, do formulário à sessão persistida. A autenticação é **E-mail/senha gerenciada pelo back-end próprio** (decisão da Seção 5.3), **sem provedor externo**.

### Fora de escopo
- Cadastro de conta, recuperação de senha e perfil físico (fases posteriores).
- Implementação real do back-end Go (spec própria depois) — aqui o front consome um **mock/stub** que respeita o contrato REST.
- Permissão de GPS e mapa (spec 3).
- OAuth/login social (decisão: não usar no MVP 1).

## Pressupostos

- Contrato REST de login definido: `POST /auth/login` recebendo `{ email, password }` e respondendo `{ token, user }` em caso de sucesso, e erro com status apropriado (401 para credenciais inválidas).
- A URL base da API vem de configuração de ambiente (igual ao padrão de segredos do setup), com um valor de *mock* para desenvolvimento.

---

## Requisitos

### Requisito 1 — Formulário de login com validação
**História:** Como usuário, quero informar e-mail e senha em um formulário validado, para entrar na minha conta com feedback claro de erros.

#### Critérios de Aceitação
1. QUANDO a tela de Login for exibida, ENTÃO DEVE conter campos de e-mail e senha usando `<text-input>` ligados a Signal Forms (`[formField]`).
2. ONDE o e-mail for inválido ou vazio, ou a senha estiver vazia, o formulário DEVE marcar o campo como inválido e exibir mensagem de erro após o campo ser tocado.
3. O campo de senha DEVE ocultar o texto digitado (entrada protegida) e o de e-mail DEVE usar teclado de e-mail.
4. ENQUANTO o formulário for inválido, o botão "Entrar" DEVE impedir o envio (via `submit()`, que marca os campos como tocados e não chama a ação).

### Requisito 2 — Autenticação contra o contrato REST (via mock/stub)
**História:** Como usuário, quero que minhas credenciais sejam verificadas pela API, para ser autenticado com segurança.

#### Critérios de Aceitação
1. QUANDO o usuário enviar credenciais válidas, ENTÃO o app DEVE chamar `POST /auth/login` via HttpClient nativo (`provideNativeHttpClient()`) com `{ email, password }`.
2. QUANDO a API responder sucesso, ENTÃO o app DEVE receber e tratar o `{ token, user }` retornado.
3. QUANDO a API responder 401 (credenciais inválidas), ENTÃO o app DEVE exibir mensagem de erro de autenticação sem navegar para a Home.
4. QUANDO houver falha de rede ou erro de servidor (5xx/timeout), ENTÃO o app DEVE exibir mensagem de erro recuperável e permitir nova tentativa.
5. ENQUANTO a chamada estiver em andamento, o app DEVE indicar carregamento e desabilitar o botão "Entrar" (`f().submitting()`).
6. ONDE o back-end real ainda não existir, a chamada DEVE ser atendida por um *mock/stub* configurável que respeita o mesmo contrato, trocável pela API Go sem alterar a tela.

### Requisito 3 — Persistência segura da sessão (JWT)
**História:** Como usuário, quero continuar logado após fechar o app, para não precisar entrar toda vez.

#### Critérios de Aceitação
1. QUANDO a autenticação for bem-sucedida, ENTÃO o token (JWT) DEVE ser gravado em armazenamento seguro (`SecureStorage` — keychain/keystore), nunca em armazenamento comum nem em log.
2. QUANDO o app iniciar e houver token válido persistido, ENTÃO o usuário DEVE ser levado direto à Home, sem passar pela tela de Login.
3. QUANDO não houver token persistido, ENTÃO a tela de Login DEVE ser a rota inicial.
4. ONDE existir ação de "sair" (logout), ENTÃO o token DEVE ser removido do armazenamento seguro e o usuário retornado ao Login. *(Logout pode ser um gatilho mínimo nesta fase.)*

### Requisito 4 — Navegação condicionada à sessão
**História:** Como usuário autenticado, quero ir para a Home ao entrar, e não conseguir voltar ao Login pelo gesto de voltar.

#### Critérios de Aceitação
1. QUANDO a autenticação for concluída, ENTÃO o app DEVE navegar para a Home com `NativeNavigation.reset(...)`, sem deixar o Login no histórico.
2. ONDE a rota de Home for acessada sem sessão válida, ENTÃO um guard DEVE redirecionar para o Login.
3. ONDE a rota de Login for acessada com sessão válida, ENTÃO o app DEVE redirecionar para a Home.

### Requisito 5 — Segurança e tratamento de dados sensíveis
**História:** Como responsável pelo produto, quero que credenciais e token sejam tratados com cuidado, para reduzir risco de vazamento.

#### Critérios de Aceitação
1. A senha NÃO DEVE ser persistida em lugar algum do dispositivo.
2. Token e credenciais NÃO DEVEM aparecer em logs nem em mensagens de erro exibidas ao usuário.
3. ONDE a URL base da API for configurada, ela DEVE vir de configuração de ambiente (não fixada em código versionado).
4. O token DEVE ser enviado como `Authorization: Bearer <token>` nas chamadas autenticadas subsequentes (preparando as specs futuras), via interceptor.

### Requisito 6 — Verificação automatizada
**História:** Como desenvolvedor, quero testes cobrindo o fluxo de login, para garantir o comportamento sem simulador.

#### Critérios de Aceitação
1. QUANDO `npm test` for executado, ENTÃO DEVEM existir testes (Node, `@ng-native/testing` + `HttpTestingController` + `Store` fake) cobrindo: login com sucesso (navega e persiste token), credenciais inválidas (401, mensagem, sem navegação) e validação de formulário (envio bloqueado).
2. QUANDO `npm run typecheck` for executado, ENTÃO DEVE concluir sem erros.
3. ONDE houver guard de sessão, ENTÃO DEVE haver teste de navegação cobrindo acesso com e sem token.
