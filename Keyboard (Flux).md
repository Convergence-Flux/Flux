module flux_keyboard

// Keyboard I/O ports
const KBD_DATA_PORT = 0x60
const KBD_STATUS_PORT = 0x64

// Global keyboard state
var key_buffer: u8 = 0
var key_ready: bool = false

// Scancode → ASCII table (partial, US QWERTY)
const scancode_map: [128]u8 = [
    0, 27, '1', '2', '3', '4', '5', '6',  // 0x00–0x07
    '7', '8', '9', '0', '-', '=', 8, 9,   // 0x08–0x0F
    'q', 'w', 'e', 'r', 't', 'y', 'u', 'i',
    'o', 'p', '[', ']', 13, 0, 'a', 's',
    'd', 'f', 'g', 'h', 'j', 'k', 'l', ';',
    '\'', '`', 0, '\\', 'z', 'x', 'c', 'v',
    'b', 'n', 'm', ',', '.', '/', 0, '*',
    0, ' ', 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, '7',
    '8', '9', '-', '4', '5', '6', '+', '1',
    '2', '3', '0', '.', 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0
]

// Wait until output buffer is full (data available)
def kbd_wait_readable():
    loop:
        status: u8
        asm("inb", KBD_STATUS_PORT, &status)
        if (status & 0x01) != 0:
            return
        goto loop

// Read one byte from keyboard data port
def kbd_read_scancode() -> u8:
    kbd_wait_readable()
    result: u8
    asm("inb", KBD_DATA_PORT, &result)
    return result

// Convert scancode to ASCII (ignores shift for now)
def scancode_to_ascii(scan: u8) -> u8:
    if scan >= 128:
        return 0
    return scancode_map[scan]

// Poll keyboard — must be called by kernel scheduler or loop
def keyboard_poll():
    scan = kbd_read_scancode()

    // Ignore key releases (high bit set)
    if (scan & 0x80) != 0:
        return

    ascii = scancode_to_ascii(scan)
    if ascii != 0:
        key_buffer = ascii
        key_ready = true