# JSLang Compiler — Documentacao do Projeto

Projeto de Compiladores 2025.1 — Analisadores Lexico e Sintatico para a
linguagem **JSLang**, implementados com **JavaCC** (Java Compiler Compiler).

---

## Visao Geral

JSLang e uma linguagem de programacao baseada em JavaScript que contempla:

| Requisito | Implementado em |
|-----------|----------------|
| Tipos basicos: string, inteiro, real | Tokens `LIT_STRING`, `LIT_INT`, `LIT_REAL` |
| Atribuicao | Regra `Expr` (IDENT "=" Expr) |
| Declaracao de funcoes | Regra `FuncDecl` |
| Chamada de funcoes | Regra `PrimaryExpr` (LOOKAHEAD 2) |
| Laco de repeticao (while) | Regra `WhileStmt` |
| Condicional simples (if) | Regra `IfStmt` |
| Condicional composta (if-else) | Regra `IfStmt` (com else) |
| Entrada (read) | Regra `ReadStmt` |
| Saida (print) | Regra `PrintStmt` |
| Operadores aritmeticos | `+` `-` `*` `/` `%` |
| Operadores relacionais | `<` `>` `<=` `>=` `==` `!=` |
| Operadores booleanos | `&&` `\|\|` `!` |

---

## Estrutura do Projeto

```
projeto-compiladores-2025.1/
├── pom.xml                              # Build Maven com plugin JavaCC
├── CLAUDE.md                            # Esta documentacao
├── README.md                            # Guia rapido
└── src/
    ├── main/
    │   └── javacc/
    │       └── compiler/
    │           └── JSLang.jj            # Gramatica JavaCC (lexico + sintatico)
    └── examples/
        ├── hello.jsl                    # Tipos basicos e print
        ├── fatorial.jsl                 # Funcoes recursivas
        ├── fibonacci.jsl                # Laco while
        ├── condicionais.jsl             # if / if-else / read
        └── expressoes.jsl               # Todos os operadores
```

O plugin Maven `javacc-maven-plugin` le `JSLang.jj` e gera automaticamente:
- `JSLang.java` — classe principal do parser
- `JSLangTokenManager.java` — analisador lexico (automato DFA)
- `Token.java`, `SimpleCharStream.java`, etc. — infraestrutura

---

## Como Compilar e Executar

### Pre-requisitos

- Java 8+
- Maven 3.6+

### Build

```bash
mvn clean package
```

Isso executa:
1. `javacc:javacc` — gera as classes Java a partir de `JSLang.jj`
2. `compiler:compile` — compila tudo em `target/classes/`
3. `jar:jar` — empacota em `target/jslang-compiler.jar`

### Menu interativo (sem argumentos)

```bash
java -jar target/jslang-compiler.jar
```

Exibe o menu principal:

```
  +------------------------------------------+
  |       JSLang Compiler  -  2025.1         |
  +------------------------------------------+
  |  1. Analisar arquivo (.jsl)              |
  |  2. Exibir arvore sintatica de arquivo   |
  |  3. Listar tokens (analise lexica)       |
  |  4. Analisar codigo digitado             |
  |  0. Sair                                 |
  +------------------------------------------+
```

| Opcao | Descricao |
|-------|-----------|
| 1 | Analisa um arquivo `.jsl` e reporta SUCESSO ou erro |
| 2 | Analisa e exibe a arvore sintatica do arquivo |
| 3 | Lista todos os tokens com tipo, linha e coluna |
| 4 | Aceita codigo digitado no terminal e exibe a arvore |
| 0 | Encerra o programa |

### Modo direto (com argumento)

```bash
# Analisa o arquivo sem exibir o menu
java -jar target/jslang-compiler.jar src/examples/fatorial.jsl
```

### Exemplo de arvore sintatica (opcao 2)

Para `fatorial.jsl`:

