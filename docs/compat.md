# Совместимость с версиями Plasticity

MCP работает с внутренностями Plasticity через отладочный протокол, поэтому любое обновление Plasticity может что-то сломать. Этот документ — процедура переноса на новую версию.

## Что проверено

Список проверенных версий — профили со статусом `supported` в `src/plasticity/compat/profiles/`. Сейчас это 26.1.3 (macOS, Apple Silicon).

## Как понять, что версия поменялась

- `plasticity_connect` и `plasticity_diagnose` возвращают поле `plasticity`: версию, статус (`supported` / `untested`) и профиль.
- При статусе `untested` в `warnings` есть предупреждение, а инструменты, которым не хватает фабрик, отвечают ошибкой с именем фабрики.

## Перенос на новую версию

1. До обновления сохраните установщик текущей версии Plasticity (чтобы можно было откатиться).
2. Установите новую версию и запустите `npm run compat:report -- --isolated`. Отчёт ляжет в `compat-reports/`.
3. Отсутствующие фабрики: найдите новое имя в разделе «New factories not used by this server» и добавьте псевдоним в новый профиль `src/plasticity/compat/profiles/<версия>.ts` (скопируйте предыдущий профиль, поменяйте `version`, зарегистрируйте его в `profiles/index.ts`).
4. Отсутствующие пути редактора: исправьте `editorPaths` в новом профиле (пути указываются от объекта редактора, через точку).
5. Повторяйте шаг 2, пока разделы «Missing …» не опустеют.
6. Прогоните `npm run test:live:full` и сравните с эталоном. Падения, не связанные с фабриками и путями, означают, что изменилось поведение Plasticity, — это отдельные задачи.
7. Поставьте новому профилю `status: "supported"`, обновите эталоны стенда (`--update-baseline`).

## Если не удаётся подключиться

- «Plasticity rejected the loopback CDP startup hook (Electron X)» — изменился запуск Electron. Смотрите `unlockMainProcess` в `src/plasticity/launcher.ts`.
- «Plasticity editor was not found (…)» — изменилась структура окна. Смотрите стратегии в `src/plasticity/compat/discovery.ts`; новую стратегию добавьте в список `STRATEGIES`.
