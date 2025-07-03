enum TokenType:
    LET
    IDENT
    ASSIGN
    NUMBER
    PLUS
    MINUS
    MUL
    DIV
    LPAREN
    RPAREN
    PRINT
    EOF

struct Token:
    type: TokenType
    value: string

def lexer(input: string) -> list<Token>:
    tokens = []
    i = 0
    while i < len(input):
        c = input[i]
        if c.isspace():
            i += 1
            continue
        if c.isalpha():
            start = i
            while i < len(input) and input[i].isalnum():
                i += 1
            word = input[start:i]
            if word == "let":
                tokens.append(Token(TokenType.LET, word))
            elif word == "print":
                tokens.append(Token(TokenType.PRINT, word))
            else:
                tokens.append(Token(TokenType.IDENT, word))
            continue
        if c.isdigit():
            start = i
            while i < len(input) and input[i].isdigit():
                i += 1
            num = input[start:i]
            tokens.append(Token(TokenType.NUMBER, num))
            continue
        if c == '+':
            tokens.append(Token(TokenType.PLUS, c))
        elif c == '-':
            tokens.append(Token(TokenType.MINUS, c))
        elif c == '*':
            tokens.append(Token(TokenType.MUL, c))
        elif c == '/':
            tokens.append(Token(TokenType.DIV, c))
        elif c == '=':
            tokens.append(Token(TokenType.ASSIGN, c))
        elif c == '(':
            tokens.append(Token(TokenType.LPAREN, c))
        elif c == ')':
            tokens.append(Token(TokenType.RPAREN, c))
        else:
            error("Unknown character: " + c)
        i += 1
    tokens.append(Token(TokenType.EOF, ""))
    return tokens

enum NodeType:
    PROGRAM
    LET_STMT
    PRINT_STMT
    IDENT
    NUMBER
    BINARY_EXPR

struct ASTNode:
    type: NodeType
    value: any
    left: ASTNode
    right: ASTNode
    name: string

struct Parser:
    tokens: list<Token>
    pos: int

def current(parser: Parser) -> Token:
    return parser.tokens[parser.pos]

def advance(parser: Parser):
    parser.pos += 1

def match(parser: Parser, type: TokenType) -> bool:
    return current(parser).type == type

def eat(parser: Parser, type: TokenType):
    if match(parser, type):
        advance(parser)
    else:
        error("Expected token: " + str(type))

def parse_factor(parser: Parser) -> ASTNode:
    tok = current(parser)
    if tok.type == TokenType.NUMBER:
        advance(parser)
        return ASTNode(NodeType.NUMBER, int(tok.value), None, None, "")
    elif tok.type == TokenType.IDENT:
        advance(parser)
        return ASTNode(NodeType.IDENT, None, None, None, tok.value)
    elif tok.type == TokenType.LPAREN:
        advance(parser)
        expr = parse_expr(parser)
        eat(parser, TokenType.RPAREN)
        return expr
    else:
        error("Invalid factor")

def parse_term(parser: Parser) -> ASTNode:
    node = parse_factor(parser)
    while match(parser, TokenType.MUL) or match(parser, TokenType.DIV):
        op = current(parser)
        advance(parser)
        node = ASTNode(NodeType.BINARY_EXPR, op.type, node, parse_factor(parser), "")
    return node

def parse_expr(parser: Parser) -> ASTNode:
    node = parse_term(parser)
    while match(parser, TokenType.PLUS) or match(parser, TokenType.MINUS):
        op = current(parser)
        advance(parser)
        node = ASTNode(NodeType.BINARY_EXPR, op.type, node, parse_term(parser), "")
    return node

def parse_stmt(parser: Parser) -> ASTNode:
    tok = current(parser)
    if tok.type == TokenType.LET:
        advance(parser)
        ident = current(parser)
        eat(parser, TokenType.IDENT)
        eat(parser, TokenType.ASSIGN)
        expr = parse_expr(parser)
        return ASTNode(NodeType.LET_STMT, None, expr, None, ident.value)
    elif tok.type == TokenType.PRINT:
        advance(parser)
        eat(parser, TokenType.LPAREN)
        expr = parse_expr(parser)
        eat(parser, TokenType.RPAREN)
        return ASTNode(NodeType.PRINT_STMT, None, expr, None, "")
    else:
        error("Unknown statement")

def parse_program(parser: Parser) -> ASTNode:
    stmts = []
    while not match(parser, TokenType.EOF):
        stmts.append(parse_stmt(parser))
    return ASTNode(NodeType.PROGRAM, stmts, None, None, "")

# Simple symbol table for variables (name -> memory location)
struct SymbolTable:
    table: map<string, int>
    next_addr: int

def symtable_init() -> SymbolTable:
    return SymbolTable({}, 0x1000)  # start variables at 0x1000 memory address

def symtable_add(symtab: SymbolTable, name: string) -> int:
    if name in symtab.table:
        return symtab.table[name]
    addr = symtab.next_addr
    symtab.table[name] = addr
    symtab.next_addr += 4  # assuming 4 bytes per variable
    return addr

# Code Generator: emits Flux ISA assembly code lines
def generate_expr(node: ASTNode, symtab: SymbolTable, output: list<string>) -> string:
    # Returns register name where result is stored (e.g. r0, r1, ...)
    if node.type == NodeType.NUMBER:
        reg = "r0"
        output.append(f"load {reg}, {node.value}")
        return reg
    elif node.type == NodeType.IDENT:
        reg = "r0"
        addr = symtab.table[node.name]
        output.append(f"loadm {reg}, {addr}")
        return reg
    elif node.type == NodeType.BINARY_EXPR:
        left_reg = generate_expr(node.left, symtab, output)
        right_reg = generate_expr(node.right, symtab, output)
        dest_reg = "r0"
        op = node.value
        if op == TokenType.PLUS:
            output.append(f"add {dest_reg}, {left_reg}, {right_reg}")
        elif op == TokenType.MINUS:
            output.append(f"sub {dest_reg}, {left_reg}, {right_reg}")
        elif op == TokenType.MUL:
            output.append(f"mul {dest_reg}, {left_reg}, {right_reg}")
        elif op == TokenType.DIV:
            output.append(f"div {dest_reg}, {left_reg}, {right_reg}")
        return dest_reg
    else:
        error("Unknown expression node")

def generate_stmt(node: ASTNode, symtab: SymbolTable, output: list<string>):
    if node.type == NodeType.LET_STMT:
        addr = symtable_add(symtab, node.name)
        reg = generate_expr(node.left, symtab, output)
        output.append(f"store {addr}, {reg}")
    elif node.type == NodeType.PRINT_STMT:
        reg = generate_expr(node.left, symtab, output)
        output.append(f"print {reg}")
    else:
        error("Unknown statement node")

def generate_program(node: ASTNode) -> list<string>:
    output = []
    symtab = symtable_init()
    for stmt in node.value:
        generate_stmt(stmt, symtab, output)
    output.append("halt")
    return output

def main():
    source = """
    let x = 10 + 5 * 2
    print(x)
    let y = x - 3
    print(y)
    """

    tokens = lexer(source)
    parser = Parser(tokens, 0)
    ast = parse_program(parser)

    asm_code = generate_program(ast)
    for line in asm_code:
        print(line)

main()