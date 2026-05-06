# Milestone 3: Modelação e Avaliação
 
## 1. Estratégia de Modelação
 
Nesta fase foi definida a metodologia de modelação e avaliação com o objetivo de prever a concentração de monóxido de carbono (`CO(GT)`), assegurando rigor técnico, reprodutibilidade e coerência com o problema de negócio identificado no Milestone 1.
 
* **Divisão do dataset:**  
Foi utilizada uma divisão de **80% para treino** e **20% para teste**.  
Dado que o dataset possui uma componente temporal, a partição foi realizada de forma **cronológica**, garantindo que o modelo é treinado com observações passadas e avaliado com observações futuras.
Esta opção foi considerada mais adequada do que uma divisão aleatória, uma vez que aproxima o processo de avaliação a um cenário real de previsão e reduz o risco de fugas de informação (*data leakage*).
 
Adicionalmente, foi utilizada validação cruzada com **`TimeSeriesSplit`** no conjunto de treino, permitindo avaliar o comportamento dos modelos em diferentes segmentos temporais e aumentar a robustez das conclusões. Desta forma, os resultados obtidos não dependem apenas de uma divisão ocasionalmente favorável dos dados, mas de um processo mais estável e repetível.
 
Antes do treino dos modelos foi ainda definido um pipeline de pré-processamento, com o objetivo de garantir consistência nas transformações aplicadas aos dados. Esse pipeline inclui:
 
- imputação de valores em falta com a mediana nas variáveis numéricas;
- tratamento de outliers com base no método do IQR;
- codificação de variáveis categóricas com One-Hot Encoding;
- escalonamento das variáveis numéricas com StandardScaler.
Todas estas transformações foram ajustadas exclusivamente com base no conjunto de treino e posteriormente aplicadas ao conjunto de teste, assegurando a integridade metodológica do processo e a correspondência entre o notebook `2.0_modelacao_treino.ipynb` e o presente relatório.
 
* **Métrica de Sucesso:**  
Tratando-se de um problema de regressão, foi escolhida como métrica principal o **RMSE (Root Mean Squared Error)**.
O RMSE foi privilegiado em detrimento do MAE por penalizar mais fortemente erros de maior magnitude, o que é particularmente relevante neste contexto, uma vez que erros mais elevados podem corresponder a previsões incorretas em situações de maior concentração de poluição atmosférica. Assim, esta métrica mede de forma mais adequada o sucesso do objetivo definido inicialmente: prever com rigor a concentração de `CO(GT)`.
 
Como métricas complementares foram utilizadas:
 
- **MAE (Mean Absolute Error)** — permite interpretar diretamente o erro médio em unidades da variável-alvo;
- **R² (Coeficiente de determinação)** — avalia a capacidade explicativa do modelo e a proporção da variabilidade dos dados capturada pelas previsões.
A utilização conjunta destas métricas permite avaliar simultaneamente precisão, robustez e qualidade global dos modelos.
 
---
 
## 2. Experiências Realizadas
 
### 2.1. Modelo Baseline
 
Como ponto de partida, foi utilizado um modelo simples de referência.
 
* **Algoritmo:** `DummyRegressor` (`strategy="mean"`)
Este modelo prevê sempre o valor médio da variável-alvo observada no conjunto de treino, não utilizando qualquer informação das variáveis preditoras.
 
* **Resultado:**  
- RMSE: **1.3576**  
- MAE: **1.0831**  
- R²: **-0.0196**
Este baseline define o nível mínimo de desempenho esperado e serve de referência obrigatória para avaliar se os modelos mais complexos acrescentam, de facto, valor preditivo.
 
### 2.2. Modelos Candidatos
 
Foram testados diferentes algoritmos de regressão com o objetivo de superar o desempenho do baseline.
 
A seleção dos modelos teve em conta a diversidade de abordagens e a sua adequação ao problema:
 
- **modelo linear** (*Regressão Linear*), como referência simples e interpretável;
- **modelos de ensemble** (*Random Forest* e *Gradient Boosting*), por serem capazes de captar relações não lineares e interações mais complexas entre variáveis.
| Algoritmo | Parâmetros Base | Métrica (Treino) | Métrica (Teste) | Notas |
| :--- | :--- | :--- | :--- | :--- |
| Regressão Linear | `default` | RMSE = 0.4968 | RMSE = 0.7649 | Indícios de underfitting |
| Random Forest | `n_estimators=100` | RMSE = 0.1345 | RMSE = 0.5387 | Evidência de overfitting |
| Gradient Boosting | `default` | RMSE = 0.3366 | RMSE = 0.5071 | Melhor capacidade de generalização |
 