```
[Program]
  [FuncDecl: fatorial(n)]
    [Block]
      [IfStmt]
        cond: n <= 1
        [Block]
          [ReturnStmt: 1]
        else
        [Block]
          [ReturnStmt: n * fatorial(n - 1)]
  [VarDecl: var i = 0]
  [WhileStmt]
    cond: i <= 10
    [Block]
      [PrintStmt: fatorial(i)]
      [ExprStmt: i = i + 1]
```

### Exemplo de listagem de tokens (opcao 3)

```
  === ANALISE LEXICA: src/examples/fatorial.jsl ===

  Linha  Col    Token                  Lexema
  ----------------------------------------------------------
  5      1      KW_FUNCTION            "function"
  5      10     IDENT                  "fatorial"
  5      18     LPAREN                 "("
  5      19     IDENT                  "n"
  ...
  ----------------------------------------------------------
  Total de tokens: 58
```

### Mensagens de erro

Erro lexico (caractere invalido):
```
  [ERRO LEXICO] Lexical error at line 1, column 9. Encountered: '@'
```

Erro sintatico (estrutura invalida):
```
  [ERRO SINTATICO] Encountered " <IDENT> "x "" at line 1, column 4.
  Was expecting: "(" ...
```

---

## Analisador Lexico

O analisador lexico e definido na secao de TOKEN/SKIP do arquivo `JSLang.jj`.
O JavaCC gera automaticamente um DFA (automato finito deterministico) para
reconhecer os tokens.

### Regras de Prioridade

No JavaCC, tokens sao escolhidos por:
1. **Maior correspondencia** (longest match): `<=` vence `<`
2. **Ordem de declaracao**: palavras-chave antes de identificadores;
   `LIT_REAL` antes de `LIT_INT`; operadores de 2 chars antes de 1 char

### Tokens da Linguagem

#### Ignorados (SKIP)
| Padrao | Descricao |
|--------|-----------|
| ` ` `\t` `\r` `\n` | Espacos em branco |
| `// ate fim da linha` | Comentario de linha |
| `/* ... */` | Comentario de bloco (estado BLOCK_COMMENT) |

#### Palavras-chave
| Token | Lexema |
|-------|--------|
| `KW_FUNCTION` | `function` |
| `KW_VAR` | `var` |
| `KW_LET` | `let` |
| `KW_IF` | `if` |
| `KW_ELSE` | `else` |
| `KW_WHILE` | `while` |
| `KW_RETURN` | `return` |
| `KW_TRUE` | `true` |
| `KW_FALSE` | `false` |
| `KW_PRINT` | `print` |
| `KW_READ` | `read` |

#### Literais
| Token | Expressao Regular | Exemplos |
|-------|-------------------|---------|
| `LIT_REAL` | `[0-9]+ '.' [0-9]+` | `3.14` `0.5` `100.0` |
| `LIT_INT` | `[0-9]+` | `0` `42` `1000` |
| `LIT_STRING` | `"..."` ou `'...'` com escapes | `"ola"` `'mundo'` `"tab\there"` |

Sequencias de escape suportadas em strings: `\n \t \r \\ \" \' \0`

#### Identificadores
| Token | Expressao Regular | Exemplos |
|-------|-------------------|---------|
| `IDENT` | `[a-zA-Z_][a-zA-Z0-9_]*` | `x` `soma` `_tmp` `valor2` |

Identificadores NAO podem comecar com digito. Palavras-chave sao reconhecidas
antes dos identificadores e NAO podem ser usadas como nomes de variaveis.

#### Operadores
| Token | Lexema | Categoria |
|-------|--------|-----------|
| `OP_EQ` | `==` | Relacional |
| `OP_NEQ` | `!=` | Relacional |
| `OP_LE` | `<=` | Relacional |
| `OP_GE` | `>=` | Relacional |
| `OP_LT` | `<` | Relacional |
| `OP_GT` | `>` | Relacional |
| `OP_AND` | `&&` | Booleano |
| `OP_OR` | `\|\|` | Booleano |
| `OP_NOT` | `!` | Booleano unario |
| `OP_ADD` | `+` | Aritmetico |
| `OP_SUB` | `-` | Aritmetico / unario |
| `OP_MUL` | `*` | Aritmetico |
| `OP_DIV` | `/` | Aritmetico |
| `OP_MOD` | `%` | Aritmetico |
| `ASSIGN` | `=` | Atribuicao |

