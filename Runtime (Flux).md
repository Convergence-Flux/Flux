module runtime

# Constants
const STACK_TOP = 0xFF00            # Top of stack memory
const HEAP_START = 0x8000           # Heap start address
const HEAP_SIZE = 0x1000            # 4 KB heap size

# Memory allocator state
var heap_ptr: u32 = HEAP_START

# Simple out port for console output
const CONSOLE_OUT_PORT = 0x01

# === Hardware Initialization ===
def init_hardware():
    # Set stack pointer (SP = R13)
    sp = STACK_TOP
    asm("mov", 13, sp)   # mov sp, STACK_TOP
    
    # Initialize other hardware registers or peripherals here
    # (e.g., init serial port, timers...)

    # Clear heap pointer
    heap_ptr = HEAP_START

# === Simple malloc ===
def malloc(size: u32) -> *u8:
    global heap_ptr
    if heap_ptr + size > HEAP_START + HEAP_SIZE:
        return 0  # Out of memory
    
    addr = heap_ptr
    heap_ptr += size
    return addr as *u8

# === Console output ===
def putchar(c: u8):
    asm("out", CONSOLE_OUT_PORT, c)

def puts(s: *u8):
    i = 0
    while s[i] != 0u8:
        putchar(s[i])
        i += 1

# === Console input (stub) ===
def getchar() -> u8:
    # In real hardware, implement port input or interrupt-driven input
    ret: u8 = 0
    asm("in", CONSOLE_OUT_PORT, &ret)
    return ret

# === Interrupt Handling (stub) ===
def enable_interrupts():
    asm("sti")

def disable_interrupts():
    asm("cli")

def interrupt_handler():
    # Placeholder for interrupt logic
    # In real system, would read interrupt vector, dispatch handlers, etc.
    pass

# === Main event loop ===
def main_loop():
    puts("Flux Runtime Started\n\0")

    loop:
        # Could check for input, timers, interrupts, or tasks here
        
        asm("halt")   # Halt CPU until next interrupt
        jmp loop

# === Runtime Entry Point ===
def main():
    disable_interrupts()
    init_hardware()
    enable_interrupts()
    main_loop()

# Automatically start runtime
main()