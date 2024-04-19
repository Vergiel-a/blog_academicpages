---
title: "Lendo as cartas dos gestores com IA usando Python"
excerpt: "Neste artigo mostramos como usar um modelo LLM para ler e sumarizar as Cartas do Gestor, publicadas em fundos de investimento no Brasil. Em menos de 60 linhas de código, criamos uma aplicação completa em Python que pega um arquivo PDF da carta do gestor, processa os dados e usa inteligência artificial para sumarizar os pontos chave do texto. <br/><img src='/images/portfolio/cartas/app.png'>"
date: 18/04/2024
---

**autor: Fernando da Silva**

O que os gestores de fundos de investimentos pensam sobre os mercados e sobre a economia? Todo mês, dezenas de gestores publicam suas cartas relatando sua visão, desempenho e outras informações relevantes para os cotistas. No entanto, ler estes relatórios pode facilmente tomar 1 dia inteiro ou mais de trabalho. Para economizar este tempo, podemos utilizar inteligência artificial e sumarizar os principais tópicos do documento em poucos segundos!

Neste artigo mostramos como usar um modelo LLM para ler e sumarizar as "Cartas do Gestor", publicadas em fundos de investimento no Brasil. Em menos de 60 linhas de código, criamos uma aplicação completa em Python que pega um arquivo PDF da carta do gestor, processa os dados e usa inteligência artificial para sumarizar os pontos chave do texto.

![](/images/portfolio/cartas/app.png)

## Passo 01: bibliotecas de Python necessárias

Para criar o app, utilizaremos as seguintes bibliotecas no Python:

