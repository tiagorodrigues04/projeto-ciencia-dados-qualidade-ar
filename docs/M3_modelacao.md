# Milestone 3: Modelação e Avaliação

## 1. Estratégia de Modelação

Nesta fase o objetivo era construir e comparar modelos capazes de prever a concentração de monóxido de carbono (`CO(GT)`). Antes de treinar qualquer modelo, tivemos de definir como íamos dividir os dados e como íamos avaliar os resultados.

* Divisão do dataset:  
Dividimos os dados em 80% para treino e 20% para teste.  
Como o dataset tem uma componente temporal, são medições horárias ao longo do tempo, fizemos a divisão de forma cronológica, ou seja, o modelo é treinado com os dados mais antigos e testado com os mais recentes. Achámos que fazia mais sentido do que uma divisão aleatória, porque é assim que funcionaria na realidade.

Para além disso, usámos validação cruzada com `TimeSeriesSplit`, o que nos permitiu testar o modelo em diferentes períodos temporais e ter mais confiança nos resultados, em vez de depender de uma única divisão que poderia ser favorável por acaso.

Definimos também um pipeline de pré-processamento para garantir que as transformações eram sempre feitas da mesma forma e apenas com base nos dados de treino. Esse pipeline inclui:

- imputação de valores em falta com a mediana;
- tratamento de outliers com o método IQR;
- codificação de variáveis categóricas com One-Hot Encoding;
- escalonamento das variáveis numéricas com StandardScaler.

Todas estas transformações foram ajustadas exclusivamente com base no conjunto de treino e posteriormente aplicadas ao conjunto de teste, assegurando a integridade metodológica do processo e a correspondência entre o notebook `2.0_modelacao_treino.ipynb` e o presente relatório.

* Métricas utilizadas:  
Para avaliar os modelos usámos principalmente o RMSE (Root Mean Squared Error), porque penaliza mais os erros grandes e neste tipo de problema (poluição atmosférica), erros mais elevados podem corresponder a previsões incorretas em situações de maior concentração de poluição atmosférica. Assim, esta métrica mede melhor o sucesso do nosso objetivo definido inicialmente

Como métricas complementares usámos:

- MAE (Mean Absolute Error) — diz-nos diretamente qual é o erro médio nas mesmas unidades do CO;
- R² (Coeficiente de determinação) — indica quanto da variabilidade do CO o modelo consegue explicar.

---

## 2. Experiências Realizadas

### 2.1. Modelo Baseline

O ponto de partida foi um modelo de referência muito simples, o `DummyRegressor`, que que prevê sempre o valor médio do CO, sem usar nenhuma informação das variáveis preditoras. Serviu para perceber qual o nível mínimo que qualquer modelo decente teria de superar.

* Resultado:  
- RMSE: 1.3576  
- MAE: 1.0831  
- R²: -0.0196

O R² negativo confirma que este modelo não tem qualquer capacidade preditiva, basicamente até é pior do que prever a média.

### 2.2. Modelos Candidatos

Quando vimos os resultados dos três modelos ficámos bastante satisfeitos porque todos eram claramente melhores do que o modelo baseline.

Testámos três algoritmos diferentes:

- Regressão linear, como modelo mais simples e fácil de interpretar;
- Random Forest, um modelo de ensemble que combina várias árvores de decisão;
- Gradient Boosting, outro modelo de ensemble, mas que constrói as árvores de forma sequencial.

| Algoritmo | Parâmetros Base | Métrica (Treino) | Métrica (Teste) | Notas |
| :--- | :--- | :--- | :--- | :--- |
| Regressão Linear | `default` | RMSE = 0.4968 | RMSE = 0.7805 | Indícios de underfitting |
| Random Forest | `n_estimators=100` | RMSE = 0.1345 | RMSE = 0.5472 | Overfitting |
| Gradient Boosting | `default` | RMSE = 0.3366 | RMSE = 0.5183 | Melhor equilíbrio |

O Random Forest surpreendeu-nos logo com um erro muito baixo no treino, pensávamos que ia ter um erro maior. No início não percebemos muito bem o porquê daquele resultado, mas depois de alguma pesquisa percebemos que se tratava de overfitting, este modelo tinha praticamente "decorado" os dados de treino mas não conseguia generalizar para dados novos, daí o erro ser bem maior no teste.

A Regressão Linear apresentou resultados mais modestos em ambos os conjuntos, o que sugere que a relação entre as variáveis e o CO não é puramente linear.

