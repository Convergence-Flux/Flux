module hal

# --- Hardware Ports ---
const TIMER_PORT = 0x10      # Hypothetical timer hardware port
const KEYBOARD_PORT = 0x00   # Keyboard input port
const SERIAL_OUT_PORT = 0x01 # Serial output port
const SERIAL_IN_PORT = 0x02  # Serial input port

# --- Timer Driver ---
var timer_ticks: u32 = 0

# Read timer value (e.g. elapsed ticks)
def read_timer() -> u32:
    low = asm_in(TIMER_PORT)
    high = asm_in(TIMER_PORT + 1)
    return (high << 8) | low

# Simple delay loop based on timer ticks
def delay(ticks: u32):
    start = timer_ticks
    while (timer_ticks - start) < ticks:
        asm("nop")

# Timer interrupt handler increments ticks
def timer_interrupt_handler():
    timer_ticks += 1
    # Acknowledge timer interrupt if needed
    asm("out", TIMER_PORT, 0xFF)

# --- Keyboard Driver ---
# Poll keyboard input (returns 0 if no key)
def poll_key() -> u8:
    key = asm_in(KEYBOARD_PORT)
    return key

# Wait and get a keypress (blocking)
def get_key() -> u8:
    key = 0u8
    while key == 0u8:
        key = poll_key()
    return key

# --- Serial Driver ---

def serial_putchar(c: u8):
    asm("out", SERIAL_OUT_PORT, c)

def serial_puts(s: *u8):
    i = 0u32
    while s[i] != 0u8:
        serial_putchar(s[i])
        i += 1

def serial_getchar() -> u8:
    # Busy wait for input availability
    ch = 0u8
    while ch == 0u8:
        ch = asm_in(SERIAL_IN_PORT)
    return ch

# --- HAL Initialization ---

def hal_init():
    timer_ticks = 0
    # Initialize hardware if necessary (e.g. enable timer interrupts)
    # For now, assume hardware auto-starts timer

# --- Assembly I/O Helpers ---

def asm_in(port: u8) -> u8:
    # Inline assembly for input from port
    # Flux asm wrapper - customize per ISA
    val = 0u8
    asm("in", port, val)
    return val