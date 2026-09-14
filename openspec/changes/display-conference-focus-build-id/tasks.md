## 1. `jitsi-control`

- [ ] 1.1 Добавить публикацию `focus-build-id` в существующие свойства конференции
  через XMPP presence, используя существующий идентификатор сборки сервера
  (Design, решения 1–2). Точечный автоматический тест публичного контракта
  подтверждает передачу исходного значения и сохранение остальных свойств.
  Задача обеспечивает серверную часть Scenario
  `display-conference-focus-build-id-001`.

## 2. `lib-jitsi-meet`

- [ ] 2.1 Добавить точечные автоматические тесты существующей передачи свойств
  конференции без изменения рабочей логики (Design, решение 3). Проверить, что
  потребитель получает исходное значение `focus-build-id`, включая окружающие
  пробелы, а при сообщении без поля значение отсутствует и остальные свойства
  сохраняются. Задача обеспечивает передачу для Scenarios
  `display-conference-focus-build-id-001`, `display-conference-focus-build-id-002`,
  `display-conference-focus-build-id-003` и `display-conference-focus-build-id-006`.

## 3. `jitsi-web`

- [ ] 3.1 Добавить строку «Focus build id» в существующее представление Conference
  info, доступную по умолчанию всем участникам при наличии хотя бы одного
  непробельного символа в значении (Design, решение 4). Точечные тесты UI-компонента
  подтверждают отображение `build-2026.09` без изменения — Scenario
  `display-conference-focus-build-id-001`; сохранение окружающих пробелов
  в `"  build-2026.09  "` — Scenario `display-conference-focus-build-id-006`;
  одинаковую видимость для модератора и немодератора — Scenario
  `display-conference-focus-build-id-004`; использование существующего
  представления длинного значения без отдельного обрезания — Scenario
  `display-conference-focus-build-id-007` из Requirement
  «Long Focus build ids use the existing presentation».
- [ ] 3.2 Не показывать новую строку, заглушку или ошибку при отсутствующем,
  пустом или пробельном значении, сохраняя остальные строки Conference info
  (Design, раздел «Interaction / API contracts»). Точечные тесты UI-компонента
  подтверждают отсутствие значения — Scenario `display-conference-focus-build-id-002`;
  отдельно пустую строку и строку из пробелов — Scenario
  `display-conference-focus-build-id-005`; получение свойств без нового поля,
  как от сервера без его поддержки, — Scenario `display-conference-focus-build-id-003`.
