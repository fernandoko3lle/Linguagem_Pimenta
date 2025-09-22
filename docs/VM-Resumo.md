# VM Alvo — BotCarVM (Resumo para a parcial)

> A Pimenta compila para a **BotCarVM** (VM didática, simples e adequada ao +1 conceito de nova VM).

## 1. Componentes
- **Registradores:** `R0`, `R1` (32 bits).
- **PC** (program counter), **FLAGS**: `Z` (zero), `N` (negativo).
- **Memória RAM:** 64 KiB (endereçada por inteiros).
- **Sensores (read-only, mapeados em memória):**
  - `forno_temp` → `0xFF00`
  - `estoque` → `0xFF01`
  - `cheiro` → `0xFF02`
- **Saída:** `PRINT R` (stdout), `HALT`.

## 2. ISA (Instruções)
- **Memória/movimento**
  - `LOAD R, [addr]`  // R ← MEM[addr]
  - `STORE [addr], R` // MEM[addr] ← R  (proibido se addr for sensor)
  - `MOV Rdst, Rsrc`
- **Aritm./lógica** (atualizam `Z/N`)
  - `ADD R, imm|[addr]|R2`
  - `SUB R, imm|[addr]|R2`
  - `INC R`, `DEC R`
- **Comparação & Saltos**
  - `CMP R, imm|[addr]|R2`
  - `JZ label`, `JNZ label`, `JMP label`
- **Pilha (opcional para funções):** `PUSH R`, `POP R`
- **I/O & Controle:** `PRINT R`, `HALT`

## 3. Mapa de Memória (sugestão)
- `0x0000–0x7FFF` → variáveis/heap  
- `0x8000–0xFEFF` → livre/stack  
- `0xFF00–0xFF02` → sensores (RO)

## 4. Mapeamento de construções Pimenta → BotCarVM
- **Variáveis**: cada identificador recebe um endereço fixo (`STORE/LOAD`).
- **Sensores**: lidos via `LOAD [0xFF0x]` (sem `STORE`).
- **`provar (cond) … senao …`**:
  ```
  ; cond em R0
  CMP R0, 0
  JZ  L_else
    ; bloco-then
    JMP L_end
  L_else:
    ; bloco-else
  L_end:
  ```
- **`mexerEnquanto (cond) { corpo }`**:
  ```
  L_cond:
    ; avalia cond em R0
    CMP R0, 0
    JZ  L_end
    ; corpo
    JMP L_cond
  L_end:
  ```
- **`assarPor(init; cond; update) { corpo }`** ≡ `init; while(cond){ corpo; update; }`.
- **`servir(expr)`**: avalia `expr` em `R0` e `PRINT R0`.

## 5. Exemplo: Pimenta → Assembly BotCarVM

**Pimenta**
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

**Assembly (comentado)**
```
; passos em [0x0000]

; passos = 0
SUB R0, R0          ; R0 <- 0
STORE [0x0000], R0

L_cond:
LOAD R0, [0xFF00]   ; R0 = forno_temp
CMP R0, 0
JZ  L_end

LOAD R0, [0x0000]
INC R0
STORE [0x0000], R0

JMP L_cond

L_end:
LOAD R0, [0x0000]
PRINT R0
HALT
```
