# MVP de Engenharia de Dados - PUC-Rio

# Objetivo

Este trabalho tem como objetivo aplicar práticas de engenharia de dados aprendidas no curso e constuir um pipeline completo de dados utilizando uma tecnologia de nuvem.
O pipeline irá envolver a busca, coleta, modelagem, carga e análise dos dados. 

Foram selecionados dados históricos de partidas de tênis profissional (ATP), um assunto que me interessa, e a partir disso foram elaboradas questões sobre as conquistas dos jogadores. 

Perguntas:
1. Quantos jogadores já ganharam os 4 grand slams? ("Career Grand Slam")
2. Quantos já conseguiram os 4 grand slams no mesmo ano e qual foi o ultimo ano que isso ocorreu?
3. Que jogador permaneceu mais tempo no top 1? E no top 5?
4. Que caracteristicas em comum tem essses jogadores?



# Coleta

## Coleta do dataset

A coleta dos dados utilizados neste trabalho é realizada a partir do **repositório do Kaggle**, por meio da API oficial da plataforma. Para garantir uma autenticação segura e automatizada dentro do ambiente do **Databricks**, foi necessário criar um processo específico para carregar as variáveis de ambiente contendo as credenciais da API do Kaggle.

Como o Databricks não consegue acessar diretamente os arquivos `.env` armazenados no DBFS de forma nativa, foi implementada uma função personalizada (`load_env_from_dbfs`) para ler e interpretar o conteúdo do arquivo `.env` como texto. Essa abordagem se mostrou eficaz para popular as variáveis de ambiente dentro da sessão ativa do notebook.

Após a configuração do ambiente, o dataset **"tennis"** foi baixado do Kaggle e descompactado no diretório temporário `/tmp`. O arquivo principal, um banco de dados no formato **SQLite**, foi então carregado e inspecionado. As tabelas disponíveis — `matches`, `players` e `rankings` — foram extraídas utilizando consultas SQL diretas, convertidas para DataFrames do Pandas e posteriormente carregadas para o DBFS utilizando spark.



# Modelagem

### Modelo Estrela

### Tabela Fato
- **f_Matches**: contém o resultado e estatísticas das partidas
  - FK: `tourney_id`, `winner_id`, `loser_id`, `tourney_date`
  - Métricas: aces, double faults, pontos de saque, tempo, score, ranking etc.

### Tabelas Dimensão
- **d_Players**: dados dos jogadores (mão, altura, país, data de nascimento)
- **d_Tournaments**: informações dos torneios (superfície, nível, tamanho da chave)
- **d_Calendario**: tempo em granularidade de dia, mês, ano
- **d_Ranking**: posição e pontos de ranking por jogador por data (1 data por semana)




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




# Script: Recreate Metastore

## Objetivo

Este script tem como objetivo **recriar o metastore** do Spark/Hive com base na estrutura de diretórios existente no caminho do warehouse.  
Ele é útil em cenários onde o metastore foi perdido ou corrompido, como no caso da reinicialização do cluster, mas os dados físicos (arquivos Delta) ainda estão presentes no armazenamento DBFS.

## Funcionamento

O script percorre o diretório base (`/user/hive/warehouse/`), onde os databases e tabelas do Hive/Spark são armazenados fisicamente, e recria:

- **Databases**, a partir de pastas com sufixo `.db`
- **Tabelas Delta**, baseando-se nas subpastas dos databases

O Spark infere o schema automaticamente para recriar as tabelas no catálogo, assumindo que os dados estão em formato **Delta Lake**.

## Observações 
- O script assume que todas as tabelas são do tipo Delta Lake
- Utiliza dbutils.fs.ls, que é específico do ambiente Databricks
- Requer que os arquivos estejam íntegros e acessíveis no caminho do warehouse
