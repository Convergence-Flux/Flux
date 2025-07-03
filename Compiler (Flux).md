module flux_compiler

# --- Constants ---
const OUT_PORT = 0x01
const CODE_BASE = 0x8000  # Where machine code will be written
const MAX_TOKENS = 256

# --- Data Structures ---
enum TokenType:
    IDENT
    NUMBER
    ASSIGN
    PLUS
    MINUS
    MUL
    DIV
    SEMI
    LPAREN
    RPAREN
    LBRACE
    RBRACE
    FN
    LET
    RETURN
    PRINT
    EOF

struct Token:
    kind: TokenType
    value: *u8

struct Var:
    name: *u8
    addr: u16

# --- I/O ---
def putchar(c: u8):
    asm("out", OUT_PORT, c)

def puts(s: *u8):
    i = 0u32
    while s[i] != 0u8:
        putchar(s[i])
        i += 1

def print_hex(n: u16):
    # dummy simple number printer
    puts("0x")
    putchar('0' + ((n >> 4) & 0xF))
    putchar('0' + (n & 0xF))
    putchar('\n')

# --- Lexer ---
def is_alpha(c: u8) -> bool:
    return (c >= 'a' and c <= 'z') or (c >= 'A' and c <= 'Z')

def is_digit(c: u8) -> bool:
    return c >= '0' and c <= '9'

def tokenize(src: *u8, tokens: *Token) -> u32:
    i = 0u32
    t = 0u32
    while src[i] != 0u8 and t < MAX_TOKENS:
        c = src[i]
        if c == ' ' or c == '\n':
            i += 1
        elif is_alpha(c):
            start = i
            while is_alpha(src[i]): i += 1
            tokens[t].kind = TokenType.IDENT
            tokens[t].value = start
            t += 1
        elif is_digit(c):
            tokens[t].kind = TokenType.NUMBER
            tokens[t].value = i
            while is_digit(src[i]): i += 1
            t += 1
        elif c == '+':
            tokens[t].kind = TokenType.PLUS
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '-':
            tokens[t].kind = TokenType.MINUS
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '*':
            tokens[t].kind = TokenType.MUL
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '/':
            tokens[t].kind = TokenType.DIV
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '=':
            tokens[t].kind = TokenType.ASSIGN
            tokens[t].value = i
            i += 1
            t += 1
        elif c == ';':
            tokens[t].kind = TokenType.SEMI
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '(':
            tokens[t].kind = TokenType.LPAREN
            tokens[t].value = i
            i += 1
            t += 1
        elif c == ')':
            tokens[t].kind = TokenType.RPAREN
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '{':
            tokens[t].kind = TokenType.LBRACE
            tokens[t].value = i
            i += 1
            t += 1
        elif c == '}':
            tokens[t].kind = TokenType.RBRACE
            tokens[t].value = i
            i += 1
            t += 1
    tokens[t].kind = TokenType.EOF
    return t

# --- Parser / Codegen Globals ---
var pos: u32 = 0
var current_addr: u16 = CODE_BASE
var vars: [Var; 16]
var var_count: u32 = 0

# --- Code Generator Helpers ---
def emit(instr: u8, op1: u8, op2: u8):
    mem[current_addr] = instr
    mem[current_addr + 1] = op1
    mem[current_addr + 2] = op2
    current_addr += 3

def lookup_var(name: *u8) -> u16:
    i = 0u32
    while i < var_count:
        if strcmp(vars[i].name, name) == 0:
            return vars[i].addr
        i += 1
    return 0xFFFF

# --- Parser & Generator ---
def expect(tokens: *Token, kind: TokenType):
    if tokens[pos].kind != kind:
        puts("Syntax Error\n")
        halt()
    pos += 1

def parse_number(tokens: *Token) -> u8:
    tok = tokens[pos]
    val = src[tok.value] - '0'
    pos += 1
    return val

def parse_expr(tokens: *Token):
    if tokens[pos].kind == TokenType.NUMBER:
        val = parse_number(tokens)
        emit(0x01, 1, val)  # li r1, val
    else:
        puts("Expected expression\n")
        halt()

def parse_stmt(tokens: *Token):
    if tokens[pos].kind == TokenType.LET:
        pos += 1
        name = tokens[pos].value
        pos += 1
        expect(tokens, TokenType.ASSIGN)
        parse_expr(tokens)
        addr = CODE_BASE + (var_count * 2)
        vars[var_count].name = name
        vars[var_count].addr = addr
        var_count += 1
        emit(0x02, 1, addr)  # store r1 -> addr
        expect(tokens, TokenType.SEMI)
    elif tokens[pos].kind == TokenType.PRINT:
        pos += 1
        parse_expr(tokens)
        emit(0x05, 0x01, 1)  # out port 1, r1
        expect(tokens, TokenType.SEMI)

# --- Main Compile ---
def compile(src: *u8):
    tokens = alloc(MAX_TOKENS * sizeof(Token))
    tokenize(src, tokens)
    while tokens[pos].kind != TokenType.EOF:
        parse_stmt(tokens)

# --- Entry Point ---
def main():
    program = "let x = 5; print 5;\0"
    compile(program)
    puts("Compiled to memory!\n")