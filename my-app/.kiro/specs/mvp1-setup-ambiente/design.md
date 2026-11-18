# Design — MVP 1: Setup do Ambiente

**Spec:** `mvp1-setup-ambiente`
**Base:** `requirements.md` desta spec + `especificao_tecnica_note_run.md` (v1.1) + steering (`tech.md`, `structure.md`)
**Plataforma-alvo:** Android (iOS adiado, mas mantido resgatável)

## 1. Visão geral

O setup transforma o template atual (um contador de exemplo em `src/app/app.ts`) em um app com **shell de navegação nativa** e duas telas esqueleto — **Login** (rota inicial) e **Home** (com área reservada para o mapa). Instala e declara os módulos nativos que as specs 2 e 3 vão usar (`react-native-screens`, `expo-location`, `expo-maps`), configura a chave da Google Maps do Android a partir de uma fonte não versionada, e deixa `typecheck` e `test` verdes com um teste de navegação Login → Home.

Nada de lógica de autenticação, permissão de GPS ou renderização de mapa entra aqui — apenas a fundação.

## 2. Dependências a instalar

| Pacote | Motivo | Observação |
|--------|--------|-----------|
| `@ng-native/router`, `@angular/router` | Navegação nativa | Shell desta spec |
| `react-native-screens` | Lado nativo das stacks/telas | `npx expo install`; sem ele o *dev build* renderiza `Unimplemented component` |
| `expo-location` | GPS (spec 3) | Instalado agora para o *prebuild* já linká-lo |
| `expo-maps` | Mapa (spec 3) | Instalado agora; *alpha*, não roda em Expo Go |

> Instalar módulos nativos agora (mesmo que usados só nas specs 2/3) garante que o *development build* gerado neste setup já contenha tudo, evitando um rebuild a cada spec.

Comandos:
```sh
npm install @ng-native/router @angular/router
npx expo install react-native-screens expo-location expo-maps
```

## 3. Configuração nativa (Android)

### 3.1. Chave da Google Maps fora do versionamento
`app.json` é estático e não lê variáveis de ambiente. Para injetar a chave sem versioná-la, adiciono um **`app.config.js`** que importa o `app.json` e sobrescreve o campo da chave a partir de `process.env` (o Expo CLI carrega arquivos `.env` automaticamente para o `app.config`).

```js
// app.config.js
const base = require('./app.json');

module.exports = () => ({
  ...base.expo,
  android: {
    ...base.expo.android,
    config: {
      googleMaps: { apiKey: process.env.GOOGLE_MAPS_API_KEY ?? '' },
    },
  },
  plugins: [
    ...(base.expo.plugins ?? []),
    [
      'expo-maps',
      {
        requestLocationPermission: true,
        locationPermission: 'Permitir que o Note Run mostre sua posição no mapa',
      },
    ],
  ],
});
```

- `.env` e variações entram no `.gitignore`; um **`.env.example`** versionado documenta a variável `GOOGLE_MAPS_API_KEY`.
- O `expo-maps` config plugin escreve as permissões de localização no manifesto Android (não precisamos editá-lo à mão).
- `expo-location` adiciona `ACCESS_COARSE_LOCATION`/`ACCESS_FINE_LOCATION` pelo próprio config plugin.

### 3.2. `app.json`
- Mantém `platforms: ["ios", "android"]` como está — **não removemos iOS** para mantê-lo resgatável (Requisito 1, critério 4). Apenas não configuramos nem buildamos iOS nesta fase.
- `scheme: "myapp"` já existe e serve para deep links depois.

### 3.3. Development build
- Fluxo de desenvolvimento passa a ser `npx expo run:android` (não Expo Go).
- README/AGENTS são atualizados para registrar: mapa exige *development build* + `GOOGLE_MAPS_API_KEY` no `.env`.

## 4. Estrutura de pastas

Segue a steering (`structure.md`): um componente por pasta, template/estilos inline, teste `*.test.ts` ao lado.

```
src/
├── main.ts                     # Passa a prover o router nativo (provideNativeRouter)
└── app/
    ├── app.ts                  # Shell raiz: <native-stack-outlet />
    ├── app.routes.ts           # Routes: '' → Login, 'home' → Home
    ├── app.test.ts             # Teste de navegação Login → Home (substitui o do contador)
    ├── core/
    │   └── routes.ts           # Constantes de caminho de rota (local comum compartilhado)
    ├── login/
    │   ├── login.ts            # Tela esqueleto de Login (rota inicial)
    │   └── login.test.ts       # Teste do esqueleto de Login
    └── home/
        └── home.ts             # Tela esqueleto de Home (placeholder do mapa)
```

- `core/` nasce como o "local comum" exigido pelo Requisito 3. Seed inicial: `routes.ts` com os caminhos nomeados, consumidos pelo `app.routes.ts` e pela navegação (evita strings mágicas e já serve as specs seguintes).

