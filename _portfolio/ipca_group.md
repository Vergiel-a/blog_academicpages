---
title: "Analisando os grupos do IPCA com o Python"
excerpt: "Neste artigo mostramos como coletar dados de inflação segmentados por faixa de renda e como calcular a variação acumulada em 12 meses usando a linguagem de programação Python. <br/><img src='/images/portfolio/ipca_group/output_10_0.png'>"
date: 18/04/2024
---

**autor: Luiz Henrique**

A inflação é conhecida como o termo que representa a taxa de crescimento do nível geral de preços entre dois períodos distintos. No Brasil, o indicador que consolidou-se como o principal índice de preços é o Índice de Preços ao Consumidor Amplo (IPCA), divulgado pelo IBGE e amplamente utilizado pela autoridade monetária como referência para realizar o controle da inflação. Neste artigo mostramos como obter a contribuição de cada grupo do IPCA usando o Python.

O IPCA é divulgado mensalmente pelo IBGE, e podemos importar os dados diretamente do SIDRA, através de sua API. Para auxiliar no processo de extração de dados usamos a biblioteca sidrapy, que permite obtermos os dados inserindo os parâmetros sobre a API da tabela de interesse.

Nosso objetivo aqui será o de buscar a série de peso e variação de cada grupo do IPCA, e de posse dos dados, calculamos a contribuição de cada grupo sobre o IPCA. Ao fim, criamos um gráfico de barras que permite avaliarmos o IPCA por grupos.


```python
# Importa as bibliotecas necessárias
!pip install sidrapy
import sidrapy

import numpy as np
import pandas as pd

import datetime as dt

import seaborn as sns
from matplotlib import pyplot as plt
from plotnine import *
```

# IPCA Contribuição por grupo

O primeiro passo será buscar a série na plataforma do sidra de forma que possamos resgatar os códigos do parâmetros.

Uma vez obtida a API da tabela 1737, e seus respectivos códigos, utilizamos a função get_table para obter a série. A API que gerou os dados foi a seguinte: `/t/7060/n1/all/v/63,66/p/all/c315/7170,7445,7486,7558,7625,7660,7712,7766,7786/d/v63%202,v66%204`.




```python
# Importa as variações e os pesos dos grupos do IPCA
ipca_gp_raw = sidrapy.get_table(table_code = '7060',
                             territorial_level = '1',
                             ibge_territorial_code = 'all',
                             variable = '63,66',
                             period = 'all',
                             classification = '315/7170,7445,7486,7558,7625,7660,7712,7766,7786'
                             )
```


```python
# Realiza a limpeza e manipulação da tabela
ipca_gp =  (
    ipca_gp_raw
    .loc[1:,['V', 'D2C', 'D3N', 'D4N']]
    .rename(columns = {'V': 'value',
                       'D2C': 'date',
                       'D3N': 'variable',
                       'D4N': 'groups'})
    .assign(variable = lambda x: x['variable'].replace({'IPCA - Variação mensal' : 'variacao',
                                                        'IPCA - Peso mensal': 'peso'}),
            date  = lambda x: pd.to_datetime(x['date'],
                                              format = "%Y%m"),
            value = lambda x: x['value'].astype(float),
            groups = lambda x: x['groups'].astype(str)
           )
    .pipe(lambda x: x.loc[x.date > '2007-01-01'])
        )
```


```python
# Torna em formato wide e calcula a contribuição de cada grupo pro IPCA
ipca_gp_wider = (
    ipca_gp
    .pivot_table(index = ['date', 'groups'],
                 columns = 'variable',
                 values = 'value')
    .reset_index()
    .assign(contribuicao = lambda x: (x.peso * x.variacao) / 100)
                )
```


```python
ipca_gp_wider[['date', 'groups', 'contribuicao']].tail(n = 9)
```





  <div id="df-a0c50c3d-1847-4963-950f-d2838e02b03e" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>variable</th>
      <th>date</th>
      <th>groups</th>
      <th>contribuicao</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>423</th>
      <td>2023-12-01</td>
      <td>1.Alimentação e bebidas</td>
      <td>0.233151</td>
    </tr>
    <tr>
      <th>424</th>
      <td>2023-12-01</td>
      <td>2.Habitação</td>
      <td>0.052217</td>
    </tr>
    <tr>
      <th>425</th>
      <td>2023-12-01</td>
      <td>3.Artigos de residência</td>
      <td>0.028743</td>
    </tr>
    <tr>
      <th>426</th>
      <td>2023-12-01</td>
      <td>4.Vestuário</td>
      <td>0.033312</td>
    </tr>
    <tr>
      <th>427</th>
      <td>2023-12-01</td>
      <td>5.Transportes</td>
      <td>0.100522</td>
    </tr>
    <tr>
      <th>428</th>
      <td>2023-12-01</td>
      <td>6.Saúde e cuidados pessoais</td>
      <td>0.046640</td>
    </tr>
    <tr>
      <th>429</th>
      <td>2023-12-01</td>
      <td>7.Despesas pessoais</td>
      <td>0.048705</td>
    </tr>
    <tr>
      <th>430</th>
      <td>2023-12-01</td>
      <td>8.Educação</td>
      <td>0.014062</td>
    </tr>
    <tr>
      <th>431</th>
      <td>2023-12-01</td>
      <td>9.Comunicação</td>
      <td>0.001929</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-a0c50c3d-1847-4963-950f-d2838e02b03e')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-a0c50c3d-1847-4963-950f-d2838e02b03e button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-a0c50c3d-1847-4963-950f-d2838e02b03e');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-95406191-7769-43e4-abff-aa08e8ca0db8">
  <button class="colab-df-quickchart" onclick="quickchart('df-95406191-7769-43e4-abff-aa08e8ca0db8')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-95406191-7769-43e4-abff-aa08e8ca0db8 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>





```python
# Cores para gráficos
colors = {'blue': '#282f6b',
          'yellow': '#eace3f',
          'red'   : "#b22200",
          'green': '#224f20',
          'purple' : "#5f487c",
          'gray': '#666666',
          'orange' : '#b35c1e',
          'turquoise' : "#419391",
          'green_two' : "#839c56"
          }
```


```python
# Cria o gráfico
(ggplot(ipca_gp_wider) +
        aes(x='date') +
        geom_col(aes(y='contribuicao', fill='groups', colour='groups')) +
        geom_hline(yintercept=0, colour='black', linetype='dashed') +
        scale_x_date(date_breaks = "3 month", date_labels = "%b/%Y") +
        scale_colour_manual(values=list(colors.values())) +
        scale_fill_manual(values=list(colors.values())) +
        labs(x='', y='', title='Contribuição dos Grupos do IPCA para a Inflação Mensal', caption = 'Elaborado por: analisemacro.com.br | Fonte: IBGE/SIDRA')+
        theme(figure_size = (14, 6),
              legend_position = "bottom")
        )
```


    
![png](/images/portfolio/ipca_group/output_10_0.png)
    





    <Figure Size: (1400 x 600)>



