# Banco de Dados Relacionais — Teoria Formal Completa
### Contexto: Plataforma de Streaming Musical (Spotify)

> Cada tópico apresenta primeiro a **definição formal matemática**, depois a **intuição** e, por fim, a **aplicação direta** nas tabelas do modelo Spotify desenvolvido em aula.

---

## Sumário

1. [Modelo Relacional](#1-modelo-relacional)
2. [Chaves: Candidata, Super, Primária, Alternada e Estrangeira](#2-chaves)
3. [Dependência Funcional](#3-dependência-funcional)
4. [Fechamento de Atributos](#4-fechamento-de-atributos)
5. [Equivalência de Dependências Funcionais](#5-equivalência-de-dependências-funcionais)
6. [Cobertura Canônica](#6-cobertura-canônica)
7. [Anomalias no Modelo Relacional](#7-anomalias-no-modelo-relacional)
8. [Mapeamento do Modelo ER para o Relacional](#8-mapeamento-do-modelo-er-para-o-relacional)
9. [Estratégias para Design de Esquema e Normalização](#9-estratégias-para-design-de-esquema-e-normalização)
10. [Integração de Esquemas](#10-integração-de-esquemas)

---

## 1. Modelo Relacional

### 1.1 Fundamento Matemático

O modelo relacional é fundamentado na **teoria dos conjuntos** e na **lógica de predicados de primeira ordem**, proposto por E. F. Codd (1970) no artigo *"A Relational Model of Data for Large Shared Data Banks"*.

#### Definição formal de Domínio

> **Definição 1.1 — Domínio:**  
> Um domínio `D` é um conjunto de valores atômicos (indivisíveis). Formalmente, é um conjunto nomeado de valores escalares.  
> Exemplos: `D_inteiros = {1, 2, 3, ...}`, `D_plano = {'gratuito', 'premium'}`.

#### Definição formal de Relação

> **Definição 1.2 — Esquema de Relação:**  
> Um esquema de relação `R(A₁, A₂, ..., Aₙ)` é um nome `R` associado a uma lista de atributos `{A₁, ..., Aₙ}`, onde cada atributo `Aᵢ` é associado a um domínio `dom(Aᵢ)`.

> **Definição 1.3 — Relação (instância):**  
> Uma relação (ou instância) `r` sobre o esquema `R(A₁, ..., Aₙ)` é um subconjunto finito do produto cartesiano dos domínios:
>
> `r ⊆ dom(A₁) × dom(A₂) × ... × dom(Aₙ)`
>
> Cada elemento de `r` é uma **tupla** `t = (v₁, v₂, ..., vₙ)` onde `vᵢ ∈ dom(Aᵢ)`.

> **Definição 1.4 — Tupla e notação de acesso:**  
> Para uma tupla `t` e atributo `Aᵢ`, a notação `t[Aᵢ]` ou `t.Aᵢ` denota o valor de `t` no atributo `Aᵢ`.  
> Para um conjunto de atributos `X = {Aᵢ, Aⱼ}`, `t[X]` denota a subtupla `(t[Aᵢ], t[Aⱼ])`.

#### Propriedades formais de uma relação

Uma relação válida satisfaz **obrigatoriamente**:

| Propriedade | Formalização |
|---|---|
| **Atomicidade (1FN)** | `∀ t ∈ r, ∀ Aᵢ : t[Aᵢ] ∈ dom(Aᵢ)` — nenhum valor é conjunto ou lista |
| **Unicidade de tuplas** | `∀ t₁, t₂ ∈ r : t₁ ≠ t₂` — `r` é um **conjunto**, não multiconjunto |
| **Ordem irrelevante** | `r` como conjunto não tem ordenação entre tuplas |
| **Atributos nomeados** | Acesso por nome, não por posição |

#### Grau, Cardinalidade e Esquema de Banco de Dados

> **Definição 1.5 — Grau e Cardinalidade:**  
> O **grau** de uma relação é o número de atributos `n` no esquema.  
> A **cardinalidade** de uma instância é o número de tuplas `|r|`.

> **Definição 1.6 — Esquema de Banco de Dados:**  
> Um esquema de banco de dados relacional `S` é um conjunto de esquemas de relação:  
> `S = {R₁, R₂, ..., Rₘ}`  
> Uma **instância** de banco de dados `DB` é um conjunto `{r₁, r₂, ..., rₘ}` onde cada `rᵢ` é uma instância válida de `Rᵢ`.

### 1.2 Intuição e Analogia

Matematicamente, uma relação é um **subconjunto do produto cartesiano** dos seus domínios. Na prática isso significa que cada linha da tabela é uma combinação possível de valores, e o banco garante que essa combinação satisfaz todas as restrições declaradas.

A diferença fundamental entre uma **tabela** (implementação) e uma **relação** (conceito matemático) é que a relação é um **conjunto** — sem duplicatas, sem ordem — enquanto implementações físicas podem ter ambas por razões de desempenho.

### 1.3 Aplicação no Spotify

```
Esquema formal:
  MUSICA(id_musica: D_int, titulo: D_varchar200, duracao_seg: D_int_pos, letra: D_text_null)

  dom(id_musica)   = ℤ⁺  (inteiros positivos)
  dom(titulo)      = {strings de até 200 caracteres}
  dom(duracao_seg) = {n ∈ ℤ | n > 0}
  dom(letra)       = {strings} ∪ {NULL}

Grau de MUSICA    = 4
Grau de REPRODUCAO = 6

Instância r(MUSICA) — exemplo:
┌───────────┬─────────────────────────┬─────────────┬────────┐
│ id_musica │ titulo                  │ duracao_seg │ letra  │
├───────────┼─────────────────────────┼─────────────┼────────┤
│ 1         │ Bohemian Rhapsody       │ 354         │ ...    │
│ 2         │ Blinding Lights         │ 200         │ ...    │
│ 3         │ Shape of You            │ 234         │ NULL   │
└───────────┴─────────────────────────┴─────────────┴────────┘
Cardinalidade atual: |r(MUSICA)| = 3
```

> **Violação de atomicidade — exemplo:**  
> Se `generos` fosse armazenado como `"pop, rock, eletrônico"` dentro de MUSICA, isso violaria a 1FN pois `dom(generos)` conteria listas. A solução no Spotify foi criar `GENERO_MUSICA` como tabela separada.

### 1.4 Restrições de Integridade

Além das propriedades estruturais, o modelo impõe **restrições de integridade**:

| Restrição | Definição formal | Exemplo no Spotify |
|---|---|---|
| **Domínio** | `t[Aᵢ] ∈ dom(Aᵢ)` | `plano ∈ {'gratuito','premium'}` |
| **Chave** | Unicidade da PK (ver seção 2) | `id_musica` único em MUSICA |
| **Integridade de entidade** | PK ≠ NULL | `id_musica IS NOT NULL` |
| **Integridade referencial** | FK referencia PK existente (ver seção 2.5) | `REPRODUCAO.id_musica` deve existir em MUSICA |
| **Semântica** | Restrições de negócio | `duracao_seg > 0`, `CHECK(tipo IN ('solo','banda'))` |

---

## 2. Chaves

### 2.1 Definições Formais

> **Definição 2.1 — Superchave:**  
> Seja `R(A₁, ..., Aₙ)` um esquema e `r` uma instância de `R`.  
> Um subconjunto `SK ⊆ {A₁, ..., Aₙ}` é uma **superchave** de `R` se e somente se:
>
> `∀ t₁, t₂ ∈ r : t₁ ≠ t₂ ⟹ t₁[SK] ≠ t₂[SK]`
>
> Ou seja: não existem duas tuplas distintas com o mesmo valor em `SK`.

> **Definição 2.2 — Chave Candidata:**  
> Uma superchave `CK ⊆ {A₁, ..., Aₙ}` é uma **chave candidata** se e somente se:
>
> `CK` é superchave  **E**  `∀ X ⊊ CK : X` **não** é superchave
>
> Ou seja: é uma superchave **mínima** — remover qualquer atributo a invalida.

> **Definição 2.3 — Chave Primária:**  
> A **chave primária** `PK` é a chave candidata **escolhida** pelo designer.  
> Restrição adicional: `∀ t ∈ r : t[PK] ≠ NULL` (integridade de entidade).

> **Definição 2.4 — Chave Alternada:**  
> Toda chave candidata que **não** foi escolhida como primária.  
> Deve ser implementada com `UNIQUE NOT NULL`.

> **Definição 2.5 — Chave Estrangeira:**  
> Sejam `R₁` e `R₂` dois esquemas, `PK₂` a chave primária de `R₂`.  
> Um subconjunto `FK ⊆ atributos(R₁)` é uma **chave estrangeira** de `R₁` referenciando `R₂` se:
>
> `∀ t₁ ∈ r₁ : t₁[FK] ≠ NULL ⟹ ∃ t₂ ∈ r₂ : t₁[FK] = t₂[PK₂]`
>
> Esta é a **restrição de integridade referencial**: todo valor não-nulo da FK deve corresponder a uma tupla existente na relação referenciada.

### 2.2 Propriedades e Teoremas

> **Teorema 2.1:**  
> Todo conjunto de todos os atributos de uma relação `R` é sempre uma superchave (trivial) de `R`.  
> *Demonstração:* Se duas tuplas diferem em ao menos um atributo, elas diferem em seu conjunto total de atributos. □

> **Teorema 2.2:**  
> Em qualquer relação `R`, existe ao menos uma chave candidata.  
> *Demonstração:* O conjunto total de atributos é superchave. Por processo de eliminação mínima, existe ao menos uma superchave mínima (chave candidata). □

> **Corolário 2.1:**  
> A chave primária sempre existe e é única (como escolha), mas podem existir múltiplas chaves candidatas.

> **Definição 2.6 — Violação de Integridade Referencial:**  
> Diz-se que uma instância `DB = {r₁, r₂}` viola a integridade referencial se:  
> `∃ t₁ ∈ r₁ : t₁[FK] ≠ NULL ∧ ¬∃ t₂ ∈ r₂ : t₁[FK] = t₂[PK₂]`

### 2.3 Intuição

A hierarquia de chaves segue uma relação de inclusão:

```
Superchave
  └─ Chave Candidata  (superchave mínima)
       ├─ Chave Primária   (candidata escolhida, NOT NULL)
       └─ Chave Alternada  (candidata não-escolhida, UNIQUE NOT NULL)

Chave Estrangeira: atributo em R₁ que aponta para PK de R₂
```

**Por que a minimalidade importa?** Uma PK com atributos desnecessários causa problemas nas tabelas associativas — uma PK composta com atributo extra exige mais espaço em índices e complica JOINs. A minimalidade garante que cada atributo da chave é **semanticamente necessário** para a identificação.

### 2.4 Aplicação no Spotify

```
USUARIO(id_usuario, nome, email, data_nascimento, pais, plano)

Superchaves:
  {id_usuario}                        ← mínima
  {email}                             ← mínima
  {id_usuario, nome}                  ← não-mínima (nome é redundante)
  {id_usuario, email, pais, plano}    ← não-mínima
  {todos os atributos}                ← não-mínima (trivial)

Chaves candidatas:
  CK₁ = {id_usuario}   → mínima: remover id_usuario → {} não é superchave
  CK₂ = {email}        → mínima: emails são únicos por regra de negócio

Chave primária escolhida:  id_usuario  (inteiro, eficiente para JOINs)
Chave alternada:           email       (UNIQUE NOT NULL)
```

```
REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo)

Chaves candidatas:
  CK₁ = {id_reproducao}
        → {id_reproducao}⁺ = todos os atributos ✅
        → mínima (atributo único) ✅

  CK₂ = {id_usuario, id_musica, data_hora}
        → determina segundos_ouvidos e dispositivo ✅
        → mínima: nenhum subconjunto sozinho determina tudo ✅

PK escolhida: id_reproducao  (surrogate key — mais simples para FKs futuras)
```

```
Verificação de integridade referencial — exemplo formal:

r(REPRODUCAO) contém t₁ com t₁[id_usuario] = 42
Exige: ∃ t₂ ∈ r(USUARIO) tal que t₂[id_usuario] = 42

Se tentarmos INSERT INTO REPRODUCAO com id_usuario = 999 (inexistente):
  → SGBD rejeita: violação de integridade referencial
  → Garante: nenhuma reprodução 'órfã' sem usuário correspondente
```

```sql
-- Implementação SQL das restrições de chave no Spotify

-- Chave primária simples
CREATE TABLE MUSICA (
    id_musica    INTEGER     PRIMARY KEY,   -- PK
    titulo       VARCHAR(200) NOT NULL,
    duracao_seg  INTEGER     NOT NULL CHECK (duracao_seg > 0),
    letra        TEXT
);

-- Chave alternada
CREATE TABLE USUARIO (
    id_usuario      INTEGER     PRIMARY KEY,        -- PK
    email           VARCHAR(150) NOT NULL UNIQUE,   -- Chave alternada
    nome            VARCHAR(100) NOT NULL,
    plano           VARCHAR(20)  DEFAULT 'gratuito'
                    CHECK (plano IN ('gratuito','premium'))
);

-- PK composta (tabela associativa)
CREATE TABLE USUARIO_CURTE_MUSICA (
    id_usuario   INTEGER REFERENCES USUARIO(id_usuario),  -- FK
    id_musica    INTEGER REFERENCES MUSICA(id_musica),    -- FK
    data_curtida TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (id_usuario, id_musica)                   -- PK composta
);

-- FK com ação referencial
CREATE TABLE REPRODUCAO (
    id_reproducao    INTEGER PRIMARY KEY,
    id_usuario       INTEGER NOT NULL REFERENCES USUARIO(id_usuario) ON DELETE CASCADE,
    id_musica        INTEGER NOT NULL REFERENCES MUSICA(id_musica)   ON DELETE RESTRICT,
    data_hora        TIMESTAMP NOT NULL,
    segundos_ouvidos INTEGER   NOT NULL,
    dispositivo      VARCHAR(30)
);
```

> **Ações referenciais:** ao definir uma FK, o designer declara o que acontece se a tupla referenciada for deletada/atualizada:  
> - `CASCADE`: propaga a operação (deletar usuário deleta suas reproduções)  
> - `RESTRICT` / `NO ACTION`: impede a operação se houver dependentes  
> - `SET NULL`: anula a FK (somente se ela aceitar NULL)  
> - `SET DEFAULT`: atribui valor padrão

---

## 3. Dependência Funcional

### 3.1 Definição Formal

> **Definição 3.1 — Dependência Funcional:**  
> Seja `R(A₁, ..., Aₙ)` um esquema de relação e `X, Y ⊆ {A₁, ..., Aₙ}`.  
> Dizemos que **X determina funcionalmente Y** (notação: `X → Y`) se e somente se:
>
> `∀ r` instância válida de `R`, `∀ t₁, t₂ ∈ r :`  
> `t₁[X] = t₂[X]  ⟹  t₁[Y] = t₂[Y]`
>
> Equivalentemente: para cada valor de `X`, existe **no máximo um** valor de `Y` na relação.

> **Definição 3.2 — Dependência Funcional Trivial:**  
> `X → Y` é **trivial** se e somente se `Y ⊆ X`.  
> Toda DF trivial é satisfeita por qualquer instância, pois as mesmas colunas sempre têm os mesmos valores.

> **Definição 3.3 — Dependência Funcional Não-Trivial:**  
> `X → Y` é **não-trivial** se `Y ⊄ X` (Y contém ao menos um atributo fora de X).  
> É **completamente não-trivial** se `X ∩ Y = ∅`.

> **Definição 3.4 — Dependência Parcial:**  
> Seja `K` uma chave candidata composta `K = {A₁, ..., Aₖ}`.  
> Um atributo não-primo `B` (não pertencente a nenhuma chave candidata) tem **dependência parcial** em relação a `K` se:  
> `∃ X ⊊ K : X → B`  
> Ou seja: `B` depende de um **subconjunto próprio** da chave, não da chave inteira.

> **Definição 3.5 — Dependência Transitiva:**  
> Dizemos que `B` tem **dependência transitiva** de `K` via `Y` se:  
> `K → Y`,  `Y → B`,  `Y ↛ K`  e  `B ∉ K`  
> (`Y` determina `B`, mas `Y` não é chave — é um intermediário não-chave).

### 3.2 Axiomas de Armstrong (Sistema Completo e Sólido)

Os **Axiomas de Armstrong** formam um sistema de inferência **sólido** (toda DF derivada é válida) e **completo** (toda DF válida pode ser derivada).

> **Axioma A1 — Reflexividade:**  
> `Y ⊆ X  ⟹  X → Y`  
> (Toda DF trivial vale.)

> **Axioma A2 — Aumentatividade:**  
> `X → Y  ⟹  XZ → YZ`,  para qualquer `Z`  
> (Adicionar os mesmos atributos dos dois lados preserva a DF.)

> **Axioma A3 — Transitividade:**  
> `X → Y  ∧  Y → Z  ⟹  X → Z`  
> (Determinação é transitiva.)

**Regras derivadas** (provadas a partir dos axiomas):

> **Regra R1 — União:**  
> `X → Y  ∧  X → Z  ⟹  X → YZ`  
> *Prova:* `X → Y` (dado); por A2, `X → XY`; `X → Z` (dado); por A2, `XY → YZ`; por A3, `X → YZ`. □

> **Regra R2 — Decomposição:**  
> `X → YZ  ⟹  X → Y  ∧  X → Z`  
> *Prova:* `Y ⊆ YZ`, então por A1, `YZ → Y`; por A3 com `X → YZ`, temos `X → Y`. Análogo para Z. □

> **Regra R3 — Pseudotransitividade:**  
> `X → Y  ∧  WY → Z  ⟹  WX → Z`  
> *Prova:* `X → Y`; por A2, `WX → WY`; por A3 com `WY → Z`, temos `WX → Z`. □

### 3.3 Conjunto de DFs vs. Instâncias

> **Definição 3.6 — Satisfação:**  
> Uma instância `r` **satisfaz** (ou **respeita**) uma DF `X → Y` se a condição da Def. 3.1 vale para todas as tuplas em `r`.

> **Definição 3.7 — DF válida em esquema:**  
> Uma DF `X → Y` é válida no esquema `R` se **toda** instância possível de `R` satisfaz `X → Y`.  
> Isso é uma **restrição semântica** — não basta verificar uma instância atual; é preciso que a regra valha para qualquer estado futuro do banco.

**Implicação prática:** observar uma instância pode sugerir DFs, mas não prová-las. A validação definitiva vem do **domínio do problema** (regra de negócio), não dos dados.

### 3.4 Tipos de Dependência — Quadro Completo

| Tipo | Definição | Impacto na Normalização |
|---|---|---|
| **Trivial** | `Y ⊆ X` | Nenhum — sempre satisfeita |
| **Não-trivial** | `Y ⊄ X` | Base das restrições reais |
| **Parcial** | `X ⊊ K → B` (K chave composta) | Viola a **2FN** |
| **Transitiva** | `K → Y → B`, Y não-chave | Viola a **3FN** |
| **Total** | `B` depende da chave inteira, não de subconjunto | Satisfaz a **2FN** |
| **Multivalorada** | `X →→ Y` (Y independente do resto) | Base da **4FN** |

### 3.5 Aplicação Sistemática no Spotify

```
─── MUSICA(id_musica, titulo, duracao_seg, letra) ───

DFs válidas (regras de negócio):
  id_musica → titulo          [não-trivial, total]
  id_musica → duracao_seg     [não-trivial, total]
  id_musica → letra           [não-trivial, total — mesmo quando NULL]
  id_musica → {titulo, duracao_seg, letra}   [por Regra R1 — União]

DFs inválidas (contraexemplos existem):
  titulo → id_musica          ✗  duas músicas podem ter o mesmo título
  duracao_seg → id_musica     ✗  muitas músicas têm 200 segundos
  duracao_seg → titulo        ✗  idem

DF trivial:
  {id_musica, titulo} → titulo  [trivial: titulo ⊆ {id_musica, titulo}]
```

```
─── ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago) ───

DFs válidas:
  id_assinatura → id_usuario      [determinante é chave → total]
  id_assinatura → tipo_plano
  id_assinatura → data_inicio
  id_assinatura → data_fim
  id_assinatura → valor_pago
  tipo_plano → valor_pago         ← DEPENDÊNCIA TRANSITIVA!
                                    id_assinatura → tipo_plano → valor_pago
                                    viola 3FN

  Por Regra R1 (União):
  id_assinatura → {id_usuario, tipo_plano, data_inicio, data_fim, valor_pago}
```

```
─── PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao) ───
PK: {id_playlist, id_musica}

DFs:
  (id_playlist, id_musica) → ordem        [total em relação à PK composta]
  (id_playlist, id_musica) → data_adicao  [total]

  id_playlist → ordem     ✗  a mesma playlist tem músicas em posições diferentes
  id_musica → ordem       ✗  a mesma música pode estar em posições diferentes em playlists diferentes

  → Sem dependências parciais: atende 2FN ✅
```

```
─── REPRODUCAO ─── Exemplo de dependência transitiva potencial ───

Se adicionássemos a regra: "dispositivo_sistema depende do dispositivo"
  dispositivo → sistema_operacional  (ex: 'celular' → 'Android/iOS')

Então:
  id_reproducao → dispositivo → sistema_operacional
  → sistema_operacional teria dependência TRANSITIVA de id_reproducao via dispositivo
  → Violaria 3FN → sistema_operacional deveria ir para tabela DISPOSITIVO separada
```

### 3.6 Conexão com Chaves

> **Teorema 3.1:**  
> `X` é superchave de `R` se e somente se `X → A` para todo atributo `A` de `R`.  
> Equivalentemente (usando Regra R1): `X` é superchave sse `X → R` (X determina toda a relação).

> **Teorema 3.2:**  
> `X` é chave candidata de `R` se e somente se:  
> 1. `X → R` (X é superchave), **E**  
> 2. `∀ Y ⊊ X : Y ↛ R` (nenhum subconjunto próprio é superchave)

Estes teoremas ligam formalmente o conceito de chave com o de dependência funcional — chaves são exatamente os determinantes mínimos de toda a relação.

---

## 4. Fechamento de Atributos

### 4.1 Definição Formal

> **Definição 4.1 — Fechamento de um Conjunto de DFs:**  
> Seja `F` um conjunto de DFs sobre `R`.  
> O **fechamento de F**, denotado `F⁺`, é o conjunto de **todas** as DFs que podem ser derivadas de `F` usando os Axiomas de Armstrong:
>
> `F⁺ = { X → Y | F ⊢ X → Y }`
>
> onde `⊢` denota derivabilidade pelo sistema de Armstrong.

> **Definição 4.2 — Fechamento de um Conjunto de Atributos:**  
> Seja `X ⊆` atributos de `R` e `F` um conjunto de DFs.  
> O **fechamento de X em relação a F**, denotado `X⁺` (ou `X⁺_F`), é:
>
> `X⁺ = { A | X → A ∈ F⁺ }`
>
> Ou seja: todos os atributos que X determina (direta ou transitivamente) sob F.

> **Teorema 4.1 — Utilidade do Fechamento:**  
> `X → Y ∈ F⁺  ⟺  Y ⊆ X⁺`  
> *Demonstração:*  
> (⟹) Se `X → Y ∈ F⁺`, então por Regra R2 (Decomposição), `X → A ∈ F⁺` para cada `A ∈ Y`, logo `A ∈ X⁺` para cada A, portanto `Y ⊆ X⁺`.  
> (⟸) Se `Y ⊆ X⁺`, então para cada `A ∈ Y`, `X → A ∈ F⁺`; por Regra R1 (União), `X → Y ∈ F⁺`. □

**Importância:** o Teorema 4.1 transforma o problema de verificar se `X → Y ∈ F⁺` (que exigiria computar o conjunto potencialmente enorme `F⁺`) no problema mais simples de calcular `X⁺`.

### 4.2 Algoritmo de Fechamento

> **Algoritmo CLOSURE(X, F):**
>
> ```
> Entrada: X ⊆ atributos(R),  F = conjunto de DFs
> Saída:   X⁺
>
> 1. resultado ← X
> 2. mudou ← true
> 3. enquanto mudou fazer:
>      mudou ← false
>      para cada DF (α → β) em F fazer:
>        se α ⊆ resultado então:
>          se β ⊄ resultado então:
>            resultado ← resultado ∪ β
>            mudou ← true
> 4. retornar resultado
> ```

> **Teorema 4.2 — Corretude e Completude do Algoritmo:**  
> O algoritmo CLOSURE(X, F) computa corretamente `X⁺`:  
> - **Corretude (solidez):** todo atributo adicionado ao resultado pode ser derivado de X via F usando Armstrong.  
> - **Completude:** todo atributo em `X⁺` será adicionado ao resultado.  
> - **Complexidade:** O(|F| · |atributos|²) no pior caso — polinomial.

> **Corolário 4.1 — Verificação de Superchave:**  
> `X` é superchave de `R`  ⟺  `X⁺ = atributos(R)`

> **Corolário 4.2 — Verificação de Chave Candidata:**  
> `X` é chave candidata de `R` ⟺  
> 1. `X⁺ = atributos(R)` (superchave), **E**  
> 2. `∀ A ∈ X : (X − {A})⁺ ≠ atributos(R)` (minimalidade)

### 4.3 Aplicação Detalhada no Spotify

```
Esquema: REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo)
Abreviações: R=id_reproducao, U=id_usuario, M=id_musica, D=data_hora, S=segundos_ouvidos, V=dispositivo

Conjunto F:
  F1: R → U
  F2: R → M
  F3: R → D
  F4: (U, M, D) → S
  F5: (U, M, D) → V

Calcular {R}⁺:

  Iteração 1:
    resultado = {R}
    F1: R ⊆ {R} ✓ → resultado = {R, U}
    F2: R ⊆ {R, U} ✓ → resultado = {R, U, M}
    F3: R ⊆ {R, U, M} ✓ → resultado = {R, U, M, D}
    F4: {U,M,D} ⊆ {R,U,M,D} ✓ → resultado = {R, U, M, D, S}
    F5: {U,M,D} ⊆ {R,U,M,D,S} ✓ → resultado = {R, U, M, D, S, V}
    mudou = true

  Iteração 2:
    Nenhuma DF adiciona novo atributo → mudou = false → PARA

  {R}⁺ = {R, U, M, D, S, V} = todos os atributos ✅
  → id_reproducao é SUPERCHAVE
```

```
Verificar minimalidade de {R} = {id_reproducao}:
  Só tem 1 atributo → nenhum subconjunto próprio não-vazio → é CHAVE CANDIDATA ✅
```

```
Verificar se {U, M, D} = {id_usuario, id_musica, data_hora} é chave candidata:

  Calcular {U, M, D}⁺:
    resultado = {U, M, D}
    F4: {U,M,D} ⊆ {U,M,D} ✓ → resultado = {U, M, D, S}
    F5: {U,M,D} ⊆ {U,M,D,S} ✓ → resultado = {U, M, D, S, V}
    Nenhuma DF adicionada adiciona R, U, M, D já estão
    → resultado final = {U, M, D, S, V}
    → R ∉ resultado → {U,M,D}⁺ ≠ todos os atributos ✗
    → NÃO é superchave (a menos que R não seja atributo independente)

  Observação: se R for gerado como SERIAL/autoincremento,
  é independente e {U,M,D} não o determina.
  Portanto {U,M,D} não é chave candidata neste esquema.
  Se removêssemos R, {U,M,D} seria a única chave candidata.
```

```
Verificar se tipo_plano → valor_pago ∈ F⁺ (para ASSINATURA):

  Conjunto F de ASSINATURA inclui:
    G1: id_assinatura → tipo_plano
    G2: tipo_plano → valor_pago

  Calcular {tipo_plano}⁺:
    resultado = {tipo_plano}
    G2: tipo_plano ⊆ {tipo_plano} ✓ → resultado = {tipo_plano, valor_pago}
    → valor_pago ∈ {tipo_plano}⁺ ✅

  Portanto: tipo_plano → valor_pago ∈ F⁺

  Por Transitividade:
    id_assinatura → tipo_plano (G1)
    tipo_plano → valor_pago (G2)
    ∴ id_assinatura → valor_pago ∈ F⁺ ← dependência TRANSITIVA
```

```
Calcular {id_playlist, id_musica}⁺ em PLAYLIST_MUSICA:

  F = { (id_playlist, id_musica) → ordem,
        (id_playlist, id_musica) → data_adicao }

  resultado = {id_playlist, id_musica}
  Aplica DF1 → {id_playlist, id_musica, ordem}
  Aplica DF2 → {id_playlist, id_musica, ordem, data_adicao}

  = todos os atributos ✅ → é chave candidata (e PK)
```

### 4.4 Uso Prático: Descoberta de Chaves

O fechamento de atributos é a ferramenta central para **descobrir todas as chaves candidatas** de um esquema dado um conjunto F de DFs. O processo geral é:

```
Para cada subconjunto X de atributos(R):
  1. Calcular X⁺
  2. Se X⁺ = todos os atributos → X é superchave
  3. Verificar minimalidade → se mínima, é chave candidata

Problema: exponencial no número de atributos no pior caso.
Otimização: começar pelos subconjuntos menores e podar.
```

---

## 5. Equivalência de Dependências Funcionais

### 5.1 Definições Formais

> **Definição 5.1 — Cobertura:**  
> Um conjunto de DFs `F` **cobre** um conjunto `G` (notação: `F ↠ G` ou `G ⊆ F⁺`) se e somente se:
>
> `∀ (X → Y) ∈ G : X → Y ∈ F⁺`
>
> Equivalentemente (pelo Teorema 4.1): `∀ (X → Y) ∈ G : Y ⊆ X⁺_F`  
> (O fechamento de X calculado usando F contém Y.)

> **Definição 5.2 — Equivalência de Conjuntos de DFs:**  
> Dois conjuntos `F` e `G` são **equivalentes** (notação: `F ≡ G`) se e somente se:
>
> `F ↠ G  ∧  G ↠ F`
>
> Ou seja: `F⁺ = G⁺` — ambos derivam exatamente as mesmas DFs.

> **Teorema 5.1 — Equivalência via Fechamentos:**  
> `F ≡ G  ⟺  F⁺ = G⁺`  
> *Demonstração:*  
> (⟹) F ↠ G implica G ⊆ F⁺; G ↠ F implica F ⊆ G⁺. Por Armstrong, F⁺ ⊆ (G⁺)⁺ = G⁺ e G⁺ ⊆ F⁺, logo F⁺ = G⁺.  
> (⟸) Se F⁺ = G⁺, então G ⊆ G⁺ = F⁺ (F cobre G) e F ⊆ F⁺ = G⁺ (G cobre F). □

### 5.2 Algoritmo para Verificar Equivalência

```
Algoritmo EQUIVALENTE(F, G):
  Retorna true se F ≡ G

  Passo 1: Verificar se F cobre G
    Para cada (X → Y) em G:
      Calcular X⁺ usando F
      Se Y ⊄ X⁺: retornar false  ← F não cobre G

  Passo 2: Verificar se G cobre F
    Para cada (X → Y) em F:
      Calcular X⁺ usando G
      Se Y ⊄ X⁺: retornar false  ← G não cobre F

  Passo 3: retornar true
```

**Complexidade:** O(|F|·|G|·|atributos|²) — eficiente na prática.

### 5.3 Intuição

Dois conjuntos de DFs são equivalentes quando capturam exatamente a mesma **semântica** do esquema, mesmo que escritos de formas diferentes. É como ter duas representações diferentes da mesma lei de negócio — podem parecer distintas, mas impõem as mesmas restrições sobre os dados.

**Por que isso importa?** Ao normalizar, podemos substituir F por qualquer G ≡ F que seja mais simples (sua cobertura canônica), sem alterar as garantias de integridade do esquema.

### 5.4 Aplicação no Spotify

```
Esquema: ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago)

Conjunto F (forma expandida):
  F1: id_assinatura → id_usuario
  F2: id_assinatura → tipo_plano
  F3: id_assinatura → data_inicio
  F4: id_assinatura → data_fim
  F5: id_assinatura → valor_pago
  F6: tipo_plano → valor_pago

Conjunto G (forma compacta proposta):
  G1: id_assinatura → {id_usuario, tipo_plano, data_inicio, data_fim}
  G2: tipo_plano → valor_pago

─── Verificar F cobre G ───

  G1: X = {id_assinatura}, Y = {id_usuario, tipo_plano, data_inicio, data_fim}
    {id_assinatura}⁺ usando F:
      F1 → +id_usuario
      F2 → +tipo_plano
      F3 → +data_inicio
      F4 → +data_fim
      F5 → +valor_pago
      F6: tipo_plano ∈ resultado → +valor_pago (já está)
    = {id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago}
    Y ⊆ resultado ✅

  G2: X = {tipo_plano}, Y = {valor_pago}
    {tipo_plano}⁺ usando F:
      F6: tipo_plano → valor_pago → resultado = {tipo_plano, valor_pago}
    valor_pago ∈ resultado ✅

  → F cobre G ✅

─── Verificar G cobre F ───

  F1: {id_assinatura}⁺ usando G:
    G1 → {id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim}
    G2: tipo_plano ∈ resultado → +valor_pago
    = todos ✅ id_usuario ∈ resultado ✅

  F5: {id_assinatura}⁺ usando G (mesmo cálculo acima):
    valor_pago ∈ resultado ✅

  (Demais DFs de F: análogo — todas cobertas por G) ✅

  → G cobre F ✅

Conclusão: F ≡ G ✅
G é uma representação mais compacta da mesma semântica.
```

```
Exemplo de conjuntos NÃO equivalentes no Spotify:

H1 = { id_musica → titulo,
        id_musica → duracao_seg }

H2 = { id_musica → titulo }

H1 cobre H2? Sim (H2 ⊂ H1).
H2 cobre H1? 
  Verificar: id_musica → duracao_seg ∈ H2⁺?
  {id_musica}⁺ usando H2 = {id_musica, titulo}
  duracao_seg ∉ {id_musica, titulo} ✗

H1 ≢ H2  (H2 perde a DF sobre duracao_seg)
```

---

## 6. Cobertura Canônica

### 6.1 Definições Formais

> **Definição 6.1 — Atributo Extrâneo:**  
> Seja `F` um conjunto de DFs e `X → Y ∈ F`.  
>
> - Um atributo `A ∈ X` é **extrâneo no lado esquerdo** se:  
>   `Y ⊆ (X − {A})⁺_F`  
>   (Y ainda pode ser determinado sem A)
>
> - Um atributo `A ∈ Y` é **extrâneo no lado direito** se:  
>   `A ∈ X⁺_{F − {X→Y} ∪ {X→(Y−{A})}}`  
>   (A pode ser derivado mesmo removendo-o do consequente desta DF)  
>   Equivalentemente: `F` com `X → Y` substituída por `X → (Y − {A})` ainda implica `X → A`.

> **Definição 6.2 — Cobertura Canônica (Forma Canônica / Minimal Cover):**  
> Um conjunto `Fc` é a **cobertura canônica** de `F` se e somente se:
>
> 1. `Fc ≡ F`  (equivalência)  
> 2. Nenhuma DF em `Fc` possui atributo extrâneo no lado esquerdo  
> 3. Nenhuma DF em `Fc` possui atributo extrâneo no lado direito  
> 4. Nenhuma DF em `Fc` é redundante (pode ser removida mantendo equivalência)  
> 5. Cada DF em `Fc` tem **lado direito com exatamente 1 atributo** (forma canônica)

> **Teorema 6.1 — Existência:**  
> Todo conjunto finito de DFs `F` possui ao menos uma cobertura canônica `Fc`.  
> *Nota:* A cobertura canônica pode **não ser única** — podem existir múltiplas Fc equivalentes mínimas.

> **Teorema 6.2 — Uso na Síntese de 3FN:**  
> O algoritmo de síntese para 3FN utiliza a cobertura canônica `Fc` como entrada:  
> para cada DF `X → Y` em `Fc`, cria-se uma relação `R_i(X ∪ Y)` com `X` como chave.  
> O resultado preserva todas as DFs e está em 3FN.

### 6.2 Algoritmo Completo

```
Algoritmo CANONICAL_COVER(F):

─── Passo 1: Decomposição do lado direito ───
  Para cada DF X → {A₁, A₂, ..., Aₖ} com k > 1:
    Substituir por: X → A₁,  X → A₂,  ...,  X → Aₖ

─── Passo 2: Remoção de atributos extrâneos no lado esquerdo ───
  Para cada DF X → A em F (|X| > 1):
    Para cada B ∈ X:
      Calcular (X − {B})⁺ usando F atual
      Se A ∈ (X − {B})⁺:
        Substituir X → A por (X − {B}) → A em F

─── Passo 3: Remoção de DFs redundantes ───
  Para cada DF X → A em F:
    Calcular X⁺ usando F − {X → A}
    Se A ∈ X⁺:
      Remover X → A de F  (é redundante)

─── Retornar F como Fc ───

Observação: os passos 2 e 3 podem precisar ser repetidos,
pois remoções podem abrir novas redundâncias.
```

### 6.3 Intuição

A cobertura canônica é a "forma mais enxuta" de expressar as mesmas restrições semânticas. É análoga à forma irredutível de uma equação algébrica: `2x + 4 = 10` e `x = 3` expressam a mesma restrição, mas a segunda é mínima.

No contexto de banco de dados, usar `Fc` em vez de `F` para guiar a normalização garante que criamos o **menor número possível de tabelas**, sem perder nenhuma restrição de integridade.

### 6.4 Aplicação Detalhada no Spotify

```
─── Exemplo 1: PLAYLIST_MUSICA ───
Esquema: PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao)
Abreviações: P=id_playlist, M=id_musica, O=ordem, D=data_adicao

F inicial:
  DF1: (P, M) → O
  DF2: (P, M) → D
  DF3: (P, M) → {O, D}    ← composta

─── Passo 1: Decomposição ───
  DF3 → DF1 e DF2 (já existem) → remover DF3
  F = { (P,M) → O,  (P,M) → D }

─── Passo 2: Atributos extrâneos no lado esquerdo ───
  Para (P, M) → O: testar remover P:
    {M}⁺ usando F = {M} → O ∉ {M} ✗ → P não é extrâneo
  Testar remover M:
    {P}⁺ usando F = {P} → O ∉ {P} ✗ → M não é extrâneo
  Análogo para (P, M) → D.

─── Passo 3: Redundâncias ───
  Remover (P,M) → O:
    F' = { (P,M) → D }
    {P,M}⁺ usando F' = {P, M, D} → O ∉ resultado ✗ → NÃO redundante

  Remover (P,M) → D:
    F'' = { (P,M) → O }
    {P,M}⁺ usando F'' = {P, M, O} → D ∉ resultado ✗ → NÃO redundante

Fc = { (P,M) → O,  (P,M) → D }   ← já era minimal
```

```
─── Exemplo 2: REPRODUCAO com redundância ───
Abreviações: R=id_reproducao, U=id_usuario, M=id_musica, D=data_hora, S=segundos, V=dispositivo

F = { R→U, R→M, R→D, R→S, R→V, (U,M,D)→S, (U,M,D)→V }

─── Passo 1: Todas as DFs já têm 1 atributo no lado direito ───

─── Passo 2: Atributos extrâneos no lado esquerdo ───
  (U,M,D) → S: testar remover U:
    {M,D}⁺ = {M, D} → S ∉ resultado ✗ → U necessário
  Testar remover M:
    {U,D}⁺ = {U, D} → S ∉ resultado ✗ → M necessário
  Testar remover D:
    {U,M}⁺ = {U, M} → S ∉ resultado ✗ → D necessário
  Análogo para (U,M,D) → V.

─── Passo 3: Redundâncias ───
  Testar R → S (id_reproducao → segundos_ouvidos):
    F' = F − {R→S}
    {R}⁺ usando F':
      R→U: +U  →  R→M: +M  →  R→D: +D
      Agora {U,M,D} ⊆ resultado → (U,M,D)→S: +S  ✅
    S ∈ {R}⁺ → R→S é REDUNDANTE → REMOVER

  Testar R → V (id_reproducao → dispositivo):
    Análogo: R→U, R→M, R→D no resultado; (U,M,D)→V: +V
    V ∈ {R}⁺ → R→V é REDUNDANTE → REMOVER

  Testar R → U:
    F'' = F − {R→U} − {R→S} − {R→V}  (já removidos S, V)
    {R}⁺ usando F'': R→M: +M; R→D: +D; {U,M,D}→S requer U → sem U, não aplica
    U ∉ {R}⁺ → R→U NÃO é redundante ✅

  (Demais: R→M, R→D: análogo — não redundantes)

Fc = { R→U,  R→M,  R→D,
       (U,M,D)→S,  (U,M,D)→V }

Interpretação: id_reproducao determina apenas o contexto (quem, o quê, quando);
os valores de segundos e dispositivo são determinados pela chave natural (U,M,D).
```

```
─── Exemplo 3: ASSINATURA com dependência transitiva ───
F = { id_assinatura → id_usuario,
      id_assinatura → tipo_plano,
      id_assinatura → data_inicio,
      id_assinatura → data_fim,
      id_assinatura → valor_pago,
      tipo_plano → valor_pago }

─── Passo 3: Testar id_assinatura → valor_pago ───
  F' = F − {id_assinatura → valor_pago}
  {id_assinatura}⁺ usando F':
    +id_usuario, +tipo_plano, +data_inicio, +data_fim
    tipo_plano → valor_pago: +valor_pago ✅
  valor_pago ∈ resultado → id_assinatura → valor_pago é REDUNDANTE → REMOVER

Fc = { id_assinatura → id_usuario,
       id_assinatura → tipo_plano,
       id_assinatura → data_inicio,
       id_assinatura → data_fim,
       tipo_plano → valor_pago }       ← apenas 5 DFs, era 6

─── Implicação para normalização ───
Fc indica que o esquema ASSINATURA viola 3FN:
  tipo_plano → valor_pago   com   tipo_plano não sendo superchave

Solução: decompor em:
  PLANO(tipo_plano PK, valor_pago)
  ASSINATURA(id_assinatura PK, id_usuario FK, tipo_plano FK, data_inicio, data_fim)
```

### 6.5 Conexão com as Formas Normais

A cobertura canônica é a **entrada do algoritmo de síntese de 3FN**:

```
Algoritmo de Síntese 3FN (visão geral):

Entrada: esquema R, conjunto F
Saída:   decomposição em 3FN que preserva DFs e é com-perda-nula

1. Computar Fc = cobertura canônica de F
2. Para cada X → A em Fc:
     Criar relação Rᵢ com atributos X ∪ {A}, chave X
3. Se nenhuma relação criada contém chave candidata de R:
     Adicionar relação contendo uma chave candidata de R
4. Eliminar relações redundantes (subconjuntos de outras)
```

---

## 7. Anomalias no Modelo Relacional

### 7.1 Fundamento Teórico — Por que Anomalias Surgem

Anomalias são consequências diretas da presença de **dependências funcionais problemáticas** em um esquema mal projetado. Formalmente:

> **Definição 7.1 — Esquema com Redundância:**  
> Um esquema `R` com conjunto de DFs `F` possui **redundância** se existe uma DF `X → Y ∈ F⁺` tal que `X` não é superchave de `R` e `Y ⊄ X`.  
> Ou seja: um não-superchave determina outros atributos, forçando repetição de valores.

> **Teorema 7.1 — Condição Necessária para Anomalias:**  
> Se um esquema `R` está na **BCNF** (Boyce-Codd Normal Form), então `R` não possui anomalias de atualização, inserção ou remoção.  
> *Contraposição:* se `R` possui anomalias, então `R` não está em BCNF.

As três formas normais formam uma hierarquia de proteção contra anomalias:

```
1FN ⊃ 2FN ⊃ 3FN ⊃ BCNF ⊃ 4FN ⊃ 5FN

Cada forma normal elimina uma classe específica de anomalias.
```

#### Definições das Formas Normais

> **Definição 7.2 — Primeira Forma Normal (1FN):**  
> `R` está em 1FN se e somente se todos os domínios dos atributos contêm apenas valores atômicos (indivisíveis) — nenhum atributo contém conjuntos, listas ou relações aninhadas.

> **Definição 7.3 — Segunda Forma Normal (2FN):**  
> `R` está em 2FN se está em 1FN **e** todo atributo não-primo (não pertencente a nenhuma chave candidata) é **totalmente dependente** de cada chave candidata — sem dependências parciais.  
> Formalmente: `∄ X ⊊ CK, ∄ B não-primo : X → B ∈ F⁺`

> **Definição 7.4 — Terceira Forma Normal (3FN):**  
> `R` está em 3FN se está em 2FN **e** não existem dependências transitivas de atributos não-primos em relação a chaves candidatas.  
> Formalmente: para toda `X → A ∈ F⁺` não-trivial com `A` não-primo: `X` deve ser superchave de `R`.  
> Alternativa (Codd): para toda `X → A ∈ F⁺` não-trivial: `X` é superchave **ou** `A` é primo.

> **Definição 7.5 — Forma Normal de Boyce-Codd (BCNF):**  
> `R` está em BCNF se, para toda `X → A ∈ F⁺` não-trivial: `X` é superchave de `R`.  
> BCNF é mais restrita que 3FN: elimina também anomalias envolvendo atributos primos.

### 7.2 Os Três Tipos de Anomalia

#### Anomalia de Inserção

> **Definição 7.6:**  
> Uma **anomalia de inserção** ocorre quando não é possível inserir certos dados sem a presença de outros dados não relacionados, ou quando inserir um fato obriga a repetir informações já existentes.

#### Anomalia de Remoção

> **Definição 7.7:**  
> Uma **anomalia de remoção** ocorre quando deletar uma tupla causa a perda inadvertida de informações sobre outras entidades, pois múltiplas entidades foram colapsadas em uma única tabela.

#### Anomalia de Atualização

> **Definição 7.8:**  
> Uma **anomalia de atualização** (ou *update anomaly*) ocorre quando um único fato do mundo real está representado em múltiplas tuplas, exigindo que uma alteração seja propagada para todas elas para manter consistência. Atualização parcial gera **inconsistência**.

### 7.3 Aplicação no Spotify — Esquema Desnormalizado

```
Tabela desnormalizada FAIXA_COMPLETA:
┌──────────┬───────────────────────┬──────────┬──────────┬──────────────────┬────────────┬────────────┬──────────────┐
│ id_musica│ titulo_musica         │ id_album │ tit_album│ data_lancamento   │ id_artista │ nome_artis │ pais_artista │
├──────────┼───────────────────────┼──────────┼──────────┼──────────────────┼────────────┼────────────┼──────────────┤
│ 1        │ Blinding Lights       │ 10       │ After H. │ 2020-03-20       │ 5          │ The Weeknd │ CA           │
│ 2        │ Starboy               │ 11       │ Starboy  │ 2016-11-25       │ 5          │ The Weeknd │ CA           │
│ 3        │ Save Your Tears       │ 10       │ After H. │ 2020-03-20       │ 5          │ The Weeknd │ CA           │
│ 4        │ Bohemian Rhapsody     │ 20       │ News W.  │ 1977-10-31       │ 8          │ Queen      │ GB           │
└──────────┴───────────────────────┴──────────┴──────────┴──────────────────┴────────────┴────────────┴──────────────┘

DFs problemáticas:
  id_album → tit_album, data_lancamento   (id_album não é superchave)
  id_artista → nome_artista, pais_artista (id_artista não é superchave)
  → BCNF violada → anomalias esperadas
```

```
─── Anomalia de Inserção ───

Problema: Cadastrar novo artista "Doja Cat" (id=99) sem músicas ainda.

INSERT INTO FAIXA_COMPLETA VALUES
  (NULL, NULL, NULL, NULL, NULL, 99, 'Doja Cat', 'US')

→ id_musica é PK (ou parte dela) → NOT NULL → INSERT rejeitado
→ Impossível cadastrar artista sem música associada ✗

No modelo normalizado:
  INSERT INTO ARTISTA VALUES (99, 'Doja Cat', 'US', NULL, 'solo') ✅
  (independente de existirem músicas)
```

```
─── Anomalia de Remoção ───

Problema: Remover a música "Bohemian Rhapsody" (única do álbum "News of the World" nesta tabela)

DELETE FROM FAIXA_COMPLETA WHERE id_musica = 4

→ Junto com a música, perdemos:
   - Dados do álbum: "News of the World", data 1977-10-31
   - Dados do artista: Queen, GB
   (se for a última linha do artista/álbum)

No modelo normalizado:
  DELETE FROM MUSICA WHERE id_musica = 4
  → ALBUM, ARTISTA, MUSICA_ARTISTA permanecem intactos ✅
```

```
─── Anomalia de Atualização ───

Problema: The Weeknd (id=5) atualiza país de 'CA' para 'US'

UPDATE FAIXA_COMPLETA SET pais_artista = 'US'
WHERE id_artista = 5

→ 3 linhas afetadas (músicas 1, 2, 3)
→ Se apenas linha 1 for atualizada por engano:
  ┌──────────┬───────────────────┬────────────┬──────────────┐
  │ id_musica│ titulo            │ id_artista │ pais_artista │
  ├──────────┼───────────────────┼────────────┼──────────────┤
  │ 1        │ Blinding Lights   │ 5          │ US  ← novo   │
  │ 2        │ Starboy           │ 5          │ CA  ← ANTIGO │  inconsistente!
  │ 3        │ Save Your Tears   │ 5          │ CA  ← ANTIGO │  inconsistente!
  └──────────┴───────────────────┴────────────┴──────────────┘

No modelo normalizado:
  UPDATE ARTISTA SET pais_origem = 'US' WHERE id_artista = 5
  → 1 linha atualizada → consistência garantida ✅
```

### 7.4 Diagnóstico pelo Teste de Formas Normais

```
FAIXA_COMPLETA — diagnóstico completo:

1FN: ✅ (valores atômicos — generos foram extraídos)

2FN: ❌ PK hipotética = {id_musica, id_artista}
  id_album → tit_album         (id_album ⊊ PK → dependência parcial)
  id_artista → nome_artista    (id_artista ⊊ PK → dependência parcial)
  → Viola 2FN

3FN: ❌ (por consequência da violação de 2FN)

BCNF: ❌ id_album → tit_album, mas id_album não é superchave
         id_artista → nome_artista, id_artista não é superchave
```

---

## 8. Mapeamento do Modelo ER para o Relacional

### 8.1 Fundamentação Teórica

O mapeamento ER → Relacional é um processo de **transformação semântica preservadora**:

> **Definição 8.1 — Mapeamento Preservador:**  
> Um mapeamento `M: ER → Relacional` é **preservador** se toda instância válida do esquema ER pode ser representada como instância válida do esquema relacional resultante, e vice-versa — sem perda semântica.

> **Definição 8.2 — Decomposição Sem Perda (Lossless Join):**  
> Uma decomposição de `R` em `{R₁, R₂}` é **sem perda** se para toda instância `r` de `R`:
>
> `r = π_{R₁}(r) ⋈ π_{R₂}(r)`
>
> onde `π` é projeção e `⋈` é junção natural.  
> **Condição suficiente (Teorema de Heath):** A decomposição é sem perda se `R₁ ∩ R₂ → R₁` ou `R₁ ∩ R₂ → R₂` (a interseção é chave de um dos lados).

> **Definição 8.3 — Preservação de Dependências:**  
> Uma decomposição `{R₁, ..., Rₖ}` de `R` **preserva dependências** se:  
> `(F₁ ∪ F₂ ∪ ... ∪ Fₖ)⁺ = F⁺`  
> onde `Fᵢ` são as DFs de `F` projetadas sobre `Rᵢ`.

### 8.2 As Sete Regras de Mapeamento

#### Regra 1 — Entidade Comum

> Para cada entidade `E` com atributos `{A₁, ..., Aₙ}` e chave `K`:  
> Criar relação `R(A₁, ..., Aₙ)` com `PK = K`.  
> Atributos compostos são **achatados** (sub-atributos viram colunas).  
> Atributos derivados são **omitidos** (computados por VIEW).

```
DER: MUSICA(id_musica*, titulo, duracao_seg, letra, duracao_media*)
     (* = PK, * = derivado)

SQL: MUSICA(id_musica PK, titulo, duracao_seg, letra)
     -- duracao_media omitida: VIEW v_duracao_media
```

#### Regra 2 — Atributo Multivalorado

> Para cada atributo multivalorado `{M}` de entidade `E` com chave `K_E`:  
> Criar relação `R_M(K_E, M)` onde `PK = {K_E, M}` ou `PK = surrogate + FK`.

```
DER: ARTISTA com {redes_sociais} e MUSICA com {generos}

SQL: REDES_SOCIAIS(id_rede PK, id_artista FK, plataforma, url)
     GENERO_MUSICA(id_genero PK, id_musica FK, nome)

Justificativa formal: {redes_sociais} viola 1FN se armazenado como lista;
a tabela separada restaura a atomicidade.
```

#### Regra 3 — Atributo Derivado

> Atributos derivados **não geram colunas físicas**.  
> Implementar como VIEW ou função computada.

```
DER: ARTISTA.ouvintes_mensais* = COUNT DISTINCT usuários que ouviram no mês

VIEW:
CREATE VIEW v_ouvintes_mensais AS
SELECT   ma.id_artista,
         COUNT(DISTINCT r.id_usuario) AS ouvintes_mensais
FROM     MUSICA_ARTISTA ma
JOIN     REPRODUCAO r ON r.id_musica = ma.id_musica
WHERE    r.data_hora >= date_trunc('month', NOW())
GROUP BY ma.id_artista;
```

#### Regra 4 — Relacionamento 1:N

> Para relacionamento `1:N` entre `E₁` (lado 1) e `E₂` (lado N):  
> Adicionar `FK = PK(E₁)` em `R₂` (tabela do lado N).  
> Se participação de `E₂` for **total** (mandatória): `FK NOT NULL`.  
> Se **parcial** (opcional): `FK NULL` (ou `NOT NULL` com JOIN opcional).

```
Relacionamentos 1:N no Spotify:

  USUARIO(1) ─── cria ─── PLAYLIST(N) [total→total]
    → PLAYLIST.id_usuario_criador FK NOT NULL → USUARIO

  USUARIO(1) ─── gera ─── REPRODUCAO(N) [parcial→total]
    → REPRODUCAO.id_usuario FK NOT NULL

  MUSICA(1) ─── de ─── REPRODUCAO(N) [parcial→total]
    → REPRODUCAO.id_musica FK NOT NULL

  USUARIO(1) ─── possui ─── ASSINATURA(N) [parcial→total]
    → ASSINATURA.id_usuario FK NOT NULL

Justificativa formal (Teorema de Heath):
  Decompor R(usuario, playlist, ...) em USUARIO e PLAYLIST:
  Interseção = {id_usuario} = PK de USUARIO → decomposição sem perda ✅
```

#### Regra 5 — Entidade Fraca

> Para entidade fraca `W` com entidade forte `E`:  
> Criar `R_W` com FK para `E` (NOT NULL) + atributos discriminadores.  
> PK de `R_W` = FK(E) + discriminador (ou surrogate key).  
> Usar `ON DELETE CASCADE` (a parte não existe sem o todo).

```
DER: REPRODUCAO é entidade fraca dependendo de USUARIO e MUSICA

SQL:
CREATE TABLE REPRODUCAO (
    id_reproducao    SERIAL PRIMARY KEY,
    id_usuario       INT NOT NULL REFERENCES USUARIO(id_usuario) ON DELETE CASCADE,
    id_musica        INT NOT NULL REFERENCES MUSICA(id_musica)   ON DELETE RESTRICT,
    data_hora        TIMESTAMP NOT NULL,
    segundos_ouvidos INT NOT NULL,
    dispositivo      VARCHAR(30)
);

Semântica: deletar um USUARIO remove todas as suas REPRODUCAO (CASCADE).
           deletar uma MUSICA é bloqueado se há REPRODUCAO associadas (RESTRICT).
```

#### Regra 6 — Relacionamento N:M

> Para relacionamento `N:M` entre `E₁` e `E₂` com atributos `{a₁, ..., aₖ}`:  
> Criar tabela associativa `R_assoc(PK_E₁, PK_E₂, a₁, ..., aₖ)`.  
> PK composta: `(PK_E₁, PK_E₂)`.  
> Cada atributo de relacionamento vira coluna em `R_assoc`.

```
DER: PLAYLIST ─── contém ─── MUSICA  [N:M]
     Atributos: ordem, data_adicao

SQL:
CREATE TABLE PLAYLIST_MUSICA (
    id_playlist  INT REFERENCES PLAYLIST(id_playlist),
    id_musica    INT REFERENCES MUSICA(id_musica),
    ordem        INT NOT NULL,         ← atributo do relacionamento
    data_adicao  TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (id_playlist, id_musica)
);

Justificativa: ordem pertence à COMBINAÇÃO (playlist, música), não a cada um isolado.
A mesma música pode ser a faixa 1 de uma playlist e faixa 7 de outra.
```

#### Regra 7 — Relacionamento Recursivo N:M

> Para auto-relacionamento `N:M` em `E` com papéis `papel₁` e `papel₂`:  
> Criar tabela associativa com **dois FKs para a mesma tabela**, nomeados pelos papéis.  
> Adicionar `CHECK (pk_papel₁ ≠ pk_papel₂)` para evitar auto-referência.

```
DER: ARTISTA ─── membro de ─── ARTISTA  [recursivo N:M]
     Papéis: banda / membro
     Atributos: data_entrada, data_saida

SQL:
CREATE TABLE ARTISTA_MEMBRO (
    id_banda     INT REFERENCES ARTISTA(id_artista),  ← papel: banda
    id_membro    INT REFERENCES ARTISTA(id_artista),  ← papel: membro
    data_entrada DATE NOT NULL,
    data_saida   DATE,                                ← NULL = ainda na banda
    PRIMARY KEY (id_banda, id_membro),
    CHECK (id_banda <> id_membro)                     ← artista não é membro de si mesmo
);
```

### 8.3 Tabela-Resumo do Mapeamento Spotify

| Elemento do DER | Regra | Resultado | Tabelas geradas |
|---|---|---|---|
| 7 entidades simples | R1 | Tabelas diretas | USUARIO, ARTISTA, MUSICA, ALBUM, PLAYLIST, ASSINATURA, (REPRODUCAO) |
| `{redes_sociais}`, `{generos}` | R2 | Tabelas multivalorado | REDES_SOCIAIS, GENERO_MUSICA |
| 5 derivados | R3 | VIEWs | — (sem colunas) |
| *cria*, *possui*, *gera*, *de* | R4 | FKs em tabelas existentes | — (sem novas tabelas) |
| REPRODUCAO (fraca) | R5 | Tabela com 2 FKs NOT NULL + CASCADE | REPRODUCAO |
| 8 N:M diretos | R6 | Tabelas associativas | UCM, USA, UCP, PM, MA_pl, MA_al, AA, MA_ar |
| *segue usr*, *membro* | R7 | Tabelas com FK duplo | USU, ARTISTA_MEMBRO |
| **Total** | — | — | **18 tabelas** |

---

## 9. Estratégias para Design de Esquema e Normalização

### 9.1 Fundamentação Teórica

> **Definição 9.1 — Qualidade de Esquema:**  
> Um esquema relacional `S = {R₁, ..., Rₖ}` é considerado de **boa qualidade** se satisfaz:  
> 1. Cada relação representa **uma única entidade ou relacionamento** (semântica clara)  
> 2. Minimiza **redundância** (evita anomalias)  
> 3. Garante **sem perda** (lossless join) em decomposições  
> 4. **Preserva dependências** funcionais

> **Definição 9.2 — Decomposição:**  
> Uma decomposição de `R` é um conjunto `{R₁, R₂, ..., Rₖ}` tal que `R₁ ∪ R₂ ∪ ... ∪ Rₖ = R`.

> **Teorema 9.1 — Preservação de Informação:**  
> Uma decomposição `{R₁, R₂}` é sem perda se e somente se:  
> `(R₁ ∩ R₂) → R₁ ∈ F⁺`  **ou**  `(R₁ ∩ R₂) → R₂ ∈ F⁺`  
> (A interseção dos esquemas é chave de ao menos um deles.)

> **Teorema 9.2 — 3FN sempre decomponível com preservação:**  
> O algoritmo de síntese de 3FN sempre produz uma decomposição que:  
> (a) está em 3FN,  
> (b) é sem perda,  
> (c) preserva todas as dependências funcionais.

> **Teorema 9.3 — BCNF pode não preservar DFs:**  
> Existem esquemas para os quais a decomposição em BCNF não preserva todas as dependências funcionais. Nestes casos, deve-se aceitar 3FN como compromisso.

### 9.2 As Três Estratégias

#### Top-Down (ER → Relacional)

```
Base teórica: parte de um modelo conceitual semântico rico (ER/EER) e
aplica as 7 regras de mapeamento para obter o esquema relacional.

Garantias teóricas:
  - Mapeamento sem perda (por construção)
  - Preservação de dependências (por construção)
  - Frequentemente resulta em 3FN ou BCNF diretamente

Processo no Spotify:
  Fase 1 — Requisitos:
    "usuários ouvem músicas, criam playlists, seguem artistas e outros usuários"
  
  Fase 2 — Entidades:
    USUARIO, ARTISTA, MUSICA, ALBUM, PLAYLIST
  
  Fase 3 — Refinamentos:
    ASSINATURA separada de USUARIO (histórico de planos)
    REPRODUCAO como entidade fraca (rastrear cada play)
    REDES_SOCIAIS como multivalorado de ARTISTA
  
  Fase 4 — Relacionamentos:
    N:M: MUSICA↔ARTISTA (colaborações), PLAYLIST↔MUSICA (ordem),
         USUARIO↔USUARIO (follows recursivo), ARTISTA↔ARTISTA (bandas)
  
  Fase 5 — Mapeamento pelas 7 regras → 18 tabelas
```

#### Bottom-Up (Síntese por DFs)

```
Base teórica: parte dos atributos e DFs e aplica o Algoritmo de Síntese 3FN.

Algoritmo de Síntese 3FN:
  1. Seja U = conjunto de todos os atributos
  2. Calcular Fc = cobertura canônica de F
  3. Para cada X → A em Fc:
       Criar Rᵢ = X ∪ {A}  com chave X
  4. Se ∄ Rᵢ contendo alguma chave candidata de U:
       Adicionar relação com uma chave candidata
  5. Eliminar Rᵢ ⊆ Rⱼ (redundantes)

Aplicação no Spotify (parcial):
  U = {id_musica, titulo, duracao_seg, id_artista, nome_artista, pais_artista, ...}

  DFs F:
    id_musica → titulo, duracao_seg, letra
    id_artista → nome_artista, pais_artista, biografia
    (id_musica, id_artista) → papel

  Fc = { id_musica → titulo,
          id_musica → duracao_seg,
          id_musica → letra,
          id_artista → nome_artista,
          id_artista → pais_artista,
          id_artista → biografia,
          (id_musica, id_artista) → papel }

  Síntese:
    R₁(id_musica, titulo, duracao_seg, letra)         ← chave: id_musica
    R₂(id_artista, nome_artista, pais_artista, bio)   ← chave: id_artista
    R₃(id_musica, id_artista, papel)                  ← chave: (id_musica, id_artista)
  
  = MUSICA, ARTISTA, MUSICA_ARTISTA ✅
```

#### Inside-Out

```
Estratégia empírica: parte do núcleo de negócio e expande.

Não tem teorema associado — é uma heurística de engenharia de software.
Vantagem: alinha modelagem com prioridades de produto.

No Spotify:
  Nível 0 (núcleo):  MUSICA
  Nível 1:           ALBUM, ARTISTA (produtores e contêineres)
  Nível 2:           MUSICA_ARTISTA, MUSICA_ALBUM (relacionamentos centrais)
  Nível 3:           USUARIO, PLAYLIST (consumidores)
  Nível 4:           REPRODUCAO, ASSINATURA, USUARIO_CURTE_MUSICA
  Nível 5:           REDES_SOCIAIS, GENERO_MUSICA, tabelas recursivas
```

### 9.3 Quadro das Formas Normais com Testes Formais

```
─── 1FN ───
Teste: ∀ atributo A, ∀ tupla t: t[A] é atômico
Violação no Spotify: {generos} em MUSICA como lista → resolvido com GENERO_MUSICA
SQL: não usar arrays, JSON ou listas em colunas (sem suporte a normalização automática)

─── 2FN ───
Pré-condição: relação tem PK composta
Teste: ∄ atributo não-primo B, ∄ X ⊊ PK : X → B
Violação exemplo: MUSICA_ALBUM(id_musica, id_album, titulo_musica, faixa_numero)
  id_musica → titulo_musica (depende só de id_musica, não da PK composta toda)
  → separar MUSICA: titulo_musica vai para MUSICA

─── 3FN ───
Teste: para toda X → B não-trivial: X é superchave OU B é primo
Violação no Spotify:
  ASSINATURA: tipo_plano → valor_pago
  tipo_plano não é superchave; valor_pago não é primo
  → criar PLANO(tipo_plano, valor_pago)

─── BCNF ───
Teste: para toda X → B não-trivial: X é superchave (ponto final)
Mais restrita: elimina dependências envolvendo atributos primos também
Violação clássica:
  R(aluno, disciplina, professor) com:
    professor → disciplina (professor leciona uma disciplina)
    (aluno, disciplina) → professor (PK)
  professor não é superchave → viola BCNF
  Decompor: mas isso perde a DF (aluno, disciplina) → professor → aceitar 3FN
```

---

## 10. Integração de Esquemas

### 10.1 Fundamentação Teórica

> **Definição 10.1 — Problema de Integração:**  
> Dados `n` esquemas locais `S₁, S₂, ..., Sₙ` (possivelmente desenvolvidos de forma independente), o problema de integração consiste em produzir um esquema global unificado `S_G` tal que:
> 1. **Completude:** toda informação representável em algum `Sᵢ` é representável em `S_G`
> 2. **Minimalidade:** `S_G` não contém redundâncias além das necessárias
> 3. **Correção:** `S_G` é um esquema relacional válido e bem-formado
> 4. **Compreensibilidade:** `S_G` pode ser mapeado de volta para cada `Sᵢ` (via VIEWs de compatibilidade)

> **Definição 10.2 — Conflito de Esquemas:**  
> Um **conflito** entre esquemas `Sᵢ` e `Sⱼ` ocorre quando a mesma entidade/atributo do mundo real é representada de formas incompatíveis.

> **Classificação de conflitos (Batini, Ceri & Navathe, 1992):**

| Tipo | Definição Formal | Exemplo Spotify |
|---|---|---|
| **Nomenclatura** | `∃ conceito C : nome(C, Sᵢ) ≠ nome(C, Sⱼ)` | `faixa_id` vs `track_id` vs `id_musica` |
| **Estrutural** | `∃ conceito C : estrutura(C, Sᵢ) ≠ estrutura(C, Sⱼ)` | Artista como atributo vs entidade |
| **Tipo de dado** | `dom(A, Sᵢ) ≠ dom(A, Sⱼ)` | `duracao` como VARCHAR vs INTEGER |
| **Restrição** | `restr(A, Sᵢ) ≠ restr(A, Sⱼ)` | `visibilidade` enum vs booleano |
| **Semântico** | Mesmo nome, semântica diferente | `data` = data de criação vs data de lançamento |

### 10.2 Processo de Integração em 4 Etapas

#### Etapa 1 — Pré-integração: Análise e Classificação de Conflitos

```
Módulos independentes do Spotify:

Módulo A — Time Conteúdo:
  FAIXA(faixa_id INTEGER, nome VARCHAR, duracao VARCHAR, artista_nome VARCHAR, artista_pais CHAR)
  DISCO(disco_id INTEGER, titulo VARCHAR, ano INTEGER, tipo VARCHAR, artista_nome VARCHAR)

Módulo B — Time Usuário:
  CONTA(conta_id INTEGER, email VARCHAR UNIQUE, nome_completo VARCHAR,
        nascimento DATE, pais_residencia CHAR)
  LISTA(lista_id INTEGER, conta_id INTEGER FK, nome_lista VARCHAR, visibilidade VARCHAR)
  FAIXA_LISTA(lista_id INTEGER FK, faixa_id INTEGER FK, posicao INTEGER)

Módulo C — Time Analytics:
  PLAY(play_id INTEGER, user_email VARCHAR, track_id INTEGER,
       ts TIMESTAMP, device VARCHAR, secs_played INTEGER)
  ARTISTA(artista_id INTEGER, artista_nome VARCHAR, pais CHAR, bio TEXT)

─── Inventário de conflitos ───

| Conflito | Tipo | Módulos | Descrição |
|---|---|---|---|
| faixa_id / track_id / id_musica | Nomenclatura | A, B, C | Mesmo conceito, 3 nomes |
| artista_nome em FAIXA vs ARTISTA tabela | Estrutural | A, C | Atributo vs Entidade |
| duracao VARCHAR vs INTEGER | Tipo | A, — | "3:54" vs 234 segundos |
| visibilidade VARCHAR vs publica BOOLEAN | Restrição | B, — | 'público'/'privado' vs TRUE/FALSE |
| user_email vs conta_id como identificador | Chave | B, C | email vs surrogate key |
| data em FAIXA vs DISCO (qual lançamento?) | Semântico | A | Ambiguidade |
```

#### Etapa 2 — Correspondência de Entidades (Entity Matching)

```
Processo formal: para cada par de entidades (Eᵢ, Eⱼ) de módulos diferentes,
verificar se representam o mesmo conceito do mundo real.

Métrica de similaridade (simplificada):
  sim(Eᵢ, Eⱼ) = w₁·sim_nome + w₂·sim_atributos + w₃·sim_semantica

Correspondências encontradas:

  Módulo A: FAIXA           ↔  Módulo C: PLAY.track_id    ↔  RESULTADO: MUSICA
  Módulo A: DISCO           ↔  (sem correspondente)        ↔  RESULTADO: ALBUM (novo)
  Módulo B: CONTA           ↔  Módulo C: PLAY.user_email   ↔  RESULTADO: USUARIO
  Módulo B: LISTA           ↔  (sem conflito)              ↔  RESULTADO: PLAYLIST
  Módulo C: ARTISTA         ↔  Módulo A: FAIXA.artista_nome ↔ RESULTADO: ARTISTA (entidade)
  Módulo A: FAIXA.artista   ≅  Módulo C: ARTISTA           → fusão necessária
```

#### Etapa 3 — Resolução de Conflitos

```
─── Conflito de nomenclatura: faixa_id / track_id ───
  Decisão: adotar id_musica (convenção snake_case + prefixo id_)
  Mapeamento: faixa_id → id_musica, track_id → id_musica

─── Conflito estrutural: artista como atributo (Módulo A) vs entidade (Módulo C) ───
  FAIXA.artista_nome → promover para entidade ARTISTA (Módulo C vence)
  Criar relacionamento N:M: MUSICA_ARTISTA(id_musica, id_artista, papel)
  Justificativa: uma música pode ter múltiplos artistas (colaborações)
    → atributo simples seria insuficiente

─── Conflito de tipo: duracao VARCHAR("3:54") → INTEGER ───
  Script de migração:
    SELECT EXTRACT(MINUTE FROM duracao::TIME)*60
         + EXTRACT(SECOND FROM duracao::TIME) AS duracao_seg
  Decisão: INTEGER seconds (canônico, comparável, somável)

─── Conflito de restrição: visibilidade VARCHAR → publica BOOLEAN ───
  Mapeamento:
    'público'   → TRUE
    'privado'   → FALSE
    'somente_eu'→ FALSE (com flag adicional colaborativa)

─── Conflito de chave: email vs id como identificador de usuário ───
  Decisão: criar surrogate key id_usuario INTEGER (PK) + email UNIQUE (alternada)
  Compatibilidade: PLAY.user_email referencia via JOIN:
    PLAY.user_email → JOIN USUARIO ON email = user_email → obter id_usuario
```

#### Etapa 4 — Geração do Esquema Global e VIEWs de Compatibilidade

```
Esquema global resultante da integração:

Do Módulo A: MUSICA(id_musica, titulo, duracao_seg, letra)
             ALBUM(id_album, titulo, data_lancamento, capa_url, tipo)
Do Módulo B: USUARIO(id_usuario, email, nome, data_nascimento, pais, plano)
             PLAYLIST(id_playlist, id_usuario_criador, nome, descricao, publica, colaborativa)
             PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao)
Do Módulo C: ARTISTA(id_artista, nome_artistico, pais_origem, biografia, tipo)
             REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, seg_ouvidos, dispositivo)
Novos (integração):
             MUSICA_ARTISTA(id_musica, id_artista, papel)
             ALBUM_ARTISTA(id_album, id_artista, papel)
             USUARIO_SEGUE_USUARIO(id_seguidor, id_seguido, data_inicio)

─── VIEWs de compatibilidade para transição gradual ───
```

```sql
-- VIEW que emula interface do Módulo A (FAIXA) após migração
CREATE VIEW FAIXA AS
  SELECT
    m.id_musica                          AS faixa_id,
    m.titulo                             AS nome,
    TO_CHAR(INTERVAL '1 second' * m.duracao_seg, 'MI:SS') AS duracao,
    MAX(a.nome_artistico)                AS artista_nome,
    MAX(a.pais_origem)                   AS artista_pais
  FROM MUSICA m
  JOIN MUSICA_ARTISTA ma ON ma.id_musica = m.id_musica
  JOIN ARTISTA a         ON a.id_artista = ma.id_artista
  WHERE ma.papel = 'principal'
  GROUP BY m.id_musica, m.titulo, m.duracao_seg;

-- VIEW que emula interface do Módulo C (PLAY) após migração
CREATE VIEW PLAY AS
  SELECT
    r.id_reproducao    AS play_id,
    u.email            AS user_email,
    r.id_musica        AS track_id,
    r.data_hora        AS ts,
    r.dispositivo      AS device,
    r.segundos_ouvidos AS secs_played
  FROM REPRODUCAO r
  JOIN USUARIO u ON u.id_usuario = r.id_usuario;

-- VIEW que emula CONTA do Módulo B
CREATE VIEW CONTA AS
  SELECT
    id_usuario          AS conta_id,
    email,
    nome                AS nome_completo,
    data_nascimento     AS nascimento,
    pais                AS pais_residencia
  FROM USUARIO;
```

### 10.3 Boas Práticas Teóricas para Integração

| Princípio | Base Teórica | Aplicação no Spotify |
|---|---|---|
| **Preferir entidades a atributos** | Evita dependências transitivas e parciais | `artista_nome` → entidade ARTISTA |
| **Surrogate keys** | Desacopla identidade técnica de semântica | `id_usuario` em vez de `email` como PK |
| **VIEWs de compatibilidade** | Preserva interfaces sem duplicar dados | PLAY, FAIXA, CONTA como views |
| **Documentar decisões** | Rastreabilidade | Cada conflito resolvido registrado |
| **Validar com dados reais** | Empirismo | Queries de reconciliação pré-migração |
| **Normalizar após integrar** | Teoremas 9.1-9.3 | Verificar 3FN/BCNF no esquema global |

---

## Resumo Geral — Conexões entre os Tópicos

```
Atributos e Domínios
      ↓ definem
Esquema de Relação (Modelo Relacional)
      ↓ sobre o qual se definem
Dependências Funcionais (DFs)
      ↓ usadas para calcular
Fechamento de Atributos (X⁺)
      ↓ que permite verificar
Chaves Candidatas (X⁺ = R e mínimo)
      ↓ e verificar
Equivalência de Conjuntos de DFs
      ↓ leading to
Cobertura Canônica (Fc mínimo equivalente)
      ↓ que diagnostica
Anomalias (via violações de 1FN/2FN/3FN/BCNF)
      ↓ corrigidas por
Normalização (decomposição sem perda + preservação)
      ↓ guiada por
Estratégias de Design (Top-down, Bottom-up, Inside-out)
      ↓ aplicadas em
Mapeamento ER → Relacional (7 regras)
      ↓ e no contexto de múltiplas fontes por
Integração de Esquemas (resolução de conflitos + VIEWs)
```

| Tópico | Conceito Formal Central | Ferramenta Algorítmica | Garantia Teórica |
|---|---|---|---|
| Modelo Relacional | Relação como subconjunto do produto cartesiano | Verificação de domínios | Integridade de entidade e domínio |
| Chaves | Superchave mínima; integridade referencial | Verificação de unicidade | `∀ t₁≠t₂: t₁[PK]≠t₂[PK]` |
| DFs | `X → Y`: `t₁[X]=t₂[X] ⟹ t₁[Y]=t₂[Y]` | Axiomas de Armstrong | Sistema sólido e completo |
| Fechamento | `X⁺ = {A | X→A ∈ F⁺}` | CLOSURE(X, F) | `X→Y ∈ F⁺ ⟺ Y ⊆ X⁺` |
| Equivalência | `F⁺ = G⁺` | EQUIVALENTE(F, G) | Verificação bidirecional de cobertura |
| Cob. Canônica | `Fc ≡ F`, mínimo, sem extrâneos | CANONICAL_COVER(F) | Entrada do algoritmo 3FN |
| Anomalias | Violação de BCNF → anomalias | Teste de formas normais | BCNF ⟹ sem anomalias |
| Mapeamento ER | 7 regras preservadoras | Sistemático | Sem perda + preserva DFs |
| Estratégias | Síntese 3FN (bottom-up) | Algoritmo de síntese | Teorema 9.2 — 3FN sempre atingível |
| Integração | Resolução de conflitos + VIEWs | Entity matching | Completude + minimalidade |
