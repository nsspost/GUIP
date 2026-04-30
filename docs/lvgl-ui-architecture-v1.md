# LVGL UI Architecture V1

Обновлено: 2026-04-30
Статус: рабочий архитектурный черновик

## Назначение

Этот документ нужен для перехода от экранных спецификаций и мокапов к реальной реализации интерфейса на `LVGL`.

Его задача:
- определить общую структуру UI;
- разложить экраны на компоненты;
- описать модели данных;
- зафиксировать границы между UI и логикой прибора;
- подготовить основу для кода без хаоса и дублирования.

## Главный Принцип Архитектуры

UI не должен знать детали алгоритмов поиска сигнала, обработки спектра или декодирования видео.

UI должен получать уже подготовленные данные в виде простых моделей:
- состояние прибора;
- состояние экрана;
- список сигналов;
- данные для графика;
- доступные действия.

То есть:
- backend прибора собирает и интерпретирует технические данные;
- UI только отображает их и отправляет назад пользовательские действия.

## Архитектурные Слои

Предлагаемая структура слоев:

1. `Platform layer`
2. `LVGL framework layer`
3. `UI design system layer`
4. `UI components layer`
5. `Screens layer`
6. `UI state / presenter layer`
7. `Application / device backend layer`

## 1. Platform Layer

Этот слой отвечает за:
- дисплей;
- тач, если появится;
- энкодер;
- кнопки;
- таймеры;
- аппаратные события;
- звук и вибрацию как исполняемые действия.

Типичные обязанности:
- инициализация `LVGL`;
- тик `lv_tick_inc`;
- flush framebuffer;
- input drivers для энкодера и кнопок.

UI-логика сюда не должна попадать.

## 2. LVGL Framework Layer

Этот слой содержит:
- базовую инициализацию `LVGL`;
- создание root screen;
- регистрацию тем и стилей;
- маршрутизацию переходов между экранами;
- общие utility-функции для работы с `LVGL`.

Примеры:
- `ui_init()`
- `ui_show_screen(screen_id)`
- `ui_apply_theme()`

## 3. UI Design System Layer

Этот слой должен хранить:
- токены;
- цвета;
- размеры;
- шрифты;
- базовые стили `lv_style_t`.

Этот слой связан с документом:
- [design-tokens-v1.md](C:/WORK/GUIP/docs/design-tokens-v1.md)

Что здесь должно жить:
- `ui_theme.h/.c`
- `ui_styles.h/.c`
- `ui_tokens.h`

## 4. UI Components Layer

Это переиспользуемые строительные блоки.

Нужные компоненты первой очереди:
- status bar
- screen header
- panel card
- bottom hints
- detection list item
- signal summary card
- small graph / activity graph
- modal confirm dialog
- settings row

### Почему Это Важно

Если сразу делать каждый экран руками целиком:
- быстро появится копипаст;
- статус-бар и подсказки расползутся по стилю;
- правки будут дорогими.

Если вынести общие блоки:
- все экраны будут собраны из одинаковых частей;
- стиль будет единым;
- обновлять UI станет проще.

## 5. Screens Layer

Это конкретные экраны, собранные из компонентов.

Для первой очереди:
- startup / selftest
- simple home
- detections list
- signal details
- hold monitor
- video view
- settings menu
- signal types settings
- alerts settings
- ignored signals
- spectrum pro
- service diagnostics

Каждый экран должен:
- принимать готовую модель данных;
- уметь перерисоваться при обновлении;
- отправлять наружу события от пользователя.

## 6. UI State / Presenter Layer

Это один из самых важных слоев.

Он нужен, чтобы:
- собрать данные backend в удобный вид для UI;
- преобразовать их в текст, метки, short status, флаги и доступные действия;
- не тащить в `LVGL`-экраны сырые низкоуровневые структуры.

Этот слой отвечает на вопросы:
- что должен показать экран;
- какие действия доступны сейчас;
- какой экран должен быть открыт;
- какие сигналы сортировать выше;
- как назвать состояние понятным пользователю текстом.

Именно здесь должны жить:
- сортировка сигналов для списка;
- выбор приоритетного сигнала;
- формирование коротких подписей;
- преобразование уровня сигнала в `растет/падает/стабилен`;
- решение, что показывать в `Simple`, а что только в `Pro`.

## 7. Application / Device Backend Layer

Это уже логика прибора:
- сканирование;
- детекция сигналов;
- классификация;
- история;
- правила игнорирования;
- видеопоток;
- системное состояние.

UI не должен напрямую читать регистры, буферы DMA или сырые диагностические счетчики.

UI должен получать уже нормализованные структуры.

## Предлагаемая Структура Папок

Один из практичных вариантов:

