# Trabalho PI – PCA e Clusterização de Pokémon Competitivos (VGC 2024)

Análise exploratória dos atributos base dos Pokémon mais usados no cenário competitivo de **VGC 2024**, comparando o meta online (**Smogon**) com o meta presencial (**Worlds**). O projeto usa **PCA** para identificar quais atributos mais diferenciam os Pokémon competitivos e **clusterização hierárquica** para agrupá-los em perfis táticos.

## Integrantes

| Nome | RA |
|---|---|
| Gabbriel Vicente Hiroshi Nagano | 24005804 |
| Gustavo de Paiva Beraldo | 23027668 |
| Nicole Siqueira Borges | 24013977 |
| Rodrigo Piaza Ribeiro | 24003397 |

## Dataset

[Pokémon Competitive Usage – Smogon and VGC Worlds](https://www.kaggle.com/datasets/danielsmdev/pokemon-competitive-usage-smogon-and-vcgworlds) (Kaggle), baixado automaticamente via `kagglehub`.

Principais colunas utilizadas:

- **Atributos base:** `hp`, `attack`, `defense`, `sp_atk`, `sp_def`, `speed`
- **Uso competitivo:** `Smogon_VGC_Usage_2022/2023/2024` e `Worlds_VGC_Usage_2022/2023/2024`
- **Metadados:** `name`, `type1`, `type2`, habilidades, `generation`, `legendary`, `mythical`

## Tecnologias

- **PySpark** – leitura, limpeza, `VectorAssembler`, `StandardScaler` e `PCA`
- **pandas / NumPy** – manipulação dos dados após conversão do Spark
- **scikit-learn** – padronização para a clusterização
- **SciPy** – clusterização hierárquica (`linkage` com método Ward, `fcluster`, `dendrogram`)
- **Matplotlib / Seaborn / Plotly** – dendrogramas, gráfico de barras e PCA em 3D

## Como executar

O notebook foi feito para o **Google Colab** (runtime com GPU T4, embora não seja obrigatória).

1. Abra `PCA_Pokemon.ipynb` no Colab.
2. Execute as células em ordem. A primeira instala as dependências:
   ```bash
   pip install -q pyspark findspark
   ```
3. O dataset é baixado com `kagglehub` e copiado para `/content/pokemon_competitive_analysis.csv`.

Para rodar localmente, instale também `kagglehub`, `plotly`, `seaborn`, `scipy` e `scikit-learn`, e ajuste o caminho do CSV (fora do Colab não existe `/content/`).

## Etapas da análise

### 1. Tratamento da base
- `ability2 = "No_ability"` → `"None"`.
- Colunas de uso com `"NoUsage"` → `0.0`, convertidas para `float`.
- Formas alternativas são separadas da base principal por regex no nome: Mega/Gigantamax, formas regionais (Alola, Galar, Hisui, Paldea), Pikachu/Eevee de *Let's Go*, skins do Pikachu e outras formas especiais. O resultado é o DataFrame `dfRaw`.
- São criados dois recortes: `df_smogonStatsColumns` e `df_worldsStatsColumns`.
- Listas de tipos (18), habilidades (290) e gerações (9) são extraídas e limpas.

### 2. PCA (`aplicar_pca`)
- Considera apenas Pokémon com uso > 0 no ano analisado (2024), para não enviesar com zeros.
- Padroniza os 6 atributos (média 0, desvio 1) e calcula o PCA completo.
- Escolhe o menor **K** cuja variância acumulada ≥ 75%.
- Para cada componente, aponta o atributo de maior peso (sem repetir atributos).

| Meta | K | Variância explicada | PC1 | PC2 | PC3 | PC4 |
|---|---|---|---|---|---|---|
| Smogon 2024 | 4 | 83,12% | Defense | Sp. Atk | Attack | HP |
| Worlds 2024 | 4 | 82,84% | Attack | Sp. Def | Speed | HP |

Interpretação dos autores: no Smogon, onde itens e habilidades do oponente não são visíveis, a defesa pesa mais; no Worlds, com mais informação sobre o adversário, ataque e velocidade ganham destaque.

### 3. Seleção do meta
Pokémon com uso **> 1%** em 2024:
- **Smogon:** 70 Pokémon
- **Worlds:** 51 Pokémon

### 4. Clusterização hierárquica
- Ward sobre os atributos padronizados.
- Corte do dendrograma em **metade da maior distância** de fusão.
- Resultado: **7 grupos no Smogon** e **5 grupos no Worlds**.

Cada grupo recebe um rótulo pela média dos atributos:

| Regra (média do grupo) | Categoria |
|---|---|
| `speed > 100` | Ofensivos Velozes (Sweepers) |
| `attack > 110` ou `sp_atk > 110` | Ofensivos Pesados (Wallbreakers) |
| `defense > 100` e `sp_def > 100` | Muralhas Defensivas (Tanks) |
| `hp > 90` | Suportes Robustos (Bulky Support) |
| demais casos | Versáteis / Equilibrados |

### 5. Visualizações
- Dendrogramas com a linha de corte (Smogon, Worlds e base completa com 449 Pokémon).
- Dispersão 3D (PC1 × PC2 × PC3) colorida por cluster, com Plotly.
- Gráfico de barras comparando a média de cada atributo entre o **Meta** (união Smogon + Worlds) e o restante da base.

### 6. Comparação com a clusterização global
A função `analisar_repeticao` mostra como cada grupo do meta se distribui entre os 9 grupos formados com todos os Pokémon, indicando se os perfis competitivos coincidem com os agrupamentos "naturais" da base.

## Principais conclusões

- O meta competitivo não é só estatisticamente superior: ele se organiza em perfis táticos bem definidos, com especializações extremas em vez de atributos medianos.
- **Smogon** apresenta distribuição mais equilibrada entre suportes e versáteis.
- **Worlds** concentra mais Wallbreakers e Sweepers, sugerindo um ambiente focado em pressão ofensiva.
- Quatro componentes explicam mais de 80% da variância nos dois metas, indicando que um Pokémon competitivo precisa se destacar em várias dimensões e se encaixar em um papel funcional claro.

## Estrutura

```
.
├── PCA_Pokemon.ipynb   # notebook com toda a análise
└── README.md
```
