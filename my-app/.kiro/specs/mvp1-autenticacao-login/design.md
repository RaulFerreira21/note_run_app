# Design — MVP 1: Autenticação / Login

**Spec:** `mvp1-autenticacao-login`
**Base:** `requirements.md` desta spec + `especificao_tecnica_note_run.md` (v1.1) + steering (`tech.md`).
**Depende de:** `mvp1-setup-ambiente` (router, estrutura, esqueleto de Login/Home).

## 1. Visão geral

Preenche a tela de Login com um formulário Signal Forms, um serviço de autenticação que fala com `POST /auth/login` via HttpClient nativo, persistência do JWT em `SecureStorage`, um guard de sessão no router e um interceptor que anexa `Authorization: Bearer`. Enquanto o back-end Go não existe, a chamada é atendida por um **stub trocável** que respeita o contrato — sem acoplar a tela ao mock.

## 2. Dependências a instalar

| Pacote | Motivo |
|--------|--------|
| `@angular/forms` (na versão exata do `@angular/core`) | Signal Forms |
| `expo-secure-store` (`npx expo install`) | `SecureStorage` para o JWT |

> HTTP nativo (`@ng-native/platform/http`) já faz parte do ecossistema instalado; não requer pacote novo.

## 3. Contrato da API (consumido pelo front)

```
POST {API_BASE_URL}/auth/login
Request:  { "email": string, "password": string }
200 OK:   { "token": string, "user": { "id": string, "email": string } }
401:      credenciais inválidas  (corpo de erro opcional, não exibido cru)
5xx/timeout/offline: erro recuperável
```

- `API_BASE_URL` vem de configuração de ambiente (mesmo mecanismo `.env` do setup; ex.: `EXPO_PUBLIC_API_BASE_URL` ou injeção via `app.config.js`), com valor de *mock* em dev.
- Formato/claims do JWT são opacos para o app nesta fase: ele guarda, anexa no header e (opcionalmente) checa expiração local se o `exp` estiver presente.

## 4. Camadas e arquivos

```
src/app/
├── core/
│   ├── routes.ts                 # (existente) + rota 'home' já protegida
│   ├── auth/
│   │   ├── auth.service.ts       # login(), logout(), token(), isAuthenticated()
│   │   ├── auth.api.ts           # AuthApi: contrato HTTP (token injetável p/ stub)
│   │   ├── auth.stub.ts          # Implementação mock do AuthApi (dev/sem back-end)
│   │   ├── auth.types.ts         # Credentials, AuthResult, User
│   │   ├── auth.guard.ts         # guard de sessão (canActivate)
│   │   ├── auth.interceptor.ts   # anexa Authorization: Bearer
│   │   └── auth.config.ts        # API_BASE_URL token + leitura de env
│   └── ...
├── login/
│   ├── login.ts                  # formulário real (substitui o esqueleto)
│   └── login.test.ts             # validação + sucesso + 401
└── home/
    └── home.ts                   # ganha ação mínima de logout
```

### 4.1. Tipos (`auth.types.ts`)
```ts
export interface Credentials { email: string; password: string; }
export interface User { id: string; email: string; }
export interface AuthResult { token: string; user: User; }
```

### 4.2. `AuthApi` + stub trocável
`AuthApi` é uma classe abstrata (ou `InjectionToken`) com `login(credentials): Promise<AuthResult>`. A implementação real usa `HttpClient`; o stub resolve localmente. A troca é por provider — a tela nunca sabe qual está ativa (Requisito 2.6).

```ts
// auth.api.ts (implementação HTTP real)
@Injectable()
export class HttpAuthApi implements AuthApi {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL);
  login(c: Credentials): Promise<AuthResult> {
    return firstValueFrom(
      this.http.post<AuthResult>(`${this.baseUrl}/auth/login`, c),
    );
  }
}
```

```ts
// auth.stub.ts (mock enquanto não há back-end Go)
@Injectable()
export class StubAuthApi implements AuthApi {
  async login({ email, password }: Credentials): Promise<AuthResult> {
    await delay(400);
    if (email === 'corredor@noterun.app' && password === 'senha123') {
      return { token: 'stub.jwt.token', user: { id: 'u1', email } };
    }
    throw new HttpErrorResponse({ status: 401 }); // mesmo formato que a API real
  }
}
```
Provider em dev escolhe `StubAuthApi`; ao subir o back-end, troca para `HttpAuthApi` sem tocar na tela.

### 4.3. `AuthService` (sessão)
```ts
@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly api = inject(AuthApi);
  private readonly secure = inject(SecureStorage);
  readonly token = this.secure.signal<string | null>('auth.token', null);
  readonly isAuthenticated = computed(() => !!this.token());

  async login(c: Credentials): Promise<void> {
    const { token } = await this.api.login(c);
    this.token.set(token);
    await this.secure.flush(); // garante persistência antes de navegar
  }
  async logout(): Promise<void> {
    await this.secure.remove('auth.token');
  }
}
```
- `token` é o signal persistido do `SecureStorage` — leitura síncrona possível no keychain, então a decisão de rota inicial no startup não "pisca" Login antes da Home (Requisito 3.2).
- A senha nunca é guardada (Requisito 5.1).

