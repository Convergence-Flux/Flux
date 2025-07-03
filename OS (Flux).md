module flux_os

const VGA_BUF = 0xB8000
const KBD_PORT = 0x60
const STACK_TOP = 0xFF00
const HEAP_START = 0x9000
const HEAP_END = 0xF000
const SCREEN_WIDTH = 80
const SCREEN_HEIGHT = 25

# ─────────────────────
# Memory Allocator (bump)
# ─────────────────────
var heap_ptr: u32 = HEAP_START

def alloc(size: u32) -> *u8:
    if heap_ptr + size >= HEAP_END:
        return null
    addr = heap_ptr
    heap_ptr += size
    return (addr as *u8)

# ─────────────────────
# VGA Text Output
# ─────────────────────
var cursor_x: u32 = 0
var cursor_y: u32 = 0

def vga_putc(c: u8):
    if c == 10:
        cursor_x = 0
        cursor_y += 1
        if cursor_y >= SCREEN_HEIGHT:
            scroll()
            cursor_y = SCREEN_HEIGHT - 1
        return

    pos = (cursor_y * SCREEN_WIDTH + cursor_x) * 2
    addr = VGA_BUF + pos
    *(addr as *u8) = c
    *(addr + 1 as *u8) = 0x07  # white on black
    cursor_x += 1
    if cursor_x >= SCREEN_WIDTH:
        cursor_x = 0
        cursor_y += 1
        if cursor_y >= SCREEN_HEIGHT:
            scroll()
            cursor_y = SCREEN_HEIGHT - 1

def scroll():
    for row in 1..SCREEN_HEIGHT:
        for col in 0..SCREEN_WIDTH:
            from = ((row * SCREEN_WIDTH + col) * 2)
            to = (((row - 1) * SCREEN_WIDTH + col) * 2)
            *(VGA_BUF + to as *u8) = *(VGA_BUF + from as *u8)
            *(VGA_BUF + to + 1 as *u8) = *(VGA_BUF + from + 1 as *u8)
    # Clear last line
    base = (SCREEN_HEIGHT - 1) * SCREEN_WIDTH * 2
    for i in 0..(SCREEN_WIDTH * 2):
        *(VGA_BUF + base + i as *u8) = 0

def clear_screen():
    for i in 0..(SCREEN_WIDTH * SCREEN_HEIGHT * 2):
        *(VGA_BUF + i as *u8) = 0
    cursor_x = 0
    cursor_y = 0

# ─────────────────────
# Keyboard Input (polling)
# ─────────────────────
const scancode_map: [128]u8 = [
    0, 27, '1','2','3','4','5','6','7','8','9','0','-','=','\b','\t',
    'q','w','e','r','t','y','u','i','o','p','[',']','\n',0,'a','s',
    'd','f','g','h','j','k','l',';','\'','`',0,'\\','z','x','c','v',
    'b','n','m',',','.','/',0,'*',0,' ',0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
]

def read_scancode() -> u8:
    sc: u8
    asm("inb", KBD_PORT, &sc)
    return sc

def get_key() -> u8:
    loop:
        sc = read_scancode()
        if (sc & 0x80) == 0:
            ascii = scancode_map[sc]
            if ascii != 0:
                return ascii
        goto loop

# ─────────────────────
# Syscalls
# ─────────────────────
def syscall(n: u32, arg: u32) -> u32:
    if n == 1:
        vga_putc(arg as u8)
        return 0
    elif n == 2:
        return get_key() as u32
    elif n == 3:
        clear_screen()
        return 0
    else:
        return 0xFFFFFFFF

# ─────────────────────
# Shell
# ─────────────────────
def putc(c: u8):
    syscall(1, c)

def print(s: *u8):
    i = 0
    while s[i] != 0:
        putc(s[i])
        i += 1

def read_char() -> u8:
    return syscall(2, 0) as u8

def print_prompt():
    print("> ")

def read_line(buf: *u8, max: u32):
    i = 0
    loop:
        c = read_char()
        if c == 13:
            putc(10)
            buf[i] = 0
            return
        elif c == 8 and i > 0:
            putc(8)
            putc(' ')
            putc(8)
            i -= 1
        elif i < max - 1:
            putc(c)
            buf[i] = c
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
    print("Commands:\n")
    print("  help  - show this help\n")
    print("  info  - show system info\n")
    print("  clear - clear the screen\n")
    print("  halt  - stop the CPU\n")

def cmd_info():
    print("Flux OS - Bare Metal\n")
    print("ISA: Custom\nRAM: ~64KB static\n")

def cmd_clear():
    syscall(3, 0)

def cmd_halt():
    print("Halting...\n")
    asm("hlt")

def shell():
    buf = alloc(128)
    print("Welcome to Flux OS Shell\n")
    print("Type 'help' for commands.\n")
    loop:
        print_prompt()
        read_line(buf, 128)
        if str_eq(buf, "help"):
            cmd_help()
        elif str_eq(buf, "clear"):
            cmd_clear()
        elif str_eq(buf, "info"):
            cmd_info()
        elif str_eq(buf, "halt"):
            cmd_halt()
        elif buf[0] == 0:
            pass
        else:
            print("Unknown: ")
            print(buf)
            print("\n")
        goto loop

# ─────────────────────
# Kernel Entry Point
# ─────────────────────
def main():
    asm("mov", 13, STACK_TOP)  # sp
    asm("mov", 14, STACK_TOP)  # fp
    clear_screen()
    shell()