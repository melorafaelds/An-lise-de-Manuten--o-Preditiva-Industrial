# Análise de Manutenção Preditiva Industrial

Projeto de análise exploratória de dados aplicado à manutenção preditiva de máquinas industriais. O objetivo é identificar condições operacionais associadas às falhas, priorizar sinais para monitoramento e transformar os resultados em recomendações práticas para a operação.

## Visão geral

Paradas não programadas aumentam o custo de manutenção, interrompem a produção e podem comprometer prazos de entrega. Neste projeto, são analisados registros de sensores e indicadores de falha de um processo industrial de usinagem.

A análise procura responder:

- Qual é a taxa observada de falhas?
- Quais tipos de falha aparecem com maior frequência?
- Quais diferenças existem entre os tipos de produto L, M e H?
- Como temperatura, velocidade, torque e desgaste se relacionam com as falhas?
- É possível criar grupos operacionais de baixo, médio e alto risco?
- Quais variáveis devem ser monitoradas prioritariamente?

> **Importante:** os resultados representam associações encontradas nesta base de dados. Eles não comprovam causalidade e não substituem a validação com dados reais da fábrica.

## Objetivos

### Objetivo geral

Identificar os principais fatores associados às falhas das máquinas e propor ações de manutenção preventiva baseadas em dados.

### Objetivos específicos

1. Avaliar a qualidade e a estrutura da base.
2. Medir a taxa geral de falhas.
3. Comparar a ocorrência dos tipos de falha TWF, HDF, PWF, OSF e RNF.
4. Investigar relações entre variáveis de processo e falha geral.
5. Avaliar a influência do desgaste da ferramenta.
6. Criar uma segmentação operacional simples por nível de risco.
7. Traduzir os achados em recomendações de monitoramento e manutenção.

## Base de dados

O projeto utiliza o **AI4I 2020 Predictive Maintenance Dataset**, disponibilizado no Kaggle. A base contém 10.000 registros e 14 colunas originais. Os dados são sintéticos, mas simulam variáveis de um processo industrial de usinagem.

