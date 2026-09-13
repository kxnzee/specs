## 1. `jitsi-control`

- [x] 1.1 Добавить `focus-region` в ответ создания конференции при непустом
      `jicofo.local-region` ([SC-CONFERENCE-REGION-DIAGNOSTICS-001]).
- [x] 1.2 Не добавлять свойство при отсутствующем или пустом значении
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-002]).

Проверка: backend-тесты покрывают заполненное, отсутствующее и пустое значения.

## 2. `lib-jitsi-meet`

- [x] 2.1 Разобрать непустое `focus-region` из ответа создания конференции и
      предоставить его через API конференции
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-003],
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]).
- [x] 2.2 Зафиксировать значение или его отсутствие по первому успешному ответу и
      не менять снимок при последующих ответах
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-005]).

Проверка: четыре browser-теста покрывают заполненное и пустое значение, сохранение
первого региона и сохранение первоначального отсутствия региона.

## 3. `jitsi-web`

- [x] 3.1 Добавить web-only label `Conference region: <value>` в существующую
      строку `ConferenceInfo` ([SC-CONFERENCE-REGION-DIAGNOSTICS-003]).
- [x] 3.2 Скрывать label при отсутствующем, пустом или некорректном значении без
      ошибки для участника ([SC-CONFERENCE-REGION-DIAGNOSTICS-004]).
- [x] 3.3 Получать подпись через i18n и передавать значение как обычный React text
      без интерактивности ([SC-CONFERENCE-REGION-DIAGNOSTICS-006]).
- [x] 3.4 Не менять мобильные компоненты
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-007]).
- [ ] 3.5 После выпуска lib-jitsi-meet с новым getter обновить pinned-версию в
      jitsi-web и повторить статические проверки
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-003],
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]).

Проверка: ESLint изменённых файлов, `tsc:web` и разбор изменённого JSON проходят.

## 4. Сквозная проверка

- [ ] 4.1 В запущенной конференции проверить показ региона при настроенном Jicofo
      и отсутствие label без ошибки при ненастроенном Jicofo
      ([SC-CONFERENCE-REGION-DIAGNOSTICS-001]—
      [SC-CONFERENCE-REGION-DIAGNOSTICS-004]).