---

## Analisador Sintatico

O analisador sintatico implementa uma gramatica **LL(k)** (descendente recursivo
preditivo) com k=1 ou k=2 conforme necessario. O JavaCC gera automaticamente
o parser descendente recursivo a partir das regras BNF.

### Gramatica Completa (BNF)

```
Program     -> ( FuncDecl | Statement )* EOF

FuncDecl    -> "function" IDENT "(" [ ParamList ] ")" Block
ParamList   -> IDENT ( "," IDENT )*
Block       -> "{" Statement* "}"

Statement   -> VarDecl
             | IfStmt
             | WhileStmt
             | ReturnStmt
             | PrintStmt
             | ReadStmt
             | Block
             | ExprStmt

VarDecl     -> ( "var" | "let" ) IDENT [ "=" Expr ] ";"
IfStmt      -> "if" "(" Expr ")" Block [ "else" ( IfStmt | Block ) ]
WhileStmt   -> "while" "(" Expr ")" Block
ReturnStmt  -> "return" [ Expr ] ";"
PrintStmt   -> "print" "(" Expr ")" ";"
ReadStmt    -> "read" "(" IDENT ")" ";"
ExprStmt    -> Expr ";"

Expr        -> IDENT "=" Expr    (atribuicao, direita-para-esquerda)
             | OrExpr

OrExpr      -> AndExpr  ( "||" AndExpr  )*
AndExpr     -> EqExpr   ( "&&" EqExpr   )*
EqExpr      -> RelExpr  ( ( "==" | "!=" ) RelExpr  )*
RelExpr     -> AddExpr  ( ( "<" | ">" | "<=" | ">=" ) AddExpr  )*
AddExpr     -> MulExpr  ( ( "+" | "-" ) MulExpr  )*
MulExpr     -> UnaryExpr ( ( "*" | "/" | "%" ) UnaryExpr )*

UnaryExpr   -> "!" UnaryExpr
             | "-" UnaryExpr
             | PrimaryExpr

PrimaryExpr -> LIT_INT
             | LIT_REAL
             | LIT_STRING
             | "true" | "false"
             | "(" Expr ")"
             | IDENT "(" [ ArgList ] ")"   -- chamada de funcao
             | IDENT                        -- variavel

ArgList     -> Expr ( "," Expr )*
```

### Precedencia de Operadores

A hierarquia de regras expressas acima implementa a seguinte tabela de
precedencia (maior numero = maior precedencia):

| Nivel | Operadores | Associatividade |
|-------|-----------|----------------|
| 1 | `=` | Direita |
| 2 | `\|\|` | Esquerda |
| 3 | `&&` | Esquerda |
| 4 | `==` `!=` | Esquerda |
| 5 | `<` `>` `<=` `>=` | Esquerda |
| 6 | `+` `-` | Esquerda |
| 7 | `*` `/` `%` | Esquerda |
| 8 | `!` `-` (unario) | Direita |

### Resolucao de Conflitos com LOOKAHEAD

O JavaCC usa LL(1) por padrao. Dois conflitos requerem lookahead extra:

**1. Chamada vs variavel em PrimaryExpr:**
```
LOOKAHEAD(<IDENT> <LPAREN>) <IDENT> <LPAREN> [ ArgList() ] <RPAREN>
| <IDENT>
```
Com 1 token, tanto chamada quanto variavel comecam com `IDENT`.
Com 2 tokens, `IDENT LPAREN` identifica chamada; `IDENT` seguido de
qualquer outra coisa e variavel.

**2. Atribuicao vs expressao em Expr:**
```
LOOKAHEAD(<IDENT> <ASSIGN>) <IDENT> <ASSIGN> Expr()
| OrExpr()
```
Com 1 token, `IDENT` poderia ser atribuicao ou inicio de expressao.
Com 2 tokens, `IDENT ASSIGN` identifica atribuicao.

