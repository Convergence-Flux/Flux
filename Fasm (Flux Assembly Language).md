# ===== Instruction Set (OpCodes) =====
const OPCODES = {
    "li":   0x01,
    "mov":  0x02,
    "add":  0x03,
    "sub":  0x04,
    "mul":  0x05,
    "div":  0x06,
    "and":  0x07,
    "or":   0x08,
    "xor":  0x09,
    "not":  0x0A,

    "ld":   0x10,
    "st":   0x11,

    "jmp":  0x20,
    "beq":  0x21,
    "bne":  0x22,
    "blt":  0x23,
    "bgt":  0x24,

    "call": 0x30,
    "ret":  0x31,

    "push": 0x40,
    "pop":  0x41,

    "out":  0x50,
    "in":   0x51,

    "halt": 0xFF
}

# ===== Instruction Structure =====
struct Instruction:
    opcode: int
    rd: int
    rs1: int
    rs2: int
    address: int

# ===== Assembler State =====
struct Assembler:
    instructions: list<Instruction>
    labels: map<string, int>
    pending_labels: list<(int, string, string)>  # (index, field, label)

def new_assembler():
    return Assembler([], {}, [])

# ===== Helpers =====
def parse_register(name):
    if name.startswith("r"):
        return int(name[1:])
    error("Invalid register: " + name)

def parse_immediate(value):
    if value.startswith("0x"):
        return int(value[2:], 16)
    return int(value)

def is_label(s):
    return not s.startswith("r") and not s.isdigit() and not s.startswith("0x")

def parse_line(line):
    if ";" in line:
        line = line.split(";")[0]
    return line.strip()

# ===== Parse and Assemble Lines =====
def assemble(asm: Assembler, line: string):
    line = parse_line(line)
    if line == "":
        return

    if line.endswith(":"):
        label = line[:-1]
        asm.labels[label] = len(asm.instructions) * 4
        return

    parts = line.split()
    op = parts[0]
    args = [arg.strip(",") for arg in parts[1:]]

    opcode = OPCODES.get(op, -1)
    if opcode == -1:
        error("Unknown opcode: " + op)

    rd = 0
    rs1 = 0
    rs2 = 0

    if op == "ret" or op == "halt":
        pass

    elif op == "jmp" or op == "call":
        if is_label(args[0]):
            asm.pending_labels.append((len(asm.instructions), "rd", args[0]))
        else:
            rd = parse_immediate(args[0])

    elif op == "out":
        rd = parse_immediate(args[0])
        rs1 = parse_register(args[1])

    elif op == "li":
        rd = parse_register(args[0])
        if is_label(args[1]):
            asm.pending_labels.append((len(asm.instructions), "rs2", args[1]))
        else:
            rs2 = parse_immediate(args[1])

    elif len(args) == 3:
        rd = parse_register(args[0])
        rs1 = parse_register(args[1])
        if is_label(args[2]):
            asm.pending_labels.append((len(asm.instructions), "rs2", args[2]))
        else:
            rs2 = parse_register(args[2])

    elif len(args) == 2:
        rd = parse_register(args[0])
        rs1 = parse_register(args[1])

    instr = Instruction(opcode, rd, rs1, rs2, len(asm.instructions) * 4)
    asm.instructions.append(instr)

# ===== Label Resolution =====
def resolve_labels(asm: Assembler):
    for (index, field, label) in asm.pending_labels:
        addr = asm.labels.get(label, -1)
        if addr == -1:
            error("Undefined label: " + label)
        instr = asm.instructions[index]
        if field == "rd":
            asm.instructions[index] = Instruction(instr.opcode, addr & 0xFF, instr.rs1, instr.rs2, instr.address)
        elif field == "rs2":
            asm.instructions[index] = Instruction(instr.opcode, instr.rd, instr.rs1, addr & 0xFF, instr.address)

# ===== Assemble a Program =====
def assemble_program(lines):
    asm = new_assembler()
    for line in lines:
        assemble(asm, line)
    resolve_labels(asm)

    binary = []
    for i in asm.instructions:
        binary.append(i.opcode)
        binary.append(i.rd)
        binary.append(i.rs1)
        binary.append(i.rs2)
    return binary

# ===== Main Entry Point with Example Program =====
def main():
    program = [
        "; Example Flux Assembly Program",
        "start:",
        "li r1, 10",
        "li r2, 20",
        "add r3, r1, r2",
        "call print",
        "halt",
        "",
        "print:",
        "out 0x01, r3",
        "ret"
    ]

    binary = assemble_program(program)

    print("Assembled Flux ISA Binary:")
    for i, byte in enumerate(binary):
        print("0x" + hex(byte), end=" ")
        if (i + 1) % 4 == 0:
            print()

main()