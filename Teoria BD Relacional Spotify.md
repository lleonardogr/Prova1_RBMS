# Banco de Dados Relacionais — Teoria Completa
### Contexto: Plataforma de Streaming Musical (Spotify)

> Todos os conceitos são ilustrados com tabelas e exemplos extraídos diretamente do modelo relacional do Spotify desenvolvido em aula.

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
9. [Estratégias para Design de Esquema](#9-estratégias-para-design-de-esquema)
10. [Integração de Esquemas](#10-integração-de-esquemas)

---

## 1. Modelo Relacional

O **modelo relacional** representa dados como coleções de **relações** (tabelas), onde cada relação é um conjunto de tuplas (linhas) que compartilham os mesmos atributos (colunas). Proposto por E. F. Codd em 1970, é a base de todos os SGBDs como PostgreSQL, MySQL e Oracle.

### Conceitos fundamentais

| Conceito formal | Equivalente prático | Exemplo no Spotify |
|---|---|---|
| **Relação** | Tabela | `MUSICA`, `USUARIO`, `PLAYLIST` |
| **Tupla** | Linha / registro | Uma música específica |
| **Atributo** | Coluna | `titulo`, `duracao_seg` |
| **Domínio** | Tipo + restrições do atributo | `duracao_seg`: INTEGER > 0 |
| **Grau** | Número de atributos | `MUSICA` tem grau 4 |
| **Cardinalidade** | Número de tuplas | Quantas músicas existem |
| **Esquema** | Estrutura declarada | `MUSICA(id_musica, titulo, duracao_seg, letra)` |

### Propriedades de uma relação válida

- Cada célula contém um valor **atômico** (1FN — sem listas, sem grupos repetidos)
- Não há tuplas duplicadas
- A ordem das tuplas **não importa**
- A ordem dos atributos **não importa** (identificados pelo nome, não pela posição)

### Exemplo no Spotify

```
Esquema:  MUSICA(id_musica, titulo, duracao_seg, letra)

Instância (exemplo de tuplas):
┌───────────┬─────────────────────────┬─────────────┬────────┐
│ id_musica │ titulo                  │ duracao_seg │ letra  │
├───────────┼─────────────────────────┼─────────────┼────────┤
│ 1         │ Bohemian Rhapsody       │ 354         │ ...    │
│ 2         │ Blinding Lights         │ 200         │ ...    │
│ 3         │ Shape of You            │ 234         │ NULL   │
└───────────┴─────────────────────────┴─────────────┴────────┘
```

> **Domínio de `duracao_seg`:** INTEGER NOT NULL CHECK > 0  
> **Domínio de `letra`:** TEXT, aceita NULL (nem toda faixa tem letra cadastrada)

---

## 2. Chaves

Chaves identificam tuplas de forma única dentro de uma relação. Cada tipo tem um papel específico no design do esquema.

### 2.1 Superchave

Uma **superchave** é qualquer conjunto de atributos que identifica unicamente cada tupla. Pode conter atributos redundantes.

```
Tabela: USUARIO(id_usuario, nome, email, data_nascimento, pais, plano)

Superchaves válidas:
  {id_usuario}
  {email}
  {id_usuario, nome}
  {id_usuario, email, pais}       ← redundante, mas ainda é superchave
  {nome, email, pais, plano, ...} ← todos os atributos juntos também é superchave
```

### 2.2 Chave Candidata

Uma **chave candidata** é uma superchave **mínima** — remover qualquer atributo dela faz com que ela deixe de identificar unicamente as tuplas.

```
USUARIO: chaves candidatas →  {id_usuario}   e   {email}
  ↳ id_usuario identifica unicamente cada usuário
  ↳ email também é único (restrição UNIQUE no esquema)

REPRODUCAO: chave candidata →  {id_reproducao}
  ↳ (id_usuario, id_musica, data_hora) também poderia ser candidata
     se assumirmos que um usuário não pode reproduzir a mesma música
     duas vezes no exato mesmo instante
```

### 2.3 Chave Primária

A **chave primária (PK)** é a chave candidata escolhida pelo designer para identificar as tuplas. Não pode ser NULL.

```sql
-- Chaves primárias do modelo Spotify:
USUARIO           → PK: id_usuario
ARTISTA           → PK: id_artista
MUSICA            → PK: id_musica
ALBUM             → PK: id_album
PLAYLIST          → PK: id_playlist
ASSINATURA        → PK: id_assinatura
REPRODUCAO        → PK: id_reproducao

-- Tabelas associativas com PK composta:
USUARIO_CURTE_MUSICA      → PK: (id_usuario, id_musica)
PLAYLIST_MUSICA           → PK: (id_playlist, id_musica)
MUSICA_ALBUM              → PK: (id_musica, id_album)
USUARIO_SEGUE_USUARIO     → PK: (id_seguidor, id_seguido)
ARTISTA_MEMBRO            → PK: (id_banda, id_membro)
```

### 2.4 Chave Alternada

A **chave alternada** é toda chave candidata que **não foi escolhida** como primária. É implementada com `UNIQUE NOT NULL`.

```
USUARIO:
  PK escolhida  → id_usuario
  Alternada     → email          (UNIQUE NOT NULL)

MUSICA:
  PK escolhida  → id_musica
  Alternada     → não há (título pode se repetir entre artistas diferentes)
```

### 2.5 Chave Estrangeira

A **chave estrangeira (FK)** é um atributo (ou conjunto) em uma tabela que referencia a PK de outra tabela, garantindo **integridade referencial**.

```
PLAYLIST.id_usuario_criador  →  FK referencia  USUARIO(id_usuario)
ASSINATURA.id_usuario        →  FK referencia  USUARIO(id_usuario)
REPRODUCAO.id_usuario        →  FK referencia  USUARIO(id_usuario)
REPRODUCAO.id_musica         →  FK referencia  MUSICA(id_musica)
REDES_SOCIAIS.id_artista     →  FK referencia  ARTISTA(id_artista)
GENERO_MUSICA.id_musica      →  FK referencia  MUSICA(id_musica)

-- Tabelas associativas têm múltiplos FKs:
USUARIO_CURTE_MUSICA.id_usuario  →  FK → USUARIO(id_usuario)
USUARIO_CURTE_MUSICA.id_musica   →  FK → MUSICA(id_musica)
```

> **Regra de integridade referencial:** não é possível inserir uma reprodução com `id_usuario = 99` se nenhum usuário com esse id existir em `USUARIO`.

### Resumo visual das chaves

```
USUARIO
┌──────────────┬────────────────────────────────────────────┐
│ id_usuario   │ PK  ←─── referenciada por PLAYLIST,        │
│              │          ASSINATURA, REPRODUCAO, etc.      │
│ email        │ Chave Alternada (UNIQUE NOT NULL)           │
│ nome         │ não é chave                                 │
└──────────────┴────────────────────────────────────────────┘

REPRODUCAO
┌──────────────────┬────────────────────────────────────────┐
│ id_reproducao    │ PK                                     │
│ id_usuario       │ FK → USUARIO(id_usuario)               │
│ id_musica        │ FK → MUSICA(id_musica)                  │
│ data_hora        │ atributo comum                         │
│ segundos_ouvidos │ atributo comum                         │
└──────────────────┴────────────────────────────────────────┘
```

---

## 3. Dependência Funcional

Uma **dependência funcional (DF)** expressa uma restrição semântica entre atributos: dado o valor de X, existe **exatamente um** valor possível para Y.

**Notação:** `X → Y` ("X determina Y" / "Y é funcionalmente dependente de X")

### Definição formal

`X → Y` em relação R se e somente se: para quaisquer duas tuplas t₁ e t₂ em R, se `t₁[X] = t₂[X]`, então `t₁[Y] = t₂[Y]`.

### Dependências no modelo Spotify

```
Tabela MUSICA(id_musica, titulo, duracao_seg, letra):
  id_musica → titulo           ✅ cada id tem exatamente um título
  id_musica → duracao_seg      ✅
  id_musica → letra            ✅ (mesmo que seja NULL)
  id_musica → {titulo, duracao_seg, letra}   ✅ (determinação de conjunto)

  titulo → id_musica           ❌ dois artistas podem ter músicas homônimas
  duracao_seg → titulo         ❌ várias músicas têm a mesma duração
```

```
Tabela ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago):
  id_assinatura → id_usuario       ✅
  id_assinatura → tipo_plano       ✅
  id_assinatura → data_inicio      ✅
  id_assinatura → valor_pago       ✅
  tipo_plano    → valor_pago       ✅ (regra de negócio: premium sempre custa R$ 21,90)
  id_usuario    → tipo_plano       ❌ um usuário pode ter histórico de planos diferentes
```

```
Tabela REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo):
  id_reproducao → {id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo}  ✅
  (id_usuario, id_musica, data_hora) → {segundos_ouvidos, dispositivo}               ✅
  id_usuario → dispositivo          ❌ mesmo usuário usa múltiplos dispositivos
```

### Tipos de Dependência Funcional

| Tipo | Definição | Exemplo no Spotify |
|---|---|---|
| **Trivial** | Y ⊆ X (Y já está contido em X) | `{id_musica, titulo} → titulo` |
| **Não-trivial** | Y ⊄ X | `id_musica → titulo` |
| **Parcial** | X → Y onde Y depende de subconjunto de X | Ver seção de anomalias |
| **Transitiva** | X → Z via X → Y → Z | `id_assinatura → valor_pago` via `tipo_plano` |
| **Total** | Y depende de toda a chave X, não de parte dela | `(id_usuario, id_musica) → data_curtida` |

### Axiomas de Armstrong

Os axiomas de Armstrong permitem derivar **todas** as DFs que valem em um esquema a partir de um conjunto base:

| Axioma | Regra | Exemplo |
|---|---|---|
| **Reflexividade** | Se Y ⊆ X, então X → Y | `{id_musica, titulo} → titulo` |
| **Aumentatividade** | Se X → Y, então XZ → YZ | `id_musica → titulo` ⟹ `{id_musica, duracao_seg} → {titulo, duracao_seg}` |
| **Transitividade** | Se X → Y e Y → Z, então X → Z | `id_assinatura → tipo_plano` e `tipo_plano → valor_pago` ⟹ `id_assinatura → valor_pago` |

---

## 4. Fechamento de Atributos

O **fechamento de X** (notação: X⁺) é o conjunto de **todos os atributos** que podem ser determinados funcionalmente a partir de X, usando as DFs disponíveis.

### Algoritmo de fechamento

```
Entrada: conjunto X, conjunto F de DFs
Saída:   X⁺

1. Inicialize resultado ← X
2. Repita até estabilizar:
   Para cada DF (α → β) em F:
     Se α ⊆ resultado:
       resultado ← resultado ∪ β
3. Retorne resultado
```

### Exemplo com REPRODUCAO

```
Esquema:  REPRODUCAO(id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo)

Conjunto F de DFs:
  F1: id_reproducao → id_usuario
  F2: id_reproducao → id_musica
  F3: id_reproducao → data_hora
  F4: id_reproducao → segundos_ouvidos
  F5: id_reproducao → dispositivo
  F6: (id_usuario, id_musica, data_hora) → segundos_ouvidos
  F7: (id_usuario, id_musica, data_hora) → dispositivo

Calcular {id_reproducao}⁺:
  Início:        resultado = {id_reproducao}
  Aplica F1:     resultado = {id_reproducao, id_usuario}
  Aplica F2:     resultado = {id_reproducao, id_usuario, id_musica}
  Aplica F3:     resultado = {id_reproducao, id_usuario, id_musica, data_hora}
  Aplica F6:     resultado = {id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos}
  Aplica F4:     (já contém segundos_ouvidos — sem mudança)
  Aplica F7:     resultado = {id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo}
  Aplica F5:     (já contém dispositivo — sem mudança)

  {id_reproducao}⁺ = {id_reproducao, id_usuario, id_musica, data_hora, segundos_ouvidos, dispositivo}
  → Contém TODOS os atributos → id_reproducao é superchave (e candidata, pois é mínima)
```

### Como usar o fechamento para verificar chaves candidatas

Para verificar se X é chave candidata de R:

1. Calcule X⁺
2. Se X⁺ = todos os atributos de R → X é **superchave**
3. Verifique se é **mínima**: para cada atributo A em X, calcule (X − {A})⁺
4. Se (X − {A})⁺ ≠ todos os atributos para todo A → X é **chave candidata**

```
Verificar se (id_usuario, id_musica, data_hora) é chave candidata em REPRODUCAO:

  (id_usuario, id_musica, data_hora)⁺ = todos os atributos ✅ é superchave

  Testar minimalidade:
    {id_musica, data_hora}⁺ = {id_musica, data_hora}  ← não alcança tudo → necessário
    {id_usuario, data_hora}⁺ = {id_usuario, data_hora} ← não alcança tudo → necessário
    {id_usuario, id_musica}⁺ = {id_usuario, id_musica} ← não alcança tudo → necessário

  Conclusão: (id_usuario, id_musica, data_hora) é chave candidata ✅
```

### Uso prático: verificar se uma DF pertence ao fechamento

Para verificar se `X → Y` pode ser derivada do conjunto F (i.e., se `X → Y ∈ F⁺`):

```
Basta calcular X⁺ e verificar se Y ⊆ X⁺

Exemplo: F contém {id_assinatura → tipo_plano, tipo_plano → valor_pago}
Verificar se id_assinatura → valor_pago ∈ F⁺:

  {id_assinatura}⁺:
    Aplica id_assinatura → tipo_plano  → {id_assinatura, tipo_plano}
    Aplica tipo_plano → valor_pago     → {id_assinatura, tipo_plano, valor_pago}

  valor_pago ∈ {id_assinatura, tipo_plano, valor_pago} ✅
  Portanto: id_assinatura → valor_pago ∈ F⁺
```

---

## 5. Equivalência de Dependências Funcionais

Dois conjuntos de DFs **F** e **G** são **equivalentes** (F ≡ G) se e somente se cobrem exatamente as mesmas dependências funcionais — ou seja:

- **F cobre G**: toda DF de G pode ser derivada a partir de F
- **G cobre F**: toda DF de F pode ser derivada a partir de G

### Algoritmo para verificar se F cobre G

Para cada DF `X → Y` em G:
1. Calcule X⁺ usando **apenas as DFs de F**
2. Verifique se Y ⊆ X⁺

Se isso vale para todas as DFs de G, então F cobre G.

### Exemplo com ASSINATURA

```
Esquema: ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago)

Conjunto F (original):
  F1: id_assinatura → id_usuario
  F2: id_assinatura → tipo_plano
  F3: id_assinatura → data_inicio
  F4: id_assinatura → data_fim
  F5: id_assinatura → valor_pago
  F6: tipo_plano → valor_pago

Conjunto G (alternativo proposto):
  G1: id_assinatura → {id_usuario, tipo_plano, data_inicio, data_fim}
  G2: tipo_plano → valor_pago
  G3: id_assinatura → id_usuario        ← redundante?

Verificar se F cobre G:
  G1: {id_assinatura}⁺ usando F = todos os atributos → contém {id_usuario, tipo_plano, data_inicio, data_fim} ✅
  G2: {tipo_plano}⁺ usando F = {tipo_plano, valor_pago} → contém valor_pago ✅
  G3: {id_assinatura}⁺ usando F = todos → contém id_usuario ✅
  → F cobre G ✅

Verificar se G cobre F:
  F5: {id_assinatura}⁺ usando G:
    G1 → {id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim}
    G2 → {id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago}
    → contém valor_pago ✅
  Demais DFs de F: verificação análoga ✅
  → G cobre F ✅

Conclusão: F ≡ G (são equivalentes, apesar de terem formas diferentes)
```

### Por que isso importa?

Ao normalizar um esquema, podemos substituir um conjunto de DFs por um equivalente mais simples (sua **cobertura canônica**) sem perder informação semântica.

---

## 6. Cobertura Canônica

A **cobertura canônica** (Fc) de um conjunto F é um conjunto **equivalente e mínimo** de DFs, sem redundâncias. É usada como base para a normalização em 3FN.

### Propriedades da cobertura canônica

1. **Sem atributo extrâneo no lado esquerdo** — nenhum atributo pode ser removido de X em `X → Y` sem perder poder
2. **Sem atributo extrâneo no lado direito** — nenhum atributo pode ser removido de Y sem perder poder
3. **Sem DF redundante** — nenhuma DF pode ser removida sem alterar o fechamento

### Algoritmo

```
Entrada: F
Saída:   Fc (cobertura canônica)

Passo 1 — Decomposição (lado direito atômico):
  Substituir X → {A, B, C} por X → A,  X → B,  X → C

Passo 2 — Remover atributos extrâneos do lado esquerdo:
  Para cada DF X → Y onde |X| > 1:
    Para cada atributo A em X:
      Se Y ⊆ (X − {A})⁺ usando F:
        Substituir X por (X − {A})

Passo 3 — Remover DFs redundantes:
  Para cada DF X → Y em F:
    Se Y ⊆ {X}⁺ calculado com F − {X → Y}:
      Remover X → Y de F
```

### Exemplo com PLAYLIST_MUSICA

```
Esquema: PLAYLIST_MUSICA(id_playlist, id_musica, ordem, data_adicao)
PK: (id_playlist, id_musica)

Conjunto F inicial:
  D1: (id_playlist, id_musica) → ordem
  D2: (id_playlist, id_musica) → data_adicao
  D3: (id_playlist, id_musica) → {ordem, data_adicao}   ← composta

─── Passo 1: Decomposição ───
  D1: (id_playlist, id_musica) → ordem
  D2: (id_playlist, id_musica) → data_adicao
  (D3 já foi decomposta em D1 e D2 — remover D3)

─── Passo 2: Atributos extrâneos no lado esquerdo ───
  D1: Tentar remover id_playlist:
    {id_musica}⁺ = {id_musica}  → não contém ordem ❌ → necessário
  D1: Tentar remover id_musica:
    {id_playlist}⁺ = {id_playlist} → não contém ordem ❌ → necessário
  → Nenhum atributo extrâneo

─── Passo 3: DFs redundantes ───
  Remover D1 temporariamente:
    F' = {D2}
    {id_playlist, id_musica}⁺ em F' = {id_playlist, id_musica, data_adicao}
    → não contém ordem → D1 NÃO é redundante ✅
  Remover D2 temporariamente:
    F'' = {D1}
    {id_playlist, id_musica}⁺ em F'' = {id_playlist, id_musica, ordem}
    → não contém data_adicao → D2 NÃO é redundante ✅

Fc = { (id_playlist, id_musica) → ordem,
       (id_playlist, id_musica) → data_adicao }
```

### Exemplo mais rico com REPRODUCAO

```
F original (com redundância proposital para o exemplo):
  R1: id_reproducao → id_usuario
  R2: id_reproducao → id_musica
  R3: id_reproducao → data_hora
  R4: id_reproducao → segundos_ouvidos
  R5: id_reproducao → dispositivo
  R6: (id_usuario, id_musica, data_hora) → segundos_ouvidos
  R7: (id_usuario, id_musica, data_hora) → dispositivo
  R8: id_reproducao → {id_usuario, id_musica, data_hora}   ← composta

─── Passo 1: Decomposição ───
  R8 se torna R1, R2, R3 (já existem — remover R8)

─── Passo 3: Remover redundâncias ───
  R4 (id_reproducao → segundos_ouvidos):
    Sem R4, {id_reproducao}⁺ via R1+R2+R3 nos dá {id_usuario, id_musica, data_hora}
    então R6 se aplica → chegamos a segundos_ouvidos ✅
    → R4 é REDUNDANTE → remover

  R5 (id_reproducao → dispositivo): análogo → REDUNDANTE → remover

Fc = { id_reproducao → id_usuario,
       id_reproducao → id_musica,
       id_reproducao → data_hora,
       (id_usuario, id_musica, data_hora) → segundos_ouvidos,
       (id_usuario, id_musica, data_hora) → dispositivo }
```

---

## 7. Anomalias no Modelo Relacional

**Anomalias** são problemas que surgem quando um esquema está mal projetado — geralmente por misturar informações de entidades distintas em uma só tabela. São de três tipos: inserção, remoção e atualização.

### Tabela com anomalias — Spotify "desnormalizado"

Imagine que, em vez de tabelas separadas, tivéssemos uma única tabela:

```
FAIXA_COMPLETA(
  id_musica, titulo_musica, duracao_seg,
  id_album, titulo_album, data_lancamento, tipo_album,
  id_artista, nome_artista, pais_artista,
  faixa_numero
)
```

### 7.1 Anomalia de Inserção

**Problema:** não é possível inserir dados de uma entidade sem ter dados de outra.

```
Cenário: Um novo artista foi cadastrado na plataforma mas ainda não lançou nenhum álbum.

INSERT INTO FAIXA_COMPLETA VALUES
  (NULL, NULL, NULL, NULL, NULL, NULL, NULL, 99, 'Novo Artista', 'BR', NULL)

→ PROBLEMA: id_musica, id_album e faixa_numero são NULL mas fazem parte da PK composta
→ Não é possível inserir o artista sem uma música associada
```

**Solução no Spotify normalizado:** tabelas `ARTISTA` e `MUSICA` são independentes — um artista existe sem precisar de músicas cadastradas.

### 7.2 Anomalia de Remoção

**Problema:** remover dados de uma entidade destrói inadvertidamente dados de outra.

```
Cenário: A única música do álbum "Debut" do artista Björk é removida.

DELETE FROM FAIXA_COMPLETA WHERE id_musica = 42

→ PROBLEMA: junto com a música, perdemos TODOS os dados do álbum "Debut"
   e possivelmente os dados da própria Björk se fosse a única linha dela
```

**Solução:** tabelas `ALBUM` e `ARTISTA` separadas preservam seus dados independentemente.

### 7.3 Anomalia de Atualização

**Problema:** um mesmo dado está repetido em várias linhas; atualizar em uma sem atualizar em todas gera **inconsistência**.

```
Cenário: O artista "The Weeknd" muda seu país cadastrado de 'CA' para 'US'.

┌───────────┬────────────────────────┬─────────────┬──────────────────┐
│ id_musica │ titulo_musica          │ id_artista  │ pais_artista     │
├───────────┼────────────────────────┼─────────────┼──────────────────┤
│ 1         │ Blinding Lights        │ 10          │ CA  ← atualizado │
│ 2         │ Starboy                │ 10          │ CA  ← ESQUECIDO  │
│ 3         │ Save Your Tears        │ 10          │ CA  ← ESQUECIDO  │
└───────────┴────────────────────────┴─────────────┴──────────────────┘

→ PROBLEMA: 3 linhas precisam ser atualizadas; se apenas 1 for, o banco fica inconsistente
```

**Solução:** `pais_artista` fica em `ARTISTA`, com apenas **1 linha por artista** — 1 UPDATE resolve.

### Resumo das anomalias

| Anomalia | Causa raiz | Consequência |
|---|---|---|
| **Inserção** | Dados de entidades distintas acoplados | Não consegue inserir uma entidade sem a outra |
| **Remoção** | Mesma mistura | Deletar um dado destrói dados não relacionados |
| **Atualização** | Dado repetido em múltiplas linhas | Inconsistência por atualização parcial |

> **Solução geral:** normalização (1FN → 2FN → 3FN → BCNF), separando cada entidade em sua própria tabela.

---

## 8. Mapeamento do Modelo ER para o Relacional

O mapeamento segue um conjunto de **regras sistemáticas** para converter cada elemento do DER em tabelas relacionais.

### Regra 1 — Entidade comum

Cada entidade vira uma tabela. Atributos simples viram colunas. A PK do DER vira a PK da tabela.

```
DER:   MUSICA(id_musica, titulo, duracao_seg, letra)
       └─ id_musica é PK

SQL:   MUSICA(id_musica PK, titulo, duracao_seg, letra)
```

### Regra 2 — Atributo multivalorado `{ }`

Cada atributo multivalorado vira uma nova tabela com FK para a entidade original.

```
DER:   ARTISTA com {redes_sociais}
DER:   MUSICA  com {generos}

SQL:   REDES_SOCIAIS(id_rede PK, id_artista FK, plataforma, url)
SQL:   GENERO_MUSICA(id_genero PK, id_musica FK, nome)
```

### Regra 3 — Atributo derivado

Atributos derivados **não viram colunas**. São calculados por queries ou VIEWs.

```
DER:   ARTISTA.ouvintes_mensais*   (derivado)
DER:   USUARIO.engajamento_mensal* (derivado)

SQL:   Não geram coluna.

VIEW exemplo:
  CREATE VIEW v_ouvintes_mensais AS
  SELECT id_artista, COUNT(DISTINCT r.id_usuario) AS ouvintes_mensais
  FROM   MUSICA_ARTISTA ma
  JOIN   REPRODUCAO r ON r.id_musica = ma.id_musica
  WHERE  r.data_hora >= date_trunc('month', NOW())
  GROUP  BY id_artista;
```

### Regra 4 — Relacionamento 1:N

A FK fica na tabela do lado **N**. Se a participação do lado N for total (mandatória), a FK é `NOT NULL`.

```
DER:   USUARIO (1) ─────── cria ─────── (N) PLAYLIST   [total nos dois]
DER:   USUARIO (1) ─────── possui ───── (N) ASSINATURA [ASSINATURA total, USUARIO parcial]

SQL:   PLAYLIST   recebe  id_usuario_criador FK NOT NULL → USUARIO
SQL:   ASSINATURA recebe  id_usuario         FK NOT NULL → USUARIO
```

```
DER:   USUARIO (1,parcial) ─── gera ─── (N,total) REPRODUCAO
DER:   MUSICA  (1,parcial) ─── de  ─── (N,total) REPRODUCAO

SQL:   REPRODUCAO recebe  id_usuario FK NOT NULL → USUARIO
SQL:   REPRODUCAO recebe  id_musica  FK NOT NULL → MUSICA
```

### Regra 5 — Entidade fraca

A entidade fraca vira tabela com FK `NOT NULL` para a entidade forte, integrando a PK composta.

```
DER:   REPRODUCAO é entidade fraca (depende de USUARIO e MUSICA)

SQL:   REPRODUCAO(
         id_reproducao  PK,
         id_usuario     FK NOT NULL → USUARIO,
         id_musica      FK NOT NULL → MUSICA,
         ...
       )
       ON DELETE CASCADE  ← deletar usuário deleta suas reproduções
```

### Regra 6 — Relacionamento N:M

Cria-se uma **tabela associativa** com as PKs das duas entidades como FKs, formando uma PK composta. Atributos do relacionamento viram colunas nessa tabela.

```
DER:   USUARIO ─── curte ─── MUSICA   [N:M, parcial]
       atributo do relacionamento: data_curtida

SQL:   USUARIO_CURTE_MUSICA(
         id_usuario   FK → USUARIO,
         id_musica    FK → MUSICA,
         data_curtida TIMESTAMP NOT NULL,
         PRIMARY KEY (id_usuario, id_musica)
       )

DER:   PLAYLIST ─── contém ─── MUSICA   [N:M]
       atributos: ordem, data_adicao

SQL:   PLAYLIST_MUSICA(
         id_playlist  FK → PLAYLIST,
         id_musica    FK → MUSICA,
         ordem        INTEGER NOT NULL,
         data_adicao  TIMESTAMP,
         PRIMARY KEY (id_playlist, id_musica)
       )
```

### Regra 7 — Relacionamento recursivo N:M

Tabela associativa com dois FKs apontando para a **mesma tabela**, com nomes de papel distintos.

```
DER:   USUARIO ─── segue ─── USUARIO   [N:M recursivo]
DER:   ARTISTA ─── membro de ─── ARTISTA   [N:M recursivo]

SQL:   USUARIO_SEGUE_USUARIO(
         id_seguidor  FK → USUARIO(id_usuario),
         id_seguido   FK → USUARIO(id_usuario),
         data_inicio  TIMESTAMP,
         PRIMARY KEY (id_seguidor, id_seguido),
         CHECK (id_seguidor <> id_seguido)
       )

SQL:   ARTISTA_MEMBRO(
         id_banda     FK → ARTISTA(id_artista),
         id_membro    FK → ARTISTA(id_artista),
         data_entrada DATE NOT NULL,
         data_saida   DATE,
         PRIMARY KEY (id_banda, id_membro),
         CHECK (id_banda <> id_membro)
       )
```

### Resumo do mapeamento aplicado ao Spotify

| Elemento do DER | Regra aplicada | Resultado |
|---|---|---|
| 7 entidades simples | Regra 1 | 7 tabelas diretas |
| `{redes_sociais}`, `{generos}` | Regra 2 | `REDES_SOCIAIS`, `GENERO_MUSICA` |
| 5 atributos derivados | Regra 3 | Nenhuma coluna — usar VIEWs |
| *cria*, *possui*, *gera*, *de* | Regra 4 | FKs em PLAYLIST, ASSINATURA, REPRODUCAO |
| REPRODUCAO (fraca) | Regra 5 | PK + 2 FKs NOT NULL com CASCADE |
| 8 relacionamentos N:M | Regra 6 | 8 tabelas associativas |
| *segue usuário*, *membro de* | Regra 7 | 2 tabelas com FK duplo para mesma tabela |
| **Total** | — | **18 tabelas** |

---

## 9. Estratégias para Design de Esquema

Ao projetar um banco de dados, o engenheiro pode partir de diferentes pontos. As três principais estratégias são complementares e frequentemente usadas em conjunto.

### 9.1 Top-Down (de cima para baixo)

Parte dos **requisitos de alto nível** e vai refinando até chegar às tabelas.

```
Etapas no projeto do Spotify:
  1. Requisitos: "usuários ouvem músicas, criam playlists, seguem artistas"
  2. Entidades de alto nível: USUARIO, MUSICA, ARTISTA, PLAYLIST
  3. Refinamento: ALBUM separa-se de MUSICA; ASSINATURA separa-se de USUARIO
  4. Atributos identificados: email único, plano com enum, duracao_seg > 0
  5. Relacionamentos formalizados: N:M entre MUSICA e ARTISTA (colaborações)
  6. Mapeamento para tabelas relacionais

Vantagem:  cobre toda a modelagem, menos risco de esquecer entidades
Desvantagem: demorado; requer visão completa do sistema antes de começar
```

### 9.2 Bottom-Up (de baixo para cima)

Parte dos **atributos individuais** e agrupa-os em relações usando dependências funcionais (base da normalização).

```
Etapas:
  1. Listar todos os atributos do sistema:
     {id_usuario, nome, email, id_musica, titulo, duracao_seg, id_artista,
      nome_artista, id_album, ...}

  2. Identificar dependências funcionais:
     id_musica → titulo
     id_musica → duracao_seg
     id_artista → nome_artista
     id_album → titulo_album
     ...

  3. Agrupar por chave comum:
     {id_musica}    → MUSICA(id_musica, titulo, duracao_seg, letra)
     {id_artista}   → ARTISTA(id_artista, nome_artistico, pais, ...)
     {id_album}     → ALBUM(id_album, titulo, data_lancamento, ...)

  4. Aplicar normalização para eliminar anomalias

Vantagem:  rigoroso, elimina redundâncias desde o início
Desvantagem: difícil de aplicar em sistemas grandes sem visão holística
```

### 9.3 Inside-Out (de dentro para fora)

Começa pelas **entidades centrais** (núcleo do negócio) e expande para as periféricas.

```
No Spotify:
  Núcleo:     MUSICA  (a razão de ser da plataforma)
  1ª expansão: ALBUM, ARTISTA (quem faz e agrupa a música)
  2ª expansão: USUARIO, PLAYLIST (quem consome)
  3ª expansão: REPRODUCAO, ASSINATURA (métricas e monetização)
  4ª expansão: REDES_SOCIAIS, GENERO_MUSICA (metadados)
  Associativas: MUSICA_ARTISTA, MUSICA_ALBUM, PLAYLIST_MUSICA...

Vantagem:  natural para equipes de produto; alinha com prioridades de negócio
Desvantagem: periferias podem ficar mal modeladas se não revisadas ao final
```

### 9.4 Formas Normais como Estratégia de Validação

Independente da estratégia usada, o esquema deve ser validado contra as formas normais:

| Forma Normal | Regra | Violação no Spotify (exemplo) |
|---|---|---|
| **1FN** | Atributos atômicos, sem grupos repetidos | `{generos}` em MUSICA → resolvido com `GENERO_MUSICA` |
| **2FN** | Sem dependências parciais da PK | Em tabela associativa, atributo dependendo só de parte da PK composta |
| **3FN** | Sem dependências transitivas | `tipo_plano → valor_pago` em ASSINATURA → candidato a separar |
| **BCNF** | Todo determinante é superchave | Versão mais forte da 3FN |

```
Exemplo de violação de 3FN em ASSINATURA e correção:

ANTES (viola 3FN):
  ASSINATURA(id_assinatura, id_usuario, tipo_plano, data_inicio, data_fim, valor_pago)
  DFs: id_assinatura → tipo_plano
       tipo_plano    → valor_pago   ← dependência transitiva!

DEPOIS (3FN):
  PLANO(tipo_plano PK, valor_pago)
  ASSINATURA(id_assinatura PK, id_usuario FK, tipo_plano FK, data_inicio, data_fim)
```

---

## 10. Integração de Esquemas

A **integração de esquemas** ocorre quando múltiplos esquemas independentes (desenvolvidos por equipes diferentes, para módulos distintos, ou herdados de sistemas legados) precisam ser **unificados** em um esquema global consistente.

### Cenário no Spotify

Imagine que o Spotify foi construído em três módulos independentes por times diferentes:

```
Módulo A — Time de Conteúdo:
  FAIXA(faixa_id, nome, duracao, artista_nome, artista_pais)
  DISCO(disco_id, titulo, ano, tipo, artista_nome)

Módulo B — Time de Usuário:
  CONTA(conta_id, email, nome_completo, nascimento, pais_residencia)
  LISTA(lista_id, conta_id, nome_lista, visibilidade)
  FAIXA_LISTA(lista_id, faixa_id, posicao)

Módulo C — Time de Analytics:
  PLAY(play_id, user_email, track_id, ts, device, secs_played)
  ARTISTA(artista_id, artista_nome, pais, bio)
```

### Etapas da integração

#### Etapa 1 — Identificação de conflitos

| Tipo de conflito | Exemplo | Resolução |
|---|---|---|
| **Nomenclatura** | `faixa_id` vs `track_id` vs `id_musica` | Padronizar para `id_musica` |
| **Tipo de dado** | `duracao` (VARCHAR "3:54") vs `duracao_seg` (INTEGER 234) | Converter para INTEGER |
| **Estrutura** | `artista_nome` embutido em FAIXA vs tabela ARTISTA separada | Extrair para tabela própria |
| **Chave** | Módulo B usa `email` como identificador; Módulo C usa `user_email` | Criar `id_usuario` sintético |
| **Semântico** | `visibilidade` ('público'/'privado') vs `publica` (boolean) | Padronizar para boolean |

#### Etapa 2 — Correspondência de entidades

```
Módulo A: FAIXA         ≅  Módulo C: PLAY.track_id    →  entidade unificada: MUSICA
Módulo A: DISCO         ≅  (sem correspondente direto) →  entidade nova: ALBUM
Módulo B: CONTA         ≅  Módulo C: PLAY.user_email   →  entidade unificada: USUARIO
Módulo B: LISTA         ≅  (sem conflito)              →  PLAYLIST
Módulo C: ARTISTA       ≅  Módulo A: FAIXA.artista_nome →  entidade unificada: ARTISTA
```

#### Etapa 3 — Resolução de redundâncias

```
PROBLEMA: artista_nome aparece como atributo em FAIXA (Módulo A)
          e como entidade em ARTISTA (Módulo C)

ANTES (integração ingênua):
  MUSICA(id_musica, titulo, duracao_seg, artista_nome)  ← redundância
  ARTISTA(id_artista, artista_nome, pais, bio)

DEPOIS (integração correta):
  MUSICA(id_musica, titulo, duracao_seg)
  ARTISTA(id_artista, nome_artistico, pais_origem, biografia)
  MUSICA_ARTISTA(id_musica FK, id_artista FK, papel)   ← N:M
```

#### Etapa 4 — Esquema integrado final

```
Resultado da integração dos 3 módulos:

  Do Módulo A:  → MUSICA, ALBUM  (com ARTISTA extraído de atributo para entidade)
  Do Módulo B:  → USUARIO (renomeado de CONTA), PLAYLIST (de LISTA), PLAYLIST_MUSICA
  Do Módulo C:  → ARTISTA (consolidado), REPRODUCAO (de PLAY, com id_usuario via FK)

  Novas tabelas geradas na integração:
    MUSICA_ARTISTA   (N:M que não existia em nenhum módulo isolado)
    ALBUM_ARTISTA    (idem)
    USUARIO_SEGUE_USUARIO (novo requisito descoberto durante integração)
```

### Boas práticas para integração

- **Padronizar nomes** antes de integrar: usar snake_case, prefixo `id_` para PKs, sufixo `_at` para timestamps
- **Não destruir esquemas legados imediatamente**: usar VIEWs de compatibilidade durante a transição
- **Documentar decisões**: cada conflito resolvido deve ter justificativa registrada
- **Validar com dados reais**: rodar queries de reconciliação para encontrar inconsistências antes de migrar

```sql
-- Exemplo de VIEW de compatibilidade durante migração
-- Mantém interface do Módulo C enquanto dados migram para esquema unificado

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
```

---

## Resumo Geral

| Tópico | Conceito-chave | Exemplo Spotify |
|---|---|---|
| **Modelo Relacional** | Dados como tabelas com tuplas atômicas | `MUSICA`, `USUARIO`, `ARTISTA` |
| **Chaves** | Identificação única de tuplas | `id_musica` PK; `email` alternada; `id_artista` FK em `REDES_SOCIAIS` |
| **Dependência Funcional** | `X → Y`: X determina Y | `id_musica → titulo`; `tipo_plano → valor_pago` |
| **Fechamento** | X⁺: tudo que X determina | `{id_reproducao}⁺` = todos os atributos |
| **Equivalência** | F ≡ G: mesmas DFs, formas diferentes | Conjuntos F e G cobrem-se mutuamente |
| **Cobertura Canônica** | Fc: conjunto mínimo equivalente a F | Eliminar R4, R5 de REPRODUCAO por transitividade |
| **Anomalias** | Inserção, remoção, atualização | Tabela FAIXA_COMPLETA desnormalizada |
| **Mapeamento ER→Relacional** | 7 regras sistemáticas | 18 tabelas geradas do DER |
| **Estratégias de Design** | Top-down, Bottom-up, Inside-out | Spotify partindo de MUSICA como núcleo |
| **Integração** | Unificação de esquemas conflitantes | Fusão dos módulos Conteúdo + Usuário + Analytics |
