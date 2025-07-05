module cli

import bios
import stdio
import string
import memory

const MAX_INPUT = 128

fn run_cli() {
    stdio.print("Flux CLI Ready. Type 'help' for commands.\n")

    loop {
        stdio.print("\n> ")
        var buffer = memory.alloc(MAX_INPUT)
        var length = stdio.read_line(buffer, MAX_INPUT)

        if length == 0 {
            continue
        }

        buffer[length - 1] = 0  // remove newline
        interpret_command(buffer)

        memory.free(buffer)
    }
}

fn interpret_command(cmd_ptr) {
    if string.equals(cmd_ptr, "help") {
        stdio.print("Available commands:\n")
        stdio.print("  help   - Show this help message\n")
        stdio.print("  clear  - Clear the screen\n")
        stdio.print("  reboot - Reboot the system\n")
        stdio.print("  echo   - Echo text (usage: echo hello world)\n")
        return
    }

    if string.equals(cmd_ptr, "clear") {
        bios.clear_screen()
        return
    }

    if string.equals(cmd_ptr, "reboot") {
        bios.reboot()
        return
    }

    if string.starts_with(cmd_ptr, "echo ") {
        var msg = cmd_ptr + 5
        stdio.print(msg)
        stdio.print("\n")
        return
    }

    stdio.print("Unknown command: ")
    stdio.print(cmd_ptr)
    stdio.print("\n")
}