---

Resumo: análise exploratória das viagens realizadas a serviço por servidores federais através dos dados disponibilizados pelo portal da transparência. 

---
Apesar do prórprio portal da transparêcia já oferecer uma seção com algumas visualizações interessantes, proponho o exercício de coletá-os em formato csv e analisá-los utilizando o python e algumas de suas bibliotecas. 

Quanto cada órgão federal gasta com viagens por ano?
Qual órgão viaja mais? Qual servidor viaja mais? 
Qual foi a passagem mais cara? E a diária? 

Podemos responder essas perguntas acessando os dados disponibilizados pelo portal da transparência do governo federal. UTILIZAMOS DADOS EM 2019

fonte: http://www.portaltransparencia.gov.br/download-de-dados/viagens
    
dicionário dos dados: http://www.portaltransparencia.gov.br/pagina-interna/603364-dicion%C3%A1rio-de-dados-viagens-a-Servi%C3%A7o-Pagamentos

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
pd.set_option('display.max_colwidth', -1)  #-1 pra ver tudo

#sobe os dados
zf = zipfile.ZipFile('/home/blackmamba/Documents/analise_dados/viagens_gov_federal/dados/2019_Viagens.zip') 
viagens = pd.read_csv(zf.open("2019_Viagem.csv"), sep=";", encoding="latin1")
```

Em seguida conferimos o tipo dos dados para identificar se há necessidade de alterações.


```python
#confere as colunas presentes e seus respectivos tipos
viagens.dtypes
```




    Identificador do processo de viagem    int64 
    Situação                               object
    Código do órgão superior               int64 
    Nome do órgão superior                 object
    Código órgão solicitante               int64 
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

#seleciona só as colunas que nos interessam
viagens = viagens[['Identificador do processo de viagem','Situação', 'Nome do órgão superior', 'Nome órgão solicitante', 'Nome', 'Cargo', 'Período - Data de início',
                  'Período - Data de fim', 'Destinos', 'Motivo', 'Valor diárias', 'Valor passagens', 'Valor outros gastos']]
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
      <th>Nome do órgão superior</th>
      <th>Nome órgão solicitante</th>
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
      <td>Ministério da Educação</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Nível Superior</td>
      <td>MARINA FERREIRA KITAZONO ANTUNES</td>
      <td>NaN</td>
      <td>2019-02-06</td>
      <td>2019-02-07</td>
      <td>Recife/PE</td>
      <td>Regresso de bolsista CAPES do exterior- PE ( PROGRAMAS ESTRATÉGICOS -DRI )</td>
      <td>0.0</td>
      <td>3406.33</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15100682</td>
      <td>Realizada</td>
      <td>Ministério da Educação</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Nível Superior</td>
      <td>JORGE ANDRE DE CARVALHO MENDONCA</td>
      <td>NaN</td>
      <td>2019-02-01</td>
      <td>2019-02-02</td>
      <td>Recife/PE</td>
      <td>Capacitação PDSE (Programa de Doutorado Sanduíche no Exterior).</td>
      <td>0.0</td>
      <td>2925.83</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>15114708</td>
      <td>Realizada</td>
      <td>Ministério da Educação</td>
      <td>Fundação Coordenação de Aperfeiçoamento de Pessoal de Nível Superior</td>
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



Olhando o dicionário dos dados notamos que a coluna situação possuí dois valore: realizada e não realizada.. Como só nos interessa as viagens realizadas, vamos filtrar os dados somente com o que nos interessa. 

Depois vamos agrupar por órgão superior para identificar quais orgãos viajaram mais e gastaram mais com viagens. 


```python
#seleciona somente as realizadas
viagens_r = viagens[viagens['Situação'] == "Realizada"]

#agrupa por órgão superior, conta e soma
agrupado = (viagens_r.groupby('Nome do órgão superior')
            .agg({'Nome órgão solicitante' : 'count', 'Valor passagens' : 'sum'})
            .reset_index())
