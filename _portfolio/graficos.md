---
title: "Construindo gráficos com o Plotnine"
excerpt: "Neste exercício, vamos criar gráficos com a biblioteca plotnine do Python <br/><img src='/images/posts/graficos/output_9_0.png'>"
date: 18/04/2024
---

# Introdução

A etapa de visualização de dados refere-se a uma parte fundamental da análise de dados, pois permite não somente compreender os dados que estamos analisando, mas como também é uma ferramenta útil para explanar os resultados encontrados. Mas qual a forma mais fácil de construir um gráfico, como podemos melhorar a produtividade nesta tarefa? É onde podemos aplicar a gramática dos gráficos e construir com a biblioteca plotnine no Python.

# Gramática dos Gráficos

A ideia da "gramática dos gráficos" foi popularizada por Leland Wilkinson em seu livro "The Grammar of Graphics", publicado em 1999. Neste livro, Wilkinson apresenta uma abordagem teórica para a criação de gráficos estatísticos, propondo uma estrutura unificada para representar uma ampla variedade de gráficos estatísticos e de visualização de dados.

O termo se refere aos elementos e princípios fundamentais que constituem um gráfico. Assim como a gramática de uma língua ajuda a organizar e comunicar informações de maneira clara e coerente, a gramática dos gráficos faz o mesmo para representações visuais de dados.

Para construir um gráfico, conforme Wilkinson, deve-se basear nos seguintes componentes: 

  - Dados: Todo gráfico começa com dados. A escolha dos dados certos é fundamental para a criação de uma visualização significativa.

  - Atributos visuais (aesthetics): A gramática dos gráficos identifica esses atributos e como eles podem ser mapeados para os dados.
  
  - Geometrias: As geometrias são as formas visuais usadas para representar os dados, como pontos, linhas, barras e áreas.
    
  - Sinais estatísticos: Sinais estatísticos, como médias, medianas e desvios padrão, podem ser adicionados aos gráficos para fornecer informações adicionais sobre os dados.       
    
  - Facetas: As facetas são usadas para dividir os dados em subconjuntos e representá-los em painéis separados. Isso pode ser útil para explorar padrões em dados multidimensionais.

  - Coordenadas: As coordenadas definem como os dados são mapeados nos elementos visuais. Deve ser cartersiano ou polar?
  
  - Temas: Descreve toda a parte não relacionada aos dados do gráficos.

## ggplot2 vs. plotnine

A gramática dos gráficos ganhou grande destaque em aplicações de análise de dados, e é impossível não mencionar que foi popularizada por Hadley Wickham com a criação do pacote ggplot2 na linguagem de programação R.

O ggplot2 revolucionou a visualização de dados na linguagem, seguindo a abordagem da gramática dos gráficos, onde os gráficos são construídos a partir de camadas (layers), geometrias (geoms) e mapeamentos estéticos (aesthetics).

Na comunidade do Python, houve a criação de bibliotecas bastante conhecidas, como matplotlib e seaborn. Entretanto, elas não foram tão difundidas no contexto da aplicação da gramática dos gráficos. Esse papel foi desempenhado pelo plotnine, uma biblioteca inspirada (ou mesmo uma cópia perfeita para o Python) do ggplot2.

O plotnine permite a construção de gráficos adicionando camadas, geometrias e mapeamentos estéticos. Isso facilita a criação de visualizações complexas e personalizadas.

## Passo 01: Instalar bibliotecas

Vamos agora realizar o procedimento de construir gráficos usando o plotnine no Python. Para tanto, é necessário instalar e importar os módulos da biblioteca.


```python
# !pip install plotnine
import plotnine as p9
```

## Passo 02: dados de exemplo

Vamos importar os dados `mtcars` que contém várias características de carros, como cilindros, cavalos de potência, consumo de combustível e muito mais.


```python
# Importa os dados do módulo data
from plotnine.data import mtcars

# Exibe o dataframe
mtcars.head()
```




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
      <th>name</th>
      <th>mpg</th>
      <th>cyl</th>
      <th>disp</th>
      <th>hp</th>
      <th>drat</th>
      <th>wt</th>
      <th>qsec</th>
      <th>vs</th>
      <th>am</th>
      <th>gear</th>
      <th>carb</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mazda RX4</td>
      <td>21.0</td>
      <td>6</td>
      <td>160.0</td>
      <td>110</td>
      <td>3.90</td>
      <td>2.620</td>
      <td>16.46</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mazda RX4 Wag</td>
      <td>21.0</td>
      <td>6</td>
      <td>160.0</td>
      <td>110</td>
      <td>3.90</td>
      <td>2.875</td>
      <td>17.02</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Datsun 710</td>
      <td>22.8</td>
      <td>4</td>
      <td>108.0</td>
      <td>93</td>
      <td>3.85</td>
      <td>2.320</td>
      <td>18.61</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hornet 4 Drive</td>
      <td>21.4</td>
      <td>6</td>
      <td>258.0</td>
      <td>110</td>
      <td>3.08</td>
      <td>3.215</td>
      <td>19.44</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hornet Sportabout</td>
      <td>18.7</td>
      <td>8</td>
      <td>360.0</td>
      <td>175</td>
      <td>3.15</td>
      <td>3.440</td>
      <td>17.02</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Passo 03: criando o gráfico com o plotnine

Com os dados em mãos, devemos definir cada etapa do processo de criação do gráfico conforme a gramática dos gráficos usando o plotnine. Primeiro, devemos abrir o parêntese `()` e definir toda a construção do gráfico em seu interior. Definimos os dados, inserindo com a função principal `p9.ggplot()` o dataframe `mtcars` e mapeamos os eixos x e y com a função `p9.aes()`. Lembramos que aqui as camadas são "somadas", ou seja, juntadas pelo símbolo +.


```python
(p9.ggplot(mtcars) + 
  p9.aes(x = "wt", y = "mpg")
  )
```


    
![](/images/posts/graficos/output_5_0.png)
    


Veja que o resultado está mapeando os dados; entretanto, ainda faltam alguns elementos. Para adicionar a geometria, aplicamos a função característica daquela que desejamos. Vamos aplicar um gráfico de dispersão com `geom_point()`.


```python
(p9.ggplot(mtcars) + 
  p9.aes(x = "wt", y = "mpg") +
  p9.geom_point()
  )
```

![png](/images/posts/graficos/output_7_0.png)
    


Podemos continuar definindo elementos estatísticos adicionais. No caso, construímos uma reta de regressão com a camada `stat_smooth(method="lm")`. Também realizamos o facetamento dos dados com `facet_wrap` através da coluna "gear". Para auxiliar na diferenciação das variáveis, adicionamos as cores de cada variável com a função `aes`. Para finalizar, aplicamos um tema pré-definido, `theme_minimal`, e inserimos um título com a função `labs`.


```python
(p9.ggplot(mtcars) + 
  p9.aes(x = "wt", y = "mpg", color = "factor(gear)") +
  p9.geom_point() +
  p9.stat_smooth(method = "lm") +
  p9.facet_wrap("gear") +
  p9.theme_minimal() +
  p9.labs(title = "Gráfico do dataset mtcars", 
          subtitle = "Criado com o plotnine no Python")
  )
```


    
![png](/images/posts/graficos/output_9_0.png)
    


## Referências

Wilkinson, L. (2005), The Grammar of Graphics , Springer .


Kibirige, Hassan. (2024) Plotnine: A Grammar of Graphics for Python. Acesso em: https://plotnine.org/
