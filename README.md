# Лаб. П03 — Системы контроля версий на примере Git

Практика по git: локальный репозиторий, SSH-ключ для GitHub, push в новый репозиторий,
намеренный merge-конфликт и его разрешение, ветка (branch) с последующим merge.

## Стенд

Та же VM, что и в лабораторных П01/П02: Ubuntu Server 24.04.3 LTS в **Oracle VirtualBox**
(`devops-lab01`, 1 vCPU, ~2 ГБ RAM), сеть NAT, проброс `ssh` → `127.0.0.1:2223`.

Работа выполнена пользователем `andrey`. Аккаунт на GitHub — `Andreykosberg`
(уже зарегистрирован, использовался в П01/П02 для публикации репозиториев).

## Содержимое

| Файл/папка | Назначение |
|---|---|
| `screenshots/` | подтверждение работы — реальный вывод терминала |
| `README.md` | этот отчёт |

Сам ход работы (коммиты, merge-конфликт, ветка `FEAT-123`) зафиксирован не только на
скриншотах, но и в **реальной истории коммитов** репозитория
[`devops-lab03-git-intro`](https://github.com/Andreykosberg/devops-lab03-git-intro) —
туда пушил именно скрипт на ВМ по SSH, это не имитация.

## Задание и как оно выполнено

> 1. Зарегистрироваться на GitHub.
> 2. Добавить SSH-ключ или активировать 2FA.
> 3. Создать репозиторий на GitHub.
> 4. Создать локальный репозиторий с README.md, внести изменения.
> 5. Запушить на сервер.
> 6. Склонировать репозиторий преподавателя, создать файл с любым именем, дописать своё
>    имя и имя файла в файл `list`, закоммитить, запушить (разрешая конфликты по ходу),
>    получить права коллаборатора.

| # | Шаг | Как выполнено |
|---|---|---|
| 1 | Регистрация на GitHub | аккаунт `Andreykosberg` уже создан (использовался в П01/П02) |
| 2 | SSH-ключ для GitHub | на ВМ сгенерирован отдельный ключ `~/.ssh/github_ed25519` (`ssh-keygen -t ed25519`), публичная часть добавлена в GitHub → Settings → SSH and GPG keys — скриншот 1 |
| 3 | Создание репозитория на GitHub | `gh repo create devops-lab03-git-intro --public` (с хоста, от аккаунта `Andreykosberg`) — скриншот 3 |
| 4 | Локальный репозиторий + README + изменения | `git init`, `README.md`, два коммита (`Initial commit`, `Update README`) — скриншот 2 |
| 5 | Push на сервер | `git remote add origin git@github.com:...`, `git branch -M master`, `git push -u origin master` — скриншот 3 |
| 6 | Репозиторий преподавателя (clone + `list` + коллаборатор) | **не выполнено** — см. раздел ниже |

Дополнительно (по материалам лекции, слайды 42–49) показаны два сценария, которые прямо
разбирались на занятии и логично продолжают задание:

- **Реальный merge-конфликт и его разрешение** (вместо репозитория преподавателя, доступ
  к которому недоступен — см. ниже, тот же механизм конфликта воспроизведён на своём
  репозитории): "второй разработчик" (второй клон репозитория на хосте, тот же аккаунт)
  правит строку в `README.md` и пушит первым; на ВМ та же строка правится локально и
  коммитится независимо; `git push` отклоняется (`! [rejected] ... fetch first`); `git pull`
  действительно вызывает `CONFLICT (content): Merge conflict in README.md`; конфликт
  разрешён вручную (отредактирован файл, убраны маркеры `<<<<<<<` / `=======` / `>>>>>>>`,
  смысл обеих правок объединён), `git add` + `git commit` + `git push` — скриншот 4.
- **Ветка и merge** (слайды 48–49): `git checkout -b FEAT-123`, изменение, коммит,
  `git push -u origin FEAT-123` (создаёт ветку и на GitHub), `git checkout master`,
  `git merge FEAT-123` (fast-forward), `git push origin master` — скриншот 5.

## SSH через порт 443

Исходящий порт 22 к `github.com` с сети стенда оказался недоступен
(`kex_exchange_identification: Connection closed by remote host` / таймаут при обмене
баннерами — похоже на блокировку порта 22 на уровне сети/провайдера). Решение — штатный
способ, который сама GitHub документирует для таких случаев: туннелировать SSH через порт
443 (`ssh.github.com:443`), прописав это в `~/.ssh/config`:

```
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
```

После этого `ssh -T git@github.com` и все операции `git`/`push`/`pull` с `git@github.com:...`
работают прозрачно, без изменения команд — см. скриншот 1.

## Развёртывание (команды, выполненные на VM)

```bash
# git уже установлен в образе (2.43.0)
git config --global user.name  "Andrey Kosberg"
git config --global user.email "andrey.kosberg@example.com"

ssh-keygen -t ed25519 -C "andrey@devops-lab01-github" -f ~/.ssh/github_ed25519 -N ""
cat ~/.ssh/github_ed25519.pub        # добавлен в GitHub -> Settings -> SSH keys

mkdir ~/devops-lab03-git-intro && cd ~/devops-lab03-git-intro
git init
echo "# Lab 3: Git intro" > README.md
echo "Автор: Andrey" >> README.md
git add README.md && git commit -m "Initial commit: README"

echo "Домашнее задание: добавлена вторая строка в README." >> README.md
git add README.md && git commit -m "Update README: add description line"

git remote add origin git@github.com:Andreykosberg/devops-lab03-git-intro.git
git branch -M master
git push -u origin master

# --- демонстрация конфликта ---
# ("второй разработчик" на хосте уже запушил свою правку той же строки)
git commit -am "Edit README locally on VM (will conflict with remote)"
git push origin master                       # rejected (fetch first)
git config pull.rebase false
git pull origin master                        # CONFLICT (content): README.md
# вручную редактируем README.md, убираем <<<<<<< / ======= / >>>>>>>
git add README.md
git commit -m "Resolve merge conflict in README.md"
git push origin master                        # успешно

# --- демонстрация ветки ---
git checkout -b FEAT-123
echo "Новая функциональность: добавлена заметка про FEAT-123." >> README.md
git commit -am "FEAT-123: add feature note to README"
git push -u origin FEAT-123
git checkout master
git merge FEAT-123                            # fast-forward
git push origin master
```

Итоговый граф коммитов (`git log --oneline --graph --all`):

```
* 61e9a7f FEAT-123: add feature note to README
*   bfab44c Resolve merge conflict in README.md
|\
| * a8d9549 Update README from second clone (simulated colleague edit)
* | 17d82da Edit README locally on VM (will conflict with remote)
|/
* 2c47039 Update README: add description line
* b7e6eba Initial commit: README
```

Репозиторий, созданный и наполненный этой историей: <https://github.com/Andreykosberg/devops-lab03-git-intro>.

## Что отличается от лекции / не выполнено

- **Пункт 6 задания не выполнен**: склонировать репозиторий преподавателя
  (`https://github.com/vparanoid/xxxxxxxxx`), добавить файл, дописать себя в `list`,
  закоммитить/запушить, получить права коллаборатора. Ссылка на этот репозиторий
  была передана в Telegram-чате курса и недоступна в рамках выполнения этой работы
  (нет доступа к чату и к самому репозиторию/приглашению в коллабораторы). По
  согласованию было решено этот пункт пропустить, а механику "push → конфликт →
  разрешение", которая по сути и проверяется этим пунктом, продемонстрировать на
  собственном репозитории (см. раздел выше) — тот же набор команд и тот же результат
  (реальный `CONFLICT`, реальное разрешение, реальный push), разница только в том, что
  "удалённые" изменения имитирует второй клон того же репозитория вместо отдельного
  репозитория преподавателя.
- Вместо порта 22 для `git@github.com` используется проброс через порт 443
  (`ssh.github.com:443`) — порт 22 наружу из сети стенда недоступен (см. раздел выше).
  Это официально документированный способ GitHub для таких сетей, а не отклонение от
  требований задания.
- SSH-ключ для GitHub (`~/.ssh/github_ed25519`) — отдельный от ключей, использованных в
  П01/П02 для входа на саму VM; это два разных по смыслу ключа (логин на сервер vs.
  аутентификация перед GitHub), и лекция также говорит о них как о разных.

## Скриншоты

1. `screenshots/01-ssh-key-github.png` — `git config`, генерация SSH-ключа для GitHub,
   `~/.ssh/config` (туннель через порт 443), успешная аутентификация `ssh -T git@github.com`.
2. `screenshots/02-local-repo-init.png` — `git init`, создание `README.md`, два первых
   коммита, `git log --oneline`.
3. `screenshots/03-push-to-github.png` — создание репозитория на GitHub (`gh repo create`),
   `git remote add origin`, `git push -u origin master`.
4. `screenshots/04-merge-conflict.png` — реальный merge-конфликт: отклонённый `push`,
   `pull` с `CONFLICT (content)`, содержимое файла с маркерами конфликта, ручное
   разрешение, успешный `push` после коммита слияния.
5. `screenshots/05-branch-feat123.png` — ветка `FEAT-123`: коммит, `push -u origin
   FEAT-123`, `merge` в `master` (fast-forward), `push`, итоговый `git log --graph --all`.