agrupado.rename(columns={'Nome órgão solicitante': 'Qtd Viagens' , 'Valor passagens' : 'Valor total passagens' }, inplace=True)

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
    #T_37453446_f366_11e9_a31e_a86badfc757frow21_col3 {
            background-color:  green;
            : ;
        }    #T_37453446_f366_11e9_a31e_a86badfc757frow24_col3 {
            : ;
            background-color:  #cd4f39;
        }</style><table id="T_37453446_f366_11e9_a31e_a86badfc757f" ><thead>    <tr>        <th class="col_heading level0 col0" >Nome do órgão superior</th>        <th class="col_heading level0 col1" >Qtd Viagens</th>        <th class="col_heading level0 col2" >Valor total passagens</th>        <th class="col_heading level0 col3" >valor médio viagem</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow0_col0" class="data row0 col0" >Ministério da Educação</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow0_col1" class="data row0 col1" >138853</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow0_col2" class="data row0 col2" >R$65,140,270</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow0_col3" class="data row0 col3" >R$469</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow1_col0" class="data row1 col0" >Ministério da Defesa</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow1_col1" class="data row1 col1" >78060</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow1_col2" class="data row1 col2" >R$46,645,726</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow1_col3" class="data row1 col3" >R$598</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow2_col0" class="data row2 col0" >Sem informação</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow2_col1" class="data row2 col1" >39809</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow2_col2" class="data row2 col2" >R$32,105,065</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow2_col3" class="data row2 col3" >R$806</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow3_col0" class="data row3 col0" >Ministério da Justiça e Segurança Pública</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow3_col1" class="data row3 col1" >63277</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow3_col2" class="data row3 col2" >R$27,848,772</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow3_col3" class="data row3 col3" >R$440</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow4_col0" class="data row4 col0" >Ministério da Saúde</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow4_col1" class="data row4 col1" >29686</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow4_col2" class="data row4 col2" >R$20,769,533</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow4_col3" class="data row4 col3" >R$700</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow5_col0" class="data row5 col0" >Ministério da Economia</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow5_col1" class="data row5 col1" >55256</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow5_col2" class="data row5 col2" >R$19,125,884</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow5_col3" class="data row5 col3" >R$346</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow6_col0" class="data row6 col0" >Ministério das Relações Exteriores</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow6_col1" class="data row6 col1" >3199</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow6_col2" class="data row6 col2" >R$12,366,786</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow6_col3" class="data row6 col3" >R$3,866</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow7_col0" class="data row7 col0" >Ministério da Infraestrutura</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow7_col1" class="data row7 col1" >13877</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow7_col2" class="data row7 col2" >R$12,362,585</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow7_col3" class="data row7 col3" >R$891</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow8_col0" class="data row8 col0" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow8_col1" class="data row8 col1" >8468</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow8_col2" class="data row8 col2" >R$10,895,048</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow8_col3" class="data row8 col3" >R$1,287</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow9_col0" class="data row9 col0" >Ministério da Agricultura, Pecuária e Abastecimento</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow9_col1" class="data row9 col1" >31631</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow9_col2" class="data row9 col2" >R$10,483,097</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow9_col3" class="data row9 col3" >R$331</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow10_col0" class="data row10 col0" >Presidência da República</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow10_col1" class="data row10 col1" >8246</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow10_col2" class="data row10 col2" >R$9,406,747</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow10_col3" class="data row10 col3" >R$1,141</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow11_col0" class="data row11 col0" >Ministério do Meio Ambiente</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow11_col1" class="data row11 col1" >19586</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow11_col2" class="data row11 col2" >R$9,008,210</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow11_col3" class="data row11 col3" >R$460</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow12_col0" class="data row12 col0" >Ministério de Minas e Energia</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow12_col1" class="data row12 col1" >3727</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow12_col2" class="data row12 col2" >R$7,018,544</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow12_col3" class="data row12 col3" >R$1,883</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow13_col0" class="data row13 col0" >Ministério da Cidadania</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow13_col1" class="data row13 col1" >3475</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow13_col2" class="data row13 col2" >R$2,980,346</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow13_col3" class="data row13 col3" >R$858</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow14_col0" class="data row14 col0" >Ministério do Desenvolvimento Regional</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow14_col1" class="data row14 col1" >5197</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow14_col2" class="data row14 col2" >R$2,955,565</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow14_col3" class="data row14 col3" >R$569</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow15_col0" class="data row15 col0" >Controladoria-Geral da União</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow15_col1" class="data row15 col1" >2471</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow15_col2" class="data row15 col2" >R$1,930,138</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow15_col3" class="data row15 col3" >R$781</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow16_col0" class="data row16 col0" >Ministério da Mulher, Família e Direitos Humanos</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow16_col1" class="data row16 col1" >4859</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow16_col2" class="data row16 col2" >R$1,686,921</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow16_col3" class="data row16 col3" >R$347</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow17_col0" class="data row17 col0" >Advocacia-Geral da União</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow17_col1" class="data row17 col1" >5329</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow17_col2" class="data row17 col2" >R$1,405,677</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow17_col3" class="data row17 col3" >R$264</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow18_col0" class="data row18 col0" >Ministério da Indústria, Comércio Exterior e Serviços</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow18_col1" class="data row18 col1" >240</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow18_col2" class="data row18 col2" >R$887,885</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow18_col3" class="data row18 col3" >R$3,700</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow19_col0" class="data row19 col0" >Ministério do Planejamento, Desenvolvimento e Gestão</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow19_col1" class="data row19 col1" >532</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow19_col2" class="data row19 col2" >R$714,835</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow19_col3" class="data row19 col3" >R$1,344</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow20_col0" class="data row20 col0" >Ministério do Trabalho e Emprego</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow20_col1" class="data row20 col1" >2330</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow20_col2" class="data row20 col2" >R$636,132</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow20_col3" class="data row20 col3" >R$273</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow21_col0" class="data row21 col0" >Ministério do Turismo</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow21_col1" class="data row21 col1" >139</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow21_col2" class="data row21 col2" >R$555,474</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow21_col3" class="data row21 col3" >R$3,996</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow22_col0" class="data row22 col0" >Ministério do Esporte</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow22_col1" class="data row22 col1" >8</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow22_col2" class="data row22 col2" >R$25,163</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow22_col3" class="data row22 col3" >R$3,145</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow23_col0" class="data row23 col0" >Ministério da Cultura</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow23_col1" class="data row23 col1" >2</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow23_col2" class="data row23 col2" >R$5,256</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow23_col3" class="data row23 col3" >R$2,628</td>
            </tr>
            <tr>
                                <td id="T_37453446_f366_11e9_a31e_a86badfc757frow24_col0" class="data row24 col0" >Ministério das Cidades</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow24_col1" class="data row24 col1" >1</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow24_col2" class="data row24 col2" >R$0</td>
                        <td id="T_37453446_f366_11e9_a31e_a86badfc757frow24_col3" class="data row24 col3" >R$0</td>
            </tr>
    </tbody></table>



Notamos que o Min. da Educação é o que mais viaja bem como o que mais gasta com viagem. Faz sentido enxergar educação, defesa e segurança pública no topo da lista devido ao número de servidores que esses três ministérios englobam. Apesar de gastarem mais, ao analisarmos o preço médio por viagem vemos que as passagens médias mais caras pertencem a outros órgãos.

