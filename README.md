# Previsão da Concentração de Monóxido de Carbono com Dados de Qualidade do Ar

Licenciatura em Ciência de Dados para a Gestão — Coimbra Business School | ISCAC
Unidade Curricular: Projeto em Ciência de Dados — Docente: Dora Melo (dmelo@iscac.pt)
Ano Letivo: 2025/2026

---

## Identificação da Equipa

- Grupo n.º: 11
- Membros:
  - Rodrigo Pedro — a2023129724
  - Tiago Rodrigues — a2022138676

---

## Organização do Repositório

```
projeto-ciencia-dados-qualidade-ar/
├── data/
│   ├── raw/                          ← Dataset original (AirQualityUCI)
│   └── processed/                    ← Dataset pós-processamento
├── docs/                             ← Documentação técnica por Milestone
│   ├── M1_iniciacao.md
│   ├── M2_exploracao.md
│   ├── M3_modelacao.md
│   └── M4_conclusoes.md
├── notebooks/                        ← Versões exportadas do Kaggle
│   ├── 1_0_eda_limpeza.ipynb         ← Corresponde à Fase 2
│   ├── 2_0_modelacao_treino.ipynb    ← Corresponde à Fase 3
│   └── 3_0_interpretacao.ipynb       ← Corresponde à Fase 4
├── reports/
│   └── figures/                      ← Figuras geradas nos notebooks
├── src/                              ← Reservado para módulos auxiliares
├── requirements.txt                  ← Dependências do projeto
└── README.md
```

---

## 1. Iniciação (Milestone 1)

### Contexto e Problema de Negócio

A qualidade do ar é um tema relevante para a saúde pública, para a gestão ambiental e para o apoio à decisão em contexto urbano. A existência de mecanismos que permitam antecipar níveis mais elevados de poluição pode ajudar entidades públicas e sistemas de monitorização a agir de forma mais informada.

Neste projeto pretende-se estudar a previsão da concentração de monóxido de carbono em ambiente urbano, procurando avaliar se é possível estimar esse valor com base em informação recolhida em contexto de monitorização da qualidade do ar. O problema é relevante porque a antecipação de episódios de maior poluição pode apoiar ações de prevenção, vigilância e mitigação.

### Objetivo SMART

Desenvolver um modelo de regressão capaz de prever a concentração horária de monóxido de carbono `CO(GT)`, em mg/m³, com base em leituras de sensores químicos e variáveis meteorológicas do dataset AirQualityUCI, obtendo uma redução mínima de 15% no RMSE face a um modelo de referência simples, avaliado num conjunto de teste cronologicamente separado, até ao final da unidade curricular de Projeto em Ciência de Dados (maio de 2026).

### Fonte de Dados

- Dataset: [AirQualityUCI — UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/387/air+quality) — Saverio Vito, 2008
- DOI: 10.24432/C5060Z
- Dimensão: 9357 linhas · 15 colunas úteis (após limpeza estrutural inicial)

### Perguntas de Investigação

1. Existem padrões horários ou sazonais na concentração de `CO(GT)` ao longo do período analisado?
2. Quais os atributos com maior relação com `CO(GT)`, nomeadamente os sensores `PT08.*` e as variáveis meteorológicas?
3. É possível prever `CO(GT)` com erro claramente inferior ao de um modelo que preveja sempre a média?
4. A criação de novos atributos temporais melhora o desempenho preditivo dos modelos?

📄 Documento completo: [`docs/M1_iniciacao.md`](docs/M1_iniciacao.md)

---

## 2. Exploração (Milestone 2)

### Limpeza e Preparação

O ficheiro original tinha alguns problemas que foi preciso resolver antes de avançar para a modelação. Os valores em falta não estavam identificados como tal — estavam codificados como `-200`, o que obrigou a uma conversão manual antes de qualquer análise. Havia também duas colunas completamente vazias e uma variável (`NMHC(GT)`) com quase 90% de dados em falta, que foi removida por não ter informação suficiente para ser útil.

Para a variável que queremos prever (`CO(GT)`), optámos por eliminar as linhas onde não havia medição, porque não faria sentido tentar adivinhar o valor que o próprio modelo vai aprender a prever. Nas restantes variáveis, os valores em falta foram preenchidos com a mediana — um valor central mais robusto do que a média quando existem valores extremos.

Foram ainda criadas novas variáveis a partir da data e hora de cada registo — como a hora do dia, o dia da semana e o mês — porque a concentração de CO varia ao longo do tempo e os modelos precisam dessa informação para aprender esses padrões. Por fim, foram removidas algumas variáveis que tinham informação demasiado parecida com outras, para não confundir os modelos.

### Principais Conclusões da Análise Exploratória

