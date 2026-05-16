# Milestone 2 — Análise Exploratória e Engenharia de Atributos

> Este documento pressupõe que o dataset já foi identificado e descrito no ficheiro `docs/M1_iniciacao.md`, incluindo o dicionário das 17 variáveis originais e a definição da variável-alvo `CO(GT)`. Caso seja necessário consultar o significado original das variáveis, deve consultar-se essa Milestone.

---

## 1. Análise Exploratória de Dados (EDA)

### 1.1. Análise Univariada — Distribuição das Variáveis

O dataset cobre o período de março de 2004 a fevereiro de 2005, com registos horários obtidos numa zona urbana de uma cidade italiana. A análise univariada permite compreender o comportamento individual de cada variável: a sua centralidade, dispersão, forma da distribuição e presença de valores extremos. Para cada variável numérica foram calculadas as principais estatísticas descritivas e gerados histogramas e boxplots.

#### Variável-alvo: `CO(GT)`

`CO(GT)` é a variável que o modelo pretende prever. Representa a concentração real de CO, em mg/m³, medida por analisador de referência certificado.

O histograma e o boxplot revelaram que a distribuição é assimétrica à direita: a maioria dos valores está concentrada entre 0 e 4 mg/m³, mas existem valores extremos que chegam a ultrapassar os 10 mg/m³. A mediana é inferior à média, o que confirma a assimetria positiva. Estes valores extremos não devem ser removidos pois correspondem a episódios reais de poluição elevada.

*(Figura 1 — Histograma e boxplot da variável-alvo `CO(GT)`, reports/figures/)*

#### Estatísticas Descritivas das Variáveis Numéricas

Após a substituição dos valores `-200` por `NaN` e antes de qualquer imputação, obtiveram-se as seguintes estatísticas descritivas para as variáveis numéricas:

| Variável | Tipo Estatístico | Mín. | Máx. | Média | Mediana | Desvio Padrão | Assimetria |
|----------|-----------------|------|------|-------|---------|---------------|------------|
| `CO(GT)` | Contínua | 0.1 | 11.9 | 2.15 | 1.80 | 1.44 | Positiva |
| `PT08.S1(CO)` | Contínua | 647 | 2040 | 1099 | 1053 | 217 | Positiva |
| `C6H6(GT)` | Contínua | 0.1 | 63.7 | 10.08 | 8.25 | 7.44 | Positiva |
| `PT08.S2(NMHC)` | Contínua | 383 | 2214 | 939 | 909 | 263 | Positiva |
| `NOx(GT)` | Contínua | 2.0 | 1479 | 246 | 141 | 240 | Positiva |
| `PT08.S3(NOx)` | Contínua | 322 | 2683 | 835 | 794 | 257 | Positiva |
| `NO2(GT)` | Contínua | 2.0 | 340 | 113 | 114 | 48 | Ligeira |
| `PT08.S4(NO2)` | Contínua | 551 | 2775 | 1457 | 1446 | 345 | Ligeira |
| `PT08.S5(O3)` | Contínua | 221 | 2523 | 1022 | 963 | 398 | Positiva |
| `T` | Contínua | -1.9 | 44.6 | 18.3 | 17.8 | 8.8 | Ligeira |
| `RH` | Contínua | 8.8 | 100.0 | 49.2 | 48.7 | 17.3 | Ligeira |
| `AH` | Contínua | 0.19 | 2.23 | 1.02 | 0.99 | 0.40 | Positiva |

> Variável-alvo do projeto.

O que mais nos chamou a atenção nesta análise foi perceber o quanto as escalas das variáveis de sensores são diferentes entre si — o `PT08.S1(CO)` vai de 647 a 2040 enquanto a `AH` vai de 0.19 a 2.23. À primeira vista não tínhamos bem a noção de que isso ia ser um problema, mas depois de ver a tabela percebemos logo que era necessário normalizar antes de avançar para os modelos. O `NOx(GT)` também nos surpreendeu: o desvio padrão é quase igual à média, o que indica que há momentos de poluição muito intensa misturados com períodos muito mais calmos — algo que nos pareceu importante ter em conta.

---

### 1.2. Análise Multivariada — Correlações

