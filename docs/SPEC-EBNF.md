# 🌶️ Pimenta — Especificação (EBNF)

## 1. Motivação
**Pimenta** é uma linguagem temática de cozinha: pequena, direta e “ardida”. A ideia é escrever “receitas” de alto nível e traduzi-las para uma VM simples (ex.: BotCarVM), como se estivéssemos pilotando uma cozinha robotizada. Apesar do vocabulário culinário (mexer, assar, provar, servir), a linguagem possui as estruturas fundamentais de programação: **variáveis, condicionais e loops**.

## 2. Escopo (MVP)
- **Tipos:** `ingred` (inteiro), `cond` (booleano).
- **Variáveis:** declaração e atribuição por bloco `{ }`.
- **Expressões:** aritmética (`+ - * / %`), relacionais (`< <= > >= == !=`), lógicas (`! && ||`).
- **Controle:** `provar`/`senao` (if/else), `mexerEnquanto` (while), `assarPor` (for).
- **I/O:** `servir(expr);`
- **Entrada do programa:** `receita principal() { ... }`
- **Sensores read-only:** `forno_temp`, `estoque`, `cheiro` (somente leitura).
- **Funções adicionais:** previstas na gramática, implementação futura.

---

## 3. Tabela de Tokens (Léxico)

| Categoria         | Token/Classe     | Lexema/Exemplo            | Observações |
|---                |---               |---                        |---|
| Palavra-chave     | `kw_ingred`      | `ingred`                  | tipo inteiro |
|                   | `kw_cond`        | `cond`                    | tipo booleano |
|                   | `kw_receita`     | `receita`                 | define função/receita |
|                   | `kw_principal`   | `principal`               | ponto de entrada |
|                   | `kw_provar`      | `provar`                  | `if` |
|                   | `kw_senao`       | `senao`                   | `else` |
|                   | `kw_mexer`       | `mexerEnquanto`           | `while` |
|                   | `kw_assar`       | `assarPor`                | `for` |
|                   | `kw_finalizar`   | `finalizar`               | `return` |
|                   | `kw_servir`      | `servir`                  | `print` |
| Sensores (RO)     | `sensor`         | `forno_temp`              | read-only |
|                   | `sensor`         | `estoque`                 | read-only |
|                   | `sensor`         | `cheiro`                  | read-only |
| Identificador     | `identifier`     | `massa`, `i`, `_tmp3`     | `[A-Za-z_][A-Za-z0-9_]*` |
| Inteiro           | `int_lit`        | `0`, `42`, `1005`         | decimal |
| Booleano          | `bool_lit`       | `true`, `false`           | |
| Pontuação         | `lpar` / `rpar`  | `(` `)`                   | |
|                   | `lbrace`/`rbrace`| `{` `}`                   | |
|                   | `lbrack`/`rbrack`| `[` `]`                   | |
|                   | `comma`          | `,`                       | |
|                   | `semi`           | `;`                       | |
| Atribuição        | `assign`         | `=`                       | |
| Lógicos           | `op_or`          | `||`                      | |
|                   | `op_and`         | `&&`                      | |
| Comparação        | `op_eq`/`op_ne`  | `==` `!=`                 | |
|                   | `op_lt`/`op_le`  | `<` `<=`                  | |
|                   | `op_gt`/`op_ge`  | `>` `>=`                  | |
| Aritméticos       | `op_add/sub`     | `+` `-`                   | |
|                   | `op_mul/div/mod` | `*` `/` `%`               | |
| Unário            | `op_not`         | `!`                       | |
| Comentários       | `comment_line`   | `// ...`                  | até fim da linha |
|                   | `comment_block`  | `/* ... */`               | pode ter quebras de linha |
| Espaço            | `ws`             | espaço, tab, newline      | ignorado |

> **Regra semântica importante:** tokens `sensor` são **reservados e read-only** — não podem ser LHS de `=`.

---

## 5. Gramática (EBNF)

