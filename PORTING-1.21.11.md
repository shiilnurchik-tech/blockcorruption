# Back-port Immersive Portals → Minecraft 1.21.11

Статус-документ порта. Всё ниже — состояние на 2026-09-19; ни одна строка
источника в этой песочнице **не скомпилирована** (нет JDK/Gradle, maven-сервера
недоступны). Первая честная проверка — `./gradlew compileJava` на живой машине.

## Что vendored

- Источник: `eoNaho/ImmersivePortalsMod-fork`, ветка `port/26.1.2`,
  SHA `eae593cf6a188330f37d3991d067c9c152270980` (2026-07-12), лицензия Apache-2.0.
  Лежит в `libs/immersive-portals/` (вложенный `.git` удалён).
- Это самый продвинутый из существующих портов линии 26.x (настоящий fork
  оригинала с реальным рефакторингом рендер-миксинов под 26.1; коммиты вида
  «Refactor rendering logic for Minecraft 26.1 compatibility»).

## Почему нет апстрим-варианта (аудит 2026-09-19)

- Оригинальные релизы заканчиваются на `v6.0.6-mc1.21.1` (2024-11-29); оригинал
  архивирован в апреле 2026 (по docs/migration-26.1.2-plan.md самого форка).
- ~20 активных в 2026 форков — все либо 1.21.1, либо 26.1/26.1.2. Репозитории с
  «1.21.11» в имени внутри содержат 1.21.1 (`tavalwin-hue/...-1.21.11`,
  `minecraft_version=1.21.1` в gradle.properties).
- GitHub code search: `"minecraft_version=1.21.11" "immersive-portals"
  filename:gradle.properties` → **0 результатов**.
- Причина дыры: 1.21.11 лежит между двумя рендер-эпохами — уже имеет
  data-driven рендер 1.21.2+ (оригинал 6.0.6 не соберётся), но ещё НЕ имеет
  RenderPipeline/GpuBuffer/RenderPass/frame-graph из 26.1 (их использует этот
  vendored-источник). Сообщество перепрыгнуло с 1.21.1 на 26.x.

## Пины сборки и происхождение каждого

| Ключ | Значение | Источник факта |
|---|---|---|
| minecraft_version | 1.21.11 | intermediary/yarn: ветка `1.21.11` существует |
| mappings | `loom.officialMojangMappings()` | исходник написан в official-именах (порт 26.1 убрал Yarn); mojmap — единственный набор, с которым он компилируется без переименования 506 файлов |
| loader_version | 0.19.5 | FabricMC/fabric-loader latest (2026-08-28) |
| fabric_version | 0.141.6+1.21.11 | последний релиз линии FabricMC/fabric `+1.21.11` (2026-07-28) |
| loom | 1.17.13 | оставлен из форка |
| java | 21 | 1.21.11 = Java 21 (26.x требовал 25) |
| cloth_config | 21.11.153 | два независимых 1.21.11-мода (MayankIsDumb/Freecam, mmlo/ReadMyItem) |
| modmenu | 17.0.0 | два независимых пина (cattyngmd/captcha-craft, mmlo/ReadMyItem) |
| midnightlib | 1.9.3+1.21.11-fabric | cattyngmd/captcha-craft |
| sodium (compileOnly) | mc1.21.11-0.8.2-fabric | sidit77/better-mipmaps (и 0.8.0 у Trainguy9512/locomotion) |
| iris (compileOnly) | 1.10.0+1.21.11-fabric | Trainguy9512/locomotion |

Runtime sodium/iris выключены (`enable_sodium=false`, `enable_iris=false`):
ванильный рендерер — обязательный путь альфы (по матрице из
docs/migration-26.1.2-plan.md самого форка).

## Что уже сделано этим проходом

1. `gradle.properties`, `build.gradle`, `fabric.mod.json` перенацелены на 1.21.11
   (пины выше; `depends.minecraft=1.21.11`, `fabric-api>=0.141.6`,
   `fabricloader>=0.19.5`; из `breaks` убраны пины sodium/iris).
2. В `build.gradle` добавлен `mappings loom.officialMojangMappings()`;
   Java 25→21 (options.release, source/target, toolchain).
3. Аудит API-дельты (ниже).

## Аудит API-дельты 26.1.2 → 1.21.11

### A. Имена, уже присутствующие в 1.21.11 (обратных переименований НЕ нужно)