Os campeões das passagens médias mais caras são Min. do Turismo (3996), Min. Relações Exteriores (3866)  e Min. da Indústria (3700). Também faz sentindo pois os três ministérios devem realizar mais viagens internacionais que os demais.   

Vamos fazer um gráfico para auxiliar a leitura dos dados. 


```python
#gráfico em barra para visualizar órgãos que mais gastaram
plt.style.use('fivethirtyeight')
plt.figure(figsize=(20,10))  
ax = sns.barplot(x='Valor total passagens', y='Nome do órgão superior', data=agrupado) #valor médio viagem
#ax.set_xlabel('Gastos com viagens por órgão')
```


![png](post_files/post_16_0.png)


Qual será que foram as passagens mais caras?


```python
#exibe as 10 passagens mais caras
(viagens_r.nlargest(10, 'Valor passagens')
 .style.format({'Valor passagens' : 'R${0:,.0f}', 'Valor diárias' : 'R${0:,.0f}', 'Valor outros gastos' : 'R${0:,.0f}'}))
```




<style  type="text/css" >
</style><table id="T_3c6880f4_f366_11e9_a31e_a86badfc757f" ><thead>    <tr>        <th class="blank level0" ></th>        <th class="col_heading level0 col0" >Identificador do processo de viagem</th>        <th class="col_heading level0 col1" >Situação</th>        <th class="col_heading level0 col2" >Nome do órgão superior</th>        <th class="col_heading level0 col3" >Nome órgão solicitante</th>        <th class="col_heading level0 col4" >Nome</th>        <th class="col_heading level0 col5" >Cargo</th>        <th class="col_heading level0 col6" >Período - Data de início</th>        <th class="col_heading level0 col7" >Período - Data de fim</th>        <th class="col_heading level0 col8" >Destinos</th>        <th class="col_heading level0 col9" >Motivo</th>        <th class="col_heading level0 col10" >Valor diárias</th>        <th class="col_heading level0 col11" >Valor passagens</th>        <th class="col_heading level0 col12" >Valor outros gastos</th>    </tr></thead><tbody>
                <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row0" class="row_heading level0 row0" >302646</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col0" class="data row0 col0" >16154465</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col1" class="data row0 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col2" class="data row0 col2" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col3" class="data row0 col3" >Agência Espacial Brasileira</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col4" class="data row0 col4" >CARLOS EDUARDO QUINTANILHA VAZ DE OLIVEIRA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col5" class="data row0 col5" >TECNOLOGISTA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col6" class="data row0 col6" >2019-06-09 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col7" class="data row0 col7" >2019-06-16 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col8" class="data row0 col8" >Londres/Reino Unido</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col9" class="data row0 col9" >Compor a delegação brasileira que participará da Reunião Plenária da ISO  TC20/SC14 - Space Systems and Operations, no período de 09 a 16 de junho de 2019, em Londres - Inglaterra, trânsito já incluído, com ônus.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col10" class="data row0 col10" >R$10,761</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col11" class="data row0 col11" >R$155,531</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow0_col12" class="data row0 col12" >R$0</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row1" class="row_heading level0 row1" >378436</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col0" class="data row1 col0" >16253216</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col1" class="data row1 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col2" class="data row1 col2" >Ministério da Educação</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col3" class="data row1 col3" >Fundação Coordenação de Aperfeiçoamento de Pessoal de Nível Superior</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col4" class="data row1 col4" >LUCIANA MOURAO CERQUEIRA E SILVA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col5" class="data row1 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col6" class="data row1 col6" >2019-07-16 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col7" class="data row1 col7" >2019-07-19 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col8" class="data row1 col8" >Brasília/DF</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col9" class="data row1 col9" >Preparatório Seminário Psicologia</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col10" class="data row1 col10" >R$880</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col11" class="data row1 col11" >R$117,096</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow1_col12" class="data row1 col12" >R$0</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row2" class="row_heading level0 row2" >230142</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col0" class="data row2 col0" >16058716</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col1" class="data row2 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col2" class="data row2 col2" >Ministério da Agricultura, Pecuária e Abastecimento</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col3" class="data row2 col3" >Ministério da Agricultura, Pecuária e Abastecimento - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col4" class="data row2 col4" >MARA ANDREA BERGAMASCHI</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col5" class="data row2 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col6" class="data row2 col6" >2019-05-06 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col7" class="data row2 col7" >2019-05-21 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col8" class="data row2 col8" >Tóquio/Japão, Xangai/China, Pequim/China, Hanói/Vietnã, Jacarta/Indonésia</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col9" class="data row2 col9" >Acompanhar e Assessorar a Senhora Ministra em: JAPÃO/Tóquio - Reunião Bilateral com o Ministro da Agricultura, Floresta e Pesca (MAFF). Reunião com o Ministro da Saúde, Trabalho e Bem-estar (MHLW). Econtro com a Kendaren - Federação das Indústrias do Japão. Encontro prévio da Ministra com CEO da UCC. Entrevistas com jornais locais. Niigata - Encontro Bilateral com o MAARA. G20 Agriculture Minister,s Meeting. Encontro Bilateral com o Secretário de Agricultura dos Estados Unidos da América. CHINA/Xangai - Agenda em Coordenação com o Banco do Brasil em Xangai e com o Consulado Brasileiro. Aguardando por parte da embaixada e Banco do Brasil o perfil dos investidores. Pequim - Encontro bilateral com a GACC. Visita à Academia Chinesa de Ciência da Agricultura e o Huawei Exibition Hall. VIETNÃ/Hanói - Reunião Bilateral com o Ministério da Agricultura e Desenvolvimento Rural - MARD. INDONÉSIA/Jacarta - Reunião com o Ministro da Agricultura Indonésia.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col10" class="data row2 col10" >R$20,303</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col11" class="data row2 col11" >R$50,885</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow2_col12" class="data row2 col12" >R$195</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row3" class="row_heading level0 row3" >204579</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col0" class="data row3 col0" >16024112</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col1" class="data row3 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col2" class="data row3 col2" >Ministério da Agricultura, Pecuária e Abastecimento</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col3" class="data row3 col3" >Ministério da Agricultura, Pecuária e Abastecimento - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col4" class="data row3 col4" >TEREZA CRISTINA CORREA DA COSTA DIAS</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col5" class="data row3 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col6" class="data row3 col6" >2019-05-06 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col7" class="data row3 col7" >2019-05-21 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col8" class="data row3 col8" >São Paulo/SP, Tóquio/Japão, Niigata/Japão, Tóquio/Japão, Xangai/China, Pequim/China, Hanói/Vietnã, Jacarta/Indonésia</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col9" class="data row3 col9" >JAPÃO/Tóquio - Reunião Bilateral com o Ministro da Agricultura, Floresta e Pesca (MAFF). Reunião com o Ministro da Saúde, Trabalh e Bem-estar (MHLW). Econtro com a Kendaren - Federação das Indústrias do Japão. Encontro prévio da Ministra com CEO da UCC. Entrevistas com jornais locais. Niigata - Encontro Bilateral com o MAARA. Econtro Bilateral com o Ministro da Agricultura da Rússia. G20 Agriculture Minister,s Meeting. Encontro Bilateral com o Secretário de Agricultura dos Estados Unidos da América. CHINA/Xangai - Agenda em Coordenação com o Banco do Brasil em Xangai e com o Consulado Brasileiro. Aguardando por parte da embaixada e Banco do Brasil o perfil dos investidores. Pequim - Encontro bilateral com a GACC. Visita à Academia Chinesa de Ciência da Agricultura e o Huawei Exibition Hall.  VIETNÃ/Hanói - Reunião Bilateral com o Ministério da Agricultura e Desenvolvimento Rural - MARD. INDONÉSIA/Jacarta - Reunião com o Ministro da Agricultura Indonésio.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col10" class="data row3 col10" >R$21,105</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col11" class="data row3 col11" >R$42,225</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow3_col12" class="data row3 col12" >R$2,666</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row4" class="row_heading level0 row4" >124017</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col0" class="data row4 col0" >15918458</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col1" class="data row4 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col2" class="data row4 col2" >Ministério da Saúde</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col3" class="data row4 col3" >Agência Nacional de Vigilância Sanitária</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col4" class="data row4 col4" >LIGIA LINDNER SCHREINER</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col5" class="data row4 col5" >ESP EM REGULACAO E VIGILANCIA SANITARIA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col6" class="data row4 col6" >2019-04-07 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col7" class="data row4 col7" >2019-04-12 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col8" class="data row4 col8" >Buenos Aires/Argentina</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col9" class="data row4 col9" >Trata-se da LXVIII Reunião do Subgrupo de Trabalho Nº 3 - Regulamentos Técnicos e Avaliação da Conformidade, que ocorrerá de 8 a 12 de abril de 2019, em Buenos Aires, Argentina. A reunião tem por objetivo elaborar/revisar regulamentos técnicos da área de alimentos, que fazem parte da Agenda da Comissão de Alimentos. O Brasil será responsável pelo tema  de aditivos alimentares, assim justifica-se a composição da delegação com os seguintes servidores: TIAGO LANIUS RAUBER, LÍGIA LINDNER SCHREINER, SUZANY PORTAL DA SILVA MORAES, ANTONIA MARIA DE AQUINO E REBECA ALMEIDA SILVA. Total de 5 participantes custeados pela Anvisa, conforme decisão da Diretoria Colegiada em reunião realizada por meio do Circuito Deliberativo - CD/DN 85/2019 - Afastamentos do País, de 7/3/2019 - SEI 25351.904566/2019-71.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col10" class="data row4 col10" >R$5,242</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col11" class="data row4 col11" >R$36,002</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow4_col12" class="data row4 col12" >R$160</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row5" class="row_heading level0 row5" >177506</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col0" class="data row5 col0" >15988180</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col1" class="data row5 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col2" class="data row5 col2" >Ministério da Defesa</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col3" class="data row5 col3" >Comando da Aeronáutica</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col4" class="data row5 col4" >GUILHERME ANTONIO MATOS RODRIGUES</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col5" class="data row5 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col6" class="data row5 col6" >2019-04-21 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col7" class="data row5 col7" >2019-04-26 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col8" class="data row5 col8" >Anápolis/GO</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col9" class="data row5 col9" >PARTICIPAR DAS REUNIÕES DE PDR DA MPTS DO PROJETO E-99M NAS DEPENDêNCIAS DO 2º/6º GAV</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col10" class="data row5 col10" >R$1,163</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col11" class="data row5 col11" >R$31,920</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow5_col12" class="data row5 col12" >R$1,163</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row6" class="row_heading level0 row6" >331719</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col0" class="data row6 col0" >16192165</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col1" class="data row6 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col2" class="data row6 col2" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col3" class="data row6 col3" >Ministério da Ciência, Tecnologia, Inovações e Comunicações - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col4" class="data row6 col4" >MARCELO MARCOS MORALES</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col5" class="data row6 col5" >PROFESSOR DO MAGISTERIO SUPERIOR</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col6" class="data row6 col6" >2019-06-15 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col7" class="data row6 col7" >2019-06-29 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col8" class="data row6 col8" >Paris/França, Viena/Áustria, Abu Dabi/Emirados Árabes, Doha/Catar, Genebra/Suíça</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col9" class="data row6 col9" >Acompanhar o Exmo. Sr. Ministro Marcos Pontes em Missão à Europa. - Assunto: Ciência, Formação e Pesquisa.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col10" class="data row6 col10" >R$22,749</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col11" class="data row6 col11" >R$29,500</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow6_col12" class="data row6 col12" >R$0</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row7" class="row_heading level0 row7" >333888</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col0" class="data row7 col0" >16194989</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col1" class="data row7 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col2" class="data row7 col2" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col3" class="data row7 col3" >Ministério da Ciência, Tecnologia, Inovações e Comunicações - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col4" class="data row7 col4" >VANIA GOMES DA SILVA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col5" class="data row7 col5" >ANALISTA EM CIENCIA E TECNOLOGIA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col6" class="data row7 col6" >2019-06-15 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col7" class="data row7 col7" >2019-06-29 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col8" class="data row7 col8" >Paris/França, Viena/Áustria, Abu Dabi/Emirados Árabes, Doha/Catar, Genebra/Suíça</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col9" class="data row7 col9" >Acompanhar e assessorar o ministro de estado da Ciência, Tecnologia, Inovações e Comunicações na missão para os seguintes países: França, Áustria, Emirados Árabes Unidos, Catar e Suíça, entre os dias 15 e 29 de junho próximo, com ônus para este Ministério.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col10" class="data row7 col10" >R$21,253</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col11" class="data row7 col11" >R$29,500</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow7_col12" class="data row7 col12" >R$0</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row8" class="row_heading level0 row8" >335878</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col0" class="data row8 col0" >16197634</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col1" class="data row8 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col2" class="data row8 col2" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col3" class="data row8 col3" >Ministério da Ciência, Tecnologia, Inovações e Comunicações - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col4" class="data row8 col4" >MARCOS CESAR PONTES</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col5" class="data row8 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col6" class="data row8 col6" >2019-06-15 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col7" class="data row8 col7" >2019-06-29 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col8" class="data row8 col8" >Paris/França, Viena/Áustria, Abu Dabi/Emirados Árabes, Dubai/Emirados Árabes, Doha/Catar, Genebra/Suíça</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col9" class="data row8 col9" >Visita oficial a França, Áustria, Emirados Árabes Unidos, Catar e Suíça onde cumprirá agenda de reuniões com autoridades governamentais das áreas de ciência, tecnologia e inovação; com dirigentes de instituições de pesquisa; e com representantes de empresas inovadoras que podem trazer mais investimentos ao Brasil.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col10" class="data row8 col10" >R$22,952</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col11" class="data row8 col11" >R$29,500</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow8_col12" class="data row8 col12" >R$0</td>
            </tr>
            <tr>
                        <th id="T_3c6880f4_f366_11e9_a31e_a86badfc757flevel0_row9" class="row_heading level0 row9" >340635</th>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col0" class="data row9 col0" >16204085</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col1" class="data row9 col1" >Realizada</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col2" class="data row9 col2" >Ministério da Ciência, Tecnologia, Inovações e Comunicações</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col3" class="data row9 col3" >Ministério da Ciência, Tecnologia, Inovações e Comunicações - Unidades com vínculo direto</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col4" class="data row9 col4" >CHRISTIANE GONCALVES CORREA</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col5" class="data row9 col5" >nan</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col6" class="data row9 col6" >2019-06-15 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col7" class="data row9 col7" >2019-06-29 00:00:00</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col8" class="data row9 col8" >Paris/França, Viena/Áustria, Abu Dabi/Emirados Árabes, Dubai/Emirados Árabes, Doha/Catar, Genebra/Suíça</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col9" class="data row9 col9" >Acompanhar o Sr. Ministro em visita oficial a França, Áustria, Emirados Árabes Unidos, Catar e Suíça onde cumprirá agenda de reuniões com autoridades governamentais das áreas de ciência, tecnologia e inovação; com dirigentes de instituições de pesquisa; e com representantes de empresas inovadoras que podem trazer mais investimentos ao Brasil.</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col10" class="data row9 col10" >R$22,952</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col11" class="data row9 col11" >R$29,500</td>
                        <td id="T_3c6880f4_f366_11e9_a31e_a86badfc757frow9_col12" class="data row9 col12" >R$0</td>
            </tr>
    </tbody></table>



