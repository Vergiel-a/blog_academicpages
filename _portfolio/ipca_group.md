# Analisando os grupos do IPCA com o Python

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
ipca_gp_raw
```





  <div id="df-307b2724-2cad-4939-95ef-bb59f33d9175" class="colab-df-container">
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
      <th></th>
      <th>NC</th>
      <th>NN</th>
      <th>MC</th>
      <th>MN</th>
      <th>V</th>
      <th>D1C</th>
      <th>D1N</th>
      <th>D2C</th>
      <th>D2N</th>
      <th>D3C</th>
      <th>D3N</th>
      <th>D4C</th>
      <th>D4N</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Nível Territorial (Código)</td>
      <td>Nível Territorial</td>
      <td>Unidade de Medida (Código)</td>
      <td>Unidade de Medida</td>
      <td>Valor</td>
      <td>Brasil (Código)</td>
      <td>Brasil</td>
      <td>Mês (Código)</td>
      <td>Mês</td>
      <td>Variável (Código)</td>
      <td>Variável</td>
      <td>Geral, grupo, subgrupo, item e subitem (Código)</td>
      <td>Geral, grupo, subgrupo, item e subitem</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>0.39</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202001</td>
      <td>janeiro 2020</td>
      <td>63</td>
      <td>IPCA - Variação mensal</td>
      <td>7170</td>
      <td>1.Alimentação e bebidas</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>0.55</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202001</td>
      <td>janeiro 2020</td>
      <td>63</td>
      <td>IPCA - Variação mensal</td>
      <td>7445</td>
      <td>2.Habitação</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>-0.07</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202001</td>
      <td>janeiro 2020</td>
      <td>63</td>
      <td>IPCA - Variação mensal</td>
      <td>7486</td>
      <td>3.Artigos de residência</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>-0.48</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202001</td>
      <td>janeiro 2020</td>
      <td>63</td>
      <td>IPCA - Variação mensal</td>
      <td>7558</td>
      <td>4.Vestuário</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>860</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>20.9421</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202312</td>
      <td>dezembro 2023</td>
      <td>66</td>
      <td>IPCA - Peso mensal</td>
      <td>7625</td>
      <td>5.Transportes</td>
    </tr>
    <tr>
      <th>861</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>13.3257</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202312</td>
      <td>dezembro 2023</td>
      <td>66</td>
      <td>IPCA - Peso mensal</td>
      <td>7660</td>
      <td>6.Saúde e cuidados pessoais</td>
    </tr>
    <tr>
      <th>862</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>10.1469</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202312</td>
      <td>dezembro 2023</td>
      <td>66</td>
      <td>IPCA - Peso mensal</td>
      <td>7712</td>
      <td>7.Despesas pessoais</td>
    </tr>
    <tr>
      <th>863</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>5.8592</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202312</td>
      <td>dezembro 2023</td>
      <td>66</td>
      <td>IPCA - Peso mensal</td>
      <td>7766</td>
      <td>8.Educação</td>
    </tr>
    <tr>
      <th>864</th>
      <td>1</td>
      <td>Brasil</td>
      <td>2</td>
      <td>%</td>
      <td>4.8228</td>
      <td>1</td>
      <td>Brasil</td>
      <td>202312</td>
      <td>dezembro 2023</td>
      <td>66</td>
      <td>IPCA - Peso mensal</td>
      <td>7786</td>
      <td>9.Comunicação</td>
    </tr>
  </tbody>
</table>
<p>865 rows × 13 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-307b2724-2cad-4939-95ef-bb59f33d9175')"
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
        document.querySelector('#df-307b2724-2cad-4939-95ef-bb59f33d9175 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-307b2724-2cad-4939-95ef-bb59f33d9175');
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


<div id="df-41f97b6d-c5d8-45db-ba63-52e5baadcdb0">
  <button class="colab-df-quickchart" onclick="quickchart('df-41f97b6d-c5d8-45db-ba63-52e5baadcdb0')"
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
        document.querySelector('#df-41f97b6d-c5d8-45db-ba63-52e5baadcdb0 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

  <div id="id_1299e090-fb25-4f60-9972-41a1f29c80bc">
    <style>
      .colab-df-generate {
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

      .colab-df-generate:hover {
        background-color: #E2EBFA;
        box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
        fill: #174EA6;
      }

      [theme=dark] .colab-df-generate {
        background-color: #3B4455;
        fill: #D2E3FC;
      }

      [theme=dark] .colab-df-generate:hover {
        background-color: #434B5C;
        box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
        filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
        fill: #FFFFFF;
      }
    </style>
    <button class="colab-df-generate" onclick="generateWithVariable('ipca_gp_raw')"
            title="Generate code using this dataframe."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M7,19H8.4L18.45,9,17,7.55,7,17.6ZM5,21V16.75L18.45,3.32a2,2,0,0,1,2.83,0l1.4,1.43a1.91,1.91,0,0,1,.58,1.4,1.91,1.91,0,0,1-.58,1.4L9.25,21ZM18.45,9,17,7.55Zm-12,3A5.31,5.31,0,0,0,4.9,8.1,5.31,5.31,0,0,0,1,6.5,5.31,5.31,0,0,0,4.9,4.9,5.31,5.31,0,0,0,6.5,1,5.31,5.31,0,0,0,8.1,4.9,5.31,5.31,0,0,0,12,6.5,5.46,5.46,0,0,0,6.5,12Z"/>
  </svg>
    </button>
    <script>
      (() => {
      const buttonEl =
        document.querySelector('#id_1299e090-fb25-4f60-9972-41a1f29c80bc button.colab-df-generate');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      buttonEl.onclick = () => {
        google.colab.notebook.generateWithVariable('ipca_gp_raw');
      }
      })();
    </script>
  </div>

    </div>
  </div>





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
ipca_gp
```





  <div id="df-db550acd-03e1-4122-a2cf-8f07f5d381f7" class="colab-df-container">
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
      <th></th>
      <th>value</th>
      <th>date</th>
      <th>variable</th>
      <th>groups</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0.3900</td>
      <td>2020-01-01</td>
      <td>variacao</td>
      <td>1.Alimentação e bebidas</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.5500</td>
      <td>2020-01-01</td>
      <td>variacao</td>
      <td>2.Habitação</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.0700</td>
      <td>2020-01-01</td>
      <td>variacao</td>
      <td>3.Artigos de residência</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.4800</td>
      <td>2020-01-01</td>
      <td>variacao</td>
      <td>4.Vestuário</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.3200</td>
      <td>2020-01-01</td>
      <td>variacao</td>
      <td>5.Transportes</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>860</th>
      <td>20.9421</td>
      <td>2023-12-01</td>
      <td>peso</td>
      <td>5.Transportes</td>
    </tr>
    <tr>
      <th>861</th>
      <td>13.3257</td>
      <td>2023-12-01</td>
      <td>peso</td>
      <td>6.Saúde e cuidados pessoais</td>
    </tr>
    <tr>
      <th>862</th>
      <td>10.1469</td>
      <td>2023-12-01</td>
      <td>peso</td>
      <td>7.Despesas pessoais</td>
    </tr>
    <tr>
      <th>863</th>
      <td>5.8592</td>
      <td>2023-12-01</td>
      <td>peso</td>
      <td>8.Educação</td>
    </tr>
    <tr>
      <th>864</th>
      <td>4.8228</td>
      <td>2023-12-01</td>
      <td>peso</td>
      <td>9.Comunicação</td>
    </tr>
  </tbody>