-   [**shiny**](https://analisemacro.com.br/curso/producao-de-dashboards-automaticos-usando-python/) para desenvolver o app/dashboard.
-   **langchain** para processar os dados e o prompt de LLM.
-   **langchain_community** para ler o PDF.
-   **langchain_openai** para usar a API do ChatGPT (OpenAI).
-   **os** para gerenciar chave de token da API da OpenAI.
-   **pypdf** para trabalhar com PDFs.

Certifique-se de ter todas as bibliotecas instaladas.


```python
# Bibliotecas ----
from shiny import App, reactive, render, ui
from langchain.chains.combine_documents.stuff import StuffDocumentsChain
from langchain.chains.llm import LLMChain
from langchain.prompts import PromptTemplate
from langchain_community.document_loaders import PyPDFLoader
from langchain_openai import ChatOpenAI
import os
```

## Passo 02: chave de autenticação da API da OpenAI

Para utilizar o modelo de inteligência artificial ChatGPT, é necessário obter uma chave de token na plataforma OpenAI (veja mais informações [aqui](https://analisemacro.com.br/data-science/como-integrar-o-chatgpt-no-r-e-no-python/)).

Com a chave de token em mãos, defina uma variável de ambiente com nome `OPENAI_API_KEY` para armazenar a chave.


```python
# Define chave de autenticação na API OpenAI
os.environ["OPENAI_API_KEY"] = "sk-XXXXXXXXXXXXXXXX" # COLOQUE AQUI SUA CHAVE
```

**Atenção**: a utilização da API da OpenAI não é gratuita. Verifique o site da empresa para mais informações.

## Passo 03: interface visual do app/dashboard

Para simplificar o exemplo, vamos criar o app com um visual simples, contendo apenas:

-   Título do app
-   Descrição breve sobre a funcionalidade
-   Campo de texto para inserir o link do PDF
-   Painel para exibir o sumário do PDF gerado pela inteligência artificial

Tudo isso pode ser feito em cerca de 12 linhas de código utilizando a biblioteca [Shiny](https://analisemacro.com.br/curso/producao-de-dashboards-automaticos-usando-python/).


```python
# Interface do Usuário ----
app_ui = ui.page_fluid(
    ui.h1(ui.strong("App Cartas do Gestor")),
    ui.markdown(("Este app utiliza Inteligência Artificial para sumarizar os pontos " +
                 "chaves da carta do gestor de um fundo de investimento. Basta " +
                 "inserir o link do PDF no campo abaixo.")),
    ui.input_text(
        id = "url_pdf", 
        label = "Link PDF:", 
        value = "https://aluno.analisemacro.com.br/download/59224/?tmstv=1710776237",
        placeholder = "https://aluno.analisemacro.com.br/download/59224/?tmstv=1710776237",
        width = '100%'
        ),
    ui.output_ui("sumario"))
```

## Passo 04: conversão de PDF para texto

Para que a carta do gestor possa ser lida pela inteligência artificial, precisamos converter o conteúdo do arquivo PDF para o formato de texto. Fazemos isso da seguinte forma:

-   Definimos o componente Servidor do [Shiny](https://analisemacro.com.br/curso/producao-de-dashboards-automaticos-usando-python/) para implementar a lógica de funcionamento do app.
-   Definimos uma função reativa para ler o arquivo PDF informado pelo usuário através do link e converter para o formato de texto.


```python
# Servidor ----
def server(input, output, session):
    # Função para importar o texto de um PDF a partir de um link (URL)
    @reactive.Calc
    def pdf_para_texto():
        pdf = PyPDFLoader(input.url_pdf())
        texto = pdf.load_and_split()
        return texto
```

## Passo 05: engenharia de prompt e modelo de inteligência artificial

Ainda no componente Servidor, iniciado no passo anterior, definimos qual será a instrução enviada para a inteligência artificial e qual modelo cumprirá esta tarefa. Estes procedimentos são realizados da seguinte forma:

-   Utilizamos o [prompt](https://analisemacro.com.br/data-science/introducao-a-prompt-engineering-para-inteligencia-artificial/) a seguir para instruir o modelo de inteligência artificial a sumarizar o texto fornecendo uma breve descrição sobre o mesmo.

    | *Escreva um sumário do texto a seguir delimitado com 3 crases. Retorne sua resposta em 3 ou 5 marcadores com uma breve descrição dos pontos chave do texto.*
    | *\`\`\`{TEXTO}\`\`\`*
    | *SUMÁRIO:*

-   Definimos o `gpt-3.5-turbo`, da OpenAI, como modelo de inteligência artificial a ser utilizado.

-   Utilizamos o **langchain** para processar automaticamente o prompt e o texto, além de fazer a comunicação com a API da OpenAI para retornar a resposta do modelo.


```python
    # Função para sumarizar o texto com um prompt pré definido usando GPT 3.5}
    @render.ui
    def sumario():
        prompt = PromptTemplate.from_template(
            """Escreva um sumário do texto a seguir delimitado com 3 crases.
            Retorne sua resposta em 3 ou 5 marcadores com uma breve descrição dos
            pontos chave do texto.

            ```{TEXTO}```

            SUMÁRIO:""")

        llm = ChatOpenAI(temperature = 0, model_name = "gpt-3.5-turbo")
        llm_chain = LLMChain(llm = llm, prompt = prompt)

        modelo = StuffDocumentsChain(
            llm_chain = llm_chain,
            document_variable_name = "TEXTO"
            )

        resultado = modelo.run(pdf_para_texto())
        return ui.TagList(ui.h3("Sumário:"), ui.HTML(ui.markdown(resultado)))
```

## Passo 06: execução e teste do app

Para rodar o app/dashboard localmente e testar o funcionamento com diferentes cartas do gestor, utilizamos o Shiny. A biblioteca se encarrega de unir as definições da interface e da lógica de servidor para gerar a dashboard que pode ser visualizada pelo navegador, conforme abaixo:


```python
# Dashboard
app = App(app_ui, server)
```

![](/images/portfolio/cartas/app.png)

## Conclusão

Neste artigo mostramos como usar um modelo LLM para ler e sumarizar as "Cartas do Gestor", publicadas em fundos de investimento no Brasil. Em menos de 60 linhas de código, criamos uma aplicação completa em Python que pega um arquivo PDF da carta do gestor, processa os dados e usa inteligência artificial para sumarizar os pontos chave do texto.