Para perceber quais as variáveis com maior relação linear com `CO(GT)`, foi gerada uma matriz de correlação de Pearson e gráficos de dispersão entre a variável-alvo e as restantes variáveis.

*(Figura 2 — Matriz de correlação (heatmap) das variáveis numéricas, reports/figures/)*

#### Correlações com `CO(GT)` (coeficiente de Pearson r)

| Variável | Correlação com `CO(GT)` | Interpretação |
|----------|------------------------|---------------|
| `C6H6(GT)` | +0.93 | Muito forte positiva |
| `PT08.S2(NMHC)` | +0.92 | Muito forte positiva |
| `NMHC(GT)` | +0.89 | Forte positiva |
| `PT08.S1(CO)` | +0.88 | Forte positiva |
| `PT08.S5(O3)` | +0.85 | Forte positiva |
| `NOx(GT)` | +0.80 | Forte positiva |
| `NO2(GT)` | +0.68 | Moderada positiva |
| `PT08.S4(NO2)` | +0.63 | Moderada positiva |
| `PT08.S3(NOx)` | −0.70 | Forte negativa |
| `T` | +0.02 | Muito fraca |
| `RH` | +0.05 | Muito fraca |
| `AH` | +0.05 | Muito fraca |

Foram também identificadas correlações muito elevadas entre variáveis preditoras, que podem causar instabilidade nos modelos lineares:
- `C6H6(GT)` ↔ `PT08.S2(NMHC)`: r = 0.98
- `PT08.S1(CO)` ↔ `PT08.S5(O3)`: r = 0.90

Estas correlações foram consideradas na fase de seleção de atributos (secção 3.2).

O que mais nos surpreendeu nesta análise foi a correlação negativa do `PT08.S3(NOx)` com o CO — nunca tínhamos pensado nisso mas depois de pesquisar percebemos que faz sentido do ponto de vista da química atmosférica. Também ficámos satisfeitos ao ver as correlações altas dos sensores químicos com a variável-alvo, porque nos deu logo uma indicação de que o modelo ia ter informação suficiente para aprender. Por outro lado, as variáveis meteorológicas terem correlações tão baixas foi algo que nos surpreendeu um pouco — intuitivamente esperávamos que a temperatura ou a humidade tivessem mais influência direta.

---

## 2. Qualidade dos Dados e Limpeza

Esta foi a fase que demorou mais tempo em todo o projeto, mas também aquela que nos deu a perceber mais sobre os dados. Existiam muitas decisões a tomar que podiam definir o rumo do trabalho — não podíamos fazê-las sem perceber realmente o porquê de cada uma, porque cada decisão poderia ter impacto no resultado final.

Na fase de iniciação (M1), já tínhamos identificado antecipadamente alguns problemas de qualidade — nomeadamente o código `-200` para valores em falta, as colunas vazias e a elevada percentagem de nulos em `NMHC(GT)`. A M2 confirmou e quantificou esses problemas.

### 2.1. Diagnóstico de Qualidade — Evolução do Dataset

| Etapa | Linhas | Colunas | Observação |
|-------|--------|---------|------------|
| Dataset original | 9471 | 17 | Inclui 2 colunas completamente vazias |
| Após remoção de colunas vazias e linhas em branco | 9357 | 15 | Limpeza estrutural inicial |
| Após substituição de `-200` por `NaN` | 9357 | 15 | Valores em falta agora identificáveis |
| Após remoção de `NMHC(GT)` | 9357 | 14 | Variável com percentagem excessiva de nulos |
| Após remoção de linhas com `CO(GT)` = NaN | ~8991 | 14 | Sem imputação da variável-alvo |
| Dataset final (após toda a limpeza) | ~8991 | 14 | Sem valores em falta |

### 2.2. Tratamento de Valores em Falta

| Variável | % de Nulos | Decisão Tomada |
|----------|------------|----------------|
| `NMHC(GT)` | ~90% | Removida — percentagem inviável para imputação |
| `CO(GT)` | ~3.9% | Linhas removidas — é a variável-alvo, não deve ser imputada |
| Restantes variáveis | ~3.9% | Preenchidas com a mediana de cada variável |

A mediana foi preferida à média pela sua robustez face à presença de outliers e à assimetria das distribuições.

