---
layout: post
title: Viajou - Uma Análise Exploratória das Viagens a Serviço do Governo Federal
date: 2019-09-20 00:00:00 +0300
description:  # Add post description (optional)
img: treeplot.png # Add image post (optional)
tags: [pandas, dados_abertos, EDA] # add tag
---
---

#### Resumo -> análise exploratória dos dados das viagens realizadas a serviço por servidores federais através dos dados disponibilizados pelo portal da transparência. 

---
Apesar do prórprio site já oferecer uma seção com algumas visualizações interessantes, proponho o exercício de coletá-os em formato csv e analisá-los utilizando o python e algumas de duas bibliotecas. 

Quanto cada órgão federal gasta com viagens por ano? 
Qual órgão viaja mais? Qual servidor viaja mais? 
Qual foi a passagem mais cara? E a diária? 

Podemos responder essas perguntas acessando os dados disponibilizados pelo portal da transparência do governo federal. UTILIZAMOS DADOS EM 2019

fonte: http://www.portaltransparencia.gov.br/download-de-dados/viagens



### Índice

1. Extração e limpeza 
2. Análise 
3. Visualização
4. Conclusão 

### Extração e limpeza

Criei um script que acessa o site do portal da transparência e faz o download do arquivo em formato zip. Como a periodicidade de atualização é  diária, é conveniente ter um script que baixei os dados de forma simples sempre que eu quiser atualizar a análise. Em um segundo momento posso até escrever uma rotina em bash para que o arquivo rode todo dia e mantenha a base sempre atualizada. 


```python
import requests

print('Iniciando o download dos dados')

url = 'http://www.portaltransparencia.gov.br/download-de-dados/viagens/2019'
r = requests.get(url)

with open('/home/blackmamba/Documents/analise_dados/viagens_gov_federal/dados/2019_Viagens.zip', 'wb') as f:
    f.write(r.content)

print('Download concluído com sucesso')
```

    Iniciando o download dos dados
    Download concluído com sucesso


Com os dados baixados no meu computador, inicio a análise importanto as bibliotecas necessárias e subindo os dados para o jupyter. 


```python
#@pbragamiranda
#10-2019
#carrega as bibliotecas necessárias
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib
import matplotlib.pyplot as plt
import zipfile
%matplotlib inline

#define as opções de display
pd.set_option("display.precision", 2)  # casas decimais
pd.set_option('display.max_colwidth', 60)  #-1 pra ver tudo

#sobe os dados
zf = zipfile.ZipFile('/home/blackmamba/Documents/analise_dados/viagens_gov_federal/dados/2019_Viagens.zip') 
viagens = pd.read_csv(zf.open("2019_Viagem.csv"), sep=";", encoding="latin1")
```

Em seguida conferimos o tipo dos dados para identificar se há necessidade de alterações.


```python
#confere as colunas presentes e seus respectivos tipos
viagens.dtypes
```




    Identificador do processo de viagem     int64
    Situação                               object
    Código do órgão superior                int64
    Nome do órgão superior                 object
    Código órgão solicitante                int64
    Nome órgão solicitante                 object
    CPF viajante                           object
    Nome                                   object
    Cargo                                  object
    Período - Data de início               object
    Período - Data de fim                  object
    Destinos                               object
    Motivo                                 object
    Valor diárias                          object
    Valor passagens                        object
    Valor outros gastos                    object
    dtype: object



Quase todas as colunas estão sendo lidas como object. Precisamos converter as colunas para que possamos trabalhar com os dados. O que for referente aos valores transformaremos para número (float ou int), o que data para data (datetime) e o resto deixaremos como objeto (object) mesmo. 

Importante lembrar também que a notação matemática no Brasil difere da estados unidense, por isso é necessário substituir a vírgula pelo ponto.