Três passagens nos chamam atenção: 1) passagem de 155mil para Londres, 2) passagem de 117mil para Brasília e 3) passagem de 36mil para Buenos Aires. Sabemos o destino das viagens, mas qual seria as origens?

O portal da transparência oferece outros três arquivo com informações sobre as viagens com dados mais detalhados sobre os trechos, o pagemente e a passagem. Todos os arquivos utilizam uma coluna denomida "Identificador do processo de viagem" como chave, possibilitando o cruzamento de informações. Vamos descobrir a origem e destino dessas viagens para entender se realmente os valores são justificáveis ou não.    

As três viagens que queremos mais informações possuem os identificadores '16154465', '15918458' e '15988180'.


```python
#sobe os dados de passagem
zf = zipfile.ZipFile('/home/blackmamba/Documents/analise_dados/viagens_gov_federal/dados/2019_Viagens.zip') 
passagens = pd.read_csv(zf.open("2019_Passagem.csv"), sep=";", encoding="latin1")
```


```python
#seleciona somente as trẽs viagens que queremos mais informações
passagens["Valor da passagem"] = passagens["Valor da passagem"].str.replace(',' , '.').astype(float)
passagens[passagens['Identificador do processo de viagem'].isin(['16154465', '15918458','15988180'])]
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
      <th>Meio de transporte</th>
      <th>País - Origem ida</th>
      <th>UF - Origem ida</th>
      <th>Cidade - Origem ida</th>
      <th>País - Destino ida</th>
      <th>UF - Destino ida</th>
      <th>Cidade - Destino ida</th>
      <th>País - Origem volta</th>
      <th>UF - Origem volta</th>
      <th>Cidade - Origem volta</th>
      <th>Pais - Destino volta</th>
      <th>UF - Destino volta</th>
      <th>Cidade - Destino volta</th>
      <th>Valor da passagem</th>
      <th>Taxa de serviço</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>70082</th>
      <td>15918458</td>
      <td>Aéreo</td>
      <td>Brasil</td>
      <td>Distrito Federal</td>
      <td>Brasília</td>
      <td>Argentina</td>
      <td>NaN</td>
      <td>Buenos Aires</td>
      <td>Argentina</td>
      <td>NaN</td>
      <td>Buenos Aires</td>
      <td>Brasil</td>
      <td>Distrito Federal</td>
      <td>Brasília</td>
      <td>36002.21</td>
      <td>0,00</td>
    </tr>
    <tr>
      <th>96692</th>
      <td>15988180</td>
      <td>Aéreo</td>
      <td>Brasil</td>
      <td>Goiás</td>
      <td>Goiânia</td>
      <td>Brasil</td>
      <td>São Paulo</td>
      <td>Guarulhos</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>375.04</td>
      <td>0,00</td>
    </tr>
    <tr>
      <th>96693</th>
      <td>15988180</td>
      <td>Aéreo</td>
      <td>Brasil</td>
      <td>São Paulo</td>
      <td>Guarulhos</td>
      <td>Brasil</td>
      <td>Goiás</td>
      <td>Goiânia</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>31544.86</td>
      <td>0,00</td>
    </tr>
    <tr>
      <th>163498</th>
      <td>16154465</td>
      <td>Aéreo</td>
      <td>Brasil</td>
      <td>Distrito Federal</td>
      <td>Brasília</td>
      <td>Reino Unido</td>
      <td>NaN</td>
      <td>Londres</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>46252.40</td>
      <td>0,00</td>
    </tr>
    <tr>
      <th>163499</th>
      <td>16154465</td>
      <td>Aéreo</td>
      <td>Reino Unido</td>
      <td>NaN</td>
      <td>Londres</td>
      <td>Brasil</td>
      <td>Distrito Federal</td>
      <td>Brasília</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>Sem Informação</td>
      <td>109278.96</td>
      <td>0,00</td>
    </tr>
  </tbody>
</table>
</div>



Os valores continuam me parecendo altíssimos:

1) '15918458' = Brasília - Buenos Aires ida e volta por 36mil reais.
2) '15988180' = Um trecho Guarulhos - Goiânia por quase 32mil reais. 
3) '16154465' = Londres - Brasília ida e volta por 155mil reais.

Quais seriam os trechos mais comuns? Qual é o preço médio que o governo federal paga nas ponte aéreas mais frequentes?

Parece que a maioria algumas passagens são lançadas individualmente, sem colocar os trechos de ida e de volta na mesma linha. Vamos conferir.


```python
print(passagens.shape)
print(passagens[passagens['Cidade - Origem volta'] == 'Sem Informação'].shape)
```

    (276310, 16)
    (254305, 16)


Das 276310 viagens lançadas, 254305 não possuem informação o trecho de volta. Esse número de ocorrência parece satisfatório para trabalharmos. Então vamos usar somente as cidades de origem e destino da ida. 

Para calcular os trechos mais viajos, criaremos uma nova coluna concatenando as cidades de origem e destino. 


```python
#cria nova coluna concatenando origem e destino
passagens['ida_volta'] = passagens['Cidade - Origem ida'] + '-' + passagens['Cidade - Destino ida'] 
```


```python
#agrupa por ida_volta + soma e conta
ida_volta = (passagens.groupby('ida_volta')
             .agg({'Meio de transporte' : 'count', 'Valor da passagem' : 'sum'})
             .reset_index())

