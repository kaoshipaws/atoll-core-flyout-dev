# Синхронизация форка с upstream

Этот гайд описывает как обновить `main` до актуального `upstream/main` и перенести свои изменения из ветки `atollcore` поверх него.

## Подготовка (один раз)

```powershell
git remote add upstream https://github.com/0x5bfa/DesktopFlyouts.git
```

---

## Шаг 1 — Сохрани точку отката

Перед любыми операциями запомни текущий SHA своей ветки:

```powershell
git log --oneline atollcore -1
# например: abc1234 мой коммит
```

Если что-то пойдёт не так — вернёшься сюда:

```powershell
git checkout atollcore
git reset --hard abc1234
git push origin atollcore --force
```

---

## Шаг 2 — Обнови upstream

```powershell
git fetch upstream
```

---

## Шаг 3 — Синхронизируй main

```powershell
git checkout main
git reset --hard upstream/main
git push origin main --force
```

---

## Шаг 4 — Перебазируй свои коммиты

```powershell
git checkout atollcore

# Посмотри граф чтобы найти SHA родителя твоего первого коммита
git log --oneline --graph atollcore main -15
```

Ищи точку расхождения: самый нижний коммит из твоих — это `<parent-sha>`.

```powershell
git rebase --onto main <parent-sha> atollcore
```

**Пример:** если граф выглядит так:
```
* abc1234 (atollcore) — мой коммит
* def5678             — старый лишний коммит
* 7dd9a1a (main)      — upstream
```
то `<parent-sha>` = `def5678`.

---

## Шаг 5 — Разрешение конфликтов (если есть)

```powershell
# Git покажет какие файлы конфликтуют
git status

# Открой файл, найди маркеры <<<<<<, реши вручную.
# Затем:
git add <файл>
git rebase --continue

# Если хочешь отменить всё и вернуться к исходному состоянию:
git rebase --abort
```

---

## Шаг 6 — Запуши

```powershell
git push origin atollcore --force
```

---

## Быстрая проверка результата

```powershell
git log --oneline --graph atollcore main -10
```

Должно выглядеть так:

```
* xxxxxxx (atollcore) — твой коммит
* yyyyyyy (main, upstream/main) — последний upstream коммит
```

---

## Правило большого пальца

Чем меньше своих коммитов поверх upstream — тем проще rebase.
Старайся держать **один squash-коммит** со всеми своими изменениями:

```powershell
# Объединить несколько своих коммитов в один
git rebase -i main
# В редакторе: первый `pick`, остальные `squash`
```