```python
#primeiro tratamento dos dados
#transforma vírgula em ponto e depois transforma em float
viagens["Valor passagens"] = viagens["Valor passagens"].str.replace(',' , '.').astype(float)
viagens["Valor diárias"] = viagens["Valor diárias"].str.replace(',' , '.').astype(float)
viagens["Valor outros gastos"] = viagens["Valor outros gastos"].str.replace(',' , '.').astype(float)

#transforma colunas em objeto
viagens["Identificador do processo de viagem"] = viagens["Identificador do processo de viagem"].astype(str)
viagens["Identificador do processo de viagem"] = viagens["Identificador do processo de viagem"].astype(str)

#transforma colunas em datetime
viagens['Período - Data de início'] = pd.to_datetime(viagens['Período - Data de início'], format='%d/%m/%Y')
viagens['Período - Data de fim'] = pd.to_datetime(viagens['Período - Data de fim'], format='%d/%m/%Y')
```

### Análise

Agora que já aplicamos o tratamento inicial aos dados podemos começar as análises. Primeiro vamos olhar a cara dos dados. 


```python
viagens.head(3)
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
      <th>Identificador do processo de viagem</th>
      <th>Situação</th>
      <th>Código do órgão superior</th>
      <th>Nome do órgão superior</th>
      <th>Código órgão solicitante</th>
      <th>Nome órgão solicitante</th>
      <th>CPF viajante</th>
      <th>Nome</th>
      <th>Cargo</th>
      <th>Período - Data de início</th>
      <th>Período - Data de fim</th>
      <th>Destinos</th>
      <th>Motivo</th>
      <th>Valor diárias</th>
      <th>Valor passagens</th>
      <th>Valor outros gastos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15045825</td>
      <td>Realizada</td>
      <td>26000</td>
      <td>Ministério da Educação</td>
      <td>26291</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Ní...</td>
      <td>***.377.624-**</td>
      <td>MARINA FERREIRA KITAZONO ANTUNES</td>
      <td>NaN</td>
      <td>2019-02-06</td>
      <td>2019-02-07</td>
      <td>Recife/PE</td>
      <td>Regresso de bolsista CAPES do exterior- PE ( PROGRAMAS E...</td>
      <td>0.0</td>
      <td>3406.33</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15100682</td>
      <td>Realizada</td>
      <td>26000</td>
      <td>Ministério da Educação</td>
      <td>26291</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Ní...</td>
      <td>***.831.975-**</td>
      <td>JORGE ANDRE DE CARVALHO MENDONCA</td>
      <td>NaN</td>
      <td>2019-02-01</td>
      <td>2019-02-02</td>
      <td>Recife/PE</td>
      <td>Capacitação PDSE (Programa de Doutorado Sanduíche no Ext...</td>
      <td>0.0</td>
      <td>2925.83</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15114708</td>
      <td>Realizada</td>
      <td>26000</td>
      <td>Ministério da Educação</td>
      <td>26291</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Ní...</td>
      <td>***.325.718-**</td>
      <td>MARCO ANTONIO COUTO JUNIOR</td>
      <td>PESQUISADOR EM GEOCIENCIA</td>
      <td>2019-02-01</td>
      <td>2019-02-01</td>
      <td>São Paulo/SP</td>
      <td>Capacitação no exterior - PDSE</td>
      <td>0.0</td>
      <td>2760.02</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



Podemos notar que existe uma coluna referente a situação da viagem. Como só nos interessa as viagens realizadas, vamos conferir quais opções existem para situação e selecionar somente o que nos interessa. 

Depois vamos agrupar por órgão superior para identificar quais orgãos viajaram mais e gastaram mais com viagens. 


```python
#confere as possibilidades da situação
viagens.Situação.unique()
```




    array(['Realizada', 'Não realizada'], dtype=object)




```python
#seleciona somente as realizadas
viagens_r = viagens[viagens.Situação == "Realizada"]

#agrupa por órgão superior, conta e soma
agrupado = (viagens_r.groupby('Nome do órgão superior')
            .agg({'Código do órgão superior' : 'count', 'Valor passagens' : 'sum'})
            .reset_index())
agrupado.rename(columns={'Código do órgão superior': 'Qtd Viagens' , 'Valor passagens' : 'Valor total passagens' }, inplace=True)