#renomeia coluna
ida_volta.rename(columns={'Meio de transporte' : 'qtd_viagens'}, inplace=True)

#adiciona coluna com valor médio dos trechos
ida_volta['media_trecho'] = ida_volta['Valor da passagem'] / ida_volta['qtd_viagens']
```


```python
#mostra os 20 trechos mais viajados + valor total + média do trecho
(ida_volta.nlargest(20, 'qtd_viagens')  
          .style.format({'Valor da passagem' : 'R${0:,.0f}', 'media_trecho' : 'R${0:,.0f}'})
          .hide_index()
          .highlight_max(subset='media_trecho',color='green')
          .highlight_min(subset='media_trecho',color='#cd4f39'))
```




<style  type="text/css" >
    #T_1d6f4464_f378_11e9_a31e_a86badfc757frow1_col3 {
            background-color:  green;
            : ;
        }    #T_1d6f4464_f378_11e9_a31e_a86badfc757frow6_col3 {
            : ;
            background-color:  #cd4f39;
        }</style><table id="T_1d6f4464_f378_11e9_a31e_a86badfc757f" ><thead>    <tr>        <th class="col_heading level0 col0" >ida_volta</th>        <th class="col_heading level0 col1" >qtd_viagens</th>        <th class="col_heading level0 col2" >Valor da passagem</th>        <th class="col_heading level0 col3" >media_trecho</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow0_col0" class="data row0 col0" >Brasília-Rio de Janeiro</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow0_col1" class="data row0 col1" >12551</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow0_col2" class="data row0 col2" >R$13,577,411</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow0_col3" class="data row0 col3" >R$1,082</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow1_col0" class="data row1 col0" >Rio de Janeiro-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow1_col1" class="data row1 col1" >12205</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow1_col2" class="data row1 col2" >R$13,469,962</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow1_col3" class="data row1 col3" >R$1,104</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow2_col0" class="data row2 col0" >Brasília-São Paulo</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow2_col1" class="data row2 col1" >10187</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow2_col2" class="data row2 col2" >R$10,488,552</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow2_col3" class="data row2 col3" >R$1,030</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow3_col0" class="data row3 col0" >São Paulo-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow3_col1" class="data row3 col1" >9916</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow3_col2" class="data row3 col2" >R$9,892,731</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow3_col3" class="data row3 col3" >R$998</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow4_col0" class="data row4 col0" >Porto Alegre-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow4_col1" class="data row4 col1" >3686</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow4_col2" class="data row4 col2" >R$3,493,061</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow4_col3" class="data row4 col3" >R$948</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow5_col0" class="data row5 col0" >Brasília-Porto Alegre</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow5_col1" class="data row5 col1" >3613</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow5_col2" class="data row5 col2" >R$3,237,268</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow5_col3" class="data row5 col3" >R$896</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow6_col0" class="data row6 col0" >Belo Horizonte-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow6_col1" class="data row6 col1" >3339</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow6_col2" class="data row6 col2" >R$1,668,533</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow6_col3" class="data row6 col3" >R$500</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow7_col0" class="data row7 col0" >Brasília-Belo Horizonte</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow7_col1" class="data row7 col1" >3285</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow7_col2" class="data row7 col2" >R$1,655,628</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow7_col3" class="data row7 col3" >R$504</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow8_col0" class="data row8 col0" >Rio de Janeiro-São Paulo</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow8_col1" class="data row8 col1" >3237</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow8_col2" class="data row8 col2" >R$1,963,562</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow8_col3" class="data row8 col3" >R$607</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow9_col0" class="data row9 col0" >São Paulo-Rio de Janeiro</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow9_col1" class="data row9 col1" >3020</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow9_col2" class="data row9 col2" >R$1,744,826</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow9_col3" class="data row9 col3" >R$578</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow10_col0" class="data row10 col0" >Recife-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow10_col1" class="data row10 col1" >2501</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow10_col2" class="data row10 col2" >R$2,484,876</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow10_col3" class="data row10 col3" >R$994</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow11_col0" class="data row11 col0" >Brasília-Recife</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow11_col1" class="data row11 col1" >2403</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow11_col2" class="data row11 col2" >R$2,164,666</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow11_col3" class="data row11 col3" >R$901</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow12_col0" class="data row12 col0" >Curitiba-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow12_col1" class="data row12 col1" >2320</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow12_col2" class="data row12 col2" >R$1,802,470</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow12_col3" class="data row12 col3" >R$777</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow13_col0" class="data row13 col0" >Brasília-Curitiba</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow13_col1" class="data row13 col1" >2318</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow13_col2" class="data row13 col2" >R$1,711,886</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow13_col3" class="data row13 col3" >R$739</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow14_col0" class="data row14 col0" >Salvador-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow14_col1" class="data row14 col1" >2066</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow14_col2" class="data row14 col2" >R$1,809,005</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow14_col3" class="data row14 col3" >R$876</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow15_col0" class="data row15 col0" >Brasília-Campinas</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow15_col1" class="data row15 col1" >2042</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow15_col2" class="data row15 col2" >R$1,198,706</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow15_col3" class="data row15 col3" >R$587</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow16_col0" class="data row16 col0" >Campinas-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow16_col1" class="data row16 col1" >2019</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow16_col2" class="data row16 col2" >R$1,299,375</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow16_col3" class="data row16 col3" >R$644</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow17_col0" class="data row17 col0" >Brasília-Salvador</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow17_col1" class="data row17 col1" >1986</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow17_col2" class="data row17 col2" >R$1,655,260</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow17_col3" class="data row17 col3" >R$833</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow18_col0" class="data row18 col0" >Brasília-Fortaleza</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow18_col1" class="data row18 col1" >1947</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow18_col2" class="data row18 col2" >R$1,708,205</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow18_col3" class="data row18 col3" >R$877</td>
            </tr>
            <tr>
                                <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow19_col0" class="data row19 col0" >Fortaleza-Brasília</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow19_col1" class="data row19 col1" >1921</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow19_col2" class="data row19 col2" >R$1,723,885</td>
                        <td id="T_1d6f4464_f378_11e9_a31e_a86badfc757frow19_col3" class="data row19 col3" >R$897</td>
            </tr>
    </tbody></table>



Brasília - Rio & Rio - Brasília são os trechos mais viajados. Esse resultado é esperado dado ao grande número de servidores federais trabalhando no Rio.

A média da passagem de uma passagem ida e volta RIO-BSB é 2186 reais. Bem caro. 

Destaque para o trecho mais barato entre os 20 destinos mais comuns para Belo Horizonte - Brasília; 500 reais pra ir e 504 pra voltar. 

Voltando para a análise dos que mais gastam, quem são os top 20 servidoras(es) que mais viajam? 


```python
#agrupa por Nome, conta e soma
pessoas = (viagens_r.groupby('Nome')
            .agg({'Nome órgão solicitante' : 'count', 'Valor passagens' : 'sum'})
            .reset_index())