O Gradient Boosting foi o que nos pareceu a melhor escolha, tinha o melhor equilíbrio entre o treino e o teste e os números diziam precisamente isso. Tivemos alguma dúvida inicial porque o Random Forest parecia melhor à primeira vista, mas depois de perceber o problema do overfitting ficou claro que a melhor escolha era o Gradient Boosting.

---

## 3. Otimização (Tuning)

Depois de escolher o Gradient Boosting tentámos melhorá-lo ainda mais através da otimização dos seus hiperparâmetros.

* Técnica Utilizada:  
Usámos o `GridSearchCV` com `TimeSeriesSplit` (`n_splits=5`). Esta foi a parte mais difícil de implementar, havia muitos parâmetros para testar e analisar, e demorou algum tempo a correr. Mas o resultado que nos deu compensou, porque melhorou o desempenho do modelo.

Testámos combinações dos seguintes parâmetros:

- `n_estimators`;
- `learning_rate`;
- `max_depth`.

* Configuração ideal encontrada:  
- `n_estimators`: 100  
- `learning_rate`: 0.05  
- `max_depth`: 3

* Resultado na validação cruzada:  

- RMSE médio na validação cruzada: 0.5445
- Desvio padrão do RMSE: 0.2467

* Melhoria obtida no conjunto de teste:  
Comparando o modelo base com o modelo otimizado, observou-se a seguinte evolução no conjunto de teste:

- RMSE: de 0.5183 para 0.4879
- MAE: de 0.3510 para 0.3276
- R²: de 0.8514 para 0.8683

A melhoria não foi muito acentuada mas foi consistente nas três métricas, o que reforçou a nossa confiança na escolha do modelo.

---

## 4. Avaliação do Modelo Final

### 4.1. Análise de erros e resíduos

Como se trata de um problema de regressão, não existe matriz de confusão, em vez disso analisámos os resíduos (diferença entre os valores reais e os previstos).

O gráfico de valores reais versus previstos confirmou que o modelo segue bem a tendência geral, mas perde alguma precisão nos valores mais altos.

<p align="center"><img src="../reports/figures/real_vs_pred_gradient_boosting.png" width="600"/></p>

Gerámos também curvas de aprendizagem para o Gradient Boosting. Mostraram que o erro de treino fica sempre abaixo do erro de validação, sem convergirem de forma clara, o que sugere que o modelo ainda poderia beneficiar de mais dados ou de ajustes adicionais.

<p align="center"><img src="../reports/figures/learning_curve_gradient_boosting.png" width="600"/></p>

### 4.2. Importância dos Atributos (Feature Importance)

Analisámos quais as variáveis que mais contribuíram para as previsões do modelo. Os resultados foram coerentes com o que já tínhamos visto na análise exploratória, os sensores químicos e os poluentes são os mais importantes.

<p align="center"><img src="../reports/figures/feature_importance_gradient_boosting.png" width="600"/></p>

<p align="center"><img src="../reports/figures/residual_plot_gradient_boosting.png" width="600"/></p>

As cinco variáveis com maior importância foram:

1. `C6H6(GT)`
2. `PT08.S1(CO)`
3. `NOx`
4. `NO2(GT)`
5. `hour`

Variáveis como `month_12` e `PT08.S3(NOx)` também contribuíram, embora com menor peso, confirmando que a estrutura temporal dos dados também tem alguma influência nas previsões. 

---

## 5. Conclusão da Fase de Modelação

Esta foi provavelmente a fase em que tivemos mais interesse em todo o projeto, foi aqui que conseguimos ver os resultados a funcionar e a fazer sentido. Também foi a que exigiu mais pesquisa, nomeadamente para perceber conceitos como overfitting, validação cruzada temporal e otimização de hiperparâmetros.

O modelo final escolhido foi o Gradient Boosting otimizado , com base em três razões:

- Desempenho: melhores resultados no conjunto de teste, RMSE = 0.4879, MAE = 0.3276 e R² = 0.8683;
- Estabilidade: melhor equilíbrio entre treino e teste, sem os problemas de overfitting do Random Forest;
- Interpretabilidade: permite analisar os erros, os resíduos e a importância das variáveis, o que nos dá mais confiança nos resultados

Em comparação com o baseline, o modelo final reduziu o RMSE em 64.1% (de 1.3576 para 0.4879) e o MAE em 69.8% (de 1.0831 para 0.3276), superando amplamente o objetivo SMART de 15% definido no início do projeto.
Ficaram algumas limitações, nomeadamente na previsão de valores extremos de CO, mas no geral consideramos que o modelo está sólido e pronto para ser apresentado como solução final desta fase.

---

*Data de última atualização: 16/05/2026*
