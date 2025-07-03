# Load immediate value into a register
load rX, value           # rX = value

# Arithmetic
add rX, rY, rZ           # rX = rY + rZ
sub rX, rY, rZ           # rX = rY - rZ
mul rX, rY, rZ           # rX = rY * rZ
div rX, rY, rZ           # rX = rY / rZ

# Comparison
cmp rX, rY               # sets flags: eq, lt, gt

# Control Flow
jmp addr                 # jump to address
je addr                  # jump if last cmp was equal
jne addr                 # jump if not equal
jlt addr                 # jump if less than
jgt addr                 # jump if greater than

# Output
print rX                 # print value in rX

# Memory
store addr, rX           # memory[addr] = rX
loadm rX, addr           # rX = memory[addr]

# Function Calls (optional for future)
call addr                # push ip, jump to addr
ret                      # pop ip, return

# Halt Program
halt                     # end execution