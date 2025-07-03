module stdlib

# --- Constants ---
const HEAP_START = 0x6000
const HEAP_END = 0x8000
const HEADER_SIZE = 3

# --- Memory Allocator ---

# malloc: allocate `size` bytes from heap; returns pointer or 0 if no space
def malloc(size: u16) -> *u8:
    ptr = HEAP_START
    while ptr < HEAP_END:
        block_size = mem[ptr] | (mem[ptr+1] << 8)
        is_free = mem[ptr+2]
        if is_free == 1 and block_size >= size + HEADER_SIZE:
            mem[ptr+2] = 0  # mark block as used
            return ptr + HEADER_SIZE
        ptr += block_size
    return 0

# free: mark block as free for reuse
def free(ptr: *u8):
    block_ptr = ptr - HEADER_SIZE
    mem[block_ptr+2] = 1

# init_heap: set up heap with a single large free block
def init_heap():
    block_size = HEAP_END - HEAP_START
    mem[HEAP_START] = block_size & 0xFF
    mem[HEAP_START+1] = (block_size >> 8) & 0xFF
    mem[HEAP_START+2] = 1  # free

# --- Simple I/O Helpers ---

const OUT_PORT = 0x01

def putchar(c: u8):
    asm("out", OUT_PORT, c)

def puts(s: *u8):
    i = 0u32
    while s[i] != 0u8:
        putchar(s[i])
        i += 1

# Print hex (nibble) for debugging
def print_hex(n: u16):
    puts("0x")
    hex_chars = "0123456789ABCDEF\0"
    putchar(hex_chars[(n >> 12) & 0xF])
    putchar(hex_chars[(n >> 8) & 0xF])
    putchar(hex_chars[(n >> 4) & 0xF])
    putchar(hex_chars[n & 0xF])
    putchar('\n')

# --- String Utilities ---

def strlen(s: *u8) -> u32:
    i = 0u32
    while s[i] != 0u8:
        i += 1
    return i

def strcmp(a: *u8, b: *u8) -> i32:
    i = 0u32
    while a[i] != 0u8 and b[i] != 0u8:
        if a[i] != b[i]:
            return a[i] - b[i]
        i += 1
    return a[i] - b[i]

# --- Misc Utilities ---

def halt():
    loop:
        asm("halt")
        jmp loop