# MVP_Eng_Dados











# Dicionário de Dados – Camada Business/Gold

**Origem dos dados**: camada `trusted`  
**Destino**: camada `business`  
**Padrão de nomenclatura**:  
- `d_` = tabelas dimensão  
- `f_` = tabelas fato

---

## 🔷 Tabela: `business.d_players` — Dimensão Jogadores

| Campo         | Tipo     | Descrição                              | Domínio / Exemplo               |
|---------------|----------|----------------------------------------|---------------------------------|
| `player_id`   | STRING   | Identificador único do jogador         | "101501", "200123"              |
| `player_name` | STRING   | Nome completo do jogador               | "Roger Federer"                 |
| `hand`        | CHAR(1)  | Mão dominante                          | "R" = destro, "L" = canhoto     |
| `height`      | INT      | Altura em centímetros                  | 180–210 cm                      |
| `ioc`         | STRING   | Código do país (ISO 3 letras)          | "SUI", "ESP", "USA"             |
| `date_of_birth` | DATE   | Data de nascimento do jogador          | "1981-08-08"                    |

---

## 🔷 Tabela: `business.d_tournaments` — Dimensão Torneios

| Campo           | Tipo    | Descrição                                | Domínio / Exemplo                   |
|-----------------|---------|------------------------------------------|-------------------------------------|
| `tourney_id`    | STRING  | Identificador único do torneio           | "2021-888", "2019-030"              |
| `tourney_name`  | STRING  | Nome do torneio                          | "Wimbledon", "Roland Garros"        |
| `surface`       | STRING  | Tipo de quadra                           | "Clay", "Grass", "Hard"             |
| `draw_size`     | INT     | Nº de jogadores no torneio               | 32, 64, 128                         |
| `tourney_level` | STRING  | Nível do torneio                         | "G" (Grand Slam), "M", "A", "C"...  |

---

## 🔷 Tabela: `business.d_calendario` — Dimensão Calendario

| Campo         | Tipo   | Descrição                                | Exemplo            |
|---------------|--------|------------------------------------------|--------------------|
| `date_id`     | DATE   | Data da semana do torneio                | "2021-06-28"       |
| `ano`         | INT    | Ano do torneio                           | 2021               |
| `mes`         | INT    | Mês do torneio                           | 6                  |
| `nome_mes`    | STRING | Nome do Mês do torneio                   | June               |

---

## 🔷 Tabela: `business.d_ranking` — Dimensão Ranking

| Campo             | Tipo   | Descrição                              | Exemplo                |
|-------------------|--------|----------------------------------------|------------------------|
| `ranking_id`      | STRING | Chave única composta (player+data)     | "101501_2021-06-28"    |
| `player_id`       | STRING | ID do jogador                          | "101501"               |
| `date`            | DATE   | Data do ranking                        | "2021-06-28"           |
| `rank`            | INT    | Posição no ranking                     | 1, 25, 105             |
| `rank_points`     | INT    | Pontos de ranking                      | 12000, 850, 55         |

---

##  🔷 Tabela: `business.f_matches` — Fato Partidas

| Campo                | Tipo     | Descrição                                 |
|----------------------|----------|-------------------------------------------|
| `match_id`           | STRING   | Identificador único da partida            |
| `tourney_id`         | STRING   | FK → `d_tournaments.tourney_id`           |
| `tourney_date`       | DATE     | FK → `d_calendario.date_id`               |
| `winner_id`          | STRING   | FK → `d_players.player_id` (vencedor)     |
| `loser_id`           | STRING   | FK → `d_players.player_id` (perdedor)     |
| `best_of`            | INT      | Número máximo de sets (3 ou 5)            |
| `round`              | STRING   | Fase do torneio (e.g. "QF", "SF", "F")     |
| `score`              | STRING   | Resultado textual do jogo                 |
| `minutes`            | INT      | Duração da partida (minutos)              |

###  Estatísticas do vencedor (`w_`)

| Campo       | Tipo | Descrição                       |
|-------------|------|---------------------------------|
| `w_ace`     | INT  | Aces                            |
| `w_df`      | INT  | Duplas faltas                   |
| `w_svpt`    | INT  | Pontos de saque                 |
| `w_1stIn`   | INT  | Primeiro saque dentro           |
| `w_1stWon`  | INT  | Pontos vencidos no 1º saque     |
| `w_2ndWon`  | INT  | Pontos vencidos no 2º saque     |
| `w_SvGms`   | INT  | Games de saque                  |
| `w_bpSaved` | INT  | Break points salvos             |
| `w_bpFaced` | INT  | Break points enfrentados        |

###  Estatísticas do perdedor (`l_`)

| Campo       | Tipo | Descrição                       |
|-------------|------|---------------------------------|
| `l_ace`     | INT  | Aces                            |
| `l_df`      | INT  | Duplas faltas                   |
| `l_svpt`    | INT  | Pontos de saque                 |
| `l_1stIn`   | INT  | Primeiro saque dentro           |
| `l_1stWon`  | INT  | Pontos vencidos no 1º saque     |
| `l_2ndWon`  | INT  | Pontos vencidos no 2º saque     |
| `l_SvGms`   | INT  | Games de saque                  |
| `l_bpSaved` | INT  | Break points salvos             |
| `l_bpFaced` | INT  | Break points enfrentados        |

###  Rankings

| Campo                 | Tipo | Descrição                             |
|-----------------------|------|---------------------------------------|
| `winner_rank`         | INT  | Ranking do vencedor                   |
| `winner_rank_points`  | INT  | Pontos de ranking do vencedor         |
| `loser_rank`          | INT  | Ranking do perdedor                   |
| `loser_rank_points`   | INT  | Pontos de ranking do perdedor         |