Os resultados mostram que todos os modelos testados superam de forma clara o baseline, o que evidencia a existência de padrões preditivos relevantes nos dados.
 
Os modelos baseados em ensemble apresentaram desempenho superior ao modelo linear, sugerindo que a relação entre os atributos e a variável-alvo não é puramente linear.
 
O modelo **Gradient Boosting** destacou-se como a melhor solução nesta fase inicial, ao apresentar:
- o menor erro no conjunto de teste;
- o melhor equilíbrio entre desempenho de treino e desempenho de teste;
- uma capacidade de generalização superior à dos restantes candidatos.
O modelo **Random Forest** apresentou erro muito reduzido no treino, mas uma degradação visível no teste, o que constitui evidência de **overfitting**.
 
Já a **Regressão Linear** apresentou resultados significativamente mais modestos, sugerindo **underfitting parcial**, isto é, uma capacidade limitada para captar a complexidade do fenómeno em análise.
 
Comparativamente ao baseline, a maior complexidade dos modelos candidatos trouxe benefícios estatísticos claros, justificando o seu custo computacional adicional.
 
---
 
## 3. Otimização (Tuning)
 
Após a comparação inicial entre os modelos candidatos, foi selecionado o **Gradient Boosting** para a fase de otimização, por ter apresentado o melhor desempenho no conjunto de teste e a melhor capacidade de generalização.
 
* **Técnica Utilizada:**  
Foi utilizado `GridSearchCV` com validação cruzada baseada em **`TimeSeriesSplit`** (`n_splits=5`), de forma a respeitar a natureza temporal do problema e a garantir consistência metodológica com a estratégia definida anteriormente.
O espaço de pesquisa incluiu os hiperparâmetros mais influentes do modelo:
 
- `n_estimators`;
- `learning_rate`;
- `max_depth`.
* **Configuração ideal encontrada:**  
- `n_estimators`: **100**  
- `learning_rate`: **0.05**  
- `max_depth`: **3**
* **Validação Cruzada:**  
O melhor modelo apresentou:
- **RMSE médio na validação cruzada:** **0.5445**
- **Desvio padrão do RMSE:** **0.2467**
Este resultado mostra um desempenho médio sólido, embora com alguma variabilidade entre diferentes divisões temporais, o que é compatível com a própria variabilidade do fenómeno em estudo.
 
* **Melhoria obtida:**  
Comparando o modelo base com o modelo otimizado, observou-se a seguinte evolução no conjunto de teste:
- RMSE: de **0.5071** para **0.4806**
- MAE: de **0.3438** para **0.3210**
- R²: de **0.8578** para **0.8722**
A otimização permitiu, assim, melhorar de forma consistente o desempenho do modelo nas três métricas consideradas. Embora a melhoria não seja muito acentuada, é estatisticamente coerente e reforça a escolha do **Gradient Boosting** como principal candidato a modelo final.
 
---
 
## 4. Avaliação do Modelo Final
 
### 4.1. Matriz de Confusão / Erros
 
Tratando-se de um problema de regressão, a avaliação do modelo final foi realizada com base na análise dos erros e resíduos (diferença entre valores reais e previstos), bem como na comparação entre desempenho de treino e desempenho de teste.
 
> **Análise:**  
A auditoria ao desempenho do modelo revelou que os maiores erros se concentram em observações onde a variável `CO(GT)` assume valores mais extremos, indicando maior dificuldade em prever situações de elevada variabilidade na qualidade do ar.
 
O gráfico de resíduos mostrou uma distribuição globalmente centrada em torno de zero, o que sugere ausência de enviesamento sistemático muito acentuado. No entanto, observa-se maior dispersão dos resíduos à medida que os valores previstos aumentam, o que indica uma redução da fiabilidade do modelo em observações de maior magnitude.
 
A comparação entre treino e teste permitiu concluir que:
 
