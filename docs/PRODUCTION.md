# NEXORA — Производство (Фаза 3)

> Карта милстоунов + атомарные промпты для Cursor. Правила: D-088…D-091.
> Один промпт = одна проверяемая фича. DoD: тесты зелёные + deploy + curl URL.

## Лестница

| M | Что | DoD |
|---|---|---|
| M0 | Каркас (Vite+React+TS, Vitest, Vercel) | curl URL → 200, "NEXORA" |
| M1 | Детерминированный движок (чистый TS, без внешних зависимостей) | сценарные тесты зелёные |
| M2 | Парс-пайплайн (edge: классификатор → LLM → структура v1 → хэш → кэш кластеров) | curl edge с фразой → валидный JSON |
| M3 | «Мир отвечает»: поле ввода + прекаст на живом URL | магический момент воспроизводим |
| M4 | Дух: дуэль против ИИ-наставника, полный кор-луп | 10 дуэлей играбельны |
| — | **ГЕЙТ ФАНА**: 10 дуэлей → хочется 11-ю? Нет → стоп проекта | честный ответ Женьчика |
| M5 | Трёхпанельный стрип из лога (D-086) | реплей рендерится детерминированно |
| M6 | Дуэли в Supabase: RLS-одновременность, коммит-по-хэшу, RPC-наблюдение (D-071/D-077) | негативные тесты RLS зелёные |
| M7 | Асинхрон: matchmade + инбокс-переписка + web-push | две учётки играют с двух устройств |
| M8 | Инициация: онбординг-сцена, гости, time-to-magic | секундомер < 60 сек |
| M9 | Безмолвие: Чернила-лимиты, ворота модерации, Лаборатория | предсезон запускаем |

Летопись/печати/роялти/восхищения — НЕ здесь: включаются событием Открытия Гримуара после Безмолвия.

## Промпты M0–M1 (копипаста в Cursor, дословно)

### P-001 (M0)
```
Scaffold a Vite + React + TypeScript project for a web game called NEXORA.
Add: Tailwind, Vitest, ESLint. Single page rendering "NEXORA" centered on
dark background (#0a0a0f). Add vercel.json for SPA. Write README.md with
one-line dev/build/test commands. Do not add any game logic yet.
Commit: "chore: scaffold vite react ts + vitest + tailwind".
```
DoD: `npm test` проходит (пустой набор), Vercel deploy, curl URL → 200 + "NEXORA".

### P-002 (M1)
```
Read docs/COMBAT-CORE.md and docs/ARCHITECTURE.md first.
Create src/engine/types.ts implementing spell structure v1 exactly as
specified (D-059): closed enums for verb (10), substrate (6), target (5),
magnitude (4), duration (4); aspects as string[]; optional rider with
closed trigger enum (4); safety, confidence, v fields. No numeric fields.
Create src/engine/config.ts: cost base table, mana curve (start 6, +3/turn,
cap 12), resolve/Стойкость 20, duration/rider taxes, fatigue step — all
marked "// Q-08 hypothesis, tune in Silence".
Write vitest tests asserting enum completeness and config shape.
Commit: "feat(engine): structure v1 types + hypothesis config".
```
DoD: тесты зелёные; типы 1:1 со спекой (сверка глазами по COMBAT-CORE).

### P-003 (M1)
```
Read docs/COMBAT-CORE.md (cost formula D-082).
Implement src/engine/cost.ts: canonical cost formula in exact modifier
order: ceil(base[verb][magnitude] * affinity(aspects, arena)) +
duration_tax + rider_tax + erosion_step + fatigue_step. Erosion and
fatigue passed in as inputs (engine is pure). Return cost breakdown
object for precast display. Vitest: 10+ cases including affinity
discounts/penalties and fatigue repeats (+1 class per repeat).
Commit: "feat(engine): canonical cost formula with breakdown".
```
DoD: тесты зелёные, разложение цены возвращается.

### P-004 (M1)
```
Read docs/COMBAT-CORE.md (layers D-054, ordinal arithmetic D-068,
anchors D-069, state D-066/067).
Implement src/engine/resolve.ts: pure function
(stateA, stateB, structA, structB, arena) -> {newState, log}.
Fixed layer order: word > space > ward > transmute > bind/veil >
strike/drain/infuse > riders > duration tick. Ordinal rules: ward absorbs
<= its magnitude on its own substrate channel, higher penetrates by
difference; dispel removes <= its magnitude; bind blocks its substrate's
verbs. Anchored zone effects miss if target left anchor (flight).
Manifested entities: barrier = ward for both, hazard = conditional strike.
No RNG anywhere. Log entries: layer, actor, outcome, one-line reason.
Vitest scenario tests from docs: (1) glass-under-feet vs fly-up = miss +
attacker mana spent; (2) heavy strike vs medium ward = light damage
through; (3) mind-bind reduces opponent word limit state 12->8;
(4) repeat same cluster = +1 class cost via fatigue input;
(5) mutual lethal = draw. 25+ tests total.
Commit: "feat(engine): layered deterministic resolve + scenario tests".
```
DoD: все сценарные тесты зелёные — дизайн-документ стал исполняемым.

### P-005 (M1)
```
Implement src/engine/narrate.ts: deterministic English template narration
from resolve log (D-065): one short line per significant layer event, e.g.
"Space outpaces matter — the glass seizes empty ground." Template map by
(layer, outcome); aspect words injected from a primitives library
(unknown aspect -> "arcane"). No LLM. Vitest: narration snapshots for the
5 scenario tests. Commit: "feat(engine): template narration".
```
DoD: снапшоты стабильны при повторных прогонах.

## Правила выдачи следующих промптов

Промпты M2+ выдаются пачками после закрытия текущего милстоуна (реальность M1 уточнит M2). Запрос: «промпты M2».
