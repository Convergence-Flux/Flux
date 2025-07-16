module flux.bootstrap

////////////////////////////////////////////////////////////
// 0. MINI RUNTIME
////////////////////////////////////////////////////////////

extern sys_write_console(msg: String)
extern sys_alloc(size: Int): Ptr
extern fs_read(path: String): String
extern fs_write(path: String, data: String)
extern load_binary_to_memory(bin: List<Byte>)
extern jump_to_entry()

func println(msg: String) {
    sys_write_console(msg + "\n")
}

func malloc(size: Int): Ptr {
    return sys_alloc(size)
}

////////////////////////////////////////////////////////////
// 1. TOKENIZER
////////////////////////////////////////////////////////////

enum TokenType {
    Module, Import, Func, Struct, Let, Var, Return,
    If, Else, While, For, In,
    True, False,
    Ident, Number, String,
    LParen, RParen, LBrace, RBrace, Comma, Dot, Colon, Semicolon,
    Plus, Minus, Star, Slash,
    Eq, EqEq, NotEq, Lt, Gt, Le, Ge,
    AndAnd, OrOr,
    Range,
    EOF
}

struct Token { let kind: TokenType; let value: String }

func tokenize(src: String): List<Token> {
    let tokens = core.list.create_list<Token>()
    var i = 0

    func add(kind: TokenType, val: String) {
        core.list.push(tokens, Token(kind, val))
    }

    while i < src.length() {
        let c = src[i]

        // skip whitespace
        if c.is_space() {
            i += 1
            continue
        }

        // keywords
        if src.starts_with("module", i) { add(TokenType.Module, "module"); i += 6; continue }
        if src.starts_with("import", i) { add(TokenType.Import, "import"); i += 6; continue }
        if src.starts_with("func", i)   { add(TokenType.Func, "func"); i += 4; continue }
        if src.starts_with("struct", i) { add(TokenType.Struct, "struct"); i += 6; continue }
        if src.starts_with("let", i)    { add(TokenType.Let, "let"); i += 3; continue }
        if src.starts_with("var", i)    { add(TokenType.Var, "var"); i += 3; continue }
        if src.starts_with("return", i) { add(TokenType.Return, "return"); i += 6; continue }
        if src.starts_with("if", i)     { add(TokenType.If, "if"); i += 2; continue }
        if src.starts_with("else", i)   { add(TokenType.Else, "else"); i += 4; continue }
        if src.starts_with("while", i)  { add(TokenType.While, "while"); i += 5; continue }
        if src.starts_with("for", i)    { add(TokenType.For, "for"); i += 3; continue }
        if src.starts_with("in", i)     { add(TokenType.In, "in"); i += 2; continue }
        if src.starts_with("true", i)   { add(TokenType.True, "true"); i += 4; continue }
        if src.starts_with("false", i)  { add(TokenType.False, "false"); i += 5; continue }

        // numbers
        if c.is_digit() {
            var j = i
            while j < src.length() && src[j].is_digit() { j += 1 }
            add(TokenType.Number, src.substring(i, j))
            i = j
            continue
        }

        // strings
        if c == '"' {
            var j = i + 1
            while j < src.length() && src[j] != '"' { j += 1 }
            let strval = src.substring(i + 1, j)
            add(TokenType.String, strval)
            i = j + 1
            continue
        }

        // identifiers
        if c.is_alpha() {
            var j = i
            while j < src.length() && (src[j].is_alnum() || src[j] == '_') { j += 1 }
            add(TokenType.Ident, src.substring(i, j))
            i = j
            continue
        }

        // operators & punctuation
        if src.starts_with("==", i) { add(TokenType.EqEq, "=="); i += 2; continue }
        if src.starts_with("!=", i) { add(TokenType.NotEq, "!="); i += 2; continue }
        if src.starts_with("<=", i) { add(TokenType.Le, "<="); i += 2; continue }
        if src.starts_with(">=", i) { add(TokenType.Ge, ">="); i += 2; continue }
        if src.starts_with("..", i) { add(TokenType.Range, ".."); i += 2; continue }
        if src.starts_with("&&", i) { add(TokenType.AndAnd, "&&"); i += 2; continue }
        if src.starts_with("||", i) { add(TokenType.OrOr, "||"); i += 2; continue }

        // single char
        match c {
            '(' => add(TokenType.LParen, "("),
            ')' => add(TokenType.RParen, ")"),
            '{' => add(TokenType.LBrace, "{"),
            '}' => add(TokenType.RBrace, "}"),
            ',' => add(TokenType.Comma, ","),
            '.' => add(TokenType.Dot, "."),
            ':' => add(TokenType.Colon, ":"),
            ';' => add(TokenType.Semicolon, ";"),
            '+' => add(TokenType.Plus, "+"),
            '-' => add(TokenType.Minus, "-"),
            '*' => add(TokenType.Star, "*"),
            '/' => add(TokenType.Slash, "/"),
            '=' => add(TokenType.Eq, "="),
            '<' => add(TokenType.Lt, "<"),
            '>' => add(TokenType.Gt, ">"),
            _ => println("[Tokenizer] Unknown char: " + c)
        }
        i += 1
    }

    add(TokenType.EOF, "")
    return tokens
}

////////////////////////////////////////////////////////////
// 2. AST NODES
////////////////////////////////////////////////////////////

enum ASTKind {
    Program, Module, Import,
    FuncDecl, StructDecl,
    Block, VarDecl, Assign, Return,
    IfStmt, WhileStmt, ForStmt,
    Call, BinOp, UnaryOp,
    NumberLit, StringLit, BoolLit, IdentExpr,
    StructInit, MemberAccess
}

struct ASTNode {
    let kind: ASTKind
    let value: String
    let children: List<ASTNode>
}

////////////////////////////////////////////////////////////
// 3. PARSER
////////////////////////////////////////////////////////////

// (Parser functions would parse the full grammar into AST)

////////////////////////////////////////////////////////////
// 4. SEMANTIC ANALYZER
////////////////////////////////////////////////////////////

// (Type checking, symbol resolution)

////////////////////////////////////////////////////////////
// 5. CODEGEN → FLUXASM
////////////////////////////////////////////////////////////

// (Walk AST, generate FluxASM instructions)

////////////////////////////////////////////////////////////
// 6. ASSEMBLER → FLUX BINARY
////////////////////////////////////////////////////////////

func assemble(asm: String): List<Byte> {
    let bin = core.list.create_list<Byte>()
    for c in asm.bytes() { core.list.push(bin, c) }
    return bin
}

////////////////////////////////////////////////////////////
// 7. FLUXOS LOADER
////////////////////////////////////////////////////////////

func fluxos_loader(path: String) {
    let bin = fs_read(path)
    load_binary_to_memory(bin.bytes())
    jump_to_entry()
}

////////////////////////////////////////////////////////////
// 8. COMPILER DRIVER
////////////////////////////////////////////////////////////

func compile_flux(src: String): List<Byte> {
    let tokens = tokenize(src)
    let ast = parse(tokens)
    analyze(ast)
    let asm = generate_asm(ast)
    println("[Compiler] Generated ASM:\n" + asm)
    return assemble(asm)
}

////////////////////////////////////////////////////////////
// 9. SELF-HOST MAIN
////////////////////////////////////////////////////////////

func main() {
    println("[FluxBootstrap] Self-host compiler starting...")
    let src = fs_read("flux_bootstrap.flux")
    let bin = compile_flux(src)
    fs_write("flux_bootstrap.fluxbin", bin.to_string())
    println("[FluxBootstrap] Rebuilt itself successfully!")
}