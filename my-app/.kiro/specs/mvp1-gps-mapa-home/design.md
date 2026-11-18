# Design — MVP 1: GPS e Mapa na Home

**Spec:** `mvp1-gps-mapa-home`
**Base:** `requirements.md` desta spec + `especificao_tecnica_note_run.md` (v1.1) + steering (`tech.md`).
**Depende de:** `mvp1-setup-ambiente` e `mvp1-autenticacao-login`.
**Plataforma-alvo:** Android (iOS adiado).

## 1. Visão geral

A Home passa a orquestrar uma máquina de estados pequena — **permissão → posição → mapa** — usando o serviço `Location` (que já pede a permissão de foreground e expõe a posição como signal), o `<expo-map>` centralizado na posição, e `injectPageFocused()` para parar o GPS quando a tela não está visível. O registro do `<expo-map>` acontece no startup (`registerExpoMap`).

## 2. Dependências

Nenhuma nova: `expo-maps` e `expo-location` foram instalados na spec 1. Esta spec apenas os usa.

- Mapa: `MapView`/`<expo-map>` de `@ng-native/expo/map-view`.
- Localização: `Location` de `@ng-native/expo/location`.
- Visibilidade de tela: `injectPageFocused()` de `@ng-native/router`; `AppState` de `@ng-native/device` para o background.
- Permissão (estado/UX): o próprio `Location` pede a permissão; para `blocked` → Configurações, usar `DeepLinks.openSettings()` de `@ng-native/device`.

## 3. Registro do mapa (startup)

Em `main.ts`, antes do `mount()` (junto de `registerPlatformComponents`):
```ts
import { Platform } from 'react-native';
import { registerExpoMap } from '@ng-native/expo/map-view';
registerExpoMap(Platform.OS as 'ios' | 'android');
```

## 4. Máquina de estados da Home

Estado derivado de signals (sem zone.js):

```
'permission-pending'  → permissão ainda não resolvida
'permission-denied'   → negada (oferece pedir de novo)
'permission-blocked'  → bloqueada (oferece abrir Configurações)
'locating'            → permitida, sem primeira posição ainda
'ready'               → há posição; mapa centralizado
'error'               → falha de localização (permite retry)
```

```ts
protected readonly status = computed<HomeStatus>(() => {
  const perm = this.locationPermission.status();     // Permission sobre expo-location
  if (perm === 'denied' && this.permBlocked()) return 'permission-blocked';
  if (perm === 'denied') return 'permission-denied';
  if (perm === 'unknown' || perm === 'undetermined') return 'permission-pending';
  if (this.error()) return 'error';
  return this.location.position() ? 'ready' : 'locating';
});
```

> O serviço `Location` já pede a permissão em `current()`/`start()`. Mantemos também um `Permission` explícito (`Permission.of(getForegroundPermissionsAsync, requestForegroundPermissionsAsync)` de `expo-location`) para **ler status/blocked** e reagir à volta das Configurações — ele, por ser criado em contexto de injeção, revalida um "não" quando o app volta ao primeiro plano (Requisito 1.5).

## 5. Ciclo de vida da localização

- Ao ter permissão e a Home visível, iniciar acompanhamento contínuo:
  `this.stop = await this.location.start({ accuracy: 'balanced', distance: 10 });`
  (`balanced`/`high` — não `navigation`, para não manter o GPS quente à toa — Requisito 2.5).
- Parar quando a tela não está em foco ou o app vai para background, via `effect` com `injectPageFocused()` + `AppState.active()` (Requisito 2.4):

```ts
constructor() {
  effect(async (onCleanup) => {
    if (!this.focused() || !this.appState.active()) return;
    if (!(await this.locationPermission.ensure())) return;   // pede/garante permissão
    const stop = await this.location.start({ accuracy: 'balanced', distance: 10 });
    onCleanup(stop);                                         // para ao sair/again background
  });
}
```

## 6. Mapa e câmera

