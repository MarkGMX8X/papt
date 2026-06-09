#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Paptman - обёртка для pacman с синтаксисом apt
# Copyright (C) 2026 INSPACE95
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.

import sys
import subprocess

# ---------- НАСТРОЙКИ ----------
REAL_PACMAN = "/usr/bin/pacman"

COMMAND_MAP = {
    "install": "-S",
    "remove": "-Rns",
    "update": "-Sy",
    "search": "-Ss",
}

# ---------- ФУНКЦИИ ----------
def run_cmd(cmd, use_sudo=True):
    """Запускает внешнюю команду. При use_sudo=True добавляет sudo в начало."""
    if use_sudo and cmd[0] != "sudo":
        cmd = ["sudo"] + cmd
    try:
        subprocess.run(cmd, check=True)
    except subprocess.CalledProcessError as e:
        sys.exit(e.returncode)

def parse_args(args):
    """
    Анализирует аргументы команды и возвращает отфильтрованный список.
    Поддерживает флаги -pcm, -p, -r (игнорируются для совместимости).
    """
    filtered_args = []
    ignore_flags = {"-pcm", "-p", "-r", "-aur", "-yay"}
    for arg in args:
        if arg not in ignore_flags:
            filtered_args.append(arg)
    return filtered_args

# ---------- ОСНОВНАЯ ЛОГИКА ----------
def main():
    if len(sys.argv) < 2:
        print("Использование: paptman {install|remove|update|search} [пакеты]")
        print("")
        print("Примеры:")
        print("  paptman install firefox")
        print("  paptman remove vlc")
        print("  paptman update")
        print("  paptman update -Sys     # полное обновление системы")
        print("  paptman search gimp")
        sys.exit(1)

    command = sys.argv[1]

    # Если команда не из нашего списка — передаём прямой вызов настоящему pacman
    if command not in COMMAND_MAP:
        run_cmd([REAL_PACMAN] + sys.argv[1:], use_sudo=True)
        return

    args = sys.argv[2:]
    pkg_args = parse_args(args)

    # ---- UPDATE (с поддержкой -Sys или -Syu для полного обновления) ----
    if command == "update":
        is_full_upgrade = "-Sys" in args or "-Syu" in args
        clean_args = [a for a in pkg_args if a not in ("-Sys", "-Syu")]
        
        if is_full_upgrade:
            print("[*] Полное обновление системы...")
            run_cmd([REAL_PACMAN, "-Syu"] + clean_args, use_sudo=True)
        else:
            print("[*] Обновление баз репозиториев...")
            run_cmd([REAL_PACMAN, "-Sy"] + clean_args, use_sudo=True)
        return

    # ---- SEARCH ----
    if command == "search":
        print("--- Официальные репозитории ---")
        run_cmd([REAL_PACMAN, "-Ss"] + pkg_args, use_sudo=True)
        return

    # ---- INSTALL ----
    if command == "install":
        run_cmd([REAL_PACMAN, "-S"] + pkg_args, use_sudo=True)
        return

    # ---- REMOVE ----
    if command == "remove":
        run_cmd([REAL_PACMAN, "-Rns"] + pkg_args, use_sudo=True)
        return

    # ---- FALLBACK ----
    run_cmd([REAL_PACMAN, COMMAND_MAP.get(command, "")] + pkg_args, use_sudo=True)

if __name__ == "__main__":
    main()