### 2.3. Registos Duplicados

Verificámos se existiam linhas completamente duplicadas no dataset, porque era algo que podia afetar os modelos sem darmos conta. Não encontrámos nenhum registo duplicado, o que faz sentido já que os dados são registos horários com um timestamp único para cada linha.

### 2.4. Outliers e Inconsistências

Para identificar os outliers, foi utilizado o método do intervalo interquartil (IQR). Nas variáveis preditoras, foi aplicado clipping para limitar os valores ao intervalo `[Q1 − 1.5×IQR, Q3 + 1.5×IQR]`. A variável-alvo `CO(GT)` foi mantida sem alterações — os valores extremos de CO podem corresponder a episódios reais de poluição elevada e não devem ser removidos.

*(Figura 3 — Análise de outliers por variável preditora, reports/figures/)*

### 2.5. Sumário da Limpeza

| Operação | Resultado |
|----------|-----------|
| Remoção de colunas vazias | 2 colunas removidas |
| Substituição de -200 por NaN | Aplicada a todo o dataset |
| Remoção de `NMHC(GT)` | 1 coluna removida (~90% de nulos) |
| Remoção de linhas com `CO(GT)` nulo | ~366 linhas removidas |
| Imputação com mediana | Aplicada às restantes 12 variáveis numéricas |
| Registos duplicados | Nenhum encontrado |
| Clipping de outliers (IQR) | Aplicado às variáveis preditoras |

No final desta etapa ficámos bastante satisfeitos porque conseguimos resolver todos os problemas que tínhamos identificado na M1. O dataset ficou sem valores em falta, sem duplicados e com os outliers das variáveis preditoras controlados. A decisão de não tocar na variável-alvo foi consciente — alterar o `CO(GT)` estaria a mudar aquilo que queremos prever, o que não faria sentido nenhum.

---

## 3. Engenharia de Atributos

### 3.1. Transformações Realizadas

Depois de limpar os dados, trabalhámos na criação e transformação de variáveis para tornar o dataset mais útil para a modelação.

- Foi criada a variável `timestamp` a partir da junção das colunas `Date` e `Time`.
- Foram extraídas variáveis temporais a partir do `timestamp`.
- Foi aplicado One-Hot Encoding às variáveis categóricas `day_of_week` e `month`.
- A variável-alvo foi mantida na escala original.

O escalonamento das variáveis numéricas com StandardScaler foi feito apenas na fase de modelação, dentro de um pipeline, para garantir que o scaler é ajustado só com os dados de treino e não "vê" os dados de teste antes do tempo — evitando data leakage.

### 3.2. Criação de Novos Atributos

Foram criadas variáveis novas com potencial relevância para a modelação:

- `hour`: representa a hora do dia — a poluição varia muito ao longo do dia, com picos nas horas de ponta.
- `day_of_week`: identifica o dia da semana — padrões de tráfego e atividade industrial diferem entre dias.
- `month`: identifica o mês do ano — para captar possíveis padrões sazonais.
- `is_weekend`: indicador binário de fim de semana — aos fins de semana há menos tráfego, o que pode influenciar os níveis de CO.
- `is_warm_season`: indicador binário de estação quente (junho–setembro) — a temperatura e a radiação solar influenciam a dispersão de poluentes na atmosfera.
- `sensor_mean`: média das leituras dos sensores químicos como indicador agregado do comportamento geral dos sensores.

Na fase de seleção de atributos, `PT08.S2(NMHC)` e `sensor_mean` foram removidas por correlação absoluta superior a 0.90 com outras variáveis, causando multicolinearidade severa.

*(Figura 4 — Impacto das novas variáveis temporais nas concentrações de CO, reports/figures/)*

Criar estas variáveis foi das partes que mais gostámos nesta fase porque sentimos que estávamos mesmo a acrescentar algo ao dataset e não apenas a limpar. Percebemos que havia padrões temporais que os modelos não conseguiriam captar sem esta informação, como o facto de a poluição ser diferente às 7h da manhã e às 14h. A parte da seleção foi mais difícil porque tivemos de perceber o conceito de multicolinearidade e o que é que isso causaria nos modelos — mas depois de pesquisar ficou mais claro.

---

## 4. Dicionário de Dados Final (Pós-Processamento)

