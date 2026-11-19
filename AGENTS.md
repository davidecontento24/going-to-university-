# Univers — coordinamento di due collaboratori (fase di accesso)

Le fonti di verità sono [direzione grafica](design/direzione-grafica.md),
[meccaniche di gioco](design/meccaniche-di-gioco.md) e la [tavola Fuori Aula](design/bozze-grafiche/fuori-aula.png).
Se il design non copre una scelta, chiedi al collaboratore umano di riferimento.

## Obiettivo di questa fase

Realizzare il percorso iniziale dalla cutscene alla lobby e al conto alla rovescia, con rete simulata.
La fase termina allo stub «Primo round». Minigiochi, server e punteggi sono fuori scope.

`intro` → `home` (Crea stanza / Entra) → `profilo` (nome e Univers) →
`lobby` (2–4 posti, link, Pronto) → `countdown` → stub «Primo round».

## Stack

- Vite e TypeScript, senza framework UI.
- Schermate in DOM, personaggi e scene in SVG inline.
- Animazioni con timeline GSAP.
- Nuove dipendenze solo se approvate nel PR.

## Divisione del lavoro

| Area | Collaboratore | Branch | Responsabilità |
| --- | --- | --- | --- |
| A — Fondamenta e personaggio | Secondo collaboratore | `feat/fondamenta` | `src/shared/`, `src/characters/`, scaffold e file dei contratti nel primo PR |
| B — Cutscene d'apertura | Secondo collaboratore | `feat/intro` | `src/screens/intro/`, `assets/intro/` |
| C — Flusso di accesso | Davide | `feat/accesso` | `src/app/`, `src/net/`, `src/screens/{home,profilo,lobby,countdown}/` |

### A — Fondamenta e personaggio

1. Primo PR circoscritto a scaffold Vite e TypeScript, script `npm run dev`, file dei contratti e `src/shared/tokens.css`. Dopo la review, ha priorità di merge.
2. Definire in `tokens.css` blu notte per contorni e testo, crema, terracotta, verde acqua, corallo e giallo. Includere tipografia, raggi e durate standard.
3. In `src/shared/motion.ts`, predisporre un helper per la gag in tre pose, con anticipazione, reazione e chiusura seguita da un fermo finale. Gestire `prefers-reduced-motion` e un toggle manuale.
4. In `src/characters/univers.ts`, disegnare gli Univers in SVG con contorno blu notte e ombre piatte. Le quattro sagome sono zaino grande, spalle larghe, gambe sottili e occhiali tondi, ciascuna con scelta del colore. Le pose sono attesa, movimento, azione, successo ed errore, più le tre di **Appello a sorpresa** (sicuro, scoperto e finge di leggere).
5. Preparare un bottone con area touch di almeno 48 px, icona e testo, un campo di testo e una card giocatore.

### B — Cutscene d'apertura (12–15 secondi)

Riprendere dalla tavola Fuori Aula portici terracotta, colonne crema, pavimento alla veneziana e bacheca. Mettere i dettagli ai bordi della scena.

1. Portici vuoti nella luce del mattino.
2. Entra la tutor con la cartellina dell'appello.
3. Un Univers si appoggia alla colonna, si stira dietro di essa quando vede la tutor e si siede con il libro. La gag si ferma sulla posa finale.
4. Altri due Univers spuntano dalla bacheca e da dietro un libro.
5. Entra la scritta **UNIVERS** su fondo crema con il comando «Tocca per entrare».

Tenere sempre visibile **Salta**. Dalla seconda visita saltare automaticamente l'intro tramite un flag in `localStorage` gestito con `try/catch`. Con movimento ridotto usare pose fisse e dissolvenze. L'audio è facoltativo. Fino al merge del personaggio usare placeholder che rispettano l'interfaccia `Univers`.

### C — Flusso di accesso

- **App.** In `src/app/`, la macchina a stati delle schermate gestisce tutte le transizioni.
- **Home.** Comandi **Crea stanza** ed **Entra**, con ingresso da link `?stanza=CODICE` o inserimento manuale del codice.
- **Profilo.** Inserimento del nome e scelta dell'Univers. Disabilitare le combinazioni già prese e marcarle con un simbolo oltre al colore.
- **Lobby.** Da due a quattro posti, link copiabile e stato **Pronto**. L'host avvia con almeno due giocatori pronti. Ogni ingresso attiva la posa `successo`.
- **Countdown.** Conto condiviso 3–2–1, poi stub «Primo round».
- **Rete.** In `src/net/`, predisporre `RoomService` e `MockRoomService`. I bot entrano dopo qualche secondo. Il server futuro sostituirà il mock mantenendo il contratto del servizio.

## Contratti

Il primo PR dell'area A inserisce i contratti nei percorsi qui indicati. Dopo il merge, per modificarli serve un PR dedicato concordato da entrambi i collaboratori.

```ts
// src/characters/univers.ts
export type Sagoma = 'zaino' | 'spalle' | 'gambe' | 'occhiali';
export type Posa = 'attesa' | 'movimento' | 'azione' | 'successo' | 'errore'
  | 'sicuro' | 'scoperto' | 'finge';

export interface Univers {
  el: SVGSVGElement;
  posa(p: Posa, opts?: { durata?: number }): Promise<void>;
}

export declare function createUnivers(o: { sagoma: Sagoma; colore: string }): Univers;
```

```ts
// src/net/room.ts
import type { Sagoma } from '../characters/univers';

export interface Giocatore {
  id: string;
  nome: string;
  sagoma: Sagoma;
  colore: string;
  pronto: boolean;
  host: boolean;
}

export interface RoomService {
  crea(): Promise<string>; // Ritorna il codice stanza.
  entra(codice: string): Promise<void>;
  aggiorna(p: Partial<Giocatore>): void;
  onChange(cb: (g: Giocatore[]) => void): () => void;
}
```

```ts
// src/app/screen.ts
import type { RoomService } from '../net/room';

export type ScreenId = 'intro' | 'home' | 'profilo' | 'lobby' | 'countdown';

export interface AppContext {
  goto(id: ScreenId): void;
  reducedMotion: boolean;
  room: RoomService;
}

export interface Screen {
  mount(root: HTMLElement, ctx: AppContext): void | Promise<void>;
  unmount(): void;
}
```

## Regole di lavoro

- Lavora nelle cartelle della tua area. Per intervenire su altre cartelle, apri un PR separato e coinvolgi il proprietario.
- Mantieni un solo obiettivo per PR. Fai rebase su `main` prima di aprirlo. Il merge su `main` richiede la review dell'altro collaboratore umano.
- Usa la tavola come riferimento e ridisegna gli asset in SVG, senza ricalchi raster.
- Scrivi i testi dell'interfaccia in italiano.
- Per ogni modifica all'app, prima di chiudere il task esegui `npm run build` e verifica il risultato a 360 px di larghezza e con movimento ridotto attivo.

## Ordine di lavoro

1. Apre per primo il PR di A con scaffold, contratti e token. L'altro collaboratore lo revisiona prima del merge.
2. Dopo il merge, il secondo collaboratore prosegue con personaggio e cutscene su PR distinti. In parallelo Davide sviluppa il flusso di accesso. Cutscene e flusso possono usare placeholder conformi ai contratti finché il personaggio è in lavorazione.
3. Entrambi verificano la demo completa dall'intro al countdown, con quattro giocatori mock e arrivo allo stub «Primo round».
