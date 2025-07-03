module flux_kernel

const VGA_BUF = 0xB8000u32
const SCREEN_WIDTH = 80
const SCREEN_HEIGHT = 25

var cursor_x: u32 = 0
var cursor_y: u32 = 0

var key_buffer: u8 = 0
var key_ready: bool = false

// Simple bump allocator state
var heap_start: u32 = 0x90000
var heap_end: u32 = 0xA0000
var heap_ptr: u32 = heap_start

// --- HAL: VGA Text Output ---
def vga_putc(c: u8):
    if c == 10: // newline
        cursor_x = 0
        cursor_y += 1
        if cursor_y >= SCREEN_HEIGHT:
            scroll()
            cursor_y = SCREEN_HEIGHT - 1
        return

    if c == 13: // carriage return
        cursor_x = 0
        return

    addr = VGA_BUF + (cursor_y * SCREEN_WIDTH + cursor_x) * 2
    asm("movb", addr, c)       // char byte
    asm("movb", addr + 1, 0x07) // attribute byte (light gray on black)
    cursor_x += 1
    if cursor_x >= SCREEN_WIDTH:
        cursor_x = 0
        cursor_y += 1
        if cursor_y >= SCREEN_HEIGHT:
            scroll()
            cursor_y = SCREEN_HEIGHT - 1

def scroll():
    // Move each line up by one
    src = VGA_BUF + SCREEN_WIDTH * 2
    dst = VGA_BUF
    count = (SCREEN_HEIGHT - 1) * SCREEN_WIDTH * 2
    for i in 0..count:
        asm("movb", dst + i, asm("movb", src + i))
    // Clear last line
    last_line_start = VGA_BUF + (SCREEN_HEIGHT - 1) * SCREEN_WIDTH * 2
    for i in 0..(SCREEN_WIDTH * 2):
        asm("movb", last_line_start + i, 0)

def clear_screen():
    for i in 0..(SCREEN_WIDTH * SCREEN_HEIGHT * 2):
        asm("movb", VGA_BUF + i, 0)
    cursor_x = 0
    cursor_y = 0

// --- HAL: Keyboard Input (polling) ---
def keyboard_poll():
    // Here, implement port reading to read keyboard scancode (x86)
    // For demo, just set key_ready = false
    key_ready = false

def keyboard_get_char() -> u8:
    while !key_ready:
        keyboard_poll()
    key_ready = false
    return key_buffer

// --- Memory Allocator (bump) ---
def alloc(size: u32) -> *u8:
    if heap_ptr + size > heap_end:
        return null
    addr = heap_ptr
    heap_ptr += size
    return (addr as *u8)

// --- Syscall Dispatcher ---
def syscall(num: u32, arg: u32) -> u32:
    if num == 1:    // putc
        vga_putc(arg as u8)
        return 0
    elif num == 2:  // read_char
        c = keyboard_get_char()
        return c as u32
    elif num == 3:  // clear screen
        clear_screen()
        return 0
    else:
        return 0xFFFFFFFF

// --- Shell from previous code (adapted) ---
def print(str: *u8):
    i = 0
    while str[i] != 0:
        syscall(1, str[i])
        i += 1

def read_char() -> u8:
    return syscall(2, 0) as u8

def print_prompt():
    print("> ")

def read_line(buffer: *u8, max: u32):
    i = 0
    loop:
        c = read_char()
        if c == 13:  // Enter
            syscall(1, 10)
            buffer[i] = 0
            return
        elif c == 8 and i > 0:  // Backspace
            syscall(1, 8)
            syscall(1, ' ')
            syscall(1, 8)
            i -= 1
        elif i < max - 1:
            syscall(1, c)
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
    syscall(3, 0)

def cmd_info():
    print("Flux OS Shell\n")
    print("ISA: Custom bare-metal\n")
    print("Memory: 64 KB max (static)\n")

def cmd_halt():
    print("Halting...\n")
    asm("hlt")

def shell_main():
    buffer = alloc(128)
    print("Flux OS v0.1\n")
    print("Type 'help' for commands\n")

    loop:
        print_prompt()
        read_line(buffer, 128)

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

// --- Kernel entry point ---
def main():
    clear_screen()
    shell_main()