# Compilador JSLang — Projeto de Compiladores 2025.1

Analisadores **lexico** e **sintatico** para a linguagem **JSLang** —
uma linguagem baseada em JavaScript, implementados com **JavaCC**.

## Inicio Rapido

```bash
# 1. Compilar o projeto (gera o parser e empacota)
mvn clean package

# 2. Analisar um arquivo de exemplo
java -jar target/jslang-compiler.jar src/examples/fatorial.jsl

# 3. Analisar outro exemplo
java -jar target/jslang-compiler.jar src/examples/fibonacci.jsl
```

## Funcionalidades da Linguagem

- **Tipos basicos** — inteiros (`42`), reais (`3.14`), strings (`"texto"`), booleanos (`true`/`false`)
- **Variaveis** — `var x = 10;` e `let y = 3.14;`
- **Atribuicao** — `x = x + 1;` (incluindo encadeada: `a = b = 0;`)
- **Funcoes** — declaracao com `function` e chamada com argumentos
- **Laco** — `while (cond) { ... }`
- **Condicional simples** — `if (cond) { ... }`
- **Condicional composta** — `if (cond) { ... } else { ... }`
- **Saida** — `print(expr);`
- **Entrada** — `read(variavel);`
- **Operadores** — aritmeticos `+ - * / %`, relacionais `< > <= >= == !=`, booleanos `&& || !`
- **Comentarios** — `// linha` e `/* bloco */`

## Exemplos

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

## Estrutura

```
src/main/javacc/compiler/JSLang.jj  — gramatica (lexico + sintatico)
src/examples/                        — programas de exemplo .jsl
pom.xml                              — build Maven
CLAUDE.md                            — documentacao detalhada
```

Consulte o `CLAUDE.md` para documentacao completa da gramatica, tokens,
decisoes de projeto e exemplos de erros.

## Pre-requisitos

- Java 8+
- Maven 3.6+
