# Контекст проекта (`glitch`)

Обновлено: 2026-05-25

## 1) Цель
Одностраничный лендинг IT-студии «Калининград PRO» на Alpine.js + Tailwind v4.
Периодические глитч-эффекты для элементов по `id`, а также hover-глитч для элементов по `class`.

## 2) Структура рабочей папки
- `index.html` — основная страница, стили и вся логика глитча в inline-скрипте.
- `alpine.js` — локальный рантайм Alpine.
- `browser@4.js` — браузерный рантайм Tailwind v4.
- `fonts/` — локальные шрифты: `InterVariable.woff2`, `Skolar-Regular.ttf`, `UbuntuMono-Regular.ttf`, `bebasneuecyrillic.ttf`.
- `images/` — фоны и персонажи: `bg1.png`, `bg4.png`, `img2.png`, `pers-03.png`, `pers-04.png`, `pers-05.png` и другие.

## 3) Текущая структура UI (`index.html`)

Внутри `body` (корневой `div#main`):

| Блок | id/класс | Описание |
|---|---|---|
| Навбар | — | sticky, `bg-neutral-800`; якоря: `#main`, `#stack`, `#process`, `#contacts` |
| Hero / main | — | `bg-neutral-800`; три span-заголовка с глитчем |
| — | `#title-1` | `span.title-b` «Сразу pro» |
| — | `#title-1.1` | `span.title-b.text-amber-200` «иску2стве2ный» |
| — | `#title-1.2` | `span.title-b` «#инте2лект.» |
| Цитата | — | `bg-neutral-800` + `bg4.png` + `pers-04.png`; цитата Канта |
| Стек | `#stack` | `bg-[#0000ff] text-[#55FFFF]`; таблица технологий, «не профиль» |
| Калининград | — | `bg-[#711313]` + `pers-03.png`; три строки «не в москве / не в минске / в калининграде» |
| Процесс | `#process` | `bg-amber-300`; аккордеон из 9 шагов (`acc-01` … `acc-09`) |
| Цены | — | `bg-[url(bg1.png)]` + `pers-05.png`; «От 10 000 ₽ / до долей / в бизнесе» |
| Контакты | `#contacts` | `bg-[#fafafa]`; заголовок «Контакты» |

Закомментированные элементы (в DOM отсутствуют, но есть в `glitchItems`): `#sub-1`, `#sub-2`, `#title-2`.

## 4) Текущее устройство движка глитча

Точка входа: `getData()` (Alpine `x-data`).

### Состояние

| Поле | Тип | Назначение |
|---|---|---|
| `defaultGlitchCharset` | string | Резервный набор символов (`'#%&?/\\*_+'`) |
| `defaultIntensity` | number | Резервная интенсивность (`0.3`) |
| `defaultHoverInterval` | number | Резервный интервал hover-глитча в мс (`80`) |
| `glitchItems` | object | Конфиг для каждого элемента по ключу (`id` или `class`) |
| `instances` | object | Реестр периодических глитч-инстансов |
| `hoverInstances` | array | Реестр hover-глитч-инстансов |
| `active_acc` | string\|null | ID открытой панели аккордеона (`'acc-01'` … `'acc-09'` или `null`) |

### Жизненный цикл

1. **`init()`**
   - Перебирает `glitchItems`.
   - Если `rawConfig.type === 'hover'` → вызывает `initHoverItems(key, rawConfig)`.
   - Иначе → находит элемент по `id`, сохраняет `originalText`, нормализует конфиг, регистрирует в `instances`, запускает `scheduleGlitch(key)`.

2. **`initHoverItems(key, rawConfig)`**
   - Выбирает все элементы по CSS-классу: `document.querySelectorAll('.' + key)`.
   - На каждый элемент вешает `mouseenter` / `mouseleave` / `mousedown`.
   - По `mouseenter` — `setInterval` с `buildGlitchText` каждые `hoverInterval` мс.
   - По `mouseleave` / `mousedown` — `clearInterval`, восстанавливает текст.
   - Регистрирует в `hoverInstances`.

3. **`scheduleGlitch(id)`**
   - Планирует следующее срабатывание со случайной задержкой в `[minDelay, maxDelay]`.

4. **`runGlitch(id)`**
   - `mode === 'alt'` → `pickAltText(config)` → случайная строка из `altTexts` (или `altText` как fallback).
   - `mode === 'random'` → `buildGlitchText(originalText, charset, intensity)`.
   - Возвращает исходный текст через `glitchDuration`.
   - Планирует следующий цикл.