### 4.4. Guard e rota inicial (`auth.guard.ts`)
- `homeGuard` (`canActivate` em `/home`): se `!isAuthenticated()` → `UrlTree` para `''` (Login).
- `loginGuard` (`canActivate` em `''`): se `isAuthenticated()` → `UrlTree` para `/home`.
- Com isso a "rota inicial" vira condicional à sessão (Requisitos 3.2, 3.3, 4.2, 4.3); combina com o `reset()` do fluxo de login para não deixar Login no histórico.

### 4.5. Interceptor (`auth.interceptor.ts`)
- Anexa `Authorization: Bearer <token>` quando há token, preparando chamadas autenticadas das specs futuras (Requisito 5.4). Registrado via `withInterceptors([...])` no `provideNativeHttpClient()`.

## 5. Tela de Login (`login.ts`)

Substitui o esqueleto. Usa Signal Forms + `submit()`:

```ts
protected readonly data = signal<Credentials>({ email: '', password: '' });
protected readonly f = form(this.data, (path) => {
  required(path.email); email(path.email);
  required(path.password);
});
protected readonly authError = signal<string | null>(null);

protected async enter(): Promise<void> {
  this.authError.set(null);
  await submit(this.f, {
    action: async () => {
      try {
        await this.auth.login(this.data());
        await this.nav.reset(ROUTES.home);      // Requisito 4.1
      } catch (e) {
        this.authError.set(this.toMessage(e));  // 401 vs rede/servidor
      }
    },
    onInvalid: () => {},                          // erros já aparecem por campo
  });
}
```
- `<text-input>` de senha com `secureTextEntry`, e-mail com `keyboardType="email-address"` (Requisito 1.3).
- Botão desabilitado e com rótulo de carregando via `f().submitting()` (Requisito 2.5).
- `authError` distingue mensagem de credenciais inválidas (401) de erro recuperável (rede/5xx), sem vazar corpo cru (Requisitos 2.3, 2.4, 5.2).
- Estado inválido estilizado por `text-input[data-invalid][data-touched]`.

## 6. Mapeamento de erros

| Origem | Status | Mensagem ao usuário |
|--------|--------|---------------------|
| Credenciais inválidas | 401 | "E-mail ou senha incorretos." |
| Servidor | 5xx | "Erro no servidor. Tente novamente." |
| Rede/timeout | 0/offline | "Sem conexão. Tente novamente." |

Nenhuma mensagem inclui token, senha ou corpo bruto da resposta.

## 7. Configuração de ambiente

- `API_BASE_URL` é um `InjectionToken<string>` cujo factory lê `process.env.EXPO_PUBLIC_API_BASE_URL` (prefixo `EXPO_PUBLIC_` fica disponível em runtime). Documentado no `.env.example` junto da `GOOGLE_MAPS_API_KEY`.
- Flag de dev (ex.: ausência de `API_BASE_URL`) seleciona `StubAuthApi`.

## 8. Estratégia de teste

Node + `@ng-native/testing`, seguindo [Testing HttpClient](https://ng-native.com/packages/testing/testing-http.md) e o `Store` fake de storage.

- **Sucesso:** `HttpTestingController` responde 200 com `{ token, user }`; assert: token no `Store` fake (após `flush()`) e navegação para Home.
- **401:** controller responde 401; assert: mensagem "E-mail ou senha incorretos.", sem navegação, sem token persistido.
- **Validação:** submeter vazio/ e-mail inválido; assert: ação não chamada, campos marcados inválidos.
- **Guard:** render com `Store` fake com e sem token; assert: rota inicial resolve Home (com token) ou Login (sem token).
- **Segurança:** assert de que a senha não é escrita no `Store` e que o header `Authorization` é anexado pelo interceptor quando há token.

## 9. Riscos e decisões

- **Stub vs. back-end:** contrato fixado agora evita retrabalho; a troca é só de provider. Risco: contrato real divergir — mitigado por manter `auth.types.ts` como fonte única e revisar na spec do back-end.
- **Rota inicial sem piscar:** depende da leitura síncrona do keychain pelo `SecureStorage`; se em algum device vier assíncrona, usar um splash/guard de resolução para evitar flash do Login.
- **Expiração do JWT:** fora do escopo renovar token nesta fase; se `exp` existir e estiver vencido, tratar como não autenticado (logout local).

## 10. Rastreabilidade (requisito → design)

| Requisito | Onde é atendido |
|-----------|-----------------|
| R1 Formulário com validação | §5 |
| R2 Autenticação REST (mock) | §3, §4.2, §5 |
| R3 Persistência segura (JWT) | §4.3, §4.4 |
| R4 Navegação condicionada | §4.4, §5 |
| R5 Segurança de dados | §4.3, §4.5, §6, §7 |
| R6 Verificação automatizada | §8 |