#calcula preço médio por viagem
agrupado['valor médio viagem'] = agrupado['Valor total passagens'] / agrupado['Qtd Viagens']
```


```python
#mostra resultado
(agrupado.sort_values(by='Valor total passagens', ascending=False)
 .style.format({'Valor total passagens' : 'R${0:,.0f}', 'valor médio viagem' : 'R${0:,.0f}'})
 .hide_index()
 .highlight_max(subset='valor médio viagem',color='green')
 .highlight_min(subset='valor médio viagem',color='#cd4f39')
) 
```




<style  type="text/css" >
    #T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow21_col3 {
            background-color:  green;
            : ;
        }    #T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow24_col3 {
            : ;
            background-color:  #cd4f39;
        }</style><table id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757f" ><thead>    <tr>        <th class="col_heading level0 col0" >Nome do órgão superior</th>        <th class="col_heading level0 col1" >Qtd Viagens</th>        <th class="col_heading level0 col2" >Valor total passagens</th>        <th class="col_heading level0 col3" >valor médio viagem</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow0_col0" class="data row0 col0" >Ministério da Educação</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow0_col1" class="data row0 col1" >138853</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow0_col2" class="data row0 col2" >R$65,140,270</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow0_col3" class="data row0 col3" >R$469</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow1_col0" class="data row1 col0" >Ministério da Defesa</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow1_col1" class="data row1 col1" >78060</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow1_col2" class="data row1 col2" >R$46,645,726</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow1_col3" class="data row1 col3" >R$598</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow2_col0" class="data row2 col0" >Sem informação</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow2_col1" class="data row2 col1" >39809</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow2_col2" class="data row2 col2" >R$32,105,065</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow2_col3" class="data row2 col3" >R$806</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow3_col0" class="data row3 col0" >Ministério da Justiça e Segurança Pública</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow3_col1" class="data row3 col1" >63277</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow3_col2" class="data row3 col2" >R$27,848,772</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow3_col3" class="data row3 col3" >R$440</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow4_col0" class="data row4 col0" >Ministério da Saúde</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow4_col1" class="data row4 col1" >29686</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow4_col2" class="data row4 col2" >R$20,769,533</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow4_col3" class="data row4 col3" >R$700</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow5_col0" class="data row5 col0" >Ministério da Economia</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow5_col1" class="data row5 col1" >55256</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow5_col2" class="data row5 col2" >R$19,125,884</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow5_col3" class="data row5 col3" >R$346</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow6_col0" class="data row6 col0" >Ministério das Relações Exteriores</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow6_col1" class="data row6 col1" >3199</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow6_col2" class="data row6 col2" >R$12,366,786</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow6_col3" class="data row6 col3" >R$3,866</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow7_col0" class="data row7 col0" >Ministério da Infraestrutura</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow7_col1" class="data row7 col1" >13877</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow7_col2" class="data row7 col2" >R$12,362,585</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow7_col3" class="data row7 col3" >R$891</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow8_col0" class="data row8 col0" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow8_col1" class="data row8 col1" >8468</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow8_col2" class="data row8 col2" >R$10,895,048</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow8_col3" class="data row8 col3" >R$1,287</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow9_col0" class="data row9 col0" >Ministério da Agricultura, Pecuária e Abastecimento</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow9_col1" class="data row9 col1" >31631</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow9_col2" class="data row9 col2" >R$10,483,097</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow9_col3" class="data row9 col3" >R$331</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow10_col0" class="data row10 col0" >Presidência da República</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow10_col1" class="data row10 col1" >8246</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow10_col2" class="data row10 col2" >R$9,406,747</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow10_col3" class="data row10 col3" >R$1,141</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow11_col0" class="data row11 col0" >Ministério do Meio Ambiente</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow11_col1" class="data row11 col1" >19586</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow11_col2" class="data row11 col2" >R$9,008,210</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow11_col3" class="data row11 col3" >R$460</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow12_col0" class="data row12 col0" >Ministério de Minas e Energia</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow12_col1" class="data row12 col1" >3727</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow12_col2" class="data row12 col2" >R$7,018,544</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow12_col3" class="data row12 col3" >R$1,883</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow13_col0" class="data row13 col0" >Ministério da Cidadania</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow13_col1" class="data row13 col1" >3475</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow13_col2" class="data row13 col2" >R$2,980,346</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow13_col3" class="data row13 col3" >R$858</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow14_col0" class="data row14 col0" >Ministério do Desenvolvimento Regional</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow14_col1" class="data row14 col1" >5197</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow14_col2" class="data row14 col2" >R$2,955,565</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow14_col3" class="data row14 col3" >R$569</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow15_col0" class="data row15 col0" >Controladoria-Geral da União</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow15_col1" class="data row15 col1" >2471</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow15_col2" class="data row15 col2" >R$1,930,138</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow15_col3" class="data row15 col3" >R$781</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow16_col0" class="data row16 col0" >Ministério da Mulher, Família e Direitos Humanos</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow16_col1" class="data row16 col1" >4859</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow16_col2" class="data row16 col2" >R$1,686,921</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow16_col3" class="data row16 col3" >R$347</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow17_col0" class="data row17 col0" >Advocacia-Geral da União</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow17_col1" class="data row17 col1" >5329</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow17_col2" class="data row17 col2" >R$1,405,677</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow17_col3" class="data row17 col3" >R$264</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow18_col0" class="data row18 col0" >Ministério da Indústria, Comércio Exterior e Serviços</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow18_col1" class="data row18 col1" >240</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow18_col2" class="data row18 col2" >R$887,885</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow18_col3" class="data row18 col3" >R$3,700</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow19_col0" class="data row19 col0" >Ministério do Planejamento, Desenvolvimento e Gestão</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow19_col1" class="data row19 col1" >532</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow19_col2" class="data row19 col2" >R$714,835</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow19_col3" class="data row19 col3" >R$1,344</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow20_col0" class="data row20 col0" >Ministério do Trabalho e Emprego</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow20_col1" class="data row20 col1" >2330</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow20_col2" class="data row20 col2" >R$636,132</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow20_col3" class="data row20 col3" >R$273</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow21_col0" class="data row21 col0" >Ministério do Turismo</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow21_col1" class="data row21 col1" >139</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow21_col2" class="data row21 col2" >R$555,474</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow21_col3" class="data row21 col3" >R$3,996</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow22_col0" class="data row22 col0" >Ministério do Esporte</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow22_col1" class="data row22 col1" >8</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow22_col2" class="data row22 col2" >R$25,163</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow22_col3" class="data row22 col3" >R$3,145</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow23_col0" class="data row23 col0" >Ministério da Cultura</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow23_col1" class="data row23 col1" >2</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow23_col2" class="data row23 col2" >R$5,256</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow23_col3" class="data row23 col3" >R$2,628</td>
            </tr>
            <tr>
                                <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow24_col0" class="data row24 col0" >Ministério das Cidades</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow24_col1" class="data row24 col1" >1</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow24_col2" class="data row24 col2" >R$0</td>
                        <td id="T_ca8f8954_f1ef_11e9_96e8_a86badfc757frow24_col3" class="data row24 col3" >R$0</td>
            </tr>
    </tbody></table>



Notamos Ministério da educação é o que mais viaja bem como o que mais gasta com viagem. Faz sentido enxergar educação, defesa e segurança pública no topo da lista devido ao número de servidores que esses três ministérios englobam. Apesar de gastarem mais, ao analisarmos o preço médio por viagem vemos que as passagens médias mais caras pertencem a outros órgãos.

Os campeões das passagens médias mais caras são Min. do Turismo (3996), Min. Relações Exteriores (3866)  e Min. da Indústria (3700). Também faz sentindo pois os três ministérios devem realizar mais viagens internacionais que os demais.   

Vamos fazer um gráfico para auxiliar a leitura dos dados. 


```python
#gráfico em barra para visualizar órgãos que mais gastaram
plt.style.use('fivethirtyeight')
plt.figure(figsize=(20,10))  
ax = sns.barplot(x='Valor total passagens', y='Nome do órgão superior', data=agrupado) #valor médio viagem
```


![png]({{site.baseurl}}/assets/img/viagens/barplot.png)



```python

```