```ebnf
(* ---------- LÉXICO / DEFINIÇÕES AUXILIARES ---------- *)

letter         = "A"…"Z" | "a"…"z" | "_" ;
digit          = "0"…"9" ;
alnum          = letter | digit ;

identifier     = letter , { alnum } ;

int_lit        = digit , { digit } ;
bool_lit       = "true" | "false" ;

ws             = { " " | "	" | "
" | "
" } ;
comment_line   = "//" , { ? any char except newline ? } , ( "
" | EOF ) ;
comment_block  = "/*" , { ? any char incl. newline, not "*/" ? } , "*/" ;

(* PALAVRAS-CHAVE (tema culinária) *)
kw_ingred      = "ingred" ;         (* inteiro *)
kw_cond        = "cond" ;           (* booleano *)
kw_receita     = "receita" ;
kw_principal   = "principal" ;
kw_provar      = "provar" ;         (* if *)
kw_senao       = "senao" ;          (* else *)
kw_mexer       = "mexerEnquanto" ;  (* while *)
kw_assar       = "assarPor" ;       (* for *)
kw_finalizar   = "finalizar" ;      (* return *)
kw_servir      = "servir" ;         (* print *)

(* PONTUAÇÃO / OPS *)
lpar           = "(" ;   rpar   = ")" ;
lbrace         = "{" ;   rbrace = "}" ;
lbrack         = "[" ;   rbrack = "]" ;
semi           = ";" ;   comma  = "," ;

assign         = "=" ;
op_or          = "||" ;
op_and         = "&&" ;
op_eq          = "==" ;
op_ne          = "!=" ;
op_lt          = "<" ;
op_le          = "<=" ;
op_gt          = ">" ;
op_ge          = ">=" ;
op_add         = "+" ;
op_sub         = "-" ;
op_mul         = "*" ;
op_div         = "/" ;
op_mod         = "%" ;
op_not         = "!" ;

(* SENSORES READ-ONLY *)
sensor         = "forno_temp" | "estoque" | "cheiro" ;

(* ---------- GRAMÁTICA DE ALTO NÍVEL ---------- *)

Programa       = ws , { DeclOrFunc } , kw_receita , kw_principal , "(" , ")" , Bloco ;

DeclOrFunc     = ( DeclVar | DeclFunc ) ;

Tipo           = kw_ingred | kw_cond ;

DeclVar        = Tipo , ListaVar , semi ;
ListaVar       = VarInit , { comma , VarInit } ;
VarInit        = identifier , [ assign , Expressao ] ;

(* LValue é um identificador comum; sensores não podem estar aqui (semântica). *)
LValue         = identifier ;

DeclFunc       = Tipo , identifier , "(" , [ ListaParams ] , ")" , Bloco ;
ListaParams    = Param , { comma , Param } ;
Param          = Tipo , identifier ;

Bloco          = lbrace , { Comando } , rbrace ;

Comando        = DeclVar
               | Atrib
               | Se
               | Enquanto
               | Para
               | Servir
               | Retorno
               | Bloco
               | semi  (* comando vazio *)
               ;

Atrib          = LValue , assign , Expressao , semi ;

Se             = kw_provar , "(" , Expressao , ")" , Comando , [ kw_senao , Comando ] ;

Enquanto       = kw_mexer , "(" , Expressao , ")" , Comando ;

Para           = kw_assar , "(" ,
                   [ ParaInit ] , semi ,
                   [ Expressao ] , semi ,
                   [ ParaUpdate ] ,
                 ")" , Comando ;

ParaInit       = DeclVar | AtribSemPonto | /* vazio */ ;
ParaUpdate     = AtribSemPonto | /* vazio */ ;
AtribSemPonto  = LValue , assign , Expressao ;

Servir         = kw_servir , "(" , Expressao , ")" , semi ;

Retorno        = kw_finalizar , [ Expressao ] , semi ;

(* EXPRESSÕES COM PRECEDÊNCIA *)

Expressao      = ExprOr ;

ExprOr         = ExprAnd , { op_or , ExprAnd } ;
ExprAnd        = ExprEq ,  { op_and , ExprEq } ;
ExprEq         = ExprRel , { (op_eq | op_ne) , ExprRel } ;
ExprRel        = ExprAdd , { (op_lt | op_le | op_gt | op_ge) , ExprAdd } ;
ExprAdd        = ExprMul , { (op_add | op_sub) , ExprMul } ;
ExprMul        = Unario  , { (op_mul | op_div | op_mod) , Unario } ;

Unario         = ( op_not | op_sub ) , Unario
               | Primario ;

Primario       = int_lit
               | bool_lit
               | identifier
               | sensor
               | Chamada
               | "(" , Expressao , ")"
               ;

Chamada        = identifier , "(" , [ ListaArgs ] , ")" ;
ListaArgs      = Expressao , { comma , Expressao } ;
```

---

## 6. Exemplos (tests)

### A) Variáveis + while + sensor
```pimenta
receita principal() {
  ingred passos = 0;
  mexerEnquanto (forno_temp > 0) {
    passos = passos + 1;
  }
  servir(passos);
  finalizar 0;
}
```

### B) For + booleanos + if/else
```pimenta
receita principal() {
  ingred soma = 0;
  assarPor (ingred i = 0; i < 10; i = i + 1) {
    soma = soma + i;
  }
  cond ok = (soma >= 45) && (soma <= 55);
  provar (ok) { servir(1); } senao { servir(0); }
  finalizar;
}
```

### C) Leitura de sensor read-only
```pimenta
receita principal() {
  ingred limite = 180;
  provar (forno_temp > limite) { servir(1); } senao { servir(0); }
  finalizar 0;
}
```
