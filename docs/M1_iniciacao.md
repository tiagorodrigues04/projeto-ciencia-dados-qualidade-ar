# Milestone 1: Iniciação e Definição do Projeto

## 1. Descrição Detalhada do Problema
O nosso projeto foca-se na **monitorização da qualidade do ar**, um tema que nos pareceu bastante relevante e atual. A poluição atmosférica é algo que nos preocupa cada vez mais e se nada for feito, podemos estar a caminhar para um planeta cada vez mais inabitável. Foi por isso que, de todos os datasets que estavam disponíveis na lista fornecida pela professora, escolhemos este porque é o que nos parecia mais interessante e ligado a algo que realmente importa nos dias de hoje.

O dataset escolhido, **AirQualityUCI**, contém medições de poluentes atmosféricos, como `CO(GT)`, `NOx(GT)` e `NO2(GT)`, leituras de sensores químicos (`PT08.`) e variáveis meteorológicas como temperatura (`T`), a humidade relativa (`RH`) e a humidade absoluta (`AH`). Quando abrimos o ficheiro pela primeira vez, uma das principais coisas que notámos foi a existência de algumas colunas completamente vazias, o que pareceu-nos estranho e fez-nos perceber que o dataset precisava de tratamento antes de ser utilizado.

Como o dataset não tem nenhum índice global único de qualidade do ar, definimos como variável-alvo a concentração de monóxido de carbono **`CO(GT)`**, por ser uma medição de referência diretamente disponível nos dados e por permitir trabalhar com um problema de regressão supervisionada. 

O nosso objetivo com este projeto não é apenas obter boas métricas mas também que qualquer pessoa que veja o trabalho consiga perceber o que fizemos e fique a conhecer melhor este tema. O dataset em si não é recente nem global, mas já permite tirar conclusões interessantes sobre a poluição atmosférica e o que a influencia.

### Perguntas de Investigação
- Existem padrões horários ou diários na concentração de `CO(GT)` ao longo do período analisado?
- Quais as variáveis, nomeadamente os sensores `PT08.*` e as variáveis meteorológicas `T`, `RH` e `AH`, que têm maior relação com `CO(GT)`?
- É possível prever `CO(GT)` com um erro claramente inferior ao de um modelo simples que preveja sempre a média?

## 2. Objetivos SMART

**Objetivo Principal:** Até **18/04/2026**, desenvolver um modelo de regressão capaz de prever a concentração de monóxido de carbono `CO(GT)` com uma melhoria mínima de **15% em MAE ou RMSE** face a um modelo de referência simples (que prevê sempre a média), utilizando o dataset **AirQualityUCI** e as bibliotecas disponíveis em python.

**Métrica de sucesso:** redução de pelo menos 15% no erro de previsão (RMSE ou MAE) no conjunto de teste, comparando o melhor modelo desenvolvido com o modelo de referência.

## 3. Metodologia de Gestão (PBL)

### Divisão de Tarefas
No início tivemos alguma dificuldade em perceber exatamente o que era para fazer e quais os pontos mais importantes do trabalho. Com o tempo fomos ganhado mais clareza e dividimos as responsabilidades da seguinte forma:
- **Rodrigo Pedro:** responsável pela parte dos documentos como por exemplo a análise exploratória, a interpretação dos resultados a escrita da documentação em `docs/` e a atualização do `README.md`.
- **Tiago Rodrigues:** responsável pela parte do código como a importação do dataset no kaggle, a limpeza dos dados, o tratamento dos valores em falta, a criação de variáveis temporais e a organização do repositório.

### Ferramentas de Colaboração
- **GitHub** para guardar e partilhar o trabalho de uma forma mais organizada
- **Kaggle** para desenvolver e correr os notebooks
- **Discord** e **WhatsApp** para comunicar e combinar o que cada um ia fazendo

### Ferramentas e Bibliotecas Python
- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `scikit-learn`

## 4. Análise de Viabilidade dos Dados

**Disponibilidade:** O dataset foi obtido em formato `csv` e `xlsx` e importado para um notebook no Kaggle. O ficheiro CSV tem uma formatação específica que exige `sep=';'` e `decimal=','` na leitura, foi algo que descobrimos ao tentar abrir o ficheiro pela primeira vez.

**Qualidade Inicial:** Ao analisar o dataset com `df.head()`, `df.info()` e `df.describe()` verificámos tem **9471 linhas e 17 colunas**, mas duas dessas colunas estavam completamente vazias e este ponto foi das primeiras coisas que notámos e que nos pareceu estranho. Para além disso, identificámos os seguintes aspetos a tratar:
- os valores em falta estão codificados como **`-200`** e não aparecem automaticamente como`NaN`;
- as colunas `Date` e `Time` precisam de ser combinadas numa variável temporal;
- a variável `NMHC(GT)` tem uma percentagem muito elevada em valores em falta;
- poderão existir outliers que precisam de ser analisados durante a Milestone 2.

**Ética:** O conjunto de dados é composto apenas por medições ambientais, sem qualquer dado pessoal. Não há problemas de privacidade nem risco de incumprimento do RGPD. A utilização é exclusivamente académica.

## 5. Cronograma Interno

| Fase | Data Limite | Entregável Esperado |
| :--- | :--- | :--- |
| M1: Iniciação | 24/02/2026 | Repositório estruturado e plano de projeto |
| M2: Exploração | 24/03/2026 | Notebook de EDA e dados processados |
| M3: Modelação | 21/04/2026 | Comparação de algoritmos e métricas |
| M4: Finalização | 21/05/2026 | Pitch e relatório final |

---

*Data de última atualização: 07/05/2026*
