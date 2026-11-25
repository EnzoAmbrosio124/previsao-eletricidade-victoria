# Previsão da demanda diária de eletricidade em Victoria (Austrália)

Trabalho final da disciplina **Análise de Séries Temporais** (PUC-Campinas, 2026), Problema 04.

**Autores:** Enzo Ambrosio da Costa, Isaac Pereira Cerceau Netto e Vitor Riki Araujo Furuta

## Objetivo

Modelar a **demanda média diária de eletricidade** do estado de Victoria, na Austrália, de 01/01/2012 a 31/12/2014, escolher o melhor modelo de séries temporais e prever a demanda de **2015**.

Os dados vêm da AEMO, pelo conjunto `tsibbledata::vic_elec`. Cada valor é a média das leituras de 30 minutos de um dia, em MW. O notebook baixa os dados direto do GitHub, então não é preciso nenhum arquivo local.

## Metodologia

| Etapa | Treina com | Prevê | Para que serve |
|---|---|---|---|
| Validação | 2012 | 2013 | escolher o modelo |
| Teste | 2012 + 2013 | 2014 | avaliar o modelo escolhido |
| Previsão | 2012 a 2014 | 2015 | forecast final |

- **Horizonte:** `h = 365` dias. Um ano de teste cobre todas as estações.
- **Critério de escolha:** RMSE na validação, em MW. Todos os modelos preveem os 365 dias a partir da mesma origem, sem usar valores reais do período previsto.
- **Tratamento da série:** nenhum. Os picos são ondas de calor reais e os vales são as festas de fim de ano, que se repetem todo ano.

## Principais achados da análise exploratória

- **Sazonalidade semanal forte:** a demanda do fim de semana é cerca de 14% menor que a dos dias úteis (4.200 contra 4.891 MW).
- **Ciclo das estações:** a demanda sobe no inverno (aquecimento) e nas ondas de calor do verão (ar-condicionado). A relação com a temperatura tem formato de "U", com a menor demanda entre 16 e 18 °C.
- **A correlação comum engana:** a correlação de Pearson entre demanda e temperatura é 0,00, porque o efeito do frio e o do calor se cancelam. Usando a distância da temperatura até a média (|z-score|), ela sobe para 0,57, ou 0,78 só nos dias úteis.
- **Estacionariedade:** pelos testes ADF e KPSS, a série original não é estacionária e a primeira diferença é.

## Modelos comparados (RMSE na validação de 2013)

| Modelo | Melhor configuração | RMSE (MW) |
|---|---|---|
| **Prophet** | aditivo, festas como feriado, \|z\| da temperatura do ano anterior (média móvel de 30 dias) | **366,5** |
| SARIMA | (1,0,0)(1,0,1)[7] | 414,2 |
| *Sazonal ingênuo (referência)* | repete o ano anterior | *455,3* |
| Árvore de decisão | 21 lags, profundidade 8, previsão recursiva | 492,6 |
| ARIMA | (2,0,0) | 532,0 |
| *Média (referência)* | média do treino | *534,5* |
| LSTM | LSTM(32), janela de 7 dias, previsão recursiva | 543,6 |
| ETS | (M,M,A) | 684,2 |

O **Prophet** foi o escolhido. Além do menor erro, ele é o único cuja previsão acompanha as estações e o vale das festas. A previsão do SARIMA, sem o padrão semanal, vira uma linha reta.

O regressor de temperatura usa apenas informação já conhecida na hora de prever: a temperatura da mesma data do ano anterior, suavizada por média móvel só para trás.

## Resultados

**Teste (2014):**

- RMSE de **493,5 MW**, cerca de 10,7% da demanda média;
- 92,6% dos dias ficaram dentro do intervalo de previsão de 95%;
- 12% melhor que prever a média, mas praticamente empatado com o sazonal ingênuo (487 MW). A vantagem vista na validação, que teve uma única janela, era otimista.

**Previsão (2015):**

- Média prevista de **4.538 MW**;
- Pico em fevereiro (verão), queda até maio, nova subida em julho e agosto (inverno) e vale nas festas de fim de ano;
- Intervalo de 95% de cerca de ±700 MW.

## Limitações e próximos passos

- O **pico de inverno é subestimado** (de 500 a 600 MW em julho e agosto de 2014). Separar a temperatura em dois regressores, um de frio e um de calor, reduziu o erro do inverno em cerca de 30% no teste, mas isso ainda precisa ser validado.
- **Ondas de calor pontuais** não são previstas, porque o regressor mostra a estação e não o dia quente.
- Com mais anos de histórico, seria possível usar a sazonalidade anual do Prophet e ter mais janelas de validação.

## Como executar

Abra o notebook `AST_2026_Trabalho_Final_Problema_04_Eletricidade_Victoria_EnzoIsaacVitor.ipynb` no Jupyter ou no Google Colab. Bibliotecas usadas:

```
numpy pandas matplotlib statsmodels scikit-learn prophet tensorflow
```