## 5. Navegação

### 5.1. Rotas (`app.routes.ts`)
```ts
import type { Routes } from '@angular/router';
import { Login } from './login/login.ts';
import { Home } from './home/home.ts';

export const routes: Routes = [
  { path: '', component: Login },      // rota inicial
  { path: 'home', component: Home },
];
```
(Rotas diretas por ora; `loadComponent` pode vir depois se o bundle crescer.)

### 5.2. Shell raiz (`app.ts`)
Substitui o contador. O host é a stack (`:host { flex: 1 }`):
```ts
import { Component } from '@angular/core';
import { SafeAreaProvider } from '@ng-native/components';
import { NativeStackOutlet } from '@ng-native/router';

@Component({
  selector: 'app-root',
  imports: [NativeStackOutlet, SafeAreaProvider],
  template: `
    <safe-area-provider>
      <native-stack-outlet />
    </safe-area-provider>
  `,
  styles: `:host { flex: 1; }`,
})
export class App {}
```

### 5.3. Bootstrap (`main.ts`)
Mantém a fiação existente de `mount()` (conditions, tokens, processColor, resolveAssetSource) e adiciona o provider do router:
```ts
providers: [provideNativeRouter(routes, withComponentInputBinding())],
```
- `withComponentInputBinding()` entra já agora (Requisito 4, critério 4), para params chegarem como inputs nas specs seguintes — e vale nos testes também.

### 5.4. Fluxo Login → Home
A tela de Login expõe uma ação placeholder "Entrar". No esqueleto ela navega com `NativeNavigation.reset('/home')` — `reset` porque, no fluxo real de login, não se deve voltar para a tela de login via gesto/back. Modelar isso já agora evita retrabalho na spec 2.

```ts
// login.ts (essência)
import { Component, inject } from '@angular/core';
import { Pressable, Text, View } from '@ng-native/components';
import { NativeNavigation } from '@ng-native/router';
import { ROUTES } from '../core/routes.ts';

@Component({
  selector: 'app-login',
  imports: [Pressable, Text, View],
  template: `
    <view class="screen">
      <text class="title">Note Run</text>
      <pressable accessibilityRole="button" (press)="enter()">
        <text>Entrar</text>
      </pressable>
    </view>
  `,
  styles: `/* ... */`,
})
export class Login {
  private readonly nav = inject(NativeNavigation);
  protected enter(): void {
    void this.nav.reset(ROUTES.home);
  }
}
```

A Home esqueleto apenas reserva a área do mapa:
```ts
// home.ts (essência)
template: `
  <safe-area-view class="screen">
    <text class="title">Home</text>
    <view class="map-placeholder" accessibilityLabel="Área reservada para o mapa" />
  </safe-area-view>
`
```

## 6. Estratégia de teste

Testes rodam em Node com `@ng-native/testing` (sem simulador), seguindo o padrão de [Testing the router](https://ng-native.com/packages/testing/testing-navigation.md).

- **`app.test.ts` (navegação):** renderiza `App` com o router (rotas + `withComponentInputBinding()`), confirma que o Login aparece, aciona "Entrar" via `userEvent.press(...)`, e verifica (`findBy*`) que a Home aparece.
- **`login.test.ts`:** renderiza o Login isolado e verifica título + botão "Entrar" (via `getByRole('button')`).
- **`app.test.ts` do contador:** substituído pelo teste de navegação (Requisito 6, critério 3).
- **Gate verde:** `npm run typecheck` e `npm test` concluem sem erro ao fim da spec.

> O teste de navegação valida apenas o roteamento do esqueleto — não há lógica de login/GPS/mapa para testar aqui.

## 7. Impacto e riscos

- **Mudança de fluxo de dev:** Expo Go deixa de bastar; quem clonar precisa de *dev build* + `.env`. Mitigado pela atualização do README/AGENTS.
- **`expo-maps` alpha:** fixar a versão resolvida no `package-lock.json`; mudanças de SDK podem exigir ajuste na spec 3 (não neste setup).
- **Chave ausente:** sem `GOOGLE_MAPS_API_KEY`, o build compila mas o mapa (spec 3) fica em branco. Documentado no `.env.example`.
- **iOS adiado:** nenhuma configuração Android-only irreversível; retomar iOS = configurar Apple Maps + `expo run:ios`, sem desfazer o que foi feito aqui.

## 8. Rastreabilidade (requisito → design)

| Requisito | Onde é atendido |
|-----------|-----------------|
| R1 Dev build (Android) | §2, §3.2, §3.3 |
| R2 Chave Google Maps | §3.1 |
| R3 Estrutura de pastas | §4 |
| R4 Navegação nativa | §5.1–5.3 |
| R5 Esqueletos de tela | §5.4 |
| R6 Verificação automatizada | §6 |
