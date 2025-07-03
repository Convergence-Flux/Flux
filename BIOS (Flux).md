module bios

const OUT_PORT = 0x01
const IN_PORT = 0x02
const STACK_TOP = 0xFF00
const KERNEL_LOAD_ADDR = 0x1000
const KERNEL_SECTORS = 10
const OS_START = KERNEL_LOAD_ADDR

# === Output character to console port ===
def putchar(c: u8):
    asm("out", OUT_PORT, c)

# === Output zero-terminated string ===
def puts(s: *u8):
    i = 0u32
    while s[i] != 0u8:
        putchar(s[i])
        i += 1

# === Read a byte from port ===
def port_in(port: u8) -> u8:
    ret: u8
    asm("in", port, &ret)
    return ret

# === Disk Read via BIOS Int 13h (simulated) ===
def disk_read(lba_start: u32, sectors: u8, dest: *u8) -> bool:
    # For bare-metal Flux, you must implement real disk reading,
    # here is a simplified placeholder returning success.
    # In a real Flux BIOS, replace this with actual hardware calls.
    return true

# === Halt the CPU forever ===
def halt_loop():
    loop:
        asm("halt")
        jmp loop

# === Print loading message ===
def print_loading():
    puts("Flux BIOS: Loading kernel...\n\0")

# === Print error message ===
def print_error():
    puts("Flux BIOS: Disk read error!\n\0")

# === Initialize hardware, stack and disable interrupts ===
def init():
    # Disable interrupts
    asm("cli")
    # Setup stack pointer (sp = R13)
    sp = STACK_TOP
    asm("mov", 13, sp)
    # Other hardware init here (timers, PIC etc.) if needed

# === Jump to kernel entry point ===
def jump_to_kernel():
    asm("li", 15, OS_START)  # Load kernel start to ip
    asm("jmp", 15)           # Jump to kernel

# === Main BIOS routine ===
def main():
    init()
    print_loading()
    ok = disk_read(2, KERNEL_SECTORS, KERNEL_LOAD_ADDR as *u8)
    if not ok:
        print_error()
        halt_loop()
    jump_to_kernel()
    halt_loop()

# === Entry point ===
main()