# Omarchy Migration Prompt

Готовый промт для ИИ-агента, чтобы перенести окружение **Arch Linux + Omarchy (Hyprland)** на новый ПК: темы, shell-плагины, «желейные» анимации рабочих столов, Telegram и повседневные программы (fastfetch, lavat, btop и т.д.).

## Использование

1. Открой [`prompt.md`](prompt.md).
2. Скопируй всё содержимое в ИИ-агента с доступом к shell на новом ПК.
3. Агент поставит пакеты, скопирует темы и плагины, включит анимацию и проверит результат.

## Что внутри

- список пакетов окружения
- кастомные темы Omarchy
- shell-плагины (git и локальные) с командами `omarchy plugin add/enable`
- Dimland Control — панель яркости с дополнительным затемнением (`dimland-git`)
- личные клавиатурные бинды (`bindings.lua`) и переключение раскладки Alt+Shift (`input.lua`)
- блок `hl.curve`/`hl.animation` для spring-анимации рабочих столов в `~/.config/hypr/looknfeel.lua`
- финальные проверки

## Лицензия

MIT — см. [LICENSE](LICENSE).