Após toda a limpeza, transformação e seleção de atributos, o dataset final entregue à fase de modelação contém as seguintes variáveis:

| Atributo | Tipo Python | Tipo Estatístico | Intervalo de Valores | Descrição |
|----------|-------------|-----------------|----------------------|-----------|
| `CO(GT)` | `float64` | Contínua | [0.1, 11.9] mg/m³ | Variável-alvo. Concentração de CO por analisador de referência |
| `PT08.S1(CO)` | `float64` | Contínua | [647, 2040] | Leitura do sensor SnO₂ sensível ao CO |
| `C6H6(GT)` | `float64` | Contínua | [0.1, 63.7] µg/m³ | Concentração de benzeno (referência) |
| `NOx(GT)` | `float64` | Contínua | [2.0, 1479] ppb | Concentração de NOx (referência) |
| `PT08.S3(NOx)` | `float64` | Contínua | [322, 2683] | Leitura do sensor WO₃ sensível a NOx |
| `NO2(GT)` | `float64` | Contínua | [2.0, 340] µg/m³ | Concentração de NO₂ (referência) |
| `PT08.S4(NO2)` | `float64` | Contínua | [551, 2775] | Leitura do sensor WO₂ sensível a NO₂ |
| `PT08.S5(O3)` | `float64` | Contínua | [221, 2523] | Leitura do sensor In₂O₃ sensível ao ozono |
| `T` | `float64` | Contínua | [−1.9, 44.6] °C | Temperatura do ar |
| `RH` | `float64` | Contínua | [8.8, 100.0] % | Humidade relativa |
| `AH` | `float64` | Contínua | [0.19, 2.23] g/m³ | Humidade absoluta |
| `hour` | `float64` | Discreta cíclica | [0, 23] | Hora do dia extraída do timestamp |
| `is_weekend` | `int64` | Binária | {0, 1} | 1 se fim de semana, 0 caso contrário |
| `is_warm_season` | `int64` | Binária | {0, 1} | 1 se meses de verão (junho–setembro), 0 caso contrário |
| `day_of_week_*` | `bool` | Binária (OHE) | {True, False} | Encoding do dia da semana (6 colunas) |
| `month_*` | `bool` | Binária (OHE) | {True, False} | Encoding do mês (11 colunas) |

> OHE = One-Hot Encoding.

Variáveis removidas:

| Variável | Motivo de Remoção |
|----------|------------------|
| `Unnamed: 15`, `Unnamed: 16` | Colunas completamente vazias |
| `NMHC(GT)` | ~90% de valores em falta — inviável para imputação |
| `Date`, `Time`, `timestamp` | Substituídas por variáveis temporais derivadas |
| `PT08.S2(NMHC)` | Correlação r = 0.98 com `C6H6(GT)` — multicolinearidade severa |
| `sensor_mean` | Correlação r > 0.90 com múltiplos sensores — multicolinearidade |

---

## 5. Conclusões da Fase de Exploração

Esta foi sem dúvida a fase que demorou mais tempo, mas também a que nos ensinou mais sobre os dados. Só depois de limpar e explorar tudo é que percebemos realmente o que o dataset continha e o que poderíamos fazer com ele nas fases seguintes.

As principais conclusões foram:

- O dataset cobre um ano completo (março 2004 – fevereiro 2005) com registos horários, o que introduz padrões temporais relevantes e sazonalidade.
- A variável-alvo tem distribuição assimétrica à direita, com a maior parte dos valores entre 0 e 4 mg/m³.
- Os sensores químicos e os poluentes são as variáveis que mais explicam o CO, com correlações até r = 0.93.
- As variáveis meteorológicas têm correlação muito fraca com `CO(GT)` de forma isolada.
- Não existem registos duplicados no dataset.
- Os problemas de qualidade dos dados foram todos resolvidos e o dataset final ficou sem valores em falta.
- A criação de novas variáveis temporais enriqueceu a informação disponível para os modelos.
- A seleção de atributos reduziu a multicolinearidade, removendo `PT08.S2(NMHC)` e `sensor_mean`.

No final desta fase, o dataset estava limpo, organizado e pronto para a fase de modelação.

---

*Data de última atualização: 15/05/2026*
