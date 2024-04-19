# Analisando a inflação por faixa de renda no Python

**autor: Fernando da Silva**

Neste artigo mostramos como coletar dados de inflação segmentados por faixa de renda e como calcular a variação acumulada em 12 meses usando a linguagem de programação Python.

<hr>

Existem diversas maneiras de analisar a taxa de inflação de uma economia, dentre elas:

-   Análise do índice cheio;

-   Análise dos grupos do índice;

-   Análise regional;

-   Análise dos núcleos;

-   Análise de difusão;

-   Análise das classficações de subitens.

Apesar de todas estas análises permitirem investigar a evolução do custo de vida dos brasileiros, elas colocam o cobrador de ônibus e o gerente de banco em uma sacola só. Em outras palavras, perfis diferentes de renda podem estar expostos a custos de vida diferentes, o que faz com que uma análise por faixa de renda seja relevante.

## Indicador IPEA de inflação por faixa de renda

O IPEA calcula desde 2017 indicadores mensais de inflação por faixa de renda[^1]. No total são 6 faixas de renda analisadas:

[^1]: Veja a metodologia em Lameiras et al. (2017).

| Faixa de renda    | Renda domiciliar (R\$ jan./2023)     |
|-------------------|--------------------------------------|
| Renda muito baixa | Menor que R\$ 2.015,18               |
| Renda baixa       | Entre R\$ 2,015,18 e R\$ 3.022,76    |
| Renda média-baixa | Entre R\$ 3.022,76 e R\$ 5.037,94    |
| Renda média       | Entre R\$ 5.037,94 e R\$ 10.075,88   |
| Renda média-alta  | Entre R\$ 10.075,88 e R\$ 20.151,75  |
| Renda alta        | Maior que R\$ 20.151,76              |

Estes indicadores segmentados permitem comparar o custo de vida, em termos de inflação, das faixas de renda, além de possibilitar a identificação dos itens que mais contribuiram para uma faixa ou outra.

## Coletando dados do IPEA no Python

Para coletar e analisar dados de inflação por faixa de renda na variação acumulada em 12 meses, podemos utilizar o código de Python abaixo:


```python
#!pip install ipeadatapy
```


```python
# Carregar bibliotecas
import ipeadatapy as ipea
import pandas as pd

# Códigos das séries no banco de dados
indicadores = {
  "DIMAC_INF1": "Renda muito baixa",
  "DIMAC_INF2": "Renda baixa",
  "DIMAC_INF3": "Renda média-baixa",
  "DIMAC_INF4": "Renda média",
  "DIMAC_INF5": "Renda média-alta",
  "DIMAC_INF6": "Renda alta"
}

# Coleta dados
dados_brutos = list()
for codigo in indicadores.keys():
  dados_brutos.append(ipea.timeseries(codigo))

# Tratamento de dados
dados = (
  pd.concat(dados_brutos)
  .replace(indicadores)
  .pivot(columns = "CODE", values = "VALUE ((% a.m.))")
  .rolling(window = 12)
  .apply(lambda x: ((x / 100 + 1).prod() - 1) * 100, raw = False)
  .rename_axis("", axis="columns")
)

# Visualiza dados
dados.plot(
  figsize = (10, 5),
  title = "Inflação por faixa de renda\nVar. %, acum. 12 meses, IPEA",
  xlabel = ""
)
```




    <Axes: title={'center': 'Inflação por faixa de renda\nVar. %, acum. 12 meses, IPEA'}>




    
![png](/images/portfolio/inflacao_faixa_renda/output_3_1.png)
    


## Conclusão

Neste artigo mostramos como coletar dados de inflação segmentados por faixa de renda e como calcular a variação acumulada em 12 meses usando a linguagem de programação Python.

## Referências

Lameiras, M. A. P.; Sacchet, S.; Souza-Júnior, J. R.C. Indicador Ipea de Inflação por Faixa de Renda. Carta de Conjuntura, n. 37, 16 nov. 2017. Disponível em: \<http://www.ipea.gov.br/cartadeconjuntura/index.php/2017/11/16/inflacao-por-faixa-de-renda/\>.
