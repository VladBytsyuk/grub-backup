# grub-backup

`grub-backup` — это Bash-утилита для создания, восстановления, просмотра и удаления резервных копий конфигурации GRUB по аналогии с `git stash`.

Проект предназначен для безопасного сохранения текущего состояния загрузчика перед изменениями конфигурации, обновлениями или ручной настройкой.

---

## Возможности

- Создание резервных копий текущей конфигурации GRUB
- Восстановление сохранённых конфигураций
- Просмотр списка доступных бэкапов
- Удаление одного или всех бэкапов
- Именованные и безымянные резервные копии
- Поддержка пользовательских директорий хранения
- Автоматический запуск `update-grub` после восстановления
- Полное тестирование через отдельный тестовый скрипт

---

## Какие файлы сохраняются

```text
/etc/default/grub
/etc/grub.d/
/boot/grub/grub.cfg
```

---

## Требования

* Ubuntu / Xubuntu / Debian-based Linux
* Bash 4+
* `tar`
* `update-grub`
* Root-права (`sudo`) для `save` и `restore`

---

## Установка

### Клонирование репозитория

```bash
git clone https://github.com/YOUR_USERNAME/grub-backup.git
cd grub-backup
```

### Установка в систему

```bash
sudo install -m 755 grub-backup /usr/local/bin/grub-backup
sudo install -m 755 test-grub-backup /usr/local/bin/test-grub-backup
```

---

## Использование

### Создать бэкап

```bash
sudo grub-backup save
```

### Создать именованный бэкап

```bash
sudo grub-backup save -m before-kernel-update
```

### Указать директорию хранения

```bash
sudo grub-backup save -p ~/grub-backups -m custom-backup
```

---

### Просмотреть список бэкапов

```bash
grub-backup list
```

```bash
grub-backup list -p ~/grub-backups
```

---

### Восстановить бэкап

```bash
sudo grub-backup restore -m before-kernel-update
```

```bash
sudo grub-backup restore -p ~/grub-backups -m custom-backup
```

---

### Удалить конкретный бэкап

```bash
grub-backup clear -m before-kernel-update
```

---

### Удалить все бэкапы

```bash
grub-backup clear --all
```

---

## Тестирование

### Быстрая проверка

```bash
./test-grub-backup ./grub-backup
```

### Полное тестирование (включая restore)

```bash
sudo ./test-grub-backup ./grub-backup --with-restore
```

---

## Безопасность

> **Важно:**
> Команда `restore` изменяет реальные системные файлы GRUB.
> Для безопасного тестирования рекомендуется использовать виртуальную машину.

---

## Пример рабочего процесса

```bash
sudo grub-backup save -m before-edit
sudo nano /etc/default/grub
sudo update-grub

# Если что-то пошло не так:
sudo grub-backup restore -m before-edit
```

---

## Лицензия

MIT License

Свободное использование, модификация и распространение разрешены.

---

## Почему этот проект полезен

Изменения GRUB могут привести к:

* невозможности загрузки системы;
* ошибкам меню загрузчика;
* конфликтам ядра;
* потере кастомных параметров.

`grub-backup` позволяет быстро откатиться к рабочему состоянию.

---

## Вклад в проект

Pull Requests, Issues и предложения приветствуются.