**3. Dangling-else em IfStmt:**
```
[ LOOKAHEAD(<KW_ELSE>) <KW_ELSE> ( LOOKAHEAD(<KW_IF>) IfStmt() | Block() ) ]
```
O `LOOKAHEAD(<KW_ELSE>)` garante que o `else` e consumido pelo `if`
mais proximo (comportamento padrao de C/Java — greedy).

**4. Return com e sem expressao:**
```
[ LOOKAHEAD( <LIT_INT> | <LIT_REAL> | ... | <OP_NOT> | <OP_SUB> ) Expr() ]
```
Verifica se o proximo token e um dos que podem iniciar uma expressao
(conjunto FIRST de Expr). Se for `;`, nao ha expressao.

---

## Exemplos de Programas JSLang

### hello.jsl — Tipos basicos
```javascript
var saudacao = "Ola, Mundo!";
print(saudacao);
var inteiro = 42;
var real    = 3.14;
```

### fatorial.jsl — Funcoes recursivas
```javascript
function fatorial(n) {
    if (n <= 1) {
        return 1;
    } else {
        return n * fatorial(n - 1);
    }
}
var i = 0;
while (i <= 10) {
    print(fatorial(i));
    i = i + 1;
}
```

### condicionais.jsl — if / if-else / if-else-if
```javascript
function classificarNota(nota) {
    if (nota >= 9.0) {
        print("A");
    } else if (nota >= 7.0) {
        print("B");
    } else {
        print("F");
    }
}
```

---

## Erros Produzidos

### Erro Lexico
Ocorre quando um caractere nao pertence ao alfabeto da linguagem ou forma
um padrao invalido.

Exemplo — caractere invalido `@`:
```
[ERRO LEXICO] Lexical error at line 3, column 5.
Encountered: "@" (64), after : ""
```

### Erro Sintatico
Ocorre quando a sequencia de tokens nao corresponde a nenhuma regra da gramatica.

Exemplo — `if` sem parenteses:
```
[ERRO SINTATICO] Encountered " <LBRACE> "{" " at line 1, column 8.
Was expecting: "(" ...
```

---

## Decisoes de Projeto

### Por que JavaCC?
JavaCC e uma ferramenta matura, amplamente usada em disciplinas de compiladores,
que gera parsers Java puros (sem dependencias externas). O arquivo `.jj` unifica
lexico e sintatico em um unico lugar, facilitando a leitura.

### Por que LL(k) e nao LR?
A gramatica LL(k) e mais facil de entender e depurar. Para uma linguagem
baseada em JavaScript com poucas ambiguidades, LL(2) e suficiente.

### Comentarios de bloco com estados
O JavaCC suporta automatos com estados (MORE/SKIP). O estado `BLOCK_COMMENT`
permite reconhecer `/* ... */` mesmo com `*` e `/` internos, sem usar
expressoes regulares complexas.

### Palavras-chave como tokens proprios
Ao definir `KW_IF`, `KW_WHILE`, etc. como tokens separados de `IDENT`,
o JavaCC garante que palavras reservadas NAO possam ser usadas como nomes
de variaveis. A ordem de declaracao (palavras-chave antes de IDENT) e
crucial para isso.

### LIT_REAL antes de LIT_INT
Se `LIT_INT` viesse antes, `3.14` seria tokenizado como `3` (inteiro),
`.` (erro), `14` (inteiro). A regra LIT_REAL tem prioridade por ser
declarada primeiro.

---

## Referencias

- APPEL, A. W. *Modern Compiler Implementation in Java*. 2nd ed. Cambridge, 2002.
- AHO, A. et al. *Compilers: Principles, Techniques and Tools*. 2nd ed. 2006.
- JavaCC Documentation: https://javacc.github.io/javacc/
- JavaCC Maven Plugin: https://www.mojohaus.org/javacc-maven-plugin/
