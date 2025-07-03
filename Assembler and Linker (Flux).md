# Opcode map
const OPCODES = {
    "load": 0x01,
    "add":  0x02,
    "sub":  0x03,
    "mul":  0x04,
    "div":  0x05,
    "cmp":  0x06,
    "jmp":  0x07,
    "je":   0x08,
    "jne":  0x09,
    "jlt":  0x0A,
    "jgt":  0x0B,
    "print":0x0C,
    "halt": 0xFF
}

# Convert register like "r3" → 3
def reg_to_num(r):
    if r.startswith("r"):
        return int(r[1:])
    return 0

# Instruction struct
struct Instruction:
    opcode: int
    op1: int
    op2: int
    op3: int
    addr: int    # address in binary output

# Assembler module
struct Assembler:
    instructions: list<Instruction>
    labels: map<string, int>  # label → instruction index
    unresolved: list<(int, string)>  # (instruction idx, label to resolve)

def new_assembler():
    return Assembler([], {}, [])

# Parse a line into instruction or label
def parse_line(asm, line):
    line = line.strip()
    if line == "" or line.startswith("#"):
        return
    if line.endswith(":"):
        label = line[:-1]
        asm.labels[label] = len(asm.instructions)
        return

    parts = line.split(" ")
    op = parts[0]
    args = parts[1:]
    opcode = OPCODES.get(op, 0)
    op1 = 0
    op2 = 0
    op3 = 0

    # Simple argument parsing based on instruction
    if opcode == 0:
        error("Unknown opcode: " + op)

    def parse_arg(arg):
        if arg.startswith("r"):
            return reg_to_num(arg)
        if arg.isdigit():
            return int(arg)
        # Label or unresolved
        return -1

    if len(args) > 0:
        op1 = parse_arg(args[0])
    if len(args) > 1:
        op2 = parse_arg(args[1])
    if len(args) > 2:
        op3 = parse_arg(args[2])

    instr = Instruction(opcode, op1, op2, op3, len(asm.instructions)*4)
    asm.instructions.append(instr)

    # Check unresolved labels (-1)
    for i, val in enumerate([op1, op2, op3]):
        if val == -1:
            asm.unresolved.append((len(asm.instructions)-1, args[i]))

# Resolve label references to instruction addresses
def resolve_labels(asm):
    for (idx, label) in asm.unresolved:
        if label in asm.labels:
            target_idx = asm.labels[label]
            target_addr = target_idx * 4
            instr = asm.instructions[idx]
            # Patch first unresolved operand found
            if instr.op1 == -1:
                asm.instructions[idx] = Instruction(instr.opcode, target_addr & 0xFF, (target_addr >> 8) & 0xFF, instr.op3, instr.addr)
            elif instr.op2 == -1:
                asm.instructions[idx] = Instruction(instr.opcode, instr.op1, target_addr & 0xFF, (target_addr >> 8) & 0xFF, instr.addr)
            elif instr.op3 == -1:
                asm.instructions[idx] = Instruction(instr.opcode, instr.op1, instr.op2, target_addr & 0xFF, instr.addr)
        else:
            error("Unresolved label: " + label)

# Assemble source code lines into binary
def assemble_source(lines):
    asm = new_assembler()
    for line in lines:
        parse_line(asm, line)
    resolve_labels(asm)

    binary = []
    for instr in asm.instructions:
        binary.append(instr.opcode)
        binary.append(instr.op1)
        binary.append(instr.op2)
        binary.append(instr.op3)
    return binary, asm.labels

# Linker module
struct Linker:
    binaries: list<list<int>>
    label_offsets: list<map<string, int>>
    combined_binary: list<int>

def new_linker():
    return Linker([], [], [])

# Add a binary module and its labels
def linker_add_module(linker, binary, labels):
    linker.binaries.append(binary)
    linker.label_offsets.append(labels)

# Link all added modules into one binary
def linker_link(linker):
    linker.combined_binary = []
    addr_offset = 0
    combined_labels = {}

    # Adjust labels by offset
    for i in range(len(linker.binaries)):
        binary = linker.binaries[i]
        labels = linker.label_offsets[i]

        for label, addr in labels.items():
            combined_labels[label] = addr + addr_offset

        # Patch jumps in binary (assuming jumps use absolute addresses)
        # For simplicity, no complex relocations here

        linker.combined_binary.extend(binary)
        addr_offset += len(binary)

    return linker.combined_binary, combined_labels

# Example usage

def main():
    # Module 1
    source1 = [
        "start:",
        "load r1 10",
        "jmp end",
        "halt",
        "end:",
        "print r1",
        "halt"
    ]

    bin1, labels1 = assemble_source(source1)

    # Module 2
    source2 = [
        "load r2 5",
        "add r3 r1 r2",
        "print r3",
        "halt"
    ]

    bin2, labels2 = assemble_source(source2)

    linker = new_linker()
    linker_add_module(linker, bin1, labels1)
    linker_add_module(linker, bin2, labels2)

    linked_binary, combined_labels = linker_link(linker)

    # Print linked binary bytes as hex
    for byte in linked_binary:
        print(hex(byte))

main()