```text
ui/
  core/
    ui_init.c
    ui_nav.c
    ui_app_state.c
    ui_events.c
  theme/
    ui_tokens.h
    ui_styles.c
    ui_styles.h
    ui_fonts.h
  components/
    ui_status_bar.c
    ui_status_bar.h
    ui_screen_header.c
    ui_screen_header.h
    ui_panel_card.c
    ui_panel_card.h
    ui_bottom_hints.c
    ui_bottom_hints.h
    ui_detection_item.c
    ui_detection_item.h
    ui_small_graph.c
    ui_small_graph.h
    ui_confirm_dialog.c
    ui_confirm_dialog.h
    ui_settings_row.c
    ui_settings_row.h
  screens/
    scr_startup.c
    scr_startup.h
    scr_simple_home.c
    scr_simple_home.h
    scr_detections_list.c
    scr_detections_list.h
    scr_signal_details.c
    scr_signal_details.h
    scr_hold_monitor.c
    scr_hold_monitor.h
    scr_video_view.c
    scr_video_view.h
    scr_settings_menu.c
    scr_settings_menu.h
    scr_signal_types.c
    scr_signal_types.h
    scr_alerts.c
    scr_alerts.h
    scr_ignored.c
    scr_ignored.h
    scr_spectrum_pro.c
    scr_spectrum_pro.h
    scr_service.c
    scr_service.h
  models/
    ui_models.h
  presenters/
    prs_simple_home.c
    prs_detections_list.c
    prs_signal_details.c
    prs_hold_monitor.c
    prs_settings.c
```

## Модели Данных Для UI

Лучше сразу определить собственные UI-friendly структуры.

## Общие Типы

```c
typedef enum {
    UI_PROFILE_SIMPLE = 0,
    UI_PROFILE_PRO
} ui_profile_t;

typedef enum {
    UI_TREND_UP = 0,
    UI_TREND_STABLE,
    UI_TREND_DOWN
} ui_trend_t;

typedef enum {
    UI_SIGNAL_NEW = 0,
    UI_SIGNAL_ACTIVE,
    UI_SIGNAL_FADING,
    UI_SIGNAL_IGNORED
} ui_signal_state_t;
```

## Модель Статус-Бара

```c
typedef struct {
    char time_text[6];
    uint8_t battery_percent;
    bool sound_enabled;
    uint8_t volume_level;
    bool vibration_enabled;
    ui_profile_t profile;
    bool has_new_events;
} ui_status_bar_model_t;
```

## Модель Сигнала

```c
typedef struct {
    char type_name[32];
    char frequency_text[24];
    int16_t power_dbm;
    ui_trend_t trend;
    ui_signal_state_t state;
    char short_hint[32];
    bool can_show_video;
    bool can_hold_frequency;
    bool ignored;
} ui_signal_model_t;
```

## Модель Главного Экрана Simple

```c
#define UI_ACTIVITY_POINTS 32

typedef struct {
    ui_status_bar_model_t status;
    char title[24];
    char subtitle[32];
    char state_line[32];
    uint8_t activity_points[UI_ACTIVITY_POINTS];
    uint16_t detections_count;
    bool has_priority_signal;
    ui_signal_model_t priority_signal;
} ui_simple_home_model_t;
```

## Модель Списка Обнаружений

```c
#define UI_MAX_VISIBLE_SIGNALS 8

typedef struct {
    ui_status_bar_model_t status;
    uint16_t total_count;
    uint16_t new_count;
    uint16_t active_count;
    uint16_t selected_index;
    uint16_t visible_count;
    ui_signal_model_t items[UI_MAX_VISIBLE_SIGNALS];
} ui_detections_list_model_t;
```

## Модель Карточки Сигнала

```c
typedef struct {
    ui_status_bar_model_t status;
    ui_signal_model_t signal;
    char title[24];
    char note[48];
    bool show_hold_action;
    bool show_video_action;
    bool show_ignore_action;
} ui_signal_details_model_t;
```

## Модель Удержания На Частоте

```c
#define UI_HOLD_GRAPH_POINTS 48

typedef struct {
    ui_status_bar_model_t status;
    char signal_name[32];
    char frequency_text[24];
    int16_t power_dbm;
    ui_trend_t trend;
    char hold_time_text[12];
    uint8_t graph_points[UI_HOLD_GRAPH_POINTS];
} ui_hold_monitor_model_t;
```

## События От Пользователя

Нужен единый набор UI-событий.

Пример:

```c
typedef enum {
    UI_EVT_NONE = 0,
    UI_EVT_OK,
    UI_EVT_BACK,
    UI_EVT_FN,
    UI_EVT_ENC_LEFT,
    UI_EVT_ENC_RIGHT,
    UI_EVT_SCREEN_ENTER,
    UI_EVT_SCREEN_EXIT
} ui_input_event_t;
```

Но для нормальной архитектуры лучше иметь еще и более высокоуровневые действия:

```c
typedef enum {
    UI_ACTION_NONE = 0,
    UI_ACTION_OPEN_DETECTIONS,
    UI_ACTION_OPEN_SIGNAL,
    UI_ACTION_RETURN_TO_SCAN,
    UI_ACTION_HOLD_FREQUENCY,
    UI_ACTION_OPEN_VIDEO,
    UI_ACTION_IGNORE_SIGNAL,
    UI_ACTION_OPEN_SETTINGS,
    UI_ACTION_TOGGLE_MUTE,
    UI_ACTION_SWITCH_PROFILE
} ui_action_t;
```