- Os sensores químicos e os poluentes atmosféricos são os atributos com maior correlação com `CO(GT)`, com valores de Pearson r que chegam a 0.93 (`C6H6(GT)`).
- As variáveis meteorológicas `T`, `RH` e `AH` apresentam correlações muito fracas com a variável-alvo de forma isolada.
- A variável-alvo tem distribuição assimétrica à direita, com a maior parte dos valores entre 0 e 4 mg/m³.
- Não foram encontrados registos duplicados no dataset.

📄 Documento completo: [`docs/M2_exploracao.md`](docs/M2_exploracao.md)

---

## 3. Modelação (Milestone 3)

### Estratégia de Avaliação

- Divisão cronológica dos dados: 80% treino e 20% teste
- Validação cruzada com TimeSeriesSplit para respeitar a ordem temporal dos dados
- Pipeline de pré-processamento integrado para evitar data leakage
- Otimização do melhor modelo com GridSearchCV

### Resultados Comparativos

| Modelo | RMSE Teste | MAE Teste | R² Teste |
|--------|-----------|-----------|---------|
| Baseline (DummyRegressor) | 1.3576 | 1.0831 | -0.0196 |
| Regressão Linear | 0.7805 | 0.6354 | 0.6630 |
| Random Forest | 0.5472 | 0.3577 | 0.8343 |
| Gradient Boosting (base) | 0.5183 | 0.3510 | 0.8514 |
| Gradient Boosting (otimizado) | 0.4879 | 0.3276 | 0.8683 |

### Modelo Final

O modelo selecionado foi o Gradient Boosting otimizado, com os hiperparâmetros `n_estimators=100`, `learning_rate=0.05` e `max_depth=3`, obtidos por GridSearchCV com TimeSeriesSplit. O RMSE reduziu 64.1% face ao baseline, superando amplamente o objetivo definido de 15%.

### Principais Conclusões da Modelação

- O Gradient Boosting otimizado superou todos os candidatos em RMSE, MAE e R².
- O Random Forest apresentou sinais de sobreajuste — erro muito baixo no treino e claramente superior no teste.
- A Regressão Linear revelou desempenho inferior, sugerindo que as relações entre variáveis não são exclusivamente lineares.
- Os atributos com maior importância foram `PT08.S2(NMHC)`, `C6H6(GT)` e `PT08.S1(CO)`, coerentes com as correlações identificadas na EDA.

📄 Documento completo: [`docs/M3_modelacao.md`](docs/M3_modelacao.md)

---

## 4. Finalização (Milestone 4)

### Resposta ao Problema

Os resultados obtidos demonstram que é possível prever a concentração de monóxido de carbono com boa precisão a partir de leituras de sensores e variáveis ambientais disponíveis em tempo real. O modelo final obteve um R² de 0.8683 no conjunto de teste, explicando cerca de 87% da variância da variável-alvo.

### Recomendações de Inovação

1. Integrar o modelo num sistema de monitorização com alertas automáticos quando a previsão ultrapassar um limiar definido.
2. Explorar validação temporal mais avançada e atualização periódica do modelo, de forma a lidar com sazonalidade e possível deriva do conceito.
3. Criar variáveis adicionais como médias móveis ou diferenças entre leituras consecutivas para captar dinâmicas temporais de curto prazo.
4. Aplicar técnicas de interpretabilidade como SHAP para aprofundar a análise dos fatores que determinam episódios de poluição elevada.

📄 Documento completo: [`docs/M4_conclusoes.md`](docs/M4_conclusoes.md)

---

## Como Reproduzir este Projeto

```bash
# 1. Clonar o repositório
git clone https://github.com/a2022138676/projeto-ciencia-dados-qualidade-ar.git
cd projeto-ciencia-dados-qualidade-ar

# 2. Instalar dependências
pip install -r requirements.txt

# 3. Executar os notebooks por ordem
jupyter notebook notebooks/1_0_eda_limpeza.ipynb         # Fase 2 — EDA e pré-processamento
jupyter notebook notebooks/2_0_modelacao_treino.ipynb    # Fase 3 — Modelação e avaliação
jupyter notebook notebooks/3_0_interpretacao.ipynb       # Fase 4 — Interpretação e entrega de valor
```

> O dataset original está disponível no [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/387/air+quality) e no [Kaggle](https://www.kaggle.com/datasets/jhonan/airqualityuci). Deve ser colocado em `data/raw/` antes de executar os notebooks.

---

## Referências

- Vito, S. (2008). *Air Quality* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5060Z
- Méndez, M., Merayo, M. G., & Núñez, M. (2023). *Machine learning algorithms to forecast air quality: a survey*. Artificial Intelligence Review.
- Aula, K., Mäkelä, V., et al. (2022). *Evaluation of low-cost air quality sensor calibration models*.
- Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.

---

Instituição: Coimbra Business School | ISCAC
Curso: Licenciatura em Ciência de Dados para a Gestão
Unidade Curricular: Projeto em Ciência de Dados
Professor Responsável: Dora Melo (dmelo@iscac.pt)
