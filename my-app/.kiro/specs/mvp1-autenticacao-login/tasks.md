# Tarefas — MVP 1: Autenticação / Login

**Spec:** `mvp1-autenticacao-login`
**Base:** `requirements.md` e `design.md` desta spec.
**Pré-requisito:** `mvp1-setup-ambiente` concluída.
**Como usar:** execute na ordem; marque `[x]` ao concluir. Cada tarefa referencia os requisitos que atende.

---

- [ ] 1. Instalar dependências de formulário e armazenamento seguro
  - Instalar `@angular/forms` na versão exata do `@angular/core` (conferir com `npm ls @angular/core`).
  - Instalar `expo-secure-store` via `npx expo install`.
  - _Requisitos: R1, R3_

- [ ] 2. Definir tipos e configuração de ambiente da auth
  - [ ] 2.1 Criar `src/app/core/auth/auth.types.ts` com `Credentials`, `User`, `AuthResult`.
  - [ ] 2.2 Criar `src/app/core/auth/auth.config.ts` com o token `API_BASE_URL` lendo `process.env.EXPO_PUBLIC_API_BASE_URL`; documentar a variável no `.env.example`.
  - _Requisitos: R2, R5 (critério 3)_

- [ ] 3. Implementar o contrato de API e o stub trocável
  - [ ] 3.1 Criar `AuthApi` (classe abstrata/token) com `login(credentials): Promise<AuthResult>`.
  - [ ] 3.2 Criar `HttpAuthApi` usando `HttpClient` contra `POST {API_BASE_URL}/auth/login`.
  - [ ] 3.3 Criar `StubAuthApi` que resolve credenciais de teste e lança erro 401 no mesmo formato da API real.
  - [ ] 3.4 Configurar o provider para selecionar `StubAuthApi` em dev (sem back-end) e `HttpAuthApi` quando houver API, sem alterar a tela.
  - _Requisitos: R2 (todos), R2.6_

- [ ] 4. Implementar o AuthService (sessão + persistência segura)
  - Criar `src/app/core/auth/auth.service.ts` com `token` (signal do `SecureStorage`, chave `auth.token`), `isAuthenticated` (computed), `login()` (chama `AuthApi`, grava token, `flush()`), `logout()` (`remove`).
  - Garantir que a senha nunca é persistida.
  - _Requisitos: R3 (critérios 1, 2, 4), R5 (critério 1)_

- [ ] 5. Configurar HttpClient nativo e interceptor de token
  - [ ] 5.1 Adicionar `provideNativeHttpClient()` (de `@ng-native/platform/http`) aos providers em `main.ts`.
  - [ ] 5.2 Criar `auth.interceptor.ts` que anexa `Authorization: Bearer <token>` quando houver token e registrá-lo via `withInterceptors([...])`.
  - _Requisitos: R2 (critério 1), R5 (critério 4)_

- [ ] 6. Implementar o guard de sessão e a rota inicial condicional
  - Criar `auth.guard.ts` com `homeGuard` (redireciona para Login sem sessão) e `loginGuard` (redireciona para Home com sessão).
  - Aplicar os guards nas rotas `''` (Login) e `home` em `app.routes.ts`.
  - _Requisitos: R3 (critérios 2, 3), R4 (critérios 2, 3)_

- [ ] 7. Implementar o formulário de login real
  - [ ] 7.1 Substituir o esqueleto em `src/app/login/login.ts` por Signal Forms (`form()`, `[formField]`) com campos de e-mail (teclado de e-mail, validação `required`+`email`) e senha (entrada protegida, `required`).
  - [ ] 7.2 Implementar `enter()` com `submit()`: em sucesso, `AuthService.login()` + `NativeNavigation.reset(ROUTES.home)`; em erro, setar mensagem.
  - [ ] 7.3 Ligar estado de carregamento ao `f().submitting()` (botão desabilitado + rótulo).
  - [ ] 7.4 Estilizar estado inválido via `text-input[data-invalid][data-touched]`.
  - _Requisitos: R1 (todos), R2 (critérios 2, 5), R4 (critério 1)_

- [ ] 8. Mapear e exibir erros com segurança
  - Implementar o mapeamento 401 → "E-mail ou senha incorretos.", 5xx → erro de servidor, rede/timeout → erro de conexão, sem vazar token/senha/corpo bruto.
  - _Requisitos: R2 (critérios 3, 4), R5 (critério 2)_

- [ ] 9. Adicionar logout mínimo na Home
  - Incluir uma ação de "sair" na `home.ts` que chama `AuthService.logout()` e retorna ao Login.
  - _Requisitos: R3 (critério 4)_

- [ ] 10. Testes do fluxo de autenticação
  - [ ] 10.1 `login.test.ts`: validação (envio bloqueado), sucesso (200 → token no `Store` fake após `flush()` + navegação), 401 (mensagem, sem navegação, sem token). Usar `HttpTestingController` e `Store` fake.
  - [ ] 10.2 Teste do guard: com e sem token no `Store` fake, rota inicial resolve Home ou Login.
  - [ ] 10.3 Teste de segurança: senha não escrita no `Store`; header `Authorization` anexado quando há token.
  - _Requisitos: R6 (todos)_

- [ ] 11. Gate verde
  - Rodar `npm run typecheck` (zero erros) e `npm test` (todos os testes acima passam).
  - _Requisitos: R6 (critérios 1, 2)_
```