Fonte: [AI4I 2020 Predictive Maintenance Dataset](https://www.kaggle.com/datasets/stephanmatzka/predictive-maintenance-dataset-ai4i-2020)

### Dicionário de dados

| Coluna | Descrição |
| --- | --- |
| `UDI` | Identificador único do registro. |
| `Product ID` | Identificador do produto, composto pela qualidade e pelo número de série. |
| `Type` | Tipo de produto: `L` (baixa qualidade), `M` (média) ou `H` (alta). |
| `Air temperature [K]` | Temperatura do ar, medida em Kelvin. |
| `Process temperature [K]` | Temperatura do processo, medida em Kelvin. |
| `Rotational speed [rpm]` | Velocidade rotacional da ferramenta, em rotações por minuto. |
| `Torque [Nm]` | Torque aplicado à ferramenta, em Newton-metro. |
| `Tool wear [min]` | Tempo acumulado de desgaste da ferramenta, em minutos. |
| `Machine failure` | Indicador geral: `1` para falha e `0` para ausência de falha. |
| `TWF` | Falha por desgaste da ferramenta (*Tool Wear Failure*). |
| `HDF` | Falha por dissipação de calor (*Heat Dissipation Failure*). |
| `PWF` | Falha de potência (*Power Failure*). |
| `OSF` | Falha por sobrecarga (*Overstrain Failure*). |
| `RNF` | Falha aleatória (*Random Failure*). |

## Metodologia

O notebook foi organizado em etapas para manter a análise rastreável:

1. **Importação:** carregamento de `pandas`, `numpy`, `matplotlib`, `seaborn` e `plotly`.
2. **Carregamento:** leitura do arquivo `ai4i2020_original.csv`.
3. **Qualidade dos dados:** verificação de tipos, valores ausentes e registros duplicados.
4. **Análise exploratória:** estatísticas descritivas e distribuição das variáveis.
5. **Análises por pergunta:** comparação de falhas, temperaturas, torque, rotação e desgaste.
6. **Engenharia de variável:** criação do `Delta termico [K]`, calculado como temperatura do processo menos temperatura do ar.
7. **Faixas operacionais:** agrupamento do desgaste e de variáveis de operação para facilitar a interpretação.
8. **Correlação:** cálculo da correlação linear entre as variáveis numéricas e `Machine failure`.
9. **Segmentação de risco:** pontuação baseada em valores acima do percentil 75 de variáveis operacionais.
10. **Recomendação:** tradução dos resultados em ações de manutenção e monitoramento.

## Qualidade dos dados

Na verificação inicial:

- Foram encontrados 10.000 registros.
- Não foram identificados valores ausentes.
- Não foram identificados registros duplicados.
- A coluna `Machine failure` foi utilizada como variável-alvo.

## Principais resultados

### Taxa geral de falhas

Foram identificadas **339 falhas em 10.000 registros**, equivalente a uma taxa observada de **3,39%**. Os outros 9.661 registros não apresentaram falha.

Esse resultado deve ser interpretado dentro do contexto da base. Como os dados são sintéticos e não informam a duração real de operação, número de máquinas ou horas efetivamente trabalhadas, não é possível converter essa taxa diretamente em uma taxa universal da fábrica.

### Tipos de falha

Os tipos HDF, OSF e PWF concentraram a maior parte das ocorrências de falha. Isso sugere três frentes prioritárias de investigação:

- condição térmica e capacidade de dissipação de calor;
- combinação entre esforço mecânico e velocidade;
- potência e torque exigidos durante a operação.

### Tipo de produto

Quando a comparação é feita pela taxa, e não apenas pela quantidade absoluta, os resultados observados foram:

| Tipo | Registros | Falhas | Taxa de falha |
| --- | ---: | ---: | ---: |
| `L` | 6.000 | 235 | 3,92% |
| `M` | 2.997 | 83 | 2,77% |
| `H` | 1.003 | 21 | 2,09% |

O tipo `L` apresentou a maior taxa observada. Essa diferença é um sinal para investigação, mas não permite afirmar que a qualidade do produto seja a causa das falhas.

### Temperatura e dissipação de calor

Foi criado o delta térmico:

```text
Delta termico [K] = Process temperature [K] - Air temperature [K]
```

Os registros com HDF apresentaram delta térmico médio de **8,23 K**, enquanto os demais apresentaram média de **10,02 K**. O comportamento é compatível com uma condição de menor diferença térmica para dissipação, mas deve ser analisado junto às temperaturas absolutas e às condições de operação.

### Torque e velocidade

As máquinas que falharam apresentaram:

- torque médio de **50,17 Nm**, contra **39,63 Nm** nos registros sem falha;
- velocidade média de **1.496 rpm**, contra **1.540 rpm** nos registros sem falha.

O padrão sugere maior esforço mecânico em situações de falha, principalmente quando torque elevado aparece associado a rotação baixa ou intermediária.

### Desgaste da ferramenta

O desgaste médio foi de **143,78 minutos** nos registros com falha e **106,69 minutos** nos registros sem falha.

O resultado mais importante apareceu na análise por faixa:

| Faixa de desgaste | Registros | Falhas | Taxa de falha |
| --- | ---: | ---: | ---: |
| 0-49 min | 2.349 | 52 | 2,21% |
| 50-99 min | 2.271 | 51 | 2,25% |
| 100-149 min | 2.290 | 52 | 2,27% |
| 150-199 min | 2.289 | 61 | 2,66% |
| 200 min ou mais | 801 | 123 | 15,36% |

A faixa de **200 minutos ou mais** pode ser utilizada como um limiar inicial de inspeção ou troca preventiva. Esse limite precisa ser recalibrado com dados reais de vida útil, custo de troca e custo de parada.

### Correlações

As maiores correlações lineares com `Machine failure` foram:

| Variável | Correlação |
| --- | ---: |
| `Torque [Nm]` | 0,191 |
| `Tool wear [min]` | 0,105 |
| `Air temperature [K]` | 0,083 |
| `Rotational speed [rpm]` | -0,044 |
| `Process temperature [K]` | 0,036 |

As correlações são relativamente baixas. Isso significa que uma única variável não explica todas as falhas e que regras combinadas ou modelos preditivos podem representar melhor o comportamento.

### Segmentação operacional de risco

Foi criada uma pontuação simples. Cada registro recebe um ponto quando apresenta valor acima do percentil 75 de torque, desgaste, temperatura do ar ou velocidade. O total de pontos define o grupo:

- **Baixo:** até 1 ponto;
- **Médio:** 2 pontos;
- **Alto:** 3 ou 4 pontos.

| Grupo | Registros | Falhas | Taxa de falha |
| --- | ---: | ---: | ---: |
| Baixo | 7.404 | 85 | 1,15% |
| Médio | 2.251 | 205 | 9,11% |
| Alto | 345 | 49 | 14,20% |

Essa segmentação não é um modelo preditivo validado, mas pode apoiar a priorização inicial de inspeções.

## Recomendações para a operação

1. Criar um alerta de inspeção para ferramentas com desgaste a partir de 200 minutos.
2. Monitorar torque e velocidade em conjunto, com atenção a torque elevado e rotação baixa ou intermediária.
3. Acompanhar o delta térmico junto às temperaturas do ar e do processo.
4. Priorizar inspeções nos registros classificados como risco médio e alto.
5. Registrar a causa, duração e custo de cada parada.
6. Separar falhas por tipo para direcionar ações específicas de processo, ferramenta, refrigeração e potência.
7. Validar os limiares com dados reais antes de transformá-los em regras definitivas.

### Variáveis prioritárias

Se fosse possível monitorar apenas duas ou três variáveis, as prioridades seriam:

1. **Torque:** apresentou a maior correlação com a falha e foi maior nos registros que falharam.
2. **Desgaste da ferramenta:** apresentou aumento expressivo de risco a partir de 200 minutos.
3. **Delta térmico:** ajuda a identificar condições relacionadas à dissipação de calor e HDF.

A velocidade rotacional deve ser mantida como variável complementar, especialmente para interpretar o torque.

## Impacto estimado das paradas

O notebook deixa a variável `horas_parada_por_falha` parametrizada porque o enunciado não informa o valor médio de X. Com o valor padrão de 1 hora:

```text
339 falhas x 1 hora por falha = 339 horas estimadas de parada
```

Para uma estimativa real, substitua o parâmetro pela média histórica de duração das paradas. O impacto financeiro pode então ser calculado considerando produção perdida, mão de obra, peças, atendimento técnico e custo de oportunidade.

## Estrutura do projeto

```text
AnalliseDeMaquina/
├── ai4i2020_original.csv
├── analise_manutencao_preditiva.ipynb
├── relatório.docx
├── readme.md
└── Graficos/
	├── 01_distribuicao_falhas.png
	├── 02_tipos_de_falha.png
	├── 03_torque_velocidade.png
	├── 04_taxa_por_desgaste.png
	└── 05_correlacoes.png
```

### Descrição dos arquivos

- `analise_manutencao_preditiva.ipynb`: notebook com exploração, cálculos, gráficos, interpretações e conclusão.
- `ai4i2020_original.csv`: base utilizada no projeto.
- `Graficos/`: imagens exportadas dos principais resultados.
- `relatório.docx`: relatório executivo com explicações e gráficos incorporados.
- `readme.md`: documentação do projeto.

## Como executar

### Pré-requisitos

- Python 3.10 ou superior;
- Jupyter Notebook ou VS Code com a extensão Jupyter;
- bibliotecas `pandas`, `numpy`, `matplotlib`, `seaborn` e `plotly`.

### Instalação

```bash
pip install pandas numpy matplotlib seaborn plotly
```

### Execução do notebook

1. Abra a pasta do projeto no VS Code.
2. Abra `analise_manutencao_preditiva.ipynb`.
3. Selecione um kernel Python com as dependências instaladas.
4. Execute as células na ordem apresentada.
5. Confirme que o arquivo CSV está na mesma pasta do notebook.

Os caminhos utilizados no projeto são relativos à pasta principal, o que facilita a reprodução em outro computador.

## Limitações

- A base é sintética e não representa necessariamente o comportamento de uma fábrica específica.
- Não há identificação explícita de várias máquinas acompanhadas ao longo do tempo.
- Não existe duração real das paradas nem custo financeiro por falha.
- A correlação linear não captura todos os efeitos combinados entre variáveis.
- RNF possui poucos registros, portanto não há evidência suficiente para concluir que exista um padrão estável.
- Os limiares de risco são exploratórios e precisam de validação antes de serem utilizados em produção.

## Próximos passos

1. Incorporar dados históricos reais de máquinas, turnos, lotes, intervenções e paradas.
2. Criar variáveis temporais para acompanhar a evolução dos sensores antes da falha.
3. Separar treinamento, validação e teste para construir um modelo preditivo.
4. Avaliar métricas adequadas para eventos raros, como recall, precisão, F1-score e PR-AUC.
5. Comparar modelos interpretáveis, como regressão logística e árvore de decisão, com modelos mais complexos.
6. Definir o custo de falsos alertas e falhas não detectadas.
7. Monitorar o desempenho do modelo após sua implantação na operação.

## Créditos

Dados: [AI4I 2020 Predictive Maintenance Dataset](https://www.kaggle.com/datasets/stephanmatzka/predictive-maintenance-dataset-ai4i-2020).

Projeto desenvolvido para portfólio de análise de dados aplicada à manutenção preditiva industrial.
