# Banco de Dados Relacionais — Teoria e Prática
### Contexto: Plataforma de Streaming Musical (Spotify)

> **Como usar este material:**  
> Cada tópico segue sempre a mesma estrutura:  
> 🔷 **Definição Formal** → 💡 **Intuição** → 🎵 **Aplicação no Spotify**  
> Os exemplos de código têm comentários inline que explicam cada linha.

---

## Sumário

1. [Modelo Relacional](#1-modelo-relacional)
2. [Chaves](#2-chaves)
3. [Dependência Funcional](#3-dependência-funcional)
4. [Fechamento de Atributos](#4-fechamento-de-atributos)
5. [Equivalência de Dependências Funcionais](#5-equivalência-de-dependências-funcionais)
6. [Cobertura Canônica](#6-cobertura-canônica)
7. [Anomalias no Modelo Relacional](#7-anomalias-no-modelo-relacional)
8. [Mapeamento ER → Relacional](#8-mapeamento-er--relacional)
9. [Estratégias de Design e Normalização](#9-estratégias-de-design-e-normalização)
10. [Integração de Esquemas](#10-integração-de-esquemas)

---

## 1. Modelo Relacional

### 🔷 Definição Formal

O modelo relacional é fundamentado na **teoria dos conjuntos** e na **lógica de predicados de primeira ordem** (Codd, 1970).

> **Def. 1.1 — Domínio:** Um domínio `D` é um conjunto nomeado de valores atômicos (indivisíveis).  
> Exemplos: `D_int = {1,2,3,...}`, `D_plano = {'gratuito','premium'}`, `D_texto = {strings}`

> **Def. 1.2 — Esquema de Relação:** `R(A₁, A₂, ..., Aₙ)` — um nome `R` com lista de atributos, cada um associado a um domínio `dom(Aᵢ)`.

> **Def. 1.3 — Relação (instância):** Uma relação `r` sobre `R` é um **subconjunto finito** do produto cartesiano dos domínios:  
> `r ⊆ dom(A₁) × dom(A₂) × ... × dom(Aₙ)`  
> Cada elemento é uma **tupla** `t = (v₁, v₂, ..., vₙ)` onde `vᵢ ∈ dom(Aᵢ)`.

> **Def. 1.4 — Acesso a tupla:** `t[Aᵢ]` denota o valor da tupla `t` no atributo `Aᵢ`.  
> Para um conjunto `X = {Aᵢ, Aⱼ}`, `t[X]` denota a subtupla `(t[Aᵢ], t[Aⱼ])`.

**Propriedades obrigatórias de uma relação válida:**

| Propriedade | Formalização | O que significa na prática |
|---|---|---|
| **Atomicidade (1FN)** | `∀ t ∈ r, ∀ Aᵢ : t[Aᵢ] ∈ dom(Aᵢ)` | Nenhuma célula contém lista ou conjunto |
| **Unicidade de tuplas** | `∀ t₁, t₂ ∈ r : t₁ ≠ t₂` | `r` é conjunto — sem linhas duplicadas |
| **Ordem irrelevante** | `r` é conjunto, sem posição | Não existe "linha número 3" formalmente |
| **Atributos nomeados** | Acesso por nome, não por posição | `t[titulo]`, não `t[2]` |

> **Def. 1.5 — Grau e Cardinalidade:**  
> **Grau** = número de atributos `n` no esquema.  
> **Cardinalidade** = número de tuplas `|r|` na instância.

---

### 💡 Intuição

Pense numa planilha Excel onde:
- Cada **aba** é uma relação
- Cada **coluna** é um atributo com tipo fixo
- Cada **linha** é uma tupla — e **não pode se repetir**
- A **ordem das linhas não importa** matematicamente

A diferença crucial entre "tabela" (implementação física) e "relação" (conceito matemático): a relação é um **conjunto puro** — sem duplicatas, sem ordem — enquanto uma tabela pode ter ambas por razões de performance.

---

### 🎵 Aplicação no Spotify

```
Esquema formal:
  MUSICA(id_musica, titulo, duracao_seg, letra)

  dom(id_musica)   = ℤ⁺              ← inteiros positivos
  dom(titulo)      = VARCHAR(200)     ← strings até 200 chars
  dom(duracao_seg) = {n ∈ ℤ | n > 0} ← inteiro positivo obrigatório
  dom(letra)       = TEXT ∪ {NULL}   ← aceita ausência de valor

  Grau de MUSICA = 4   (4 atributos)
```

```
Instância r(MUSICA) — exemplo de 3 tuplas:

┌───────────┬─────────────────────────┬─────────────┬────────┐
│ id_musica │ titulo                  │ duracao_seg │ letra  │
├───────────┼─────────────────────────┼─────────────┼────────┤
│ 1         │ Bohemian Rhapsody       │ 354         │ ...    │  ← tupla t₁
│ 2         │ Blinding Lights         │ 200         │ ...    │  ← tupla t₂
│ 3         │ Shape of You            │ 234         │ NULL   │  ← letra ausente: válido
└───────────┴─────────────────────────┴─────────────┴────────┘

Cardinalidade atual: |r(MUSICA)| = 3
```

> ⚠️ **Violação de atomicidade — o caso dos gêneros:**  
> Se `generos` fosse `"pop, rock, eletrônico"` dentro de MUSICA, violaria a 1FN pois `dom(generos)` conteria listas.  
> **Solução no Spotify:** tabela separada `GENERO_MUSICA(id_genero, id_musica, nome)` — cada gênero em sua própria linha.

**Restrições de integridade disponíveis:**

| Restrição | Definição | Exemplo no Spotify |
|---|---|---|
| **Domínio** | `t[Aᵢ] ∈ dom(Aᵢ)` | `CHECK (plano IN ('gratuito','premium'))` |
| **Entidade** | PK `≠ NULL` sempre | `id_musica NOT NULL` |
| **Referencial** | FK aponta para PK existente | `REPRODUCAO.id_musica` deve existir em MUSICA |
| **Semântica** | Regras de negócio | `CHECK (duracao_seg > 0)` |

---

## 2. Chaves

### 🔷 Definição Formal

> **Def. 2.1 — Superchave:** `SK ⊆ atributos(R)` é superchave se:  
> `∀ t₁, t₂ ∈ r : t₁ ≠ t₂ ⟹ t₁[SK] ≠ t₂[SK]`  
> Nenhuma superchave pode produzir valores iguais em duas tuplas distintas.

> **Def. 2.2 — Chave Candidata:** Superchave **mínima** —  
> `CK` é superchave **E** `∀ X ⊊ CK : X` não é superchave.  
> Remover qualquer atributo destrói a propriedade de identificação única.

> **Def. 2.3 — Chave Primária (PK):** Chave candidata **escolhida** pelo designer.  
> Restrição adicional: `∀ t ∈ r : t[PK] ≠ NULL` (integridade de entidade).

> **Def. 2.4 — Chave Alternada:** Toda chave candidata **não escolhida** como primária.  
> Implementada com `UNIQUE NOT NULL`.

> **Def. 2.5 — Chave Estrangeira (FK):** `FK ⊆ atributos(R₁)` referenciando `PK` de `R₂`:  
> `∀ t₁ ∈ r₁ : t₁[FK] ≠ NULL ⟹ ∃ t₂ ∈ r₂ : t₁[FK] = t₂[PK₂]`  
> Todo valor não-nulo da FK deve corresponder a uma tupla existente em `R₂`.

> **Teorema 2.1:** Em qualquer relação `R`, existe ao menos uma chave candidata.  
> *(O conjunto total de atributos sempre é superchave; por minimização, existe ao menos uma candidata mínima.)*

---

### 💡 Intuição

A hierarquia de chaves é uma relação de especialização:

```
Superchave           → qualquer conjunto que identifica unicamente
  └─ Chave Candidata → superchave mínima (sem gordura)
       ├─ Chave Primária  → candidata escolhida (NOT NULL obrigatório)
       └─ Chave Alternada → candidata rejeitada (UNIQUE NOT NULL)

Chave Estrangeira    → "ponteiro" de uma tabela para a PK de outra
```

**Por que a minimalidade importa?**  
Uma PK com atributos desnecessários encarece índices, complica JOINs e pode mascarar dependências funcionais que gerariam anomalias. Cada atributo da PK deve ser semanticamente **necessário** para a identificação.

**Regra prática:** Se você pode remover um atributo da chave e ela ainda identifica unicamente — o atributo é extrâneo e deve sair.

---

### 🎵 Aplicação no Spotify

```
USUARIO(id_usuario, nome, email, data_nascimento, pais, plano)

Superchaves válidas (identificam unicamente):
  {id_usuario}                     ← mínima ✅
  {email}                          ← mínima ✅
  {id_usuario, nome}               ← nome é redundante — não-mínima
  {id_usuario, email, pais, plano} ← muito redundante — não-mínima
  {todos os atributos}             ← trivialmente superchave — não-mínima

Chaves candidatas (mínimas):
  CK₁ = {id_usuario}  → remover id_usuario → {} não identifica nada ✅
  CK₂ = {email}       → emails são únicos por regra de negócio ✅

PK escolhida:    id_usuario  (inteiro — eficiente para índices e JOINs)
Chave alternada: email       (UNIQUE NOT NULL — ainda garante unicidade)
```

```
REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo)

Chaves candidatas:
  CK₁ = {id_reproducao}
        ↳ surrogate key, gerado automaticamente, garante unicidade ✅

  CK₂ = {id_usuario, id_musica, data_hora}
        ↳ um mesmo usuário não pode reproduzir a mesma música
          exatamente no mesmo instante de tempo ✅

PK escolhida: id_reproducao  (mais simples para referenciar em FKs futuras)
```

```
Tabelas associativas — PK composta:

USUARIO_CURTE_MUSICA
  PK = (id_usuario, id_musica)
  ↳ um usuário pode curtir cada música no máximo 1 vez (par único)
  ↳ remover id_usuario → múltiplos usuários para mesma música → não identifica
  ↳ remover id_musica  → múltiplas músicas para mesmo usuário → não identifica
  → ambos os atributos são necessários na PK composta ✅
```

```sql
-- Implementação SQL das restrições de chave no Spotify

CREATE TABLE USUARIO (
    id_usuario  INTEGER      PRIMARY KEY,                   -- PK: NOT NULL + UNIQUE
    email       VARCHAR(150) NOT NULL UNIQUE,               -- Chave alternada
    nome        VARCHAR(100) NOT NULL,
    plano       VARCHAR(20)  DEFAULT 'gratuito'
                             CHECK (plano IN ('gratuito','premium'))
);

CREATE TABLE REPRODUCAO (
    id_reproducao    INTEGER   PRIMARY KEY,
    id_usuario       INTEGER   NOT NULL
                     REFERENCES USUARIO(id_usuario)          -- FK → USUARIO
                     ON DELETE CASCADE,                      -- deletar usuário → remove plays
    id_musica        INTEGER   NOT NULL
                     REFERENCES MUSICA(id_musica)            -- FK → MUSICA
                     ON DELETE RESTRICT,                     -- não pode deletar música com plays
    data_hora        TIMESTAMP NOT NULL,
    segundos_ouvidos INTEGER   NOT NULL,
    dispositivo      VARCHAR(30)
);

CREATE TABLE USUARIO_CURTE_MUSICA (
    id_usuario   INTEGER REFERENCES USUARIO(id_usuario),    -- FK₁
    id_musica    INTEGER REFERENCES MUSICA(id_musica),      -- FK₂
    data_curtida TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (id_usuario, id_musica)                     -- PK composta
);
```

**Ações referenciais ao deletar/atualizar:**

| Ação | Comportamento | Uso no Spotify |
|---|---|---|
| `CASCADE` | Propaga a operação | Deletar usuário → remove suas reproduções |
| `RESTRICT` | Bloqueia a operação | Não deleta música se houver plays |
| `SET NULL` | Anula a FK | Somente se FK aceitar NULL |
| `SET DEFAULT` | Atribui valor padrão | Casos raros |

> **Violação de integridade referencial — exemplo:**  
> `INSERT INTO REPRODUCAO (id_usuario=999, ...)` onde usuário 999 não existe em USUARIO  
> → SGBD **rejeita** automaticamente — nenhuma reprodução "órfã" é possível.

---

## 3. Dependência Funcional

### 🔷 Definição Formal

> **Def. 3.1 — Dependência Funcional (DF):**  
> Sejam `X, Y ⊆ atributos(R)`. Dizemos que **X determina Y** (`X → Y`) se:  
> `∀ r` instância válida de `R`, `∀ t₁, t₂ ∈ r : t₁[X] = t₂[X] ⟹ t₁[Y] = t₂[Y]`  
> Para cada valor de X, existe **exatamente um** valor possível de Y.

> **Def. 3.2 — DF Trivial:** `X → Y` é trivial se `Y ⊆ X` — sempre satisfeita, sem restrição real.

> **Def. 3.3 — DF Não-Trivial:** `X → Y` com `Y ⊄ X` — impõe restrição real sobre os dados.

> **Def. 3.4 — Dependência Parcial:**  
> Atributo não-primo `B` tem dependência parcial se `∃ X ⊊ PK : X → B`.  
> (B depende de **parte** da chave composta, não da chave inteira.) → **viola 2FN**

> **Def. 3.5 — Dependência Transitiva:**  
> `B` depende transitivamente de `K` se: `K → Y`, `Y → B`, `Y` não é chave.  
> (B não depende diretamente da chave — vai por um intermediário não-chave.) → **viola 3FN**

**Axiomas de Armstrong** (sistema sólido e completo para derivar DFs):

| Axioma | Regra | Exemplo no Spotify |
|---|---|---|
| **A1 — Reflexividade** | `Y ⊆ X ⟹ X → Y` | `{id_musica, titulo} → titulo` (trivial) |
| **A2 — Aumentatividade** | `X → Y ⟹ XZ → YZ` | `id_musica → titulo` ⟹ `{id_musica, duracao_seg} → {titulo, duracao_seg}` |
| **A3 — Transitividade** | `X → Y ∧ Y → Z ⟹ X → Z` | `id_assinatura → tipo_plano` e `tipo_plano → valor_pago` ⟹ `id_assinatura → valor_pago` |

**Regras derivadas** (provadas a partir dos axiomas):

| Regra | Enunciado |
|---|---|
| **R1 — União** | `X→Y ∧ X→Z ⟹ X→YZ` |
| **R2 — Decomposição** | `X→YZ ⟹ X→Y ∧ X→Z` |
| **R3 — Pseudotransitividade** | `X→Y ∧ WY→Z ⟹ WX→Z` |

---

### 💡 Intuição

Uma DF `X → Y` é uma **lei de negócio** expressa matematicamente:  
"Dado o valor de X, o valor de Y está completamente determinado."

- `id_musica → titulo` diz: "conhecendo o ID da música, sabemos exatamente seu título"
- `titulo → id_musica` **não vale**: duas músicas diferentes podem ter o mesmo título

**Importante:** DFs são **restrições semânticas**, não observações dos dados. Ver uma instância onde `titulo → id_musica` parece valer não prova que a DF é válida — apenas uma regra de negócio confirma isso.

**Os três tipos problemáticos:**
```
Parcial (viola 2FN):
  PK composta = (id_playlist, id_musica)
  id_musica → titulo_musica   ← titulo depende só de PARTE da PK
  Solução: titulo_musica vai para tabela MUSICA separada

Transitiva (viola 3FN):
  id_assinatura → tipo_plano → valor_pago
  valor_pago não depende diretamente da PK, vai por tipo_plano
  Solução: criar tabela PLANO(tipo_plano, valor_pago)

Trivial (inofensiva):
  {id_musica, titulo} → titulo   ← titulo já está no lado esquerdo
  Sempre satisfeita, não impõe restrição real
```

---

### 🎵 Aplicação no Spotify

```
─── MUSICA(id_musica, titulo, duracao_seg, letra) ───

DFs válidas (regras de negócio garantem):
  id_musica → titulo          ✅  cada ID tem exatamente um título
  id_musica → duracao_seg     ✅  cada ID tem exatamente uma duração
  id_musica → letra           ✅  mesmo que seja NULL

  Por Regra R1 (União):
  id_musica → {titulo, duracao_seg, letra}  ✅  determinação do conjunto inteiro

DFs inválidas (contraexemplos existem nos dados):
  titulo → id_musica          ❌  "Perfect" do Ed Sheeran ≠ "Perfect" de outro artista
  duracao_seg → titulo        ❌  milhares de músicas têm 200 segundos
  duracao_seg → id_musica     ❌  idem

DF trivial (sempre satisfeita, sem restrição):
  {id_musica, titulo} → titulo  ← titulo ⊆ {id_musica, titulo} → trivial
```

```
─── ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago) ───

DFs válidas:
  id_assinatura → id_usuario      ✅  cada assinatura pertence a um usuário
  id_assinatura → tipo_plano      ✅
  id_assinatura → data_inicio     ✅
  id_assinatura → data_fim        ✅  NULL = assinatura ativa
  tipo_plano    → valor_pago      ✅  regra de negócio: premium = R$21,90 sempre

  ⚠️  DEPENDÊNCIA TRANSITIVA detectada:
  id_assinatura → tipo_plano → valor_pago
  valor_pago depende de id_assinatura via tipo_plano (intermediário não-chave)
  → viola 3FN → solução: criar PLANO(tipo_plano, valor_pago)

DFs inválidas:
  id_usuario → tipo_plano     ❌  usuário pode ter histórico de planos diferentes
  data_inicio → id_usuario    ❌  múltiplas assinaturas começam na mesma data
```

```
─── PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao) ───
PK composta: (id_playlist, id_musica)

DFs:
  (id_playlist, id_musica) → ordem       ✅  par único determina posição
  (id_playlist, id_musica) → data_adicao ✅  par único determina quando foi adicionada

  id_playlist → ordem    ❌  mesma playlist tem músicas em posições 1, 2, 3...
  id_musica → ordem      ❌  mesma música pode ser posição 1 numa playlist, 7 em outra

  ✅  Sem dependências parciais → satisfaz 2FN
  ✅  Sem dependências transitivas → satisfaz 3FN
```

**Conexão formal entre chaves e DFs:**

> **Teorema 3.1:** `X` é superchave de `R`  ⟺  `X → R` (X determina todos os atributos).  
> **Teorema 3.2:** `X` é chave candidata  ⟺  `X → R` **e** `∀ Y ⊊ X : Y ↛ R`.

```
Verificação:
  {id_musica} → {titulo, duracao_seg, letra}  e  id_musica → id_musica (trivial)
  → {id_musica} → R  →  é superchave
  → |{id_musica}| = 1 → mínima automaticamente → é chave candidata ✅
```

---

## 4. Fechamento de Atributos

### 🔷 Definição Formal

> **Def. 4.1 — Fechamento de F:** O **fechamento de F** (notação: `F⁺`) é o conjunto de **todas** as DFs deriváveis de `F` via Axiomas de Armstrong:  
> `F⁺ = { X → Y | F ⊢ X → Y }`

> **Def. 4.2 — Fechamento de X:** O **fechamento de X em relação a F** (notação: `X⁺`) é:  
> `X⁺ = { A | X → A ∈ F⁺ }`  
> Todos os atributos que X determina direta ou transitivamente.

> **Teorema 4.1 — Utilidade central:**  
> `X → Y ∈ F⁺  ⟺  Y ⊆ X⁺`  
> *(Verificar se X determina Y equivale a checar se Y está no fechamento de X — muito mais eficiente que computar F⁺ inteiro.)*

> **Corolário 4.1:** `X` é superchave de `R`  ⟺  `X⁺ = atributos(R)`

> **Corolário 4.2:** `X` é chave candidata  ⟺  `X⁺ = atributos(R)` **e** `∀ A ∈ X : (X−{A})⁺ ≠ atributos(R)`

**Algoritmo CLOSURE(X, F):**
```
Entrada: X ⊆ atributos(R),  F = conjunto de DFs
Saída:   X⁺

1. resultado ← X                     ← começa com o próprio X
2. repita até não mudar:
     para cada DF (α → β) em F:
       se α ⊆ resultado:             ← se o lado esquerdo já foi alcançado
         resultado ← resultado ∪ β   ← adiciona o lado direito
3. retornar resultado
```

Complexidade: O(|F| · |atributos|²) — **polinomial**, eficiente na prática.

---

### 💡 Intuição

O fechamento `X⁺` responde à pergunta:  
**"A partir de X, o que mais consigo descobrir usando as regras do sistema?"**

Pense numa cadeia de deduções:  
- Sei o `id_reproducao` → descubro `id_usuario`, `id_musica`, `data_hora`  
- Com `id_usuario + id_musica + data_hora` → descubro `segundos_ouvidos` e `dispositivo`  
- Resultado: a partir de `id_reproducao`, chego a **todos** os atributos → é chave!

O **Teorema 4.1** é o ganho prático: em vez de computar `F⁺` (potencialmente exponencial), basta calcular `X⁺` (polinomial) para verificar qualquer DF.

---

### 🎵 Aplicação no Spotify

```
─── Calcular {id_reproducao}⁺ ───
Abreviações: R=id_reproducao, U=id_usuario, M=id_musica, D=data_hora, S=segundos, V=dispositivo

Conjunto F:
  F1: R → U           F2: R → M           F3: R → D
  F4: (U,M,D) → S     F5: (U,M,D) → V

Passo a passo:
  Início:     resultado = {R}
  F1 aplica:  R ⊆ {R} ✓          → resultado = {R, U}
  F2 aplica:  R ⊆ {R, U} ✓       → resultado = {R, U, M}
  F3 aplica:  R ⊆ {R, U, M} ✓    → resultado = {R, U, M, D}
  F4 aplica:  {U,M,D} ⊆ resultado → resultado = {R, U, M, D, S}
  F5 aplica:  {U,M,D} ⊆ resultado → resultado = {R, U, M, D, S, V}
  ─── nenhuma DF adiciona mais → encerra ───

  {R}⁺ = {R, U, M, D, S, V} = todos os atributos ✅
  → id_reproducao é SUPERCHAVE → e como tem 1 atributo, é CHAVE CANDIDATA
```

```
─── Verificar se (id_usuario, id_musica, data_hora) é chave candidata ───

Calcular {U, M, D}⁺:
  Início:     resultado = {U, M, D}
  F4 aplica:  {U,M,D} ⊆ {U,M,D} ✓  → resultado = {U, M, D, S}
  F5 aplica:  {U,M,D} ⊆ resultado ✓ → resultado = {U, M, D, S, V}
  F1: R ⊆ resultado? R ∉ {U,M,D,S,V} ✗ → não aplica
  ─── encerra ───

  {U,M,D}⁺ = {U, M, D, S, V}  → R não está! → NÃO é superchave de R completo
  (Se removêssemos id_reproducao do esquema, {U,M,D} seria a única chave)
```

```
─── Verificar se id_assinatura → valor_pago ∈ F⁺ ───

F de ASSINATURA inclui:
  G1: id_assinatura → tipo_plano
  G2: tipo_plano → valor_pago

Calcular {id_assinatura}⁺:
  Início:     resultado = {id_assinatura}
  G1 aplica:  → resultado = {id_assinatura, tipo_plano}
  G2 aplica:  tipo_plano ∈ resultado → resultado = {id_assinatura, tipo_plano, valor_pago}

  valor_pago ∈ resultado ✅
  → id_assinatura → valor_pago ∈ F⁺  (por transitividade via tipo_plano)
  → dependência TRANSITIVA detectada → viola 3FN
```

```
─── Testar minimalidade da PK composta de PLAYLIST_MUSICA ───
PK = (id_playlist, id_musica)
F = { (id_playlist,id_musica)→ordem, (id_playlist,id_musica)→data_adicao }

Tentar remover id_playlist:
  {id_musica}⁺ = {id_musica}  → ordem ∉ resultado ✗ → id_playlist é necessário

Tentar remover id_musica:
  {id_playlist}⁺ = {id_playlist} → ordem ∉ resultado ✗ → id_musica é necessário

→ PK composta (id_playlist, id_musica) é chave candidata mínima ✅
```

---

## 5. Equivalência de Dependências Funcionais

### 🔷 Definição Formal

> **Def. 5.1 — Cobertura:** `F` **cobre** `G` (notação: `G ⊆ F⁺`) se:  
> `∀ (X → Y) ∈ G : Y ⊆ X⁺_F`  
> (Todo X → Y de G pode ser derivado usando apenas as DFs de F.)

> **Def. 5.2 — Equivalência:** `F ≡ G` se e somente se `F` cobre `G` **e** `G` cobre `F`.  
> Equivalentemente: `F⁺ = G⁺` — ambos os conjuntos derivam exatamente as mesmas DFs.

> **Teorema 5.1:** `F ≡ G ⟺ F⁺ = G⁺`

**Algoritmo para verificar se F ≡ G:**
```
1. Para cada (X → Y) em G:
     Calcular X⁺ usando apenas F
     Se Y ⊄ X⁺: retornar "F NÃO cobre G" ✗

2. Para cada (X → Y) em F:
     Calcular X⁺ usando apenas G
     Se Y ⊄ X⁺: retornar "G NÃO cobre F" ✗

3. Retornar "F ≡ G" ✅
```

---

### 💡 Intuição

Dois conjuntos de DFs são equivalentes quando capturam a **mesma semântica**, mesmo escritos de formas diferentes.

É como ter duas formas de escrever a mesma lei:
- Lei A: "Imposto = 10% do salário; salário determina cargo"
- Lei B: "Imposto = 10% do salário; salário determina cargo; salário determina imposto"

Lei B tem uma regra a mais, mas ela é **derivável** das duas primeiras — as leis são equivalentes.

**Por que isso importa na prática?**  
Ao normalizar, substituímos F pela sua **cobertura canônica** (seção 6) — um conjunto equivalente mais enxuto — sem perder nenhuma restrição de integridade.

---

### 🎵 Aplicação no Spotify

```
─── ASSINATURA: F (expandido) ≡ G (compacto)? ───

Conjunto F (todas as DFs explicitadas):
  F1: id_assinatura → id_usuario
  F2: id_assinatura → tipo_plano
  F3: id_assinatura → data_inicio
  F4: id_assinatura → data_fim
  F5: id_assinatura → valor_pago    ← derivável transitivamente
  F6: tipo_plano → valor_pago

Conjunto G (forma compacta):
  G1: id_assinatura → {id_usuario, tipo_plano, data_inicio, data_fim}
  G2: tipo_plano → valor_pago       ← a raiz da transitividade fica explícita
```

```
Verificar F cobre G:

  G1: X = {id_assinatura}, Y = {id_usuario, tipo_plano, data_inicio, data_fim}
    {id_assinatura}⁺ via F:
      F1 → +id_usuario
      F2 → +tipo_plano
      F3 → +data_inicio
      F4 → +data_fim
      F5 → +valor_pago
      F6: tipo_plano ∈ resultado → +valor_pago (já estava)
    = {id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago}
    Y ⊆ resultado ✅

  G2: X = {tipo_plano}, Y = {valor_pago}
    {tipo_plano}⁺ via F: F6 → {tipo_plano, valor_pago}
    valor_pago ∈ resultado ✅

  → F cobre G ✅
```

```
Verificar G cobre F:

  F5: X = {id_assinatura}, Y = {valor_pago}
    {id_assinatura}⁺ via G:
      G1 → +id_usuario, +tipo_plano, +data_inicio, +data_fim
      G2: tipo_plano ∈ resultado → +valor_pago  ← chega via G2!
    valor_pago ∈ resultado ✅

  (F1~F4, F6: análogo — todos cobertos por G) ✅

  → G cobre F ✅

Conclusão: F ≡ G ✅
G é mais compacto mas representa a mesma semântica.
```

```
─── Exemplo de NÃO equivalência ───

H1 = { id_musica → titulo,
        id_musica → duracao_seg }

H2 = { id_musica → titulo }   ← perdeu a DF sobre duração

H1 cobre H2? ✅ (H2 ⊂ H1 literalmente)

H2 cobre H1?
  id_musica → duracao_seg ∈ H1
  {id_musica}⁺ via H2 = {id_musica, titulo}
  duracao_seg ∉ {id_musica, titulo} ✗

H1 ≢ H2  → H2 perdeu informação semântica sobre duração
```

---

## 6. Cobertura Canônica

### 🔷 Definição Formal

> **Def. 6.1 — Atributo Extrâneo (lado esquerdo):** `A ∈ X` é extrâneo em `X → Y` se:  
> `Y ⊆ (X − {A})⁺_F`  (Y ainda é determinado sem A)

> **Def. 6.2 — Atributo Extrâneo (lado direito):** `A ∈ Y` é extrâneo em `X → Y` se:  
> A ainda pode ser derivado com `X → (Y−{A})` no lugar de `X → Y`.

> **Def. 6.3 — Cobertura Canônica (Fc):** `Fc` é a cobertura canônica de `F` se:  
> 1. `Fc ≡ F` (equivalentes)  
> 2. Nenhuma DF de `Fc` tem atributo extrâneo no lado esquerdo  
> 3. Nenhuma DF de `Fc` tem atributo extrâneo no lado direito  
> 4. Nenhuma DF de `Fc` é redundante  
> 5. Cada DF tem **exatamente 1 atributo** no lado direito (forma canônica)

> **Teorema 6.1:** Todo conjunto finito `F` possui ao menos uma cobertura canônica.  
> *(A Fc pode não ser única — múltiplas formas mínimas podem existir.)*

> **Teorema 6.2 — Uso na 3FN:** O algoritmo de síntese 3FN usa `Fc` como entrada:  
> para cada `X → A` em `Fc`, cria-se `Rᵢ(X ∪ {A})` com chave `X`.

**Algoritmo CANONICAL_COVER(F):**
```
Passo 1 — Decomposição (lado direito atômico):
  X → {A, B, C}  →  X→A, X→B, X→C

Passo 2 — Remover atributos extrâneos no lado esquerdo:
  Para cada DF X→A com |X|>1:
    Para cada B ∈ X:
      Se A ∈ (X−{B})⁺:        ← B não é necessário
        Substituir X por X−{B}

Passo 3 — Remover DFs redundantes:
  Para cada DF X→A:
    Se A ∈ X⁺ calculado com F−{X→A}: ← A ainda é alcançável sem esta DF
      Remover X→A de F

Retornar F como Fc
```

---

### 💡 Intuição

A cobertura canônica é a **versão mínima** de um conjunto de DFs — sem gordura, sem redundâncias, sem atributos desnecessários.

É como simplificar uma equação:  
`2x + 4 = 10  →  x = 3`  
Ambas expressam a mesma restrição, mas a segunda é a forma mínima.

**Por que isso importa?**  
A normalização em 3FN usa `Fc` para criar o **menor número possível de tabelas**, cada uma com uma chave bem definida. Usar `F` em vez de `Fc` poderia gerar tabelas duplicadas ou com estrutura inflada.

**Os três problemas que Fc elimina:**
```
1. Atributo extrâneo no lado esquerdo:
   (id_usuario, id_musica, data_hora) → segundos  com  id_musica sozinho → segundos
   → id_usuario e data_hora são extrâneos se id_musica já determina sozinho
   (não é o caso no Spotify, mas ilustra o conceito)

2. DF redundante:
   { R→U, R→M, R→D, (U,M,D)→S,  R→S }
   R→S é redundante: via R→U, R→M, R→D e então (U,M,D)→S chegamos a S mesmo sem R→S

3. Lado direito composto:
   R → {U, M, D}  →  R→U, R→M, R→D  (decompõe para forma canônica)
```

---

### 🎵 Aplicação no Spotify

```
─── Exemplo 1: REPRODUCAO com DFs redundantes ───
Abreviações: R=id_reproducao, U=id_usuario, M=id_musica, D=data_hora, S=segundos, V=dispositivo

F inicial (contém redundâncias):
  DF1: R→U    DF2: R→M    DF3: R→D
  DF4: R→S    ← suspeita de redundância
  DF5: R→V    ← suspeita de redundância
  DF6: (U,M,D)→S
  DF7: (U,M,D)→V
  DF8: R→{U,M,D}   ← lado direito composto

─── Passo 1: Decomposição ───
  DF8: R→{U,M,D}  →  DF1(R→U), DF2(R→M), DF3(R→D) já existem → remover DF8

─── Passo 3: Testar redundância de DF4 (R→S) ───
  F' = F − {R→S}
  {R}⁺ via F':
    DF1: +U → DF2: +M → DF3: +D         ← chegou ao antecedente de DF6
    DF6: {U,M,D} ⊆ resultado → +S ✅    ← S ainda alcançável!
  → R→S é REDUNDANTE → remover DF4

─── Passo 3: Testar redundância de DF5 (R→V) ───
  {R}⁺ via F sem DF4 e DF5:
    DF1+DF2+DF3 → {R,U,M,D} → DF7: +V ✅
  → R→V é REDUNDANTE → remover DF5

─── Passo 2: Testar extrâneos em (U,M,D)→S ───
  Remover U: {M,D}⁺ = {M,D} → S ∉ resultado → U necessário ✅
  Remover M: {U,D}⁺ = {U,D} → S ∉ resultado → M necessário ✅
  Remover D: {U,M}⁺ = {U,M} → S ∉ resultado → D necessário ✅

Fc = { R→U,  R→M,  R→D,
       (U,M,D)→S,  (U,M,D)→V }

← Interpretação: id_reproducao determina "quem, o quê, quando";
                 os valores de segundos e dispositivo emergem da chave natural.
```

```
─── Exemplo 2: ASSINATURA com dependência transitiva ───

F original:
  id_assinatura → id_usuario
  id_assinatura → tipo_plano
  id_assinatura → data_inicio
  id_assinatura → data_fim
  id_assinatura → valor_pago    ← testar redundância
  tipo_plano → valor_pago

─── Passo 3: Testar id_assinatura → valor_pago ───
  F' = F − {id_assinatura→valor_pago}
  {id_assinatura}⁺ via F':
    +id_usuario, +tipo_plano, +data_inicio, +data_fim
    tipo_plano ∈ resultado → tipo_plano→valor_pago: +valor_pago ✅
  → id_assinatura→valor_pago é REDUNDANTE → remover

Fc = { id_assinatura → id_usuario,
       id_assinatura → tipo_plano,
       id_assinatura → data_inicio,
       id_assinatura → data_fim,
       tipo_plano → valor_pago }    ← 5 DFs em vez de 6

─── Implicação: Fc revela violação de 3FN ───
  tipo_plano → valor_pago  com tipo_plano não sendo superchave
  → normalizar: separar PLANO(tipo_plano PK, valor_pago)
                         ASSINATURA(id_assinatura, id_usuario, tipo_plano FK, data_inicio, data_fim)
```

```
─── Exemplo 3: PLAYLIST_MUSICA — já é canônica ───

F = { (id_playlist,id_musica)→ordem, (id_playlist,id_musica)→data_adicao }

Passo 1: Lados direitos já atômicos ✅
Passo 2: Testado na seção 4 — sem extrâneos ✅
Passo 3: Remover qualquer DF deixa a outra sem cobrir o atributo removido ✅

Fc = F  (já estava na forma canônica)
```

**Conexão com as Formas Normais:**

| Fc revela... | Forma normal violada | Solução |
|---|---|---|
| `X→A` com `X ⊊ PK` (dep. parcial) | **2FN** | Mover A para tabela onde X é PK |
| `X→A` com `X` não-chave transitivo | **3FN** | Criar tabela para X→A |
| `X→A` com `X` não-superchave | **BCNF** | Decompor em torno de X→A |

---

## 7. Anomalias no Modelo Relacional

### 🔷 Definição Formal

> **Def. 7.1 — Redundância:** Um esquema `R` possui redundância se `∃ X → Y ∈ F⁺` com `X` não sendo superchave e `Y ⊄ X`. Um não-superchave determina outros atributos, forçando repetição de valores em múltiplas tuplas.

> **Def. 7.2 — 1FN:** `R` está em 1FN se todos os domínios contêm apenas valores atômicos — sem listas, conjuntos ou relações aninhadas.

> **Def. 7.3 — 2FN:** `R` está em 1FN **e** todo atributo não-primo tem dependência **total** de cada chave candidata — sem dependências parciais.  
> Formalmente: `∄ X ⊊ CK, ∄ B não-primo : X → B ∈ F⁺`

> **Def. 7.4 — 3FN:** `R` está em 2FN **e** para toda `X → A ∈ F⁺` não-trivial: `X` é superchave **ou** `A` é atributo primo (pertence a alguma chave candidata).

> **Def. 7.5 — BCNF:** Para toda `X → A ∈ F⁺` não-trivial: `X` é superchave. *(Mais restrita que 3FN — elimina todas as anomalias.)*

> **Teorema 7.1:** Se `R` está em BCNF, então `R` não possui anomalias de inserção, remoção ou atualização.  
> *(Contraposição: anomalias existem → R não está em BCNF.)*

**Hierarquia das formas normais:**

```
1FN ⊃ 2FN ⊃ 3FN ⊃ BCNF
  ↑       ↑       ↑       ↑
sem    sem dep. sem dep. todo
listas  parcial  transit. determin.
                          é superchave
```

---

### 💡 Intuição

Anomalias são **consequências diretas de misturar entidades diferentes em uma mesma tabela**. Sempre que um não-superchave determina outros atributos, os mesmos valores se repetem em várias linhas — e aí surgem os problemas.

Há três tipos de anomalia:

| Tipo | O problema | Pergunta diagnóstica |
|---|---|---|
| **Inserção** | Não consigo cadastrar X sem ter Y | "Preciso de outra entidade para inserir esta?" |
| **Remoção** | Deletar X apaga Y junto sem querer | "Deletar esta linha perde dados de outra entidade?" |
| **Atualização** | Mudar X exige atualizar N linhas | "Este fato aparece repetido em múltiplas linhas?" |

**Regra prática:** se ao responder "sim" para qualquer uma das perguntas acima, o esquema precisa ser normalizado.

---

### 🎵 Aplicação no Spotify

Imagine que, em vez de tabelas separadas, tivéssemos **uma única tabela desnormalizada**:

```
FAIXA_COMPLETA(
  id_musica, titulo_musica, duracao_seg,
  id_album, titulo_album, data_lancamento,
  id_artista, nome_artista, pais_artista,
  faixa_numero
)

Instância de exemplo:
┌──────────┬───────────────────┬──────────┬──────────────┬──────────────────┬────────────┬────────────┬──────────────┐
│ id_musica│ titulo_musica     │ id_album │ titulo_album │ data_lancamento   │ id_artista │ nome_artist│ pais_artista │
├──────────┼───────────────────┼──────────┼──────────────┼──────────────────┼────────────┼────────────┼──────────────┤
│ 1        │ Blinding Lights   │ 10       │ After Hours  │ 2020-03-20       │ 5          │ The Weeknd │ CA           │
│ 2        │ Starboy           │ 11       │ Starboy      │ 2016-11-25       │ 5          │ The Weeknd │ CA           │ ← nome e pais
│ 3        │ Save Your Tears   │ 10       │ After Hours  │ 2020-03-20       │ 5          │ The Weeknd │ CA           │ ← repetidos 3x
│ 4        │ Bohemian Rhapsody │ 20       │ News o.World │ 1977-10-31       │ 8          │ Queen      │ GB           │
└──────────┴───────────────────┴──────────┴──────────────┴──────────────────┴────────────┴────────────┴──────────────┘

DFs problemáticas:
  id_album → titulo_album, data_lancamento   (id_album não é superchave)
  id_artista → nome_artista, pais_artista    (id_artista não é superchave)
  → Viola BCNF e 3FN → anomalias garantidas
```

```
─── Anomalia de Inserção ───

Cenário: cadastrar artista "Doja Cat" (id=99) que ainda não lançou músicas.

INSERT INTO FAIXA_COMPLETA VALUES
  (NULL, NULL, NULL, NULL, NULL, NULL, 99, 'Doja Cat', 'US', NULL)

→ id_musica faz parte da PK → NOT NULL → INSERT rejeitado ✗
→ Impossível registrar a artista sem uma música associada

No modelo normalizado (ARTISTA separado):
  INSERT INTO ARTISTA VALUES (99, 'Doja Cat', 'US', NULL, 'solo') ✅
  ← artista existe independentemente de ter músicas cadastradas
```

```
─── Anomalia de Remoção ───

Cenário: remover "Bohemian Rhapsody" (única linha do álbum "News of the World").

DELETE FROM FAIXA_COMPLETA WHERE id_musica = 4

→ Junto com a música, perdemos:
   - Dados do álbum "News of the World" (data 1977-10-31)
   - Dados de Queen (id=8, pais GB) — se for a última linha do artista

No modelo normalizado:
  DELETE FROM MUSICA WHERE id_musica = 4
  → ALBUM e ARTISTA permanecem intactos ✅
  ← cada entidade tem sua própria tabela — não há perda colateral
```

```
─── Anomalia de Atualização ───

Cenário: The Weeknd (id=5) atualiza país de 'CA' para 'US'.

UPDATE FAIXA_COMPLETA SET pais_artista = 'US' WHERE id_artista = 5
→ 3 linhas precisam ser atualizadas (músicas 1, 2 e 3)

Se por engano apenas a linha 1 for atualizada:
┌──────────┬───────────────────┬────────────┬──────────────┐
│ id_musica│ titulo_musica     │ id_artista │ pais_artista │
├──────────┼───────────────────┼────────────┼──────────────┤
│ 1        │ Blinding Lights   │ 5          │ US  ← novo   │
│ 2        │ Starboy           │ 5          │ CA  ← ANTIGO │ ← inconsistente!
│ 3        │ Save Your Tears   │ 5          │ CA  ← ANTIGO │ ← inconsistente!
└──────────┴───────────────────┴────────────┴──────────────┘

No modelo normalizado:
  UPDATE ARTISTA SET pais_origem = 'US' WHERE id_artista = 5
  → 1 UPDATE, 1 linha, consistência garantida ✅
  ← o fato "país do artista" existe em apenas 1 lugar
```

**Diagnóstico formal de FAIXA_COMPLETA:**

| Forma Normal | Satisfaz? | Motivo |
|---|---|---|
| **1FN** | ✅ | Valores atômicos (gêneros já foram extraídos) |
| **2FN** | ❌ | `id_artista → nome_artista` — id_artista é parte da PK composta |
| **3FN** | ❌ | Consequência direta da violação de 2FN |
| **BCNF** | ❌ | `id_album → titulo_album` com id_album não sendo superchave |

---

## 8. Mapeamento ER → Relacional

### 🔷 Definição Formal

> **Def. 8.1 — Mapeamento Preservador:** Um mapeamento `M: ER → Relacional` é preservador se toda instância válida do DER tem representação válida no relacional e vice-versa — sem perda semântica.

> **Def. 8.2 — Decomposição Sem Perda (Lossless Join):**  
> Decomposição de `R` em `{R₁, R₂}` é sem perda se:  
> `r = π_{R₁}(r) ⋈ π_{R₂}(r)` para toda instância `r`.  
> **Condição suficiente (Teorema de Heath):** A decomposição é sem perda se `R₁ ∩ R₂ → R₁ ∈ F⁺` ou `R₁ ∩ R₂ → R₂ ∈ F⁺`.  
> *(A interseção dos esquemas é chave de ao menos um dos lados.)*

> **Def. 8.3 — Preservação de Dependências:** Decomposição `{R₁, ..., Rₖ}` preserva DFs se:  
> `(F₁ ∪ ... ∪ Fₖ)⁺ = F⁺`  
> onde `Fᵢ` são as DFs de `F` projetadas sobre `Rᵢ`.

---

### 💡 Intuição

O mapeamento segue **7 regras sistemáticas** — cada elemento do DER tem uma tradução direta para o modelo relacional. A lógica geral é:

- **Entidades** → tabelas próprias
- **Atributos multivalorados** → tabelas auxiliares (preservam 1FN)
- **Atributos derivados** → nunca viram colunas (calculados por VIEW)
- **1:N** → FK no lado N
- **N:M** → nova tabela associativa
- **Recursivo N:M** → tabela associativa com FK duplo para a mesma tabela
- **Entidade fraca** → tabela com FK NOT NULL + CASCADE

---

### 🎵 Aplicação no Spotify — As 7 Regras

```
─── Regra 1: Entidade Comum → Tabela ───

DER: MUSICA(id_musica*, titulo, duracao_seg, letra, duracao_media*)
     (* = PK, * = atributo derivado)

Relacional: MUSICA(id_musica PK, titulo, duracao_seg, letra)
            ← duracao_media omitida: é derivada → vai para VIEW
```

```
─── Regra 2: Atributo Multivalorado → Nova Tabela ───

DER: ARTISTA com {redes_sociais}
     MUSICA  com {generos}

Relacional: REDES_SOCIAIS(id_rede PK, id_artista FK, plataforma, url)
            GENERO_MUSICA(id_genero PK, id_musica FK, nome)

← {redes_sociais} como coluna violaria 1FN (lista em célula única)
← cada rede/gênero vira sua própria linha — atomicidade restaurada
```

```
─── Regra 3: Atributo Derivado → VIEW ───

DER: ARTISTA.ouvintes_mensais* (derivado: usuários únicos que ouviram no mês)

Relacional: nenhuma coluna criada

CREATE VIEW v_ouvintes_mensais AS
  SELECT   ma.id_artista,
           COUNT(DISTINCT r.id_usuario) AS ouvintes_mensais
  FROM     MUSICA_ARTISTA ma
  JOIN     REPRODUCAO r ON r.id_musica = ma.id_musica
  WHERE    r.data_hora >= date_trunc('month', NOW())
  GROUP BY ma.id_artista;

← armazenar derivado criaria redundância e risco de inconsistência
← VIEW sempre retorna o valor atual calculado dinamicamente
```

```
─── Regra 4: Relacionamento 1:N → FK no lado N ───

DER: USUARIO (1,parcial) ──gera── (N,total) REPRODUCAO
     MUSICA  (1,parcial) ──de──   (N,total) REPRODUCAO

Relacional: REPRODUCAO recebe:
  id_usuario  FK NOT NULL → USUARIO   (total: toda reprodução tem usuário)
  id_musica   FK NOT NULL → MUSICA    (total: toda reprodução é de uma música)

Teorema de Heath (sem perda):
  R₁ ∩ R₂ = {id_usuario} = PK de USUARIO → decomposição sem perda ✅
```

```
─── Regra 5: Entidade Fraca → FK NOT NULL + CASCADE ───

DER: REPRODUCAO é entidade fraca (depende de USUARIO e MUSICA)

CREATE TABLE REPRODUCAO (
    id_reproducao    SERIAL  PRIMARY KEY,
    id_usuario       INTEGER NOT NULL
                     REFERENCES USUARIO(id_usuario) ON DELETE CASCADE,
    id_musica        INTEGER NOT NULL
                     REFERENCES MUSICA(id_musica)   ON DELETE RESTRICT,
    data_hora        TIMESTAMP NOT NULL,
    segundos_ouvidos INTEGER   NOT NULL,
    dispositivo      VARCHAR(30)
);

← CASCADE: deletar usuário remove automaticamente suas reproduções
← RESTRICT: não permite deletar música que ainda tem histórico de plays
```

```
─── Regra 6: Relacionamento N:M → Tabela Associativa ───

DER: PLAYLIST ──contém── MUSICA  [N:M]
     atributos do relacionamento: ordem, data_adicao

CREATE TABLE PLAYLIST_MUSICA (
    id_playlist  INTEGER REFERENCES PLAYLIST(id_playlist),
    id_musica    INTEGER REFERENCES MUSICA(id_musica),
    ordem        INTEGER   NOT NULL,    ← pertence ao RELACIONAMENTO (par único)
    data_adicao  TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (id_playlist, id_musica)
);

← "ordem" pertence ao par (playlist, música), não a cada um isolado
← mesma música pode ser faixa 1 numa playlist e faixa 7 em outra
```

```
─── Regra 7: Relacionamento Recursivo N:M → FK Duplo para Mesma Tabela ───

DER: ARTISTA ──membro de── ARTISTA  [N:M recursivo]
     papéis: banda / membro
     atributos: data_entrada, data_saida

CREATE TABLE ARTISTA_MEMBRO (
    id_banda     INTEGER REFERENCES ARTISTA(id_artista),  ← papel: banda
    id_membro    INTEGER REFERENCES ARTISTA(id_artista),  ← papel: membro
    data_entrada DATE    NOT NULL,
    data_saida   DATE,                     ← NULL = ainda na banda
    PRIMARY KEY (id_banda, id_membro),
    CHECK (id_banda <> id_membro)          ← artista não pode ser membro de si mesmo
);

← duas FKs apontando para a MESMA tabela ARTISTA
← papéis distintos: id_banda é quem "contém"; id_membro é quem "pertence"
← Queen (id_banda=8) ← Freddie Mercury (id_membro=9), Brian May (id_membro=10)...
```

**Tabela-resumo do mapeamento Spotify:**

| Elemento do DER | Regra | Resultado |
|---|---|---|
| 7 entidades simples | R1 | 7 tabelas diretas |
| `{redes_sociais}`, `{generos}` | R2 | REDES_SOCIAIS, GENERO_MUSICA |
| 5 atributos derivados | R3 | Nenhuma coluna — VIEWs |
| *cria*, *possui*, *gera*, *de* | R4 | FKs em PLAYLIST, ASSINATURA, REPRODUCAO |
| REPRODUCAO (entidade fraca) | R5 | 2 FKs NOT NULL com CASCADE/RESTRICT |
| 8 relacionamentos N:M | R6 | 8 tabelas associativas |
| *segue usuário*, *membro de* | R7 | 2 tabelas com FK duplo para mesma tabela |
| **Total** | — | **18 tabelas** |

---

## 9. Estratégias de Design e Normalização

### 🔷 Definição Formal

> **Def. 9.1 — Esquema de Boa Qualidade:** `S = {R₁,...,Rₖ}` tem boa qualidade se:  
> 1. Cada relação representa **uma entidade ou relacionamento** (semântica clara)  
> 2. Minimiza **redundância**  
> 3. Garante **lossless join** em decomposições  
> 4. **Preserva dependências** funcionais

> **Teorema 9.1 — Condição de Lossless Join:**  
> Decomposição `{R₁, R₂}` é sem perda ⟺ `(R₁ ∩ R₂) → R₁ ∈ F⁺` ou `(R₁ ∩ R₂) → R₂ ∈ F⁺`.

> **Teorema 9.2 — 3FN sempre atingível:**  
> O algoritmo de síntese 3FN sempre produz decomposição que: (a) está em 3FN, (b) é lossless, (c) preserva todas as DFs.

> **Teorema 9.3 — BCNF pode perder DFs:**  
> Existem esquemas onde a decomposição em BCNF não preserva todas as DFs.  
> Nesses casos, aceitar 3FN é o compromisso correto.

**Algoritmo de Síntese 3FN** (usa Fc como entrada):
```
1. Calcular Fc = cobertura canônica de F
2. Para cada X → A em Fc:
     Criar Rᵢ com atributos X ∪ {A},  chave = X
3. Se nenhum Rᵢ contém uma chave candidata de R:
     Adicionar relação com uma chave candidata
4. Eliminar relações Rᵢ ⊆ Rⱼ (redundantes)
```

---

### 💡 Intuição

Há três formas de abordar o design de um esquema — elas não são excludentes e costumam ser usadas juntas:

| Estratégia | Ponto de partida | Melhor para |
|---|---|---|
| **Top-Down** | Requisitos → DER → tabelas | Sistemas novos com visão completa |
| **Bottom-Up** | Atributos + DFs → síntese | Sistemas com muitos dados legados |
| **Inside-Out** | Núcleo do negócio → expande | Equipes de produto ágeis |

Independente da estratégia, o esquema deve ser **validado pelas formas normais** ao final.

---

### 🎵 Aplicação no Spotify

```
─── Top-Down: do requisito à tabela ───

Requisito: "usuários ouvem músicas, criam playlists, seguem artistas"

Etapa 1 — Entidades centrais:
  USUARIO, MUSICA, ARTISTA, PLAYLIST

Etapa 2 — Refinamentos:
  ALBUM separado de MUSICA (álbum ≠ faixa)
  ASSINATURA separada de USUARIO (histórico de planos)
  REPRODUCAO como entidade fraca (rastrear cada play individualmente)

Etapa 3 — Relacionamentos:
  N:M entre MUSICA e ARTISTA (colaborações — uma música tem vários autores)
  N:M entre PLAYLIST e MUSICA (ordem importa → atributo do relacionamento)
  N:M recursivo USUARIO↔USUARIO (follows)
  N:M recursivo ARTISTA↔ARTISTA (bandas com membros)

Etapa 4 — Mapeamento pelas 7 regras → 18 tabelas
```

```
─── Bottom-Up: dos atributos à tabela via DFs ───

Passo 1 — Todos os atributos do sistema:
  U = {id_musica, titulo, duracao_seg, id_artista, nome_artista,
       pais_artista, id_album, titulo_album, data_lancamento, ...}

Passo 2 — DFs identificadas:
  id_musica  → titulo, duracao_seg, letra
  id_artista → nome_artista, pais_artista, biografia
  id_album   → titulo_album, data_lancamento, tipo
  (id_musica, id_artista) → papel    ← atributo do relacionamento

Passo 3 — Cobertura canônica Fc (cada DF já atômica):
  Fc = { id_musica→titulo,  id_musica→duracao_seg,  id_musica→letra,
          id_artista→nome_artista,  id_artista→pais_artista, ...
          (id_musica,id_artista)→papel }

Passo 4 — Síntese 3FN (uma tabela por grupo de DFs com mesma chave):
  {id_musica}        → MUSICA(id_musica, titulo, duracao_seg, letra)
  {id_artista}       → ARTISTA(id_artista, nome_artista, pais_artista, ...)
  {id_musica,id_artista} → MUSICA_ARTISTA(id_musica, id_artista, papel)

← resultado: MUSICA, ARTISTA, MUSICA_ARTISTA — exatamente o esperado ✅
```

```
─── Inside-Out: do núcleo para a periferia ───

Nível 0 — Núcleo:       MUSICA  (razão de ser da plataforma)
Nível 1 — Produção:     ALBUM, ARTISTA
Nível 2 — Vínculos:     MUSICA_ARTISTA, MUSICA_ALBUM
Nível 3 — Consumo:      USUARIO, PLAYLIST
Nível 4 — Interação:    REPRODUCAO, ASSINATURA, USUARIO_CURTE_MUSICA
Nível 5 — Metadados:    REDES_SOCIAIS, GENERO_MUSICA
Nível 6 — Sociais:      USUARIO_SEGUE_USUARIO, ARTISTA_MEMBRO
```

**Validação pelas Formas Normais — testes rápidos:**

```
─── Teste de 1FN ───
Pergunta: há listas ou grupos repetidos em alguma coluna?
  MUSICA.generos como "pop,rock" → ✗ viola 1FN → criar GENERO_MUSICA

─── Teste de 2FN (só para PKs compostas) ───
Pergunta: algum atributo não-primo depende de PARTE da PK composta?

  Hipótese ruim: PLAYLIST_MUSICA(id_playlist, id_musica, titulo_musica, ordem)
    id_musica → titulo_musica ← depende de só id_musica, não do par
    → viola 2FN → titulo_musica deve ficar em MUSICA

  Modelo correto: PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao)
    (id_playlist,id_musica) → ordem       ✅ dependência total
    (id_playlist,id_musica) → data_adicao ✅ dependência total

─── Teste de 3FN ───
Pergunta: algum atributo não-primo depende de não-superchave por transitividade?

  ASSINATURA antes da normalização:
    id_assinatura → tipo_plano → valor_pago
    tipo_plano não é superchave; valor_pago não é primo → viola 3FN

  Solução:
    PLANO(tipo_plano PK, valor_pago)
    ASSINATURA(id_assinatura PK, id_usuario FK, tipo_plano FK, data_inicio, data_fim)
    ← cada tabela tem semântica de UMA entidade; transitividade eliminada ✅

─── Teste de BCNF ───
Pergunta: existe X → A não-trivial onde X NÃO é superchave?
  (mais restrito que 3FN — inclui atributos primos)
  No Spotify, após normalização: todas as tabelas satisfazem BCNF ✅
```

---

## 10. Integração de Esquemas

### 🔷 Definição Formal

> **Def. 10.1 — Problema de Integração:** Dados `n` esquemas locais `S₁,...,Sₙ`, produzir esquema global `S_G` satisfazendo:  
> 1. **Completude:** toda informação de algum `Sᵢ` é representável em `S_G`  
> 2. **Minimalidade:** `S_G` não contém redundâncias desnecessárias  
> 3. **Correção:** `S_G` é esquema relacional válido  
> 4. **Compreensibilidade:** `S_G` mapeia de volta para cada `Sᵢ` via VIEWs

> **Def. 10.2 — Conflitos de Esquema (Batini, Ceri & Navathe, 1992):**

| Tipo | Formalização | Exemplo Spotify |
|---|---|---|
| **Nomenclatura** | `nome(C, Sᵢ) ≠ nome(C, Sⱼ)` | `faixa_id` vs `track_id` vs `id_musica` |
| **Estrutural** | `estrutura(C, Sᵢ) ≠ estrutura(C, Sⱼ)` | Artista como atributo vs entidade |
| **Tipo de dado** | `dom(A, Sᵢ) ≠ dom(A, Sⱼ)` | `duracao` VARCHAR vs INTEGER |
| **Restrição** | `restr(A, Sᵢ) ≠ restr(A, Sⱼ)` | `visibilidade` enum vs boolean |
| **Semântico** | Mesmo nome, sentido diferente | `data` = criação vs lançamento |

---

### 💡 Intuição

Integração acontece quando o sistema foi construído por **múltiplas equipes independentes** — cada uma modelando as mesmas coisas de formas diferentes. Os problemas típicos são:

- **Nomes diferentes, mesma coisa:** `faixa_id`, `track_id`, `id_musica` → todos são a PK de MUSICA
- **Mesma coisa, estrutura diferente:** artista ora é atributo (`nome_artista`), ora é entidade (`ARTISTA`)
- **Tipos incompatíveis:** duração como `"3:54"` (string) num módulo, como `234` (inteiro) em outro
- **Chaves diferentes:** módulo B usa `email` como PK de usuário; módulo C usa `user_id`

O processo segue 4 etapas: **analisar conflitos → corresponder entidades → resolver conflitos → gerar esquema global**.

---

### 🎵 Aplicação no Spotify

```
─── Cenário: 3 módulos desenvolvidos por times independentes ───

Módulo A — Time Conteúdo:
  FAIXA(faixa_id, nome, duracao, artista_nome, artista_pais)
  DISCO(disco_id, titulo, ano, tipo, artista_nome)

Módulo B — Time Usuário:
  CONTA(conta_id, email, nome_completo, nascimento, pais_residencia)
  LISTA(lista_id, conta_id FK, nome_lista, visibilidade)  ← visibilidade: 'público'/'privado'
  FAIXA_LISTA(lista_id FK, faixa_id FK, posicao)

Módulo C — Time Analytics:
  PLAY(play_id, user_email, track_id, ts, device, secs_played)
  ARTISTA(artista_id, artista_nome, pais, bio)           ← entidade separada!
```

```
─── Etapa 1: Inventário de conflitos ───

┌─────────────────────────────────┬──────────────┬──────────────────────────────────────────┐
│ Conflito                        │ Tipo         │ Descrição                                │
├─────────────────────────────────┼──────────────┼──────────────────────────────────────────┤
│ faixa_id / track_id / id_musica │ Nomenclatura │ 3 nomes para o mesmo conceito            │
│ artista como atributo vs tabela │ Estrutural   │ Mód A: atributo; Mód C: entidade         │
│ duracao VARCHAR vs INTEGER      │ Tipo         │ "3:54" vs 234 (segundos)                 │
│ visibilidade vs publica boolean │ Restrição    │ enum vs TRUE/FALSE                       │
│ email vs conta_id como PK       │ Chave        │ Mód B: email é PK; Mód C: user_email=FK  │
└─────────────────────────────────┴──────────────┴──────────────────────────────────────────┘
```

```
─── Etapa 2: Correspondência de entidades ───

  Módulo A: FAIXA     ↔  Módulo C: PLAY.track_id   →  MUSICA  (entidade unificada)
  Módulo A: DISCO     ↔  (sem correspondente)       →  ALBUM   (novo nome padronizado)
  Módulo B: CONTA     ↔  Módulo C: PLAY.user_email  →  USUARIO (chave: id_usuario gerado)
  Módulo B: LISTA     ↔  (sem conflito)             →  PLAYLIST
  Módulo C: ARTISTA   ↔  Módulo A: FAIXA.artista_nome → ARTISTA (entidade vence atributo)
```

```
─── Etapa 3: Resolução de conflitos ───

─── Nomenclatura ───
  faixa_id / track_id → id_musica  (convenção: id_ + nome da entidade)

─── Estrutural: artista como atributo (Mód A) vs entidade (Mód C) ───
  Decisão: entidade ARTISTA vence (mais expressivo)
  FAIXA.artista_nome → removido; relacionamento N:M MUSICA_ARTISTA criado
  Justificativa: uma música pode ter múltiplos artistas (colaborações)
                 → atributo simples seria insuficiente

─── Tipo: duracao VARCHAR("3:54") → INTEGER (segundos) ───
  Script de migração:
    EXTRACT(MINUTE FROM duracao::TIME)*60
    + EXTRACT(SECOND FROM duracao::TIME) AS duracao_seg
  Decisão: INTEGER é comparável, somável, padrão de mercado

─── Restrição: visibilidade VARCHAR → publica BOOLEAN ───
  'público'    → TRUE
  'privado'    → FALSE
  'somente_eu' → FALSE  (com flag colaborativa separada para cobrir casos extras)

─── Chave: email como PK (Mód B) vs surrogate key (Mód C) ───
  Decisão: criar id_usuario INTEGER (PK surrogate) + email UNIQUE NOT NULL (chave alternada)
  ← email pode mudar; surrogate key é estável para FKs
```

```
─── Etapa 4: Esquema global e VIEWs de compatibilidade ───

Resultado da integração → mesmas 18 tabelas do modelo Spotify
```

```sql
-- VIEW de compatibilidade para manter interface do Módulo A durante migração
CREATE VIEW FAIXA AS
  SELECT
    m.id_musica    AS faixa_id,
    m.titulo       AS nome,
    TO_CHAR(INTERVAL '1 second' * m.duracao_seg, 'MI:SS') AS duracao,
    a.nome_artistico AS artista_nome,
    a.pais_origem    AS artista_pais
  FROM MUSICA m
  JOIN MUSICA_ARTISTA ma ON ma.id_musica = m.id_musica AND ma.papel = 'principal'
  JOIN ARTISTA a         ON a.id_artista = ma.id_artista;

-- VIEW de compatibilidade para o Módulo C (analytics)
CREATE VIEW PLAY AS
  SELECT
    r.id_reproducao    AS play_id,
    u.email            AS user_email,  ← interface legada usa email como identificador
    r.id_musica        AS track_id,
    r.data_hora        AS ts,
    r.dispositivo      AS device,
    r.segundos_ouvidos AS secs_played
  FROM REPRODUCAO r
  JOIN USUARIO u ON u.id_usuario = r.id_usuario;

-- VIEW de compatibilidade para o Módulo B (usuário)
CREATE VIEW CONTA AS
  SELECT
    id_usuario    AS conta_id,
    email,
    nome          AS nome_completo,
    data_nascimento AS nascimento,
    pais          AS pais_residencia
  FROM USUARIO;
```

**Boas práticas para integração:**

| Princípio | Por quê | Aplicação no Spotify |
|---|---|---|
| Entidade > atributo | Evita dep. parciais e transitivas | `artista_nome` → entidade ARTISTA |
| Surrogate key como PK | Desacopla identidade técnica de semântica | `id_usuario` em vez de `email` como PK |
| VIEWs de compatibilidade | Migração gradual sem quebrar sistemas | PLAY, FAIXA, CONTA como views temporárias |
| Documentar cada decisão | Rastreabilidade e onboarding | Tabela de conflitos → decisões |
| Validar com dados reais | Encontrar inconsistências antes da migração | Queries de reconciliação entre módulos |
| Normalizar após integrar | Garantir qualidade do esquema global | Verificar 3FN/BCNF no esquema unificado |

---

## Mapa Conceitual — Como os Tópicos se Conectam

```
Domínios e Atributos
        ↓ estruturam
Esquema de Relação  (Tópico 1)
        ↓ sobre o qual se definem
Dependências Funcionais  (Tópico 3)
        ↓ usadas para calcular
Fechamento de Atributos X⁺  (Tópico 4)
        ↓ que permite identificar
Chaves Candidatas e PKs  (Tópico 2)
        ↓ e verificar
Equivalência de Conjuntos de DFs  (Tópico 5)
        ↓ que leva a
Cobertura Canônica Fc  (Tópico 6)
        ↓ que diagnostica
Anomalias e Violações de FN  (Tópico 7)
        ↓ corrigidas por normalização, guiada por
Estratégias de Design (Top-down/Bottom-up/Inside-out)  (Tópico 9)
        ↓ aplicadas via
Mapeamento ER → Relacional (7 regras)  (Tópico 8)
        ↓ e, em contextos multi-equipe, por
Integração de Esquemas  (Tópico 10)
```

**Tabela-resumo geral:**

| Tópico | Conceito formal central | Para que serve na prática |
|---|---|---|
| **1. Modelo Relacional** | `r ⊆ dom(A₁)×...×dom(Aₙ)` | Fundamento: dados como conjuntos de tuplas atômicas |
| **2. Chaves** | Superchave mínima; `∀t₁≠t₂: t₁[PK]≠t₂[PK]` | Identificar tuplas; garantir integridade referencial |
| **3. DFs** | `t₁[X]=t₂[X] ⟹ t₁[Y]=t₂[Y]` | Expressar regras de negócio matematicamente |
| **4. Fechamento** | `X⁺ = {A | X→A ∈ F⁺}` | Descobrir chaves e verificar DFs eficientemente |
| **5. Equivalência** | `F⁺ = G⁺` | Substituir F por forma mais simples sem perda |
| **6. Cob. Canônica** | `Fc ≡ F`, mínima, sem extrâneos | Entrada do algoritmo de síntese 3FN |
| **7. Anomalias** | Violação de BCNF → anomalias | Diagnosticar problemas e motivar normalização |
| **8. Mapeamento** | 7 regras preservadoras | Traduzir DER em tabelas relacionais |
| **9. Estratégias** | Síntese 3FN (Teorema 9.2) | Garantir qualidade formal do esquema final |
| **10. Integração** | Resolução de conflitos + VIEWs | Unificar esquemas de múltiplas equipes |
