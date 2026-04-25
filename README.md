# Previsão de Consultas por Síndrome Gripal — ARS Norte

Modelo preditivo de consultas diárias por síndrome gripal nos Cuidados de Saúde Primários da região da **ARS Norte**, desenvolvido no âmbito da unidade curricular de **Séries Temporais** da Licenciatura em Ciência de Dados para a Gestão (ISCAC, 2025/2026).

---

## Descrição do Projeto

O objetivo central é simular um ambiente operacional realista: gerar previsões para um horizonte de **14 dias** utilizando exclusivamente informação histórica disponível **3 dias antes** do início do período de previsão — replicando os atrasos reais de consolidação de dados administrativos no SNS.

O projeto cobre dois horizontes temporais:
- **Análise histórica alargada (2016–2025):** tendências de longo prazo, impacto da pandemia COVID-19 e padrões sazonais
- **Período de foco (setembro 2024 – dezembro 2025):** treino, validação e previsão dos modelos

---

## Dataset

- **Fonte:** [Portal da Transparência do SNS](https://transparencia.sns.gov.pt) — dataset *"Atendimentos nos CSP – Gripe"*
- **Região:** ARS Norte
- **Período:** 2016–2025
- **Frequência:** Diária
- **Extração:** Automatizada via API REST com paginação em lotes de 100 dias
- **Atraso real detetado (gap):** 2 dias entre a ocorrência e a disponibilização pública dos dados

---

## Análise Exploratória

- Evolução histórica da série (2016–2025) com identificação do impacto pandémico (2020–2022)
- Sobreposição anual com padrão sazonal em forma de "U" — picos no inverno, mínimos de junho a setembro
- Heatmap de intensidade mensal por ano
- **Efeito de Segunda-Feira:** identificado e validado estatisticamente via Violin Plots e Boxplots — as segundas-feiras apresentam sistematicamente as medianas mais elevadas, absorvendo o represamento de sintomas do fim de semana
- Decomposição STL (Seasonal-Trend decomposition using Loess) com identificação de outliers negativos em feriados a meio da semana

---

## Modelos Testados

### Fase 1 — Sem preditores exógenos

| Modelo | MAE | RMSE |
|--------|-----|------|
| AutoETS | 19.04 | 37.53 |
| AutoARIMA | 19.68 | 37.09 |
| SeasonalNaive | 20.47 | 39.15 |

### Fase 2 — Com variável exógena `Feriado`

| Modelo | MAE | RMSE |
|--------|-----|------|
| **AutoARIMA** | **18.98** | **35.69** |
| AutoETS | 19.04 | 37.53 |
| SeasonalNaive | 20.47 | 39.15 |

A inclusão da variável binária `Feriado` (dias de paragem nacional e tolerâncias de ponto) foi determinante para eliminar os grandes resíduos associados a paragens a meio da semana, reduzindo o RMSE de 37.09 para 35.69.

---

## Modelo Final: AutoARIMA Multivariado

**Configuração:**
- `season_length = 7` (sazonalidade semanal)
- Variável exógena: `Feriado` (binária)
- Janela de treino: 457 observações (01/09/2024 a 01/12/2025)
- Biblioteca: [StatsForecast — Nixtla](https://github.com/Nixtla/statsforecast)

**Validação estatística dos resíduos:**
- Teste de Ljung-Box: p-value = 0.517 → resíduos como ruído branco 
- Teste t-Student: erro médio residual = 1.07 consultas (estatisticamente indistinguível de zero) 
- Teste de Shapiro-Wilk: normalidade rejeitada — esperado em dados epidemiológicos com eventos extremos

---

## Previsão Objetivo e Deteção do Surto (6–12 dezembro 2025)

O modelo previu corretamente a capacidade basal para uma semana típica de dezembro (**1.277 episódios previstos**). A realidade registou **3.452 atendimentos** — um excesso de +170.3%.

| Data | Previsão | Realidade | Dentro do IC 95%? |
|------|----------|-----------|-------------------|
| 06/12 (Sáb) | ~100 | ~200 | |
| 08/12 (Seg — Feriado) | 92 | 201 | |
| 09/12 (Ter) | 154 | **919** | Surto |
| 10–12/12 | 150–300 | 600–900 | Surto |

> O desvio de +170% não representa uma falha do modelo — é a **métrica que quantifica a severidade do surto epidémico**. Ao definir com rigor a fronteira da normalidade, o sistema funciona como um **Sistema de Alerta Precoce**: o momento em que a realidade rompe o intervalo de confiança de 95% é a confirmação estatística de que os recursos dimensionados pelo histórico normal já não são suficientes.

---

## Tecnologias

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Statsmodels](https://img.shields.io/badge/Statsmodels-4B8BBE?style=flat)
![Nixtla](https://img.shields.io/badge/StatsForecast-Nixtla-blueviolet?style=flat)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=flat)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?style=flat&logo=plotly&logoColor=white)

- **Python** — linguagem principal
- **requests** — extração automatizada via API REST
- **pandas / numpy** — manipulação e pré-processamento
- **statsmodels** — decomposição STL e testes estatísticos
- **StatsForecast (Nixtla)** — AutoARIMA e AutoETS
- **holidays** — geração automática do calendário de feriados portugueses
- **matplotlib / seaborn / plotly** — visualização

---

## Estrutura do Repositório

```
 Previsao_dos_casos_de_gripe_norte
   Trabalho_Final_Grupo6_Norte.ipynb   # Notebook principal com todo o pipeline
   Grupo_6_-_Norte.pdf                 # Relatório técnico completo
   README.md
```

---

## Autores

| Nome | Número de Estudante |
|------|-------------------|
| Tomás Moreira | a2023143375 |
| Rodrigo Ferrão | a2022138105 |

**Instituição:** ISCAC — Instituto Superior de Contabilidade e Administração de Coimbra  
**Unidade Curricular:** Séries Temporais  
**Docente:** Joana Leite  
**Ano Letivo:** 2025/2026
