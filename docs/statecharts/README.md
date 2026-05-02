# UI Statecharts

В этой папке хранятся PlantUML-диаграммы, описывающие навигацию и крупные режимы пользовательского интерфейса.

## Назначение

Диаграммы должны быть не только иллюстрациями, но и рабочими артефактами архитектуры. Они используются для:

- формализации переходов между экранами;
- обсуждения поведения интерфейса;
- ревью изменений в Git;
- будущей генерации state machine кода через StateSmith;
- предотвращения скрытой навигационной логики в LVGL callbacks.

## Стиль имён

Состояния: `PascalCase`

```text
HomeScanning
SignalDetails
HoldMonitor
```

События: `UPPER_SNAKE_CASE`

```text
EV_BTN_BACK
EV_SIGNAL_DETECTED
EV_TIMEOUT
```

Действия: `snake_case()`

```text
show_home_screen()
start_hold_mode()
request_video_preview()
```

## Файлы

- `ui_navigation_top_level_v1.plantuml` — верхнеуровневая навигация UI.

## Рекомендация

Не стоит сразу превращать одну диаграмму в огромную карту всего интерфейса. Лучше держать верхний уровень компактным, а сложные области выносить в отдельные диаграммы:

```text
ui_home_scanning_v1.plantuml
ui_signal_details_v1.plantuml
ui_settings_menu_v1.plantuml
ui_common_dialogs_v1.plantuml
```
