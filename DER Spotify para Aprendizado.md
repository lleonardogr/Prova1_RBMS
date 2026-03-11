# DER — Plataforma de Streaming Musical (estilo Spotify)
> Notação: Mermaid `erDiagram` · Resolução completa do desafio

---

## Diagrama Entidade-Relacionamento

```mermaid
erDiagram

    USUARIO {
        int     id_usuario       PK
        string  nome
        string  email
        date    data_nascimento
        string  pais
        string  plano
        float   engajamento_mensal "DERIVADO: total min ouvidos/mês"
    }

    ARTISTA {
        int     id_artista        PK
        string  nome_artistico
        string  pais_origem
        text    biografia
        string  tipo              "solo | banda"
        int     ouvintes_mensais  "DERIVADO: ouvintes únicos/mês"
    }

    REDES_SOCIAIS {
        int     id_rede      PK
        int     id_artista   FK
        string  plataforma
        string  url
    }

    MUSICA {
        int     id_musica    PK
        string  titulo
        int     duracao_seg
        text    letra        "nullable"
        float   duracao_media "DERIVADO: média tempo ouvido"
    }

    GENERO {
        int     id_genero    PK
        int     id_musica    FK
        string  nome
    }

    ALBUM {
        int     id_album         PK
        string  titulo
        date    data_lancamento
        string  capa_url
        string  tipo             "single | EP | album"
        int     total_faixas     "DERIVADO: count de músicas"
    }

    PLAYLIST {
        int     id_playlist    PK
        string  nome
        text    descricao
        boolean publica
        boolean colaborativa
        date    data_criacao
    }

    REPRODUCAO {
        int     id_reproducao    PK
        int     id_usuario       FK
        int     id_musica        FK
        datetime data_hora
        int     segundos_ouvidos
        string  dispositivo      "celular | desktop | smart_tv | ..."
        boolean ouviu_completo   "DERIVADO: seg_ouvidos >= duracao_seg"
    }

    ASSINATURA {
        int     id_assinatura  PK
        int     id_usuario     FK
        string  tipo_plano     "gratuito | premium"
        date    data_inicio
        date    data_fim       "nullable: null = ativa"
        float   valor_pago
    }

    %% ─── Atributos multivalorados representados como entidades associativas ───
    %% {redes_sociais} de ARTISTA → entidade REDES_SOCIAIS
    %% {generos}       de MUSICA  → entidade GENERO

    %% ─── RELACIONAMENTOS ───────────────────────────────────────────────────────

    %% USUÁRIO — cria — PLAYLIST (1:N, total nos dois lados)
    USUARIO ||--o{ PLAYLIST : "cria"

    %% USUÁRIO — colabora em — PLAYLIST (N:M, parcial nos dois lados)
    %% Atributos do rel.: data_entrada
    USUARIO }o--o{ PLAYLIST : "colabora em"

    %% USUÁRIO — curte — MÚSICA (N:M, parcial)
    %% Atributos do rel.: data_curtida
    USUARIO }o--o{ MUSICA : "curte"

    %% USUÁRIO — segue — ARTISTA (N:M, parcial)
    %% Atributos do rel.: data_inicio
    USUARIO }o--o{ ARTISTA : "segue artista"

    %% USUÁRIO — segue — USUÁRIO (N:M RECURSIVO, parcial)
    %% Papéis: seguidor / seguido · Atributo: data_inicio
    USUARIO }o--o{ USUARIO : "segue usuário [recursivo]"

    %% USUÁRIO — possui — ASSINATURA (1:N; ASSINATURA total, USUÁRIO parcial)
    USUARIO ||--o{ ASSINATURA : "possui"

    %% USUÁRIO — gera — REPRODUÇÃO (1:N; REPRODUÇÃO total = entidade fraca)
    USUARIO ||--o{ REPRODUCAO : "gera"

    %% REPRODUÇÃO — de — MÚSICA (N:1; REPRODUÇÃO total, MÚSICA parcial)
    REPRODUCAO }o--|| MUSICA : "de"

    %% PLAYLIST — contém — MÚSICA (N:M, parcial)
    %% Atributos do rel.: ordem, data_adicao
    PLAYLIST }o--o{ MUSICA : "contém"

    %% MÚSICA — pertence a — ÁLBUM (N:M; ÁLBUM total, MÚSICA parcial)
    %% Atributos do rel.: faixa_numero
    MUSICA }o--o{ ALBUM : "pertence a"

    %% ÁLBUM — lançado por — ARTISTA (N:M, total nos dois lados)
    %% Atributos do rel.: papel (principal | featuring)
    ALBUM }|--|{ ARTISTA : "lançado por"

    %% MÚSICA — composta por — ARTISTA (N:M, parcial)
    %% Atributos do rel.: papel (principal | featuring)
    MUSICA }o--o{ ARTISTA : "composta por"

    %% ARTISTA — membro de — ARTISTA (N:M RECURSIVO, parcial)
    %% Papéis: banda / membro · Atributos: data_entrada, data_saida
    ARTISTA }o--o{ ARTISTA : "membro de [recursivo]"

    %% Multivalorados como entidades auxiliares
    ARTISTA ||--o{ REDES_SOCIAIS : "possui redes"
    MUSICA  ||--o{ GENERO        : "classificada em"
```