</table>
<p>864 rows × 4 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-db550acd-03e1-4122-a2cf-8f07f5d381f7')"
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
        document.querySelector('#df-db550acd-03e1-4122-a2cf-8f07f5d381f7 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-db550acd-03e1-4122-a2cf-8f07f5d381f7');
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


<div id="df-f17a1f3c-d04a-46cd-ad16-550a1ddac1bd">
  <button class="colab-df-quickchart" onclick="quickchart('df-f17a1f3c-d04a-46cd-ad16-550a1ddac1bd')"
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
        document.querySelector('#df-f17a1f3c-d04a-46cd-ad16-550a1ddac1bd button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

  <div id="id_486e36d0-5162-49b4-b3a5-96a80a90cefa">
    <style>
      .colab-df-generate {
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

      .colab-df-generate:hover {
        background-color: #E2EBFA;
        box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
        fill: #174EA6;
      }

      [theme=dark] .colab-df-generate {
        background-color: #3B4455;
        fill: #D2E3FC;
      }

      [theme=dark] .colab-df-generate:hover {
        background-color: #434B5C;
        box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
        filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
        fill: #FFFFFF;
      }
    </style>
    <button class="colab-df-generate" onclick="generateWithVariable('ipca_gp')"
            title="Generate code using this dataframe."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M7,19H8.4L18.45,9,17,7.55,7,17.6ZM5,21V16.75L18.45,3.32a2,2,0,0,1,2.83,0l1.4,1.43a1.91,1.91,0,0,1,.58,1.4,1.91,1.91,0,0,1-.58,1.4L9.25,21ZM18.45,9,17,7.55Zm-12,3A5.31,5.31,0,0,0,4.9,8.1,5.31,5.31,0,0,0,1,6.5,5.31,5.31,0,0,0,4.9,4.9,5.31,5.31,0,0,0,6.5,1,5.31,5.31,0,0,0,8.1,4.9,5.31,5.31,0,0,0,12,6.5,5.46,5.46,0,0,0,6.5,12Z"/>
  </svg>
    </button>
    <script>
      (() => {
      const buttonEl =
        document.querySelector('#id_486e36d0-5162-49b4-b3a5-96a80a90cefa button.colab-df-generate');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      buttonEl.onclick = () => {
        google.colab.notebook.generateWithVariable('ipca_gp');
      }
      })();
    </script>
  </div>

    </div>
  </div>





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




```python
# Opção interativa
# Plota a contribuição de cada grupo com plotly

# import plotly.express as px

# px.bar(ipca_gp_wider,
#        x = 'date',
#        y = 'contribuicao',
#        color = 'groups',
#        color_discrete_sequence = list(colors.values())
#       )
```


<html>
<head><meta charset="utf-8" /></head>
<body>
    <div>            <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-AMS-MML_SVG"></script><script type="text/javascript">if (window.MathJax && window.MathJax.Hub && window.MathJax.Hub.Config) {window.MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script>                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script charset="utf-8" src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>                <div id="a6d089e7-eebb-466d-8c12-0d8fd059715c" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("a6d089e7-eebb-466d-8c12-0d8fd059715c")) {                    Plotly.newPlot(                        "a6d089e7-eebb-466d-8c12-0d8fd059715c",                        [{"alignmentgroup":"True","hovertemplate":"groups=1.Alimenta\u00e7\u00e3o e bebidas\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"1.Alimenta\u00e7\u00e3o e bebidas","marker":{"color":"#282f6b","pattern":{"shape":""}},"name":"1.Alimenta\u00e7\u00e3o e bebidas","offsetgroup":"1.Alimenta\u00e7\u00e3o e bebidas","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.07545837,0.02131976,0.21870584999999998,0.35010252000000003,0.04792704,0.07635605999999999,0.00201194,0.15636972,0.45954083999999995,0.39530838999999995,0.5257419,0.3660438,0.21538218,0.05745033,0.02749981,0.0839492,0.09242067999999999,0.08996889999999999,0.12540300000000001,0.2894953,0.21355536,0.24461774999999997,-0.0083576,0.17380776,0.22991097000000005,0.26661888,0.5054460399999999,0.4336609,0.10204992,0.17008800000000002,0.27675439999999996,0.05211696,-0.11144316000000001,0.15697944,0.11569635,0.14426676,0.12900055,0.03500752,0.010866300000000002,0.15330461999999997,0.034585120000000004,-0.14255736,-0.09877534,-0.18144524999999997,-0.1499094,0.06482534000000001,0.13184829,0.23315106000000005],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=2.Habita\u00e7\u00e3o\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"2.Habita\u00e7\u00e3o","marker":{"color":"#eace3f","pattern":{"shape":""}},"name":"2.Habita\u00e7\u00e3o","offsetgroup":"2.Habita\u00e7\u00e3o","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.08576975,-0.06101979,0.020211100000000003,-0.0155544,-0.03896375,0.00624192,0.1245672,0.05630472,0.05794089,0.056231279999999995,0.06838744000000001,0.44559647999999996,-0.16805955,0.061998,0.1249911,0.03391256,0.2741378,0.17100490000000002,0.484685,0.10858172000000002,0.40800000000000003,0.16804736,0.16608853,0.11944044,0.02582768,0.08683362,0.18405404999999997,-0.18162138,-0.26493649999999996,0.06251844,-0.15971445,0.0151476,0.0912702,0.05217742,0.07806671999999999,0.0306476,0.050361960000000004,0.12489091999999999,0.08680017,0.07299936,0.10175625,0.10525535999999999,-0.15524912000000002,0.16870890000000002,0.0720604,0.0030727,0.07358112,0.052216519999999995],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=3.Artigos de resid\u00eancia\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"3.Artigos de resid\u00eancia","marker":{"color":"#b22200","pattern":{"shape":""}},"name":"3.Artigos de resid\u00eancia","offsetgroup":"3.Artigos de resid\u00eancia","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[-0.0026270300000000003,-0.00299384,-0.040281840000000006,-0.05051738,0.021162459999999998,0.0478933,0.033499799999999996,0.0209552,0.037537,0.05763204,0.03260862,0.06670928,0.032725580000000004,0.025265460000000003,0.02636007,0.021722129999999996,0.04775875,0.041814580000000004,0.03008382,0.03811203,0.0346923,0.04882515,0.039606590000000004,0.052724450000000006,0.07048496,0.06903424,0.02252754,0.05984442,0.02593998,0.021656800000000004,0.004719,0.01665258,-0.005196490000000001,0.01561521,-0.027170080000000003,0.02529664,0.027671699999999997,0.00435589,-0.010614509999999999,0.006618270000000001,-0.00891595,-0.01620654,0.00153812,-0.00153672,-0.022221539999999998,0.0174777,-0.015992759999999998,0.0287432],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=4.Vestu\u00e1rio\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"4.Vestu\u00e1rio","marker":{"color":"#224f20","pattern":{"shape":""}},"name":"4.Vestu\u00e1rio","offsetgroup":"4.Vestu\u00e1rio","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[-0.0219696,-0.033182149999999994,0.009451679999999999,0.0045075,-0.02625254,-0.02077958,-0.023321480000000002,-0.03467022,0.01627778,0.048693480000000004,0.0030781800000000002,0.025736389999999994,-0.00302995,0.0163951,0.012450570000000001,0.0200502,0.039305160000000006,0.05174202,0.022814910000000004,0.04371924,0.01330799,0.07661520000000001,0.0406638,0.08818654000000001,0.04640590000000001,0.03837064,0.07926282000000001,0.05497632,0.09225553,0.07420812,0.026026340000000002,0.07683416,0.08216163,0.05779994,0.0524392,0.07296608,-0.01307448,-0.011530799999999999,0.0147343,0.037398600000000004,0.02229351,0.01664075,-0.01145904,0.02568564,0.01812942,0.02149515,-0.016754849999999998,0.033312299999999996],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=5.Transportes\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"5.Transportes","marker":{"color":"#5f487c","pattern":{"shape":""}},"name":"5.Transportes","offsetgroup":"5.Transportes","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.06591328,-0.04742991,-0.18472950000000002,-0.54077534,-0.3772146,0.060603449999999996,0.15255006000000002,0.16104636,0.13826329999999998,0.23517612999999998,0.26370043000000004,0.27081000000000005,0.08165272999999999,0.45479615999999995,0.77069823,-0.01664144,0.23830645,0.08523695,0.31560520000000003,0.30479544000000003,0.38209808,0.55359028,0.7173489,0.1271418,-0.024079220000000002,0.10005322,0.65327432,0.41877323,0.29630616000000004,0.12715047,-1.00504899,-0.7223156900000001,-0.41159844,0.11851429999999999,0.16959805,0.043087799999999996,0.1123925,0.07563910000000001,0.4293132599999999,0.11550616000000001,-0.11750891999999999,-0.0838491,0.305745,0.07025114,0.2895886,0.07322735,0.05654502,0.10052208],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=6.Sa\u00fade e cuidados pessoais\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"6.Sa\u00fade e cuidados pessoais","marker":{"color":"#666666","pattern":{"shape":""}},"name":"6.Sa\u00fade e cuidados pessoais","offsetgroup":"6.Sa\u00fade e cuidados pessoais","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[-0.043306880000000006,0.09826676,0.02840187,-0.029796359999999997,-0.0135533,0.047570249999999994,0.05986112,0.068076,-0.08736128000000001,0.037738400000000005,-0.01742091,0.05306280000000001,0.04205408,0.08153868000000002,-0.00262416,0.15468571999999997,0.0996664,0.06683703,-0.08517015,-0.00515864,0.049848239999999995,0.04946877000000001,-0.07169060999999999,0.09291074999999999,0.04460148,0.05812208,0.10825320000000001,0.21620019000000001,0.12423606,0.15334956,0.060935420000000004,0.16482027,0.07292636999999999,0.14968756,0.00259506,0.20681280000000002,0.02088352,0.16383528000000003,0.10707068,0.19479664,0.12265956000000001,0.014609320000000002,0.03459352,0.07727919999999999,0.005348160000000001,0.042694720000000005,0.01068128,0.04663995],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=7.Despesas pessoais\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"7.Despesas pessoais","marker":{"color":"#b35c1e","pattern":{"shape":""}},"name":"7.Despesas pessoais","offsetgroup":"7.Despesas pessoais","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.0375655,0.03331694,-0.02473443,-0.01501262,-0.00429708,-0.00538925,-0.01181983,-0.0010696300000000002,0.00960093,0.02016261,0.00105415,0.06791720000000001,0.04047693000000001,0.017668950000000003,0.00412924,0.00102321,0.0214284,0.02941093,0.0455283,0.06442944,0.05623800000000001,0.0748785,0.05662322999999999,0.05541816000000001,0.0770601,0.06338304,0.05821942999999999,0.04688976,0.05050708,0.047624080000000006,0.10962807999999997,0.05332986,0.09466844999999999,0.0575073,0.02118102,0.06240176,0.076494,0.044383679999999995,0.03817936,0.01802574,0.06381376,0.03604032,0.03820672,0.038308560000000005,0.0454356,0.02731131,0.058678600000000004,0.04870512],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=8.Educa\u00e7\u00e3o\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"8.Educa\u00e7\u00e3o","marker":{"color":"#419391","pattern":{"shape":""}},"name":"8.Educa\u00e7\u00e3o","offsetgroup":"8.Educa\u00e7\u00e3o","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.0098376,0.22735760000000002,0.037500399999999996,0.0,0.00128196,0.0032173,-0.0077037600000000005,-0.22174341000000003,-0.00553779,-0.0024441199999999997,-0.00121116,0.028803359999999997,0.00773682,0.14741616000000002,-0.03140748,0.00238152,0.00356334,0.00294705,0.010558619999999998,0.016297680000000002,-0.00057851,0.00343176,0.00113028,0.0027994500000000006,0.01390425,0.3110745,0.008695650000000001,0.00342852,0.00226324,0.0050707799999999996,0.00336084,0.03440705,0.006833519999999999,0.0102924,0.00113874,0.0107749,0.02032956,0.35399104000000003,0.0059407,0.00531495,0.0029374500000000003,0.0035182800000000004,0.00763295,0.040521629999999996,0.0029495500000000004,0.0029434,0.001175,0.014062080000000001],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"groups=9.Comunica\u00e7\u00e3o\u003cbr\u003edate=%{x}\u003cbr\u003econtribuicao=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"9.Comunica\u00e7\u00e3o","marker":{"color":"#839c56","pattern":{"shape":""}},"name":"9.Comunica\u00e7\u00e3o","offsetgroup":"9.Comunica\u00e7\u00e3o","orientation":"v","showlegend":true,"textposition":"auto","x":["2020-01-01T00:00:00","2020-02-01T00:00:00","2020-03-01T00:00:00","2020-04-01T00:00:00","2020-05-01T00:00:00","2020-06-01T00:00:00","2020-07-01T00:00:00","2020-08-01T00:00:00","2020-09-01T00:00:00","2020-10-01T00:00:00","2020-11-01T00:00:00","2020-12-01T00:00:00","2021-01-01T00:00:00","2021-02-01T00:00:00","2021-03-01T00:00:00","2021-04-01T00:00:00","2021-05-01T00:00:00","2021-06-01T00:00:00","2021-07-01T00:00:00","2021-08-01T00:00:00","2021-09-01T00:00:00","2021-10-01T00:00:00","2021-11-01T00:00:00","2021-12-01T00:00:00","2022-01-01T00:00:00","2022-02-01T00:00:00","2022-03-01T00:00:00","2022-04-01T00:00:00","2022-05-01T00:00:00","2022-06-01T00:00:00","2022-07-01T00:00:00","2022-08-01T00:00:00","2022-09-01T00:00:00","2022-10-01T00:00:00","2022-11-01T00:00:00","2022-12-01T00:00:00","2023-01-01T00:00:00","2023-02-01T00:00:00","2023-03-01T00:00:00","2023-04-01T00:00:00","2023-05-01T00:00:00","2023-06-01T00:00:00","2023-07-01T00:00:00","2023-08-01T00:00:00","2023-09-01T00:00:00","2023-10-01T00:00:00","2023-11-01T00:00:00","2023-12-01T00:00:00"],"xaxis":"x","y":[0.006857519999999999,0.011990159999999998,0.0022830800000000003,-0.011411,0.01370544,0.04309575000000001,0.029451480000000002,0.03875146,0.008713350000000002,0.01214115,0.016659919999999998,0.022272120000000003,0.00113128,-0.00733681,-0.00391279,0.0044276,0.01159641,-0.006585720000000001,0.006543719999999999,0.01243955,0.0037620799999999997,0.02870694,0.00475119,0.017797300000000002,0.05475015,0.015200639999999998,-0.0026019999999999997,0.004095280000000001,0.03649392,0.0081312,0.0035397600000000003,-0.0560318,-0.10516064,-0.023831039999999998,-0.006876520000000001,0.024425500000000003,0.10198155,0.04856389999999999,0.024815,0.00396232,0.01034523,-0.00689556,0.0,-0.0044244,-0.00539077,-0.00927675,-0.024306,0.00192912],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"date"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"contribuicao"}},"legend":{"title":{"text":"groups"},"tracegroupgap":0},"margin":{"t":60},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('a6d089e7-eebb-466d-8c12-0d8fd059715c');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                            </script>        </div>
</body>
</html>