Идея:
- экран принимает на вход низкоуровневые события;
- внутри или через presenter преобразует их в понятные `ui_action_t`;
- application слой уже решает, что делать дальше.

## Навигация Между Экранами

Нужен централизованный роутер экранов.

Пример идентификаторов:

```c
typedef enum {
    UI_SCREEN_STARTUP = 0,
    UI_SCREEN_SIMPLE_HOME,
    UI_SCREEN_DETECTIONS_LIST,
    UI_SCREEN_SIGNAL_DETAILS,
    UI_SCREEN_HOLD_MONITOR,
    UI_SCREEN_VIDEO_VIEW,
    UI_SCREEN_SETTINGS_MENU,
    UI_SCREEN_SIGNAL_TYPES,
    UI_SCREEN_ALERTS,
    UI_SCREEN_IGNORED,
    UI_SCREEN_SPECTRUM_PRO,
    UI_SCREEN_SERVICE
} ui_screen_id_t;
```

Что должен уметь роутер:
- открыть новый экран;
- вернуться назад;
- хранить текущий экран;
- при необходимости помнить выбранный сигнал между экранами.

## Контекст Выбранного Сигнала

Так как сценарий сильно крутится вокруг одного сигнала, нужен общий UI-контекст:

```c
typedef struct {
    int32_t selected_signal_id;
    ui_profile_t active_profile;
    ui_screen_id_t current_screen;
    bool mute_requested;
} ui_context_t;
```

Это позволит:
- не передавать все вручную через глобальные переменные;
- удерживать выбранный сигнал при переходе из списка в карточку и дальше в hold/video.

## Simple И Pro Режимы

Лучше считать это не двумя разными приложениями, а двумя профилями отображения.

Общий принцип:
- ядро данных одно;
- набор экранов частично общий;
- глубина и детализация зависят от профиля.

### Simple

Доступно:
- startup
- simple home
- detections list
- signal details
- hold monitor
- video view
- settings menu
- alerts
- signal types
- ignored

### Pro

Дополнительно:
- spectrum pro
- service diagnostics
- возможно более глубокие настройки и техстраницы

## Обновление Данных На Экранах

Не все экраны нужно обновлять одинаково часто.

### Частое Обновление

- simple home activity graph
- hold monitor graph
- spectrum pro
- status bar

### Среднее Обновление

- detections list
- signal details

### Редкое Обновление

- settings
- ignored
- service static fields

Практический совет:
- не пересоздавать экран при каждом обновлении;
- создавать виджеты один раз;
- потом обновлять только текст, выделение и графические точки.

## Жизненный Цикл Экрана

Для каждого экрана удобно иметь три фазы:
- create
- bind/update
- destroy/hide

Примерный стиль API:

```c
void scr_simple_home_create(lv_obj_t *parent);
void scr_simple_home_bind(const ui_simple_home_model_t *model);
ui_action_t scr_simple_home_handle_event(ui_input_event_t evt);
```

## Что Должно Быть Общими Компонентами В Первую Очередь

Если выбирать минимальный обязательный набор переиспользования:
- status bar
- bottom hints
- screen title/header
- panel card
- detection list item
- confirm dialog

Этого уже хватит, чтобы не развалиться по стилю на первых экранах.

## Минимальный Порядок Реализации

Лучший практический порядок:

1. Инициализация `LVGL` и theme layer.
2. `status bar` и `bottom hints`.
3. `simple home`.
4. `detections list`.
5. `signal details`.
6. `hold monitor`.
7. `settings menu`.
8. Остальные экраны.

Причина:
- это повторяет реальный основной пользовательский путь;
- дает ранний видимый результат;
- помогает быстрее проверить жизнеспособность архитектуры.

## Риски И Типичные Ошибки

### Ошибка 1

Экран напрямую знает о backend-структурах.

Почему плохо:
- UI становится жестко связан с логикой прибора;
- изменения в backend ломают экраны.

### Ошибка 2

Каждый экран рисуется вручную без общих компонентов.

Почему плохо:
- быстро появляется копипаст;
- сложно менять стиль единообразно.

### Ошибка 3

Перерисовка всего экрана на каждое обновление.

Почему плохо:
- лишняя нагрузка;
- мерцание;
- сложнее поддерживать плавность.

### Ошибка 4

Слишком раннее усложнение под `Pro`.

Почему плохо:
- можно потерять фокус на главном сценарии;
- `Simple` режим перестанет быть простым.

## Практический Итог

Если следовать этой архитектуре:
- мокапы превращаются в повторяемую систему;
- экраны можно реализовывать последовательно;
- `Simple` и `Pro` режимы остаются управляемыми;
- UI не тонет в низкоуровневой логике прибора.

## Следующий Логичный Шаг

Теперь уже можно делать один из двух очень практичных шагов:
- `implementation-plan-v1.md` с очередностью разработки файлов и задач;
- или сразу каркас папок/файлов в репозитории под будущую реализацию.

Если цель именно начать код, следующий лучший ход:
- подготовить `implementation-plan-v1.md`,
- а затем за один проход создать каркас `ui/` с пустыми файлами и header-структурой.