5. **`destroy()`**
   - Чистит таймеры и восстанавливает тексты для `instances`.
   - Чистит интервалы и слушатели для `hoverInstances`.

### Вспомогательные методы

| Метод | Назначение |
|---|---|
| `normalizeConfig(config, originalText)` | Нормализует конфиг элемента, возвращает чистый объект под нужный `mode` |
| `toPositiveNumber(value, fallback)` | Валидирует число > 0, иначе возвращает `fallback` |
| `normalizeIntensity(value)` | Зажимает intensity в `[0.05, 1]`, fallback → `defaultIntensity` |
| `resolveCharset(charset)` | Возвращает charset или `defaultGlitchCharset` |
| `buildGlitchText(originalText, charset, intensity)` | Искажает случайные не-пробельные символы |
| `pickAltText(config)` | Выбирает случайную строку из `altTexts` или возвращает `altText` |
| `pickRandom(items)` | Случайный элемент массива |
| `randomBetween(min, max)` | Целое случайное число в `[min, max]` |

## 5) Контракт конфига элемента (`glitchItems`)

### Тип `hover` (выбор по классу)
| Поле | Описание |
|---|---|
| `type` | `"hover"` |
| `intensity` | 0..1 |
| `glitchCharset` | строка символов для подмены |
| `hoverInterval` | мс между заменами во время hover |

### Тип периодический (выбор по `id`)

Общие поля:
- `mode`: `"random"` или `"alt"`
- `minDelay`, `maxDelay` (мс)
- `glitchDuration` (мс)

Поля для режима `random`:
- `intensity` (0..1, с ограничением диапазона)
- `glitchCharset` (символы для подмены)

Поля для режима `alt`:
- `altTexts` (массив альтернативных строк)
- опционально `altText` как fallback (по умолчанию = `originalText`)

### Текущие зарегистрированные ключи

| Ключ | Тип | mode | Примечание |
|---|---|---|---|
| `title-1` | периодический | random | есть в DOM |
| `title-1.1` | периодический | random | есть в DOM |
| `title-1.2` | периодический | random | есть в DOM |
| `sub-1` | периодический | alt | **нет в DOM** (закомментирован) |
| `sub-2` | периодический | random | **нет в DOM** (закомментирован) |
| `title-2` | периодический | alt | **нет в DOM** (закомментирован) |
| `link` | hover | — | CSS-класс `.link` (навбар) |
| `stack-link` | hover | — | CSS-класс `.stack-link` (в DOM нет) |
| `process-link` | hover | — | CSS-класс `.process-link` (в DOM нет) |
| `button` | hover | — | CSS-класс `.button` |

Примечания:
- В режиме `random` длина результата равна длине исходного текста.
- В режиме `alt` длина результата может отличаться (зависит от выбранной альтернативы).
- Если элемент не найден в DOM, `init()` молча пропускает его (`if (!el) return`).
- Для hover-типа `querySelectorAll` по несуществующему классу просто вернёт пустой список — ошибок не будет.

## 6) Известные нюансы / заметки

- `sub-1`, `sub-2`, `title-2` зарегистрированы в `glitchItems`, но элементов в DOM нет — закомментированы в HTML.
- `stack-link` и `process-link` зарегистрированы в `glitchItems`, но классов в HTML пока нет.
- `sub-1.altTexts` и `title-2.altTexts` содержат тестовые placeholder-тексты — требуют замены перед релизом.
- В CSS подключаются все четыре шрифта; все четыре файла присутствуют в `fonts/`.
- IDE может показывать предупреждения `x-data is not allowed here` / `Unused function getData` — это шум статического анализа для Alpine в HTML.

## 7) Как обновлять этот файл контекста
При изменениях поведения обновляйте разделы:
1. **Текущая структура UI** (если менялись DOM-элементы/`id`/секции)
2. **Текущее устройство движка глитча** (если менялись функции/жизненный цикл)
3. **Контракт конфига** (если добавились/удалились поля или режимы)
4. **Известные нюансы** (новые предупреждения, отсутствующие ресурсы, особенности)

Рекомендуется вести мини-историю изменений внизу.

## 8) История изменений (вручную)
- 2026-05-22: Создан файл контекста.
- 2026-05-25: Полное обновление. Проект вырос из демо глитча в многосекционный лендинг. Добавлен hover-тип глитча (`type: 'hover'`, выбор по классу, `initHoverItems`). Состояние расширено: `defaultHoverInterval`, `hoverInstances`, `active_acc` (аккордеон на 9 шагов). Актуализированы структура UI, таблица ключей глитча и вспомогательные методы.
