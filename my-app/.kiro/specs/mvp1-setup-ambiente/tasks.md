# Tarefas — MVP 1: Setup do Ambiente

**Spec:** `mvp1-setup-ambiente`
**Base:** `requirements.md` e `design.md` desta spec.
**Como usar:** execute as tarefas na ordem. Cada uma referencia os requisitos que atende. Marque `[x]` ao concluir.

---

- [ ] 1. Instalar dependências de navegação e módulos nativos
  - Instalar `@ng-native/router` e `@angular/router` via `npm install`.
  - Instalar `react-native-screens`, `expo-location` e `expo-maps` via `npx expo install` (versões compatíveis com o SDK do Expo).
  - Confirmar que as versões resolvidas ficaram fixadas no `package-lock.json`.
  - _Requisitos: R1, R2_

- [ ] 2. Configurar a chave da Google Maps fora do versionamento
  - [ ] 2.1 Criar `app.config.js` que importa `app.json`, injeta `android.config.googleMaps.apiKey` a partir de `process.env.GOOGLE_MAPS_API_KEY` e adiciona o plugin `expo-maps` (`requestLocationPermission: true` + string de uso de localização).
  - [ ] 2.2 Criar `.env.example` documentando `GOOGLE_MAPS_API_KEY=` e adicionar `.env`/`.env.*` ao `.gitignore` (mantendo `.env.example` versionado).
  - [ ] 2.3 Verificar que `npx expo config --type public` reflete o plugin e o campo da chave sem expor valor real.
  - _Requisitos: R2_

- [ ] 3. Criar a estrutura base de pastas e as constantes de rota
  - Criar `src/app/core/routes.ts` exportando `ROUTES` com os caminhos nomeados (`login: ''`, `home: '/home'`).
  - Criar as pastas `src/app/login/` e `src/app/home/` (arquivos nas tarefas 6 e 7).
  - _Requisitos: R3_

- [ ] 4. Definir as rotas e o shell raiz de navegação
  - [ ] 4.1 Criar `src/app/app.routes.ts` com o `Routes` array (`'' → Login`, `'home' → Home`), usando `ROUTES`.
  - [ ] 4.2 Reescrever `src/app/app.ts` como shell: `<safe-area-provider>` com `<native-stack-outlet />`, `:host { flex: 1 }`, removendo o contador de exemplo.
  - _Requisitos: R3 (critério 4), R4_

- [ ] 5. Prover o router nativo no bootstrap
  - Editar `src/main.ts` para adicionar `providers: [provideNativeRouter(routes, withComponentInputBinding())]` ao `mount()`, mantendo a fiação existente (conditions, tokens, processColor, resolveAssetSource).
  - _Requisitos: R4_

- [ ] 6. Implementar a tela esqueleto de Login (rota inicial)
  - Criar `src/app/login/login.ts` (`selector: app-login`) com título e um `<pressable>` "Entrar" (`accessibilityRole="button"`) que chama `NativeNavigation.reset(ROUTES.home)`.
  - Usar elementos minúsculos de `@ng-native/components`, eventos nativos e estado em signals conforme a steering.
  - _Requisitos: R4 (critério 3), R5_

- [ ] 7. Implementar a tela esqueleto de Home
  - Criar `src/app/home/home.ts` (`selector: app-home`) com `<safe-area-view>`, título e uma `<view>` placeholder do mapa (`accessibilityLabel` indicando a área reservada).
  - _Requisitos: R5_

- [ ] 8. Testes do esqueleto e substituição do teste de exemplo
  - [ ] 8.1 Substituir `src/app/app.test.ts` por um teste de navegação: renderiza `App` com o router, confirma a tela de Login, aciona "Entrar" com `userEvent.press(...)` e verifica (`findBy*`) que a Home aparece.
  - [ ] 8.2 Criar `src/app/login/login.test.ts` que renderiza o Login isolado e verifica título e o botão "Entrar" via `getByRole('button')`.
  - _Requisitos: R5, R6 (critérios 2 e 3)_

- [ ] 9. Atualizar a documentação do projeto
  - Atualizar `README.md` e `AGENTS.md`: fluxo de dev passa a ser `npx expo run:android` (Expo Go não renderiza o mapa), e é necessário `GOOGLE_MAPS_API_KEY` no `.env`.
  - Registrar que o MVP 1 é Android-first.
  - _Requisitos: R1 (critério 3), R2 (critério 4)_

- [ ] 10. Verificação automatizada (gate verde)
  - Rodar `npm run typecheck` e garantir zero erros.
  - Rodar `npm test` e garantir que o teste de navegação e o de Login passam.
  - _Requisitos: R6 (critérios 1 e 2)_

- [ ] 11. (Manual) Validar o development build no Android
  - Com `GOOGLE_MAPS_API_KEY` no `.env`, rodar `npx expo run:android` em emulador/dispositivo e confirmar que o app abre no Login e navega para a Home.
  - _Observação:_ passo manual (exige ambiente Android/emulador); não é coberto pelos testes em Node.
  - _Requisitos: R1 (critérios 1 e 2)_
```
