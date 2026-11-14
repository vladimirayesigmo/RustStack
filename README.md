# RustStack

[Русский](#русский) | [English](#english)

## Русский

Портативная утилита для игры Rust на Windows 10/11 x64. Написана на C++17 с
Win32, DirectX 11 и Dear ImGui. Не требует установки, .NET и внешних DLL:
распаковал папку - запустил `RustStack.exe`.

Умеет менять гамму, рисовать внешний прицел, редактировать бинды и графические
конфиги.

- Версия: beta 1 (2.0.0).
- Лицензия: [RustStack EULA](LICENSE) - закрытый код, бесплатное личное
  использование.
- Сторонние компоненты: [NOTICE.md](NOTICE.md).

### Безопасность

**Нет сети.** Запустите диспетчер задач → «Производительность» → «Монитор
ресурсов» → вкладка «Сеть». У `RustStack.exe` не должно быть ни одного
соединения. Приложение не содержит кода для работы с сетью.

**Нет служб, драйверов и задач планировщика.** Программа не устанавливается:
после закрытия и удаления папки от неё ничего не остаётся, кроме файлов,
которые она правила.

**Нет автозапуска по умолчанию.** Он добавляется только если вы сами включите
галочку - одна запись в `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`,
удаляется снятием галочки.

**Нет хуков и внедрения.** Горячие клавиши читаются через `GetAsyncKeyState`,
окно прицела - обычное прозрачное окно. В процесс Rust ничего не грузится,
память игры не читается.

**Отчёт VirusTotal:** [ссылка на скан](https://www.virustotal.com/gui/file/0bdf1233cf1eea11c79a174c653eb869d3e3bf6467c1423c7f563ac502b55012)

**Ложные срабатывания.** Неподписанный exe из небольшого проекта часто ругают
SmartScreen и антивирусы. Если Defender или другой антивирус что-то нашёл -
пришлите скриншот в Issues, файл будет отправлен в Microsoft на проверку.

#### Что программа меняет на диске

| Что | Где | Когда |
|---|---|---|
| Настройки | `settings.ini` рядом с exe | при изменении настроек |
| Бинды | `cfg/keys.cfg` + `.bak` | по кнопке сохранения |
| Графика | `client.cfg` + `.bak` | по кнопке сохранения |
| Параметры запуска | `localconfig.vdf` Steam + бэкап | по кнопке сохранения |
| Резервные копии CFG | `cfg_backup_<время>` рядом с exe | при очистке CFG |
| Карты | `*.map` удаляются без бэкапа | по кнопке очистки карт |
| Автозапуск | одна запись в `HKCU\...\Run` | только если включить |

Всё остальное - чтение путей Steam, названий CPU/GPU, объёма ОЗУ и разрешения
экрана через стандартные API Windows.

### Как это работает

#### Гамма

Яркость меняется через `SetDeviceGammaRamp` - стандартный API Windows, который
используют драйверы и программы калибровки цвета. Перед первым изменением
сохраняется исходная таблица гаммы, при штатном выходе она восстанавливается.
Кривая строится по формуле `pow(x, gamma)` и применяется ко всем мониторам.

Горячие клавиши читаются через `GetAsyncKeyState` в отдельном потоке. Глобальных
хуков на клавиатуру нет. В режиме «только в Rust» дополнительно проверяется, что
активное окно принадлежит `RustClient.exe` или `Rust.exe`.

#### Прицел

Отдельное прозрачное окно поверх всех окон:
`WS_EX_LAYERED | WS_EX_TOPMOST | WS_EX_TRANSPARENT`. Прозрачность работает по
цветовому ключу, клики проходят сквозь окно. Прицел рисуется через GDI+ по центру
клиентской области окна Rust; позиция проверяется раз в 200 мс.

#### Бинды

Программа читает и записывает `cfg/keys.cfg` как обычный текст. При сохранении
файл пересобирается из списка `bind`-строк, исходная версия копируется в `.bak`.
Программа не нажимает клавиши за игрока и не автоматизирует игровой процесс.

#### Графика

`client.cfg` редактируется построчно: меняются только выбранные параметры,
остальные строки и их порядок сохраняются. Пресеты лежат в `assets`. Параметры
запуска пишутся в `localconfig.vdf` Steam; кодировка, BOM и переводы строк
сохраняются, перед изменением делается резервная копия.

### Быстрый старт

1. Скачайте `RustStack-beta1.zip` из раздела Releases.
2. Распакуйте в любую папку.
3. Запустите `RustStack.exe`.

Содержимое архива:

```
RustStack.exe
README.md
LICENSE
NOTICE.md
licenses/
  Dear ImGui - MIT.txt
assets/
  commands.txt
  graphics_minimal.cfg
  graphics_medium.cfg
  graphics_high.cfg
  icon52.ico
  preview_desert.jpg
  preview_jungle.jpg
  preview_temperate.jpg
  preview_winter.jpg
  test.png
```

Картинки в `assets` необязательны: без них интерфейс показывает цветные заглушки.

Если SmartScreen предупреждает о неизвестном издателе: «Подробнее» → «Выполнить
в любом случае».

### Требования

- Windows 10/11 x64.
- Около 1,2 МБ на exe и 20-25 МБ оперативной памяти.
- Права администратора не нужны, кроме случаев, когда папка Rust или Steam
  защищена от записи.

### Ограничения

- Бинды в разработке: каталог команд неполный, часть команд может отсутствовать.
- На HDR-дисплеях Windows может игнорировать изменение gamma ramp.
- Интерфейс на русском и английском; переводы могут отставать от новых пунктов.

### Лицензия

Программа распространяется как **закрытое ПО**: исходный код не публикуется.

Разрешено: использовать собранное приложение для личного некоммерческого
использования и передавать оригинальный архив без изменений и без платы.

Запрещено: изменять, пересобирать и декомпилировать приложение, продавать его
или его сборки, включать в другие продукты, убирать уведомления об авторских
правах.

Полный текст - в файле [LICENSE](LICENSE). Dear ImGui распространяется по
лицензии MIT, см. `licenses/Dear ImGui - MIT.txt` и [NOTICE.md](NOTICE.md).

### Дисклеймер

RustStack - неофициальная утилита, не связанная с Facepunch Studios. Rust и
логотипы Rust являются товарными знаками Facepunch Studios.

Программа не читает память игры, не внедряется в процесс и не автоматизирует
игровой процесс. Тем не менее сторонние оверлеи и изменение гаммы могут нарушать
правила конкретного сервера или античита. Используйте на свой риск: возможна
блокировка игрового аккаунта.

## English

A portable utility for the game Rust that allows you to adjust gamma, display an
external crosshair, and edit key bindings as well as graphics configuration
files. Written in C++17 with Win32, DirectX 11 and Dear ImGui. No installation,
no .NET, no external DLLs: unpack the folder and run `RustStack.exe`.

It can change gamma, draw an external crosshair, edit binds and graphics configs.

- Version: beta 1 (2.0.0).
- License: [RustStack EULA](LICENSE) - closed source, free for personal use.
- Third-party components: [NOTICE.md](NOTICE.md).

### Security

**No network.** Open Task Manager → Performance → Resource Monitor → Network
tab. `RustStack.exe` must not have a single connection. The application contains
no network code.

**No services, drivers or scheduled tasks.** The app is not installed: after you
close it and delete the folder, nothing remains except the files it edited.

**No autostart by default.** It is added only if you enable the checkbox
yourself - a single value in
`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`, removed by clearing the
checkbox.

**No hooks, no injection.** Hotkeys are polled with `GetAsyncKeyState`; the
crosshair is a regular transparent window. Nothing is loaded into the Rust
process and game memory is not read.

**VirusTotal report:** [scan link](https://www.virustotal.com/gui/file/0bdf1233cf1eea11c79a174c653eb869d3e3bf6467c1423c7f563ac502b55012)

**False positives.** An unsigned exe from a small project often triggers
SmartScreen and antivirus warnings. If Defender or another antivirus flags it,
post a screenshot in Issues and the file will be submitted to Microsoft for
review.

#### What the app changes on disk

| Item | Location | When |
|---|---|---|
| Settings | `settings.ini` next to the exe | when settings change |
| Binds | `cfg/keys.cfg` + `.bak` | on save |
| Graphics | `client.cfg` + `.bak` | on save |
| Launch options | Steam `localconfig.vdf` + backup | on save |
| CFG backups | `cfg_backup_<time>` next to the exe | when clearing CFG |
| Maps | `*.map` deleted without backup | when clearing maps |
| Autostart | one value in `HKCU\...\Run` | only if enabled |

Everything else is reading Steam paths, CPU/GPU names, RAM size and screen
resolution through standard Windows APIs.

### How it works

#### Gamma

Brightness is changed through `SetDeviceGammaRamp` - a standard Windows API used
by drivers and color calibration tools. The original gamma table is saved before
the first change and restored on normal exit. The curve is built as
`pow(x, gamma)` and applied to all monitors.

Hotkeys are polled with `GetAsyncKeyState` in a separate thread. There are no
global keyboard hooks. In "Rust only" mode the app additionally checks that the
foreground window belongs to `RustClient.exe` or `Rust.exe`.

#### Crosshair

A separate transparent always-on-top window:
`WS_EX_LAYERED | WS_EX_TOPMOST | WS_EX_TRANSPARENT`. Transparency uses a color
key, so clicks pass through. The crosshair is drawn with GDI+ in the center of
the Rust client area; the position is checked every 200 ms.

#### Binds

The app reads and writes `cfg/keys.cfg` as plain text. On save the file is
rebuilt from the list of `bind` lines and the original is copied to `.bak`. The
app does not press keys for the player and does not automate gameplay.

#### Graphics

`client.cfg` is edited line by line: only the selected values change, all other
lines and their order are preserved. Presets live in `assets`. Launch options
are written to Steam `localconfig.vdf`; encoding, BOM and line endings are
preserved and a backup is made before writing.

### Quick start

1. Download `RustStack-beta1.zip` from Releases.
2. Unpack it into any folder.
3. Run `RustStack.exe`.

Archive contents:

```
RustStack.exe
README.md
LICENSE
NOTICE.md
licenses/
  Dear ImGui - MIT.txt
assets/
  commands.txt
  graphics_minimal.cfg
  graphics_medium.cfg
  graphics_high.cfg
  icon52.ico
  preview_desert.jpg
  preview_jungle.jpg
  preview_temperate.jpg
  preview_winter.jpg
  test.png
```

The images in `assets` are optional: without them the UI shows colored
placeholders.

If SmartScreen warns about an unknown publisher: "More info" → "Run anyway".

### Requirements

- Windows 10/11 x64.
- About 1.2 MB for the exe and 20-25 MB of RAM.
- Administrator rights are not required, except when the Rust or Steam folder is
  write-protected.

### Limitations

- Binds are a work in progress: the command catalog is incomplete and some
  commands may be missing.
- On HDR displays Windows may ignore gamma ramp changes.
- The UI is available in Russian and English; translations may lag behind new
  items.

### License

The app is distributed as **closed-source software**: the source code is not
published.

Allowed: use the compiled application for personal, non-commercial purposes and
share the original archive unchanged and free of charge.

Not allowed: modify, repack or decompile the app, sell it or its builds, include
it in other products, remove copyright notices.

The full text is in [LICENSE](LICENSE). Dear ImGui is distributed under the MIT
License, see `licenses/Dear ImGui - MIT.txt` and [NOTICE.md](NOTICE.md).

### Disclaimer

RustStack is an unofficial utility, not affiliated with Facepunch Studios. Rust
and the Rust logos are trademarks of Facepunch Studios.

The app does not read game memory, does not inject into the game process and
does not automate gameplay. Still, third-party overlays and gamma changes may
violate the rules of a specific server or anti-cheat. Use at your own risk: a
game account ban is possible.
