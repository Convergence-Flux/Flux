module interrupts

const IVT_START = 0x100   # Interrupt Vector Table start address
const IVT_SIZE = 256      # Number of interrupt vectors supported

# Hardware IRQ lines
const IRQ_TIMER = 0x00
const IRQ_KEYBOARD = 0x01

# I/O Ports for PIC (Programmable Interrupt Controller) EOI
const PIC_COMMAND_PORT = 0x20

# CPU registers saved during interrupt
struct CPUState {
    r0: u32
    r1: u32
    r2: u32
    r3: u32
    r4: u32
    r5: u32
    r6: u32
    r7: u32
    sp: u32
    pc: u32
    flags: u32
}

# Interrupt Vector Table holds pointers to interrupt handler functions
var ivt: [IVT_SIZE]*fn(*CPUState) = [0; IVT_SIZE]

# Default interrupt handler
def default_handler(state: *CPUState):
    # Print or log interrupt number here if you want
    # For now, just halt
    loop:
        asm("halt")
        jmp loop

# Register an interrupt handler for given vector number
def register_handler(vector: u8, handler: *fn(*CPUState)):
    ivt[vector] = handler

# Assembly stub for saving CPU state and calling dispatcher
# This stub would be defined in assembly for your ISA, here simplified
def interrupt_entry_stub(vector: u8):
    # Save registers to stack or CPUState struct
    state = allocate_cpu_state()  # your function to get CPUState pointer
    save_cpu_state(state)          # pseudo functions

    # Call the dispatcher
    interrupt_dispatcher(vector, state)

    restore_cpu_state(state)       # restore CPU registers
    asm("iret")                   # return from interrupt

# Interrupt dispatcher: calls registered handler or default
def interrupt_dispatcher(vector: u8, state: *CPUState):
    handler = ivt[vector]
    if handler == 0:
        handler = default_handler
    handler(state)
    send_eoi(vector)

# Send End Of Interrupt to PIC
def send_eoi(vector: u8):
    asm("out", PIC_COMMAND_PORT, 0x20)

# Example timer interrupt handler
def timer_handler(state: *CPUState):
    # Increment system tick or call scheduler
    # For demo, just acknowledge and return
    send_eoi(IRQ_TIMER)

# Example keyboard interrupt handler
def keyboard_handler(state: *CPUState):
    # Read key from keyboard port
    key = asm_in(0x00)
    # Optionally buffer key or signal input subsystem
    send_eoi(IRQ_KEYBOARD)

# Initialize IVT with default handlers and setup specific ones
def init_interrupts():
    i = 0u32
    while i < IVT_SIZE:
        ivt[i] = default_handler
        i += 1
    register_handler(IRQ_TIMER, timer_handler)
    register_handler(IRQ_KEYBOARD, keyboard_handler)

# Dummy functions for CPU state save/restore - implement as per ISA
def allocate_cpu_state() -> *CPUState:
    # allocate from static memory or stack
    return cast(*CPUState, 0x7000)  # example fixed address

def save_cpu_state(state: *CPUState):
    # save CPU registers to state - implement inline asm for your ISA
    asm("pushall", state)

def restore_cpu_state(state: *CPUState):
    # restore CPU registers from state
    asm("popall", state)

# Helper for port input
def asm_in(port: u8) -> u8:
    val = 0u8
    asm("in", port, val)
    return val