module flux_shell

const MAX_INPUT = 128
const SCREEN_WIDTH = 80

def print(str: *u8):
    i = 0
    while str[i] != 0:
        putc(str[i])
        i += 1

def putc(c: u8):
    asm("syscall", 1, c)  # syscall 1 = write character

def read_char() -> u8:
    ret: u8
    asm("syscall", 2, &ret)  # syscall 2 = read character
    return ret

def print_prompt():
    print("> ")

def read_line(buffer: *u8, max: u32):
    i = 0
    loop:
        c = read_char()
        if c == 13:  # Enter
            putc(10)  # newline
            buffer[i] = 0
            return
        elif c == 8 and i > 0:  # Backspace
            putc(8)
            putc(' ')
            putc(8)
            i -= 1
        elif i < max - 1:
            putc(c)
            buffer[i] = c
            i += 1
        goto loop

def str_eq(a: *u8, b: *u8) -> bool:
    i = 0
    while a[i] != 0 and b[i] != 0:
        if a[i] != b[i]:
            return false
        i += 1
    return a[i] == b[i]

def cmd_help():
    print("Available commands:\n")
    print("  help  - show this help\n")
    print("  clear - clear screen\n")
    print("  info  - system info\n")
    print("  halt  - stop CPU\n")

def cmd_clear():
    asm("syscall", 3)  # syscall 3 = clear screen

def cmd_info():
    print("Flux OS Shell\n")
    print("ISA: Custom bare-metal\n")
    print("Memory: 64 KB max (static)\n")

def cmd_halt():
    print("Halting...\n")
    asm("halt")

def shell_main():
    buffer = alloc(MAX_INPUT)
    print("Flux OS v0.1\n")
    print("Type 'help' for commands\n")

    loop:
        print_prompt()
        read_line(buffer, MAX_INPUT)

        if str_eq(buffer, "help"):
            cmd_help()
        elif str_eq(buffer, "clear"):
            cmd_clear()
        elif str_eq(buffer, "info"):
            cmd_info()
        elif str_eq(buffer, "halt"):
            cmd_halt()
        elif buffer[0] == 0:
            pass
        else:
            print("Unknown command: ")
            print(buffer)
            print("\n")
        goto loop