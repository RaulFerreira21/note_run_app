# Tarefas — MVP 1: GPS e Mapa na Home

**Spec:** `mvp1-gps-mapa-home`
**Base:** `requirements.md` e `design.md` desta spec.
**Pré-requisitos:** `mvp1-setup-ambiente` e `mvp1-autenticacao-login` concluídas.
**Como usar:** execute na ordem; marque `[x]` ao concluir. Cada tarefa referencia os requisitos que atende.

---

- [ ] 1. Registrar o mapa no startup
  - Em `src/main.ts`, chamar `registerExpoMap(Platform.OS)` (de `@ng-native/expo/map-view`) antes do `mount()`, junto de `registerPlatformComponents`.
  - _Requisitos: R3_

- [ ] 2. Criar o serviço/estado de permissão de localização
  - Criar um `Permission` para a localização foreground via `Permission.of(getForegroundPermissionsAsync, requestForegroundPermissionsAsync)` do `expo-location`, exposto para a Home ler `status`/`granted`/`blocked` e chamar `ensure()`.
  - Garantir criação em contexto de injeção (revalida ao voltar das Configurações).
  - _Requisitos: R1 (critérios 1, 4, 5)_

- [ ] 3. Implementar a máquina de estados da Home
  - Em `src/app/home/home.ts`, criar o `computed` `status()` com os estados: `permission-pending`, `permission-denied`, `permission-blocked`, `locating`, `ready`, `error` (derivado de permissão + posição + erro), usando signals.
  - _Requisitos: R4 (critérios 1, 3)_

- [ ] 4. Implementar o ciclo de vida da localização
  - [ ] 4.1 Injetar `Location` (`@ng-native/expo/location`), `injectPageFocused()` e `AppState`.
  - [ ] 4.2 `effect` que, quando a tela está em foco e o app ativo, garante a permissão (`ensure()`) e inicia `location.start({ accuracy: 'balanced', distance: 10 })`, parando via `onCleanup(stop)` ao perder foco/ir para background.
  - _Requisitos: R2 (critérios 1, 2, 4, 5)_

- [ ] 5. Renderizar o mapa centralizado
  - [ ] 5.1 Adicionar `<expo-map class="map">` com `[properties]="{ isMyLocationEnabled: true }"` e `class` com `flex: 1`.
  - [ ] 5.2 Centralizar a câmera na primeira posição recebida (setar `camera()` uma vez, zoom ~15), via `[cameraPosition]` e/ou `setCameraPosition(...)` imperativo.
  - [ ] 5.3 Garantir que a interação nativa (arrastar/zoom) funciona sem recentralizar a cada fix.
  - _Requisitos: R3 (critérios 1, 2, 3, 4)_

- [ ] 6. Implementar os estados visuais e tratamento de erro
  - [ ] 6.1 `locating`: indicador de carregamento até a primeira posição.
  - [ ] 6.2 `permission-denied`: explicação + botão "tentar de novo" (`ensure()`).
  - [ ] 6.3 `permission-blocked`: explicação + botão "Abrir Configurações" (`DeepLinks.openSettings()` de `@ng-native/device`).
  - [ ] 6.4 `error`: mensagem + retry.
  - [ ] 6.5 Chave Google Maps ausente/ inválida: mapa em branco tratado com aviso não-bloqueante (sem crash).
  - _Requisitos: R1 (critérios 2, 3), R2 (critério 3), R3 (critério 5), R4 (critérios 1, 2)_

- [ ] 7. Testes da lógica da Home
  - [ ] 7.1 Permissão concedida (fake `granted` + posição emitida) → passa por `locating` e chega a `ready`; `camera()` definido.
  - [ ] 7.2 Permissão negada (fake `denied`) → estado `permission-denied`; `start()` não inicia; botão de retry presente.
  - [ ] 7.3 Visibilidade: `injectPageFocused()` falso → acompanhamento parado (`stop` chamado via `onCleanup`).
  - _Requisitos: R5 (critério 1)_

- [ ] 8. Gate verde
  - Rodar `npm run typecheck` (zero erros) e `npm test` (testes da Home passam).
  - _Requisitos: R5 (critério 2)_

- [ ] 9. (Manual) Validar o mapa no development build Android
  - Com `GOOGLE_MAPS_API_KEY` no `.env`, rodar `npx expo run:android`, autenticar, conceder a permissão e confirmar: mapa renderiza, centraliza na posição atual, ponto do usuário visível e interação nativa funcionando.
  - _Observação:_ render nativo do `<expo-map>` não é coberto pelos testes em Node.
  - _Requisitos: R5 (critério 3)_
```
