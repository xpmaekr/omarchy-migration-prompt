# Промт для другого ИИ: перенести окружение Omarchy на новый ПК

Скопируй этот текст целиком в другой ИИ (агент с доступом к shell на новом ПК).

---

Ты настраиваешь новый ПК с **Arch Linux + Omarchy (Hyprland, Wayland)**. Нужно повторить окружение со старой машины. Выполняй по шагам, проверяй каждый шаг, не спрашивай лишнего. Если чего-то не хватает — ставь/клонируй сам.

## 0. Проверка базы
```bash
omarchy --version || echo "Omarchy не установлена"
```
Если Omarchy нет — сначала поставь её по официальной инструкции (omarchy.org), потом продолжай. Нужен `yay` (AUR).

## 1. Установить программы
```bash
yay -S --needed --noconfirm \
  telegram-desktop-bin fastfetch lavat btop mpv mpv-mpris \
  obsidian chromium zen-browser-bin localsend moonlight-qt \
  visual-studio-code-bin virtualbox virtualbox-host-modules-arch \
  starship yazi eza bat fd ripgrep tldr tree 7zip \
  gpu-screen-recorder imv sushi pinta android-tools ddcutil \
  hyprpicker superwhisper-bin \
  ttf-ia-writer ttfx woff2-font-awesome
```
Примечания:
- `telegram-desktop-bin` и `lavat` — из AUR, это нормально.
- Приложения Omarchy (`hype`, `monologue`, `omacut`, `omasnap`, `omawrite`, `omacalc`, `tobi-try`, `cliamp`, `aether`, `gliff`, `disktree-bin`, `usage`, `herdr`) ставятся отдельно, если нужны: `yay -S --needed hype monologue omacut omasnap omawrite omacalc tobi-try cliamp aether gliff disktree-bin usage herdr`.
- Ретро-стек (retroarch + куча libretro-* ядер) — опционально, только если нужен эмулятор: `yay -S retroarch retroarch-joypad-autoconfig-git` и ядра по вкусу.
- Ввод корейского/японского (опционально): `fcitx5-chewing fcitx5-gtk fcitx5-hangul fcitx5-mozc fcitx5-table-extra`.

## 2. Темы Omarchy
На старом ПК лежат кастомные темы. Перенеси папки:
```
~/.config/omarchy/themes/luna-xp
~/.config/omarchy/themes/nothing
~/.config/omarchy/themes/usa
~/.config/omarchy/themes/waffle-cat
```
Команда (если копируешь с флешки/архива):
```bash
mkdir -p ~/.config/omarchy/themes
cp -r /путь/к/бэкапу/themes/* ~/.config/omarchy/themes/
omarchy theme list          # темы должны появиться в списке
omarchy theme set Vantablack   # текущая тема со старого ПК
```
Проверить: `omarchy theme current` → `Vantablack`.

## 3. Плагины Omarchy Shell
Из git:
```bash
omarchy plugin add https://github.com/Aicyraa/omarchy-workspace-gallery.git --enable --yes
omarchy plugin add https://github.com/diegopluna/omarchy-argus --enable --yes
# опционально, на старом ПК выключен:
# omarchy plugin add https://github.com/rosakodu/omarchy-dock.git --yes
```
Локальные (не из git) плагины скопируй папками в `~/.config/omarchy/plugins/`:
```
dimland-control
superwhisper-panel
```
и включи:
```bash
omarchy plugin enable dimland-control
omarchy plugin enable superwhisper.panel
omarchy plugin list
```
Ожидаемо включены third-party: `io.github.diegopluna.argus`, `dimland-control`, `superwhisper.panel`, `io.github.aicyraa.workspace-gallery`. `rosakodu.dock` — выключен.

## 4. «Желейная» анимация перехода рабочих столов
В конец `~/.config/hypr/looknfeel.lua` добавь:
```lua
-- Jelly workspace transitions: old workspace slides away, new one slides in
-- with a small spring bounce. Tune the feel with stiffness (speed) and
-- damping (lower = more bounce, ~critical = none).
hl.curve("jelly", { type = "spring", mass = 1, stiffness = 140, dampening = 16 })
hl.animation({ leaf = "workspaces", enabled = true, speed = 4, spring = "jelly", style = "slide" })
hl.animation({ leaf = "workspacesIn", enabled = true, speed = 4, spring = "jelly", style = "slide" })
hl.animation({ leaf = "workspacesOut", enabled = true, speed = 4, spring = "jelly", style = "slide" })
hl.animation({ leaf = "specialWorkspace", enabled = true, speed = 4, spring = "jelly", style = "slide" })
```
Затем:
```bash
hyprctl reload
hyprctl configerrors    # должно быть пусто
```
Проверить визуально: переключение рабочих столов (Super+1..9 / колесо) — старый уезжает, новый выезжает с пружинкой.

## 5. Клавиатурные бинды (`~/.config/hypr/bindings.lua`)
Замени содержимое файла (или добавь блок в конец):
```lua
-- File manager
o.bind("SUPER + E", "File manager", "omarchy launch nautilus cwd")

-- CLI tools in terminal. Replaces SUPER + V (universal paste) and SUPER + W (close window).
o.rebind("SUPER + V", "Vibe", "omarchy-launch-terminal vibe")
o.rebind("SUPER + W", "OpenCode", "omarchy-launch-terminal opencode")

-- Task manager (omarchy-task-manager): focus it if running, else launch.
o.bind("SUPER + ALT + DELETE", "Task manager", "omarchy-task-manager")

-- Open a terminal. Replaces SUPER + T (was: toggle floating/tiling).
o.rebind("SUPER + T", "Terminal", o.launch_terminal())

-- Blank the screen without stopping anything (DPMS off). Replaces SUPER + SHIFT + P
-- (was: Google Photos). Any key press or mouse movement turns the screen back on.
o.rebind("SUPER + SHIFT + P", "Blank screen", function()
  hl.timer(function()
    hl.dispatch(hl.dsp.dpms({ action = "disable" }))
  end, { timeout = 500, type = "oneshot" })
end)
```
Итог: `Super+E` — файловый менеджер, `Super+V` — Vibe, `Super+W` — OpenCode, `Super+Alt+Delete` — диспетчер задач, `Super+T` — терминал, `Super+Shift+P` — погасить экран (DPMS off, фоновые процессы не останавливаются), `Alt+Shift` — раскладка.

## 6. Раскладка: переключение Alt+Shift (`~/.config/hypr/input.lua`)
В секции `hl.config({ input = { ... } })` задай:
```lua
kb_options = "omarchy:shift_caps_compose,grp:alt_shift_toggle"
```
(по умолчанию Omarchy — Left Alt + Right Alt; здесь левый Alt + левый Shift).

## 7. Проверка
```bash
which telegram-desktop fastfetch lavat btop mpv
fastfetch | head -5
lavat -h | head -3
omarchy theme current
omarchy plugin list | grep third-party
hyprctl configerrors
```
Все команды должны отработать без ошибок, темы и плагины присутствовать, `configerrors` пустой.

Отчитайся коротко: что установлено, какие темы/плагины включены, пустой ли `configerrors`.
