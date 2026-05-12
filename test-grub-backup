#!/usr/bin/env bash
#
# test-grub-backup
# Тестирует скрипт grub-backup и печатает отчет.
#
# Безопасные тесты:
#   ./test-grub-backup ./grub-backup
#
# Опасный restore-тест только в VM:
#   sudo ./test-grub-backup ./grub-backup --with-restore
#

set -u

SCRIPT="${1:-./grub-backup}"
WITH_RESTORE="${2:-}"
TEST_ROOT="$(mktemp -d)"
REPORT="$TEST_ROOT/report.txt"

PASSED=0
FAILED=0
SKIPPED=0

log() {
  echo "$*" | tee -a "$REPORT"
}

pass() {
  PASSED=$((PASSED + 1))
  log "✅ PASS: $*"
}

fail() {
  FAILED=$((FAILED + 1))
  log "❌ FAIL: $*"
}

skip() {
  SKIPPED=$((SKIPPED + 1))
  log "⚠️  SKIP: $*"
}

run_test() {
  local name="$1"
  shift

  log ""
  log "TEST: $name"

  if "$@" >>"$REPORT" 2>&1; then
    pass "$name"
  else
    fail "$name"
  fi
}

cleanup() {
  log ""
  log "Временная папка: $TEST_ROOT"
}
trap cleanup EXIT

log "Отчет тестирования grub-backup"
log "Скрипт: $SCRIPT"
log "Дата: $(date)"
log "Тестовая папка: $TEST_ROOT"

if [[ ! -f "$SCRIPT" ]]; then
  fail "Файл скрипта не найден: $SCRIPT"
  exit 1
fi

if [[ ! -x "$SCRIPT" ]]; then
  chmod +x "$SCRIPT" 2>/dev/null || true
fi

# 1. Проверка синтаксиса Bash.
run_test "bash -n" bash -n "$SCRIPT"

# 2. ShellCheck, если установлен.
if command -v shellcheck >/dev/null 2>&1; then
  run_test "shellcheck" shellcheck "$SCRIPT"
else
  skip "shellcheck не установлен"
fi

# 3. Проверка help/usage.
run_test "help command" "$SCRIPT" --help

# 4. Проверка неизвестной команды.
log ""
log "TEST: unknown command"
if "$SCRIPT" unknown-command >>"$REPORT" 2>&1; then
  fail "unknown command должна завершаться ошибкой"
else
  pass "unknown command"
fi

# 5. Проверка неизвестной опции.
log ""
log "TEST: unknown option"
if "$SCRIPT" list --bad-option >>"$REPORT" 2>&1; then
  fail "unknown option должна завершаться ошибкой"
else
  pass "unknown option"
fi

# 6. save без root должен упасть, если тест запущен не от root.
if [[ "$EUID" -ne 0 ]]; then
  log ""
  log "TEST: save requires root"
  if "$SCRIPT" save -p "$TEST_ROOT" -m no-root >>"$REPORT" 2>&1; then
    fail "save без root должен завершаться ошибкой"
  else
    pass "save requires root"
  fi
else
  skip "save requires root — тест запущен от root"
fi

# 7. save/list/clear требуют sudo, потому что читают реальные GRUB-файлы.
if [[ "$EUID" -eq 0 ]]; then
  run_test "save named backup" "$SCRIPT" save -p "$TEST_ROOT" -m test-backup

  ARCHIVE_COUNT="$(find "$TEST_ROOT/.grub-backup" -type f -name '*.tar.gz' 2>/dev/null | wc -l)"
  log ""
  log "TEST: archive created"
  if [[ "$ARCHIVE_COUNT" -ge 1 ]]; then
    pass "archive created"
  else
    fail "archive created"
  fi

  ARCHIVE="$(find "$TEST_ROOT/.grub-backup" -type f -name '*.tar.gz' | head -n 1)"

  log ""
  log "TEST: archive contains grub files"
  if tar -tzf "$ARCHIVE" | grep -q 'etc/default/grub' &&
     tar -tzf "$ARCHIVE" | grep -q 'etc/grub.d' &&
     tar -tzf "$ARCHIVE" | grep -q 'boot/grub/grub.cfg'; then
    pass "archive contains grub files"
  else
    fail "archive contains grub files"
  fi

  log ""
  log "TEST: list shows backup"
  if "$SCRIPT" list -p "$TEST_ROOT" | grep -q 'test-backup'; then
    pass "list shows backup"
  else
    fail "list shows backup"
  fi

  run_test "save unnamed backup" "$SCRIPT" save -p "$TEST_ROOT"

  log ""
  log "TEST: list shows unnamed backup"
  if "$SCRIPT" list -p "$TEST_ROOT" | grep -q '__unnamed__'; then
    pass "list shows unnamed backup"
  else
    fail "list shows unnamed backup"
  fi

  run_test "clear named backup" "$SCRIPT" clear -p "$TEST_ROOT" -m test-backup

  log ""
  log "TEST: named backup removed"
  if "$SCRIPT" list -p "$TEST_ROOT" | grep -q 'test-backup'; then
    fail "named backup removed"
  else
    pass "named backup removed"
  fi

  run_test "clear all backups" "$SCRIPT" clear -p "$TEST_ROOT" --all

  log ""
  log "TEST: all backups removed"
  REMAINING="$(find "$TEST_ROOT/.grub-backup" -type f -name '*.tar.gz' 2>/dev/null | wc -l)"
  if [[ "$REMAINING" -eq 0 ]]; then
    pass "all backups removed"
  else
    fail "all backups removed"
  fi
else
  skip "save/list/clear archive tests — запустите через sudo"
fi

# 8. Опциональный restore-тест. Только для VM.
if [[ "$WITH_RESTORE" == "--with-restore" ]]; then
  if [[ "$EUID" -ne 0 ]]; then
    skip "restore test требует sudo"
  else
    log ""
    log "ВНИМАНИЕ: restore test изменяет реальные GRUB-файлы."

    cp /etc/default/grub "$TEST_ROOT/grub.original"

    "$SCRIPT" save -p "$TEST_ROOT" -m restore-test >>"$REPORT" 2>&1

    echo "# temporary test line" >> /etc/default/grub

    "$SCRIPT" restore -p "$TEST_ROOT" -m restore-test >>"$REPORT" 2>&1

    log ""
    log "TEST: restore reverted /etc/default/grub"
    if cmp -s /etc/default/grub "$TEST_ROOT/grub.original"; then
      pass "restore reverted /etc/default/grub"
    else
      fail "restore reverted /etc/default/grub"
    fi
  fi
else
  skip "restore test отключен; включается через --with-restore только в VM"
fi

log ""
log "=============================="
log "ИТОГО"
log "Успешно:   $PASSED"
log "Ошибки:    $FAILED"
log "Пропущено: $SKIPPED"
log "Отчет:     $REPORT"
log "=============================="

if [[ "$FAILED" -gt 0 ]]; then
  exit 1
fi

exit 0

