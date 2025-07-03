const MEM_SIZE = 65536  # 64 KB memory

struct CPU:
    registers: array<int>  # 16 general registers (r0 - r15)
    memory: array<int>     # Memory bytes
    running: bool

def new_cpu():
    cpu = CPU([0]*16, [0]*MEM_SIZE, false)
    cpu.registers[0] = 0      # r0 is always zero
    cpu.running = false
    return cpu

# Register indices
const R0 = 0
const SP = 13
const FP = 14
const IP = 15
const RA = 12

# Opcodes
const OPCODES = {
    0x01: "li",
    0x02: "mov",
    0x03: "add",
    0x04: "sub",
    0x05: "mul",
    0x06: "div",
    0x07: "and",
    0x08: "or",
    0x09: "xor",
    0x0A: "not",

    0x10: "ld",
    0x11: "st",

    0x20: "jmp",
    0x21: "beq",
    0x22: "bne",
    0x23: "blt",
    0x24: "bgt",

    0x30: "call",
    0x31: "ret",

    0x40: "push",
    0x41: "pop",

    0x50: "out",
    0x51: "in",

    0xFF: "halt"
}

def fetch_instruction(cpu: CPU):
    ip = cpu.registers[IP]
    if ip + 3 >= MEM_SIZE:
        error("Instruction pointer out of bounds")
    # Instructions are 4 bytes
    opcode = cpu.memory[ip]
    rd = cpu.memory[ip + 1]
    rs1 = cpu.memory[ip + 2]
    rs2 = cpu.memory[ip + 3]
    return (opcode, rd, rs1, rs2)

def execute_instruction(cpu: CPU, opcode: int, rd: int, rs1: int, rs2: int):
    def reg(r): return cpu.registers[r]

    def set_reg(r, val):
        if r != R0:
            cpu.registers[r] = val & 0xFFFFFFFF  # 32-bit registers

    # For debugging, uncomment:
    # print("Executing:", OPCODES.get(opcode, "???"), rd, rs1, rs2)

    if opcode == 0x01:  # li rd, imm (rs2 is immediate)
        set_reg(rd, rs2)

    elif opcode == 0x02:  # mov rd, rs1
        set_reg(rd, reg(rs1))

    elif opcode == 0x03:  # add rd, rs1, rs2
        set_reg(rd, reg(rs1) + reg(rs2))

    elif opcode == 0x04:  # sub rd, rs1, rs2
        set_reg(rd, reg(rs1) - reg(rs2))

    elif opcode == 0x05:  # mul rd, rs1, rs2
        set_reg(rd, reg(rs1) * reg(rs2))

    elif opcode == 0x06:  # div rd, rs1, rs2
        if reg(rs2) == 0:
            error("Division by zero")
        set_reg(rd, reg(rs1) // reg(rs2))

    elif opcode == 0x07:  # and rd, rs1, rs2
        set_reg(rd, reg(rs1) & reg(rs2))

    elif opcode == 0x08:  # or rd, rs1, rs2
        set_reg(rd, reg(rs1) | reg(rs2))

    elif opcode == 0x09:  # xor rd, rs1, rs2
        set_reg(rd, reg(rs1) ^ reg(rs2))

    elif opcode == 0x0A:  # not rd, rs1
        set_reg(rd, ~reg(rs1))

    elif opcode == 0x10:  # ld rd, [rs1 + rs2]
        addr = (reg(rs1) + reg(rs2)) % MEM_SIZE
        set_reg(rd, cpu.memory[addr])

    elif opcode == 0x11:  # st [rd + rs1], rs2
        addr = (reg(rd) + reg(rs1)) % MEM_SIZE
        cpu.memory[addr] = reg(rs2) & 0xFF

    elif opcode == 0x20:  # jmp rd (absolute address)
        cpu.registers[IP] = reg(rd)
        return True  # IP already set, skip IP increment

    elif opcode == 0x21:  # beq rs1, rs2, rd (branch if equal)
        if reg(rs1) == reg(rs2):
            cpu.registers[IP] = reg(rd)
            return True

    elif opcode == 0x22:  # bne rs1, rs2, rd
        if reg(rs1) != reg(rs2):
            cpu.registers[IP] = reg(rd)
            return True

    elif opcode == 0x23:  # blt rs1, rs2, rd
        if reg(rs1) < reg(rs2):
            cpu.registers[IP] = reg(rd)
            return True

    elif opcode == 0x24:  # bgt rs1, rs2, rd
        if reg(rs1) > reg(rs2):
            cpu.registers[IP] = reg(rd]
            return True

    elif opcode == 0x30:  # call rd (jump and save return)
        cpu.registers[RA] = cpu.registers[IP] + 4
        cpu.registers[IP] = reg(rd)
        return True

    elif opcode == 0x31:  # ret
        cpu.registers[IP] = cpu.registers[RA]
        return True

    elif opcode == 0x40:  # push rs1
        cpu.registers[SP] -= 1
        addr = cpu.registers[SP] % MEM_SIZE
        cpu.memory[addr] = reg(rs1) & 0xFF

    elif opcode == 0x41:  # pop rd
        addr = cpu.registers[SP] % MEM_SIZE
        set_reg(rd, cpu.memory[addr])
        cpu.registers[SP] += 1

    elif opcode == 0x50:  # out port, rs1 (simulate by print)
        print("OUT port", rd, "value:", reg(rs1))

    elif opcode == 0x51:  # in rd, port (simulate input = 0)
        set_reg(rd, 0)

    elif opcode == 0xFF:  # halt
        cpu.running = False

    else:
        error("Unknown opcode: " + str(opcode))

    return False  # IP not changed, increment as usual

def load_program(cpu: CPU, program: list<int>):
    for i in range(len(program)):
        cpu.memory[i] = program[i]

def run(cpu: CPU):
    cpu.running = True
    cpu.registers[IP] = 0
    cpu.registers[SP] = MEM_SIZE - 1

    while cpu.running:
        opcode, rd, rs1, rs2 = fetch_instruction(cpu)
        ip_changed = execute_instruction(cpu, opcode, rd, rs1, rs2)
        if not ip_changed:
            cpu.registers[IP] += 4
        cpu.registers[R0] = 0  # r0 always zero

def main():
    # Example binary program (assembled Flux ISA):
    # li r1, 10; li r2, 20; add r3, r1, r2; out 0x01, r3; halt
    program = [
        0x01, 0x01, 0x00, 10,   # li r1, 10
        0x01, 0x02, 0x00, 20,   # li r2, 20
        0x03, 0x03, 0x01, 0x02, # add r3, r1, r2
        0x50, 0x01, 0x03, 0x00, # out 0x01, r3
        0xFF, 0x00, 0x00, 0x00  # halt
    ]

    cpu = new_cpu()
    load_program(cpu, program)
    run(cpu)

main()