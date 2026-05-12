#!/usr/bin/env bash
#
# grub-backup
# Простая утилита для сохранения, восстановления, просмотра и удаления
# бэкапов конфигурации GRUB по аналогии с git stash.
#
# Что сохраняется:
#   - /etc/default/grub
#   - /etc/grub.d/
#   - /boot/grub/grub.cfg
#
# После restore автоматически выполняется update-grub.
#
# Использование:
#   grub-backup save    [-p DIR] [-m MSG]
#   grub-backup restore [-p DIR] [-m MSG]
#   grub-backup list    [-p DIR]
#   grub-backup clear   [-p DIR] [-m MSG] [-a]
#

set -euo pipefail

BACKUP_ROOT="."
MESSAGE=""
ALL=false

UNNAMED="__unnamed__"

die() {
  echo "Ошибка: $*" >&2
  exit 1
}

usage() {
  cat <<EOF
Использование:
  grub-backup save    [-p DIR] [-m MSG]
  grub-backup restore [-p DIR] [-m MSG]
  grub-backup list    [-p DIR]
  grub-backup clear   [-p DIR] [-m MSG] [-a]

Команды:
  save      Создать бэкап текущей конфигурации GRUB
  restore   Восстановить последний подходящий бэкап
  list      Показать список бэкапов
  clear     Удалить бэкап или бэкапы

Опции:
  -p, --path DIR       Папка для хранения или чтения бэкапов
  -m, --message MSG    Имя бэкапа
  -a, --all            Удалить все бэкапы в папке
  -h, --help           Показать справку
EOF
}

require_root() {
  # Изменение файлов GRUB требует root-прав.
  if [[ "${EUID}" -ne 0 ]]; then
    die "команда требует root-прав. Запустите через sudo."
  fi
}

safe_message() {
  # Преобразуем unsafe-символы в "_"
  local msg="${1:-$UNNAMED}"

  # Все символы кроме букв, цифр, точки, подчеркивания и дефиса заменяются
  # через встроенную подстановку Bash.
  msg="${msg//[^a-zA-Z0-9._-]/_}"

  echo "$msg"
}

backup_dir() {
  echo "$BACKUP_ROOT/.grub-backup"
}

find_latest_backup() {
  local msg_safe
  msg_safe="$(safe_message "$MESSAGE")"

  find "$(backup_dir)" \
    -maxdepth 1 \
    -type f \
    -name "grub-backup-*-${msg_safe}.tar.gz" \
    | sort \
    | tail -n 1
}

cmd_save() {
  require_root

  local dir msg_safe timestamp archive meta
  dir="$(backup_dir)"
  msg_safe="$(safe_message "$MESSAGE")"
  timestamp="$(date '+%Y%m%d-%H%M%S')"
  archive="$dir/grub-backup-${timestamp}-${msg_safe}.tar.gz"
  meta="$(mktemp)"

  mkdir -p "$dir"

  # Manifest нужен для удобного просмотра списка бэкапов.
  cat > "$meta" <<EOF
created_at=$timestamp
message=${MESSAGE:-}
hostname=$(hostname)
EOF

  tar -czf "$archive" \
    -C / \
    etc/default/grub \
    etc/grub.d \
    boot/grub/grub.cfg \
    -C "$(dirname "$meta")" "$(basename "$meta")"

  rm -f "$meta"

  echo "Бэкап создан: $archive"
}

cmd_restore() {
  require_root

  local archive
  archive="$(find_latest_backup)"

  [[ -n "$archive" ]] || die "бэкап не найден"

  echo "Восстановление из: $archive"

  # Распаковываем только системные файлы GRUB.
  tar -xzf "$archive" -C / \
    etc/default/grub \
    etc/grub.d \
    boot/grub/grub.cfg

  # Перегенерируем grub.cfg по восстановленной конфигурации.
  update-grub

  echo "GRUB восстановлен."
}

cmd_list() {
  local dir
  dir="$(backup_dir)"

  [[ -d "$dir" ]] || {
    echo "Бэкапов нет."
    return
  }

  find "$dir" -maxdepth 1 -type f -name 'grub-backup-*.tar.gz' | sort | while read -r file; do
    basename "$file"
  done
}

cmd_clear() {
  local dir msg_safe files
  dir="$(backup_dir)"

  [[ -d "$dir" ]] || {
    echo "Бэкапов нет."
    return
  }

  if [[ "$ALL" == true ]]; then
    files="$(find "$dir" -maxdepth 1 -type f -name 'grub-backup-*.tar.gz')"
  else
    msg_safe="$(safe_message "$MESSAGE")"
    files="$(find "$dir" -maxdepth 1 -type f -name "grub-backup-*-${msg_safe}.tar.gz")"
  fi

  [[ -n "$files" ]] || {
    echo "Нечего удалять."
    return
  }

  echo "$files" | xargs rm -f
  echo "Бэкапы удалены."
}

COMMAND="${1:-}"
[[ -n "$COMMAND" ]] || {
  usage
  exit 1
}
shift

while [[ $# -gt 0 ]]; do
  case "$1" in
    -p|--path)
      BACKUP_ROOT="${2:-}"
      [[ -n "$BACKUP_ROOT" ]] || die "не указан путь после $1"
      shift 2
      ;;
    -m|--message)
      MESSAGE="${2:-}"
      [[ -n "$MESSAGE" ]] || die "не указано имя после $1"
      shift 2
      ;;
    -a|--all)
      ALL=true
      shift
      ;;
    -h|--help)
      usage
      exit 0
      ;;
    *)
      die "неизвестная опция: $1"
      ;;
  esac
done

case "$COMMAND" in
  save)
    cmd_save
    ;;
  restore)
    cmd_restore
    ;;
  list)
    cmd_list
    ;;
  clear)
    cmd_clear
    ;;
  -h|--help)
    usage
    ;;
  *)
    die "неизвестная команда: $COMMAND"
    ;;
esac