- o **Random Forest** apresenta sinais claros de sobreajuste, com erro muito reduzido no treino e superior no teste;
- o **Gradient Boosting** apresenta o melhor equilíbrio entre treino e teste, evidenciando maior capacidade de generalização;
- a **Regressão Linear** apresenta desempenho mais fraco em ambos os conjuntos, o que sugere uma capacidade limitada para modelar a complexidade dos dados.
O gráfico de valores reais versus previstos confirmou esta análise, mostrando uma relação linear forte entre previsões e valores observados, mas também alguma subestimação e maior dispersão nos valores mais elevados de `CO(GT)`.
 
Foram igualmente geradas **curvas de aprendizagem** para o modelo Gradient Boosting. Essas curvas mostram que o erro de treino se mantém substancialmente abaixo do erro de validação e que a distância entre ambas não converge de forma clara. Este comportamento sugere que, apesar de o modelo apresentar bom desempenho global, continua a existir alguma sensibilidade à variabilidade temporal dos dados, o que reforça a utilidade de otimização adicional e, potencialmente, de mais dados ou de estratégias complementares de modelação.
 
### 4.2. Importância dos Atributos (Feature Importance)
 
Foi também realizada uma análise da **importância dos atributos** do modelo final, permitindo identificar quais as variáveis com maior contributo para a previsão da concentração de monóxido de carbono.
 
Os resultados obtidos são coerentes com a análise exploratória efetuada no Milestone 2, evidenciando o peso dominante de variáveis associadas a sensores químicos e poluentes atmosféricos.
 
As variáveis com maior importância no modelo final foram:
 
1. `C6H6(GT)`
2. `PT08.S1(CO)`
3. `NOx(GT)`
4. `NO2(GT)`
5. `PT08.S5(O3)`
Adicionalmente, surgem com importância secundária variáveis como `hour`, `PT08.S3(NOx)` e alguns indicadores temporais derivados do mês, o que sugere que a estrutura temporal dos dados também contribui, embora com menor peso, para a capacidade preditiva do modelo.
 
De forma geral, esta análise confirma que o modelo final baseia a sua capacidade de previsão sobretudo em variáveis diretamente relacionadas com sensores e poluentes, o que reforça a coerência técnica da solução desenvolvida.
 
---
 
## 5. Conclusão da Fase de Modelação
 
Os resultados obtidos demonstram que é possível prever a concentração de monóxido de carbono com um bom nível de precisão utilizando modelos de *machine learning*.
 
A comparação entre o modelo baseline, os modelos candidatos e a versão otimizada permitiu concluir que o **Gradient Boosting otimizado** constitui a melhor solução técnica desenvolvida nesta fase do projeto.
 
A escolha deste modelo foi sustentada em três critérios principais:
 
- **Desempenho:** apresentou os melhores resultados no conjunto de teste, com RMSE = **0.4806**, MAE = **0.3210** e R² = **0.8722**;
- **Estabilidade:** revelou melhor equilíbrio entre treino e teste do que o Random Forest, bem como resultados consistentes na validação cruzada;
- **Interpretabilidade:** embora não seja tão simples como a Regressão Linear, permite apoiar a análise com curvas de aprendizagem, gráfico de resíduos, valores reais versus previstos e importância dos atributos.
O modelo **Random Forest** apresentou sinais de sobreajuste, enquanto a **Regressão Linear**, apesar de mais simples e mais fácil de interpretar, revelou desempenho claramente inferior. Assim, o **Gradient Boosting otimizado** foi o modelo que melhor conciliou precisão, estabilidade e utilidade prática.
 
A comparação com o baseline evidenciou melhorias muito significativas, justificando plenamente a utilização de modelos mais complexos. Em termos concretos, o modelo final atingiu uma melhoria de **64.6% no RMSE** (de 1.3576 para 0.4806) e de **70.4% no MAE** (de 1.0831 para 0.3210) face ao baseline, superando amplamente o objetivo SMART definido na Milestone 1, que estabelecia uma melhoria mínima de **15%**.
 
Apesar dos bons resultados, mantêm-se algumas limitações na previsão de valores extremos da variável `CO(GT)`, o que sugere margem para melhoria futura através de refinamento adicional dos hiperparâmetros, análise mais aprofundada dos erros e eventual reforço dos dados disponíveis.
 
De forma global, considera-se que o **Gradient Boosting otimizado** se encontra suficientemente validado para ser apresentado como o modelo técnico final desta fase do projeto, mantendo plena coerência entre o relatório, o notebook `2.0_modelacao_treino.ipynb` e os artefactos visuais exportados para `reports/figures/`.
 
---
 
*Data de última atualização: 06/05/2026*
 