---

## Legenda de Cardinalidade (Mermaid)

| Símbolo Mermaid | Leitura                        |
|-----------------|--------------------------------|
| `\|\|`          | Exatamente um (mandatório)     |
| `o\|`           | Zero ou um (opcional)          |
| `\|{`           | Um ou mais (mandatório)        |
| `o{`            | Zero ou mais (opcional)        |
| `}o--o{`        | N:M parcial nos dois lados     |
| `\|\|--o{`      | 1:N — um lado total, outro parcial |
| `}o--\|\|`      | N:1 — lado N parcial, lado 1 total |

---

## Atributos Especiais

### Derivados *(não armazenados — calculados sob demanda)*

| Entidade      | Atributo             | Fórmula                                      |
|---------------|----------------------|----------------------------------------------|
| `USUARIO`     | `engajamento_mensal` | Σ `segundos_ouvidos` / 60 no mês corrente    |
| `ARTISTA`     | `ouvintes_mensais`   | COUNT DISTINCT `id_usuario` em REPRODUCAO/mês |
| `MUSICA`      | `duracao_media`      | AVG `segundos_ouvidos` em REPRODUCAO         |
| `ALBUM`       | `total_faixas`       | COUNT músicas vinculadas ao álbum            |
| `REPRODUCAO`  | `ouviu_completo`     | `segundos_ouvidos >= duracao_seg` da música  |

### Multivalorados *(representados como entidades auxiliares no Mermaid)*

| Entidade  | Atributo original | Entidade auxiliar  |
|-----------|-------------------|--------------------|
| `ARTISTA` | `{redes_sociais}` | `REDES_SOCIAIS`    |
| `MUSICA`  | `{generos}`       | `GENERO`           |

> **Nota:** A notação Chen representa multivalorados com elipse dupla. No Mermaid `erDiagram`, a convenção é criar uma entidade auxiliar ligada por 1:N.

---

## Relacionamentos com Atributos Próprios

| Relacionamento            | Atributos                    |
|---------------------------|------------------------------|
| `USUARIO — cria — PLAYLIST`        | `data_criacao`               |
| `USUARIO — colabora em — PLAYLIST` | `data_entrada`               |
| `USUARIO — curte — MUSICA`         | `data_curtida`               |
| `USUARIO — segue — ARTISTA`        | `data_inicio`                |
| `USUARIO — segue — USUARIO`        | `data_inicio`                |
| `PLAYLIST — contém — MUSICA`       | `ordem`, `data_adicao`       |
| `MUSICA — pertence a — ALBUM`      | `faixa_numero`               |
| `ALBUM — lançado por — ARTISTA`    | `papel` (principal/featuring)|
| `MUSICA — composta por — ARTISTA`  | `papel` (principal/featuring)|
| `ARTISTA — membro de — ARTISTA`    | `data_entrada`, `data_saida` |

> **Nota:** O Mermaid `erDiagram` não suporta atributos em relacionamentos nativamente. Em uma implementação real, esses relacionamentos N:M viram **tabelas associativas** com os atributos listados.

---

## Pontos de Destaque da Resolução

### Entidade Fraca — `REPRODUÇÃO`
`REPRODUCAO` é uma entidade fraca pois **sua existência depende simultaneamente de `USUARIO` e `MUSICA`**. Sem um usuário ou sem uma música referenciada, uma reprodução não tem sentido semântico. Na notação Chen, seria representada com retângulo de borda dupla.

### Relacionamentos Recursivos
- **`USUARIO — segue — USUARIO`**: auto-relacionamento N:M com papéis *seguidor* e *seguido*, ambos opcionais — um usuário pode não seguir ninguém e pode não ser seguido por ninguém.
- **`ARTISTA — membro de — ARTISTA`**: modela bandas sem criar uma entidade separada. Os atributos `data_entrada` e `data_saida` permitem rastrear o histórico de formações.

### Decisão de Modelagem — `REPRODUÇÃO` como Entidade vs. Relacionamento
Embora `REPRODUCAO` possa ser vista inicialmente como um relacionamento entre `USUARIO` e `MUSICA`, ela é modelada como **entidade** porque possui atributos próprios relevantes (`data_hora`, `segundos_ouvidos`, `dispositivo`) e é consultada de forma independente (histórico de plays, analytics). Relacionamentos com muitos atributos e que precisam ser referenciados diretamente devem virar entidades.

### Distinção `cria` vs. `colabora em`
São dois relacionamentos distintos entre `USUARIO` e `PLAYLIST` para separar o **dono** (que tem controle total, participação mandatória) dos **colaboradores** (opcionais, presentes apenas em playlists colaborativas).