Источник уже использует новые имена; введения ниже попали в ваниль между 1.21.2
и 1.21.9, то есть раньше 1.21.11: `ResourceLocation→Identifier` (1.21.2),
`DimensionTransition→TeleportTransition` (1.21.2), пакет `renderer.fog.FogRenderer`
(1.21.2+), `renderer.rendertype.RenderType` (1.21.2+), `util.Util`,
`SavedDataStorage`+`ValueInput/ValueOutput` (линия 1.21.9, 25w31a — 5 файлов
персистенции остаются как есть). Подтверждено косвенно: массовое использование
этих импортов в экосистеме и наличие их в 1.21.11-модах. Точечная проверка на
живой сборке всё равно нужна.

### B. Хребет рендера 26.1 — главный ручной фронт (13 файлов)

Символы `RenderPipeline/GpuBuffer/GpuBufferSlice/RenderPass/FrameGraphBuilder`
отсутствуют в 1.21.11. Файлы:

- `core/mixin/client/render/MixinGameRenderer.java`
- `core/mixin/client/render/MixinLevelRenderer.java`
- `core/mixin/client/render/MixinLevelRenderer_BeforeIris.java`
- `core/mixin/client/render/MixinLevelRenderer_Optional.java`
- `core/mixin/client/render/shader/MixinShaderManager.java`
- `core/render/renderer/RendererUsingStencil.java`
- `core/render/FrontClipping.java`
- `core/render/MyRenderHelper.java`
- `core/render/ViewAreaRenderer.java`
- `core/peripheral/dim_stack/DimEntryWidget.java`
- + 3 файла sodium/iris-совместимости (скомпилируются против 1.21.11-банок,
  но возможны дрейфы символов sodium 0.8.2 vs 0.9.1 и iris 1.10 vs 1.11.2):
  `compat/mixin/sodium/MixinSodiumWorldRenderer.java`,
  `compat/iris_compatibility/IrisPortalRenderer.java`,
  `compat/iris_compatibility/ExperimentalIrisPortalRenderer.java`

Стратегия: алгоритмический референс — stencil-подход оригинала 6.0.6
(DigitalWolf1313/ImmersivePortalsMod-Updated, ветка 1.21), переписанный на API
эпохи 1.21.2–1.21.8: `BufferUploader.drawWithShader(MeshData)`,
`RenderSystem.setShader*`, stencil основного render target'а. Миксины
`LevelRenderer` целимся в сигнатуры 1.21.11 (не 1.21.1 и не 26.1).

### C. Чанк-тикеты (2 файла)

`TicketStorage` — новшество 26.1. В 1.21.11 старая схема
(`ServerChunkCache`/`DistanceManager`/`TicketType` с тип-параметром). Файлы:
`core/chunk_loading/ImmPtlChunkTickets.java`,
`core/commands/PortalDebugCommands.java`. Референс обратной переписки —
оригинал 6.0.6. Замечание из renames.json форка: «TicketType/Ticket losing their
type parameter» — в 26.1; значит в 1.21.11 тип-параметр ВЕРНУТЬ.

### D. Обзор при первом компиле (API предположительно стабильны с 1.21.2)

- `FogRenderer` — 7 файлов;
- `BufferBuilder/VertexConsumer` — 14 файлов;
- `GuiGraphics` — 10 файлов (включая NOT_A_RENAME-кейс
  `extractRenderState` из renames.json — проверить, существует ли он в 1.21.11;
  если это новшество 26.1 — откатывать к `render(GuiGraphics,...)`);
- Fabric API rename `ClientCommandManager→ClientCommands` — проверить по
  fabric-api 0.141.6.

## Локальный цикл сборки (следующий шаг)

```bash
cd libs/immersive-portals
./gradlew compileJava          # -Xmaxerrs 5000 уже выставлен в build.gradle
python3 migration_tools/parse_compile_errors.py < build/log.txt
```

Порядок чинки: метаданные/пины → точечные mojmap-откаты (B/C/D) → рендер-хребет
(B). Инструменты форка `find_candidates.py`/`inspect_class.py` работают против
javap — ими сверять каждую сомнительную сигнатуру вместо угадывания.

## Чего этот документ НЕ утверждает

- Что код компилируется. Не компилируется и не проверялся.
- Что рендер после переписки будет визуально корректен — это фаза 4 из
  docs/migration-26.1.2-plan.md, только для 1.21.11.