- `<expo-map>` ocupa a área principal (`class="map"` com `flex: 1`) — sem tamanho, o mapa não aparece (Requisito 3.1).
- `properties.isMyLocationEnabled = true` para o ponto nativo do usuário (Requisito 3.3).
- Centralização: ao chegar a **primeira** posição, mover a câmera uma vez (não a cada fix, para não brigar com a interação do usuário):

```ts
@Component({
  selector: 'app-home',
  imports: [MapView, /* View, Text, Pressable, ActivityIndicator, SafeAreaView */],
  template: `
    @switch (status()) {
      @case ('ready') {
        <expo-map class="map"
          [cameraPosition]="camera()"
          [properties]="{ isMyLocationEnabled: true }" />
      }
      @case ('locating')           { <!-- spinner --> }
      @case ('permission-pending') { <!-- pedindo permissão --> }
      @case ('permission-denied')  { <!-- explica + botão tentar de novo --> }
      @case ('permission-blocked') { <!-- explica + botão abrir Configurações --> }
      @case ('error')              { <!-- mensagem + retry --> }
    }
  `,
})
export class Home {
  private readonly map = viewChild(MapView);
  protected readonly camera = signal<{ coordinates: MapCoordinates; zoom: number } | null>(null);
  // ao primeiro position(), setar camera() uma vez com zoom ~15
}
```
- Centralização da câmera via `[cameraPosition]` no primeiro fix, e/ou `map().setCameraPosition(...)` imperativo (chamadas antes do mapa aparecer são enfileiradas) (Requisito 3.2).
- Interação nativa (arrastar/zoom) é do próprio `<expo-map>` (Requisito 3.4).

## 7. Erros e degradação

| Situação | Tratamento |
|----------|-----------|
| Permissão negada | estado `permission-denied`, botão "tentar de novo" (`ensure()`) |
| Permissão bloqueada | estado `permission-blocked`, botão "Abrir Configurações" (`DeepLinks.openSettings()`) |
| Chave Google Maps ausente/ inválida | mapa em branco é esperado; mostrar aviso não-bloqueante; não crashar (Requisito 3.5) |
| Falha de localização | estado `error` + retry (Requisito 4.2) |

## 8. Estratégia de teste

Node + `@ng-native/testing`, com fakes injetados (o `Location`/`Permission` respondem como a doc descreve quando sem fake; para cenários determinísticos, prover stubs):

- **Permissão concedida:** fake de permissão `granted` + `Location` que emite uma posição → status passa por `locating` e chega a `ready`; `camera()` definido.
- **Permissão negada:** fake `denied` → status `permission-denied`; botão de tentar novamente presente; `start()` não inicia.
- **Visibilidade:** simular `injectPageFocused()` falso → acompanhamento não ativo (o `onCleanup` chamou o `stop`).
- **Mapa nativo:** render do `<expo-map>` em si é **verificação manual no dev build Android** (Requisito 5.3), documentada; os testes em Node cobrem a lógica de estado, não o pixel do mapa.

## 9. Riscos e decisões

- **`expo-maps` alpha:** API pode mudar entre SDKs; manter versão fixada (vindo do setup).
- **Centralizar uma vez vs. seguir o usuário:** optei por centralizar no primeiro fix e deixar o usuário navegar; "recentralizar" pode ser um botão futuro — fora do escopo.
- **Dupla fonte de permissão:** `Location` pede sozinho, mas usamos `Permission` para ler `blocked`/reagir à volta das Configurações. Manter a leitura de status no `Permission` como fonte da UI evita estados inconsistentes.
- **Bateria:** `accuracy: 'balanced'` + parar fora de foco atende ao MVP; rastreamento de treino (precisão alta) é outra spec.

## 10. Rastreabilidade (requisito → design)

| Requisito | Onde é atendido |
|-----------|-----------------|
| R1 Permissão (pedir/negar/bloquear/voltar) | §4, §5, §7 |
| R2 Posição atual + tempo real + ciclo de vida | §5 |
| R3 Mapa centralizado e interativo | §3, §6 |
| R4 Estados da Home | §4, §6, §7 |
| R5 Verificação automatizada | §8 |
