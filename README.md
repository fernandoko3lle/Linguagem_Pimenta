# 🌶️ Pimenta — Linguagem temática de culinária

**Pimenta** é uma linguagem minimalista e temática de cozinha para ensinar/experimentar compiladores. Você escreve “receitas” de alto nível e o compilador gera **assembly** para uma VM simples (**BotCarVM**).

## Estrutura do projeto
```
Linguagem_Pimenta/
  docs/
    SPEC-EBNF.md
    VM-Resumo.md
  tests/
    A_variaveis_while.pim
    B_for_booleanos.pim
    C_sensores_ro.pim
```

### Passos
1. **Lexer (Flex)**: reconhece *tokens* (palavras-chave, identificadores, literais, operadores etc.).
2. **Parser (Bison)**: valida a gramática (EBNF) e constrói a AST.
3. **Geração de código**: AST → **assembly BotCarVM**.
4. **Execução**: Emulador interpreta `.bcar` e produz a saída (de `servir()`).

Mais detalhes em `docs/SPEC-EBNF.md` e `docs/VM-Resumo.md`.