#renomeia colunas
pessoas.rename(columns={'Nome órgão solicitante' : 'Qtd Viagens' , 'Valor passagens' : 'Valor total passagens' }, inplace=True)

#calcula preço médio por viagem
pessoas['valor médio passagem'] = pessoas['Valor total passagens'] / pessoas['Qtd Viagens']
```


```python
#mostra as 20 pessoas que mais viajam + valor total + média do trecho
(pessoas.nlargest(20, 'Qtd Viagens')  
          .style.format({'Valor total passagens' : 'R${0:,.0f}', 'valor médio passagem' : 'R${0:,.0f}'})
          .hide_index()
          .highlight_max(subset='valor médio passagem',color='green')
          .highlight_min(subset='valor médio passagem',color='#cd4f39'))
```




<style  type="text/css" >
    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow0_col3 {
            background-color:  green;
            : ;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow1_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow2_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow3_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow4_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow5_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow6_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow7_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow8_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow9_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow10_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow11_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow12_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow13_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow14_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow15_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow16_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow17_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow18_col3 {
            : ;
            background-color:  #cd4f39;
        }    #T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow19_col3 {
            : ;
            background-color:  #cd4f39;
        }</style><table id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757f" ><thead>    <tr>        <th class="col_heading level0 col0" >Nome</th>        <th class="col_heading level0 col1" >Qtd Viagens</th>        <th class="col_heading level0 col2" >Valor total passagens</th>        <th class="col_heading level0 col3" >valor médio passagem</th>    </tr></thead><tbody>
                <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow0_col0" class="data row0 col0" >Informações protegidas por sigilo</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow0_col1" class="data row0 col1" >63759</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow0_col2" class="data row0 col2" >R$31,061,923</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow0_col3" class="data row0 col3" >R$487</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow1_col0" class="data row1 col0" >PATRICIA TIDORI MIURA</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow1_col1" class="data row1 col1" >137</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow1_col2" class="data row1 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow1_col3" class="data row1 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow2_col0" class="data row2 col0" >JOSE MILTON NATIVIDADE</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow2_col1" class="data row2 col1" >118</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow2_col2" class="data row2 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow2_col3" class="data row2 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow3_col0" class="data row3 col0" >ANTONIO TOMAZ PATRONO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow3_col1" class="data row3 col1" >116</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow3_col2" class="data row3 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow3_col3" class="data row3 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow4_col0" class="data row4 col0" >ROBERTO CARLOS DE CARVALHO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow4_col1" class="data row4 col1" >114</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow4_col2" class="data row4 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow4_col3" class="data row4 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow5_col0" class="data row5 col0" >ADILSON RAIMUNDO XAVIER</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow5_col1" class="data row5 col1" >109</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow5_col2" class="data row5 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow5_col3" class="data row5 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow6_col0" class="data row6 col0" >ANTONIO MIKIO SAVAMOTO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow6_col1" class="data row6 col1" >109</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow6_col2" class="data row6 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow6_col3" class="data row6 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow7_col0" class="data row7 col0" >JOSE RAIMUNDO TIMOTEO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow7_col1" class="data row7 col1" >99</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow7_col2" class="data row7 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow7_col3" class="data row7 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow8_col0" class="data row8 col0" >JOAO BATISTA DA SILVA</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow8_col1" class="data row8 col1" >98</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow8_col2" class="data row8 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow8_col3" class="data row8 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow9_col0" class="data row9 col0" >MARCIO FLAVIO MOL</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow9_col1" class="data row9 col1" >97</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow9_col2" class="data row9 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow9_col3" class="data row9 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow10_col0" class="data row10 col0" >NELSON MENDES CARNEIRO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow10_col1" class="data row10 col1" >97</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow10_col2" class="data row10 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow10_col3" class="data row10 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow11_col0" class="data row11 col0" >PAULO CESAR GERMINIANI DA SILVA</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow11_col1" class="data row11 col1" >94</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow11_col2" class="data row11 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow11_col3" class="data row11 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow12_col0" class="data row12 col0" >EDSON CLAUDIO GUALBERTO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow12_col1" class="data row12 col1" >89</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow12_col2" class="data row12 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow12_col3" class="data row12 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow13_col0" class="data row13 col0" >JOSE OSVALDO DA SILVA</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow13_col1" class="data row13 col1" >87</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow13_col2" class="data row13 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow13_col3" class="data row13 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow14_col0" class="data row14 col0" >ARTUR JOSE BELTRAMI</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow14_col1" class="data row14 col1" >85</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow14_col2" class="data row14 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow14_col3" class="data row14 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow15_col0" class="data row15 col0" >ANTONIO DIAS DOS SANTOS</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow15_col1" class="data row15 col1" >84</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow15_col2" class="data row15 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow15_col3" class="data row15 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow16_col0" class="data row16 col0" >GILMAR BARROS DA SILVA</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow16_col1" class="data row16 col1" >84</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow16_col2" class="data row16 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow16_col3" class="data row16 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow17_col0" class="data row17 col0" >JOSE ALDO DE HOLANDA CAVALCANTE</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow17_col1" class="data row17 col1" >84</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow17_col2" class="data row17 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow17_col3" class="data row17 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow18_col0" class="data row18 col0" >JOSE AUGUSTO FERREIRA NETO</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow18_col1" class="data row18 col1" >84</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow18_col2" class="data row18 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow18_col3" class="data row18 col3" >R$0</td>
            </tr>
            <tr>
                                <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow19_col0" class="data row19 col0" >MANUEL VIEIRA GUIMARAES</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow19_col1" class="data row19 col1" >81</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow19_col2" class="data row19 col2" >R$0</td>
                        <td id="T_bb0f76a4_f37c_11e9_a31e_a86badfc757frow19_col3" class="data row19 col3" >R$0</td>
            </tr>
    </tbody></table>



Eita! Como pode uma viagem ter sido realizada e ter custado zero reais?! Pior, 137 viagens! Existe a possibilidade desse ser alguma questão técnica, valeria a pena entrar em contato com o portal da transparência para entender. De qualquer forma é muito estranho.. no mínimo está mal explicado. 

Quantas viagens custaram zero reais em passagem?


```python
print(pessoas.shape)

print(pessoas[pessoas['Valor total passagens'] == 0].shape)


```

    (157875, 4)
    (82175, 4)


Das 157875 viagens realizadas, 82175 custaram zero reais. Quase metade. Estranho..

### Conclusão 

Apesar 
