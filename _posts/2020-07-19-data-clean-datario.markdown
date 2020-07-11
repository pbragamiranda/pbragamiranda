---
layout: post
title: Limpando e estruturando dados Data.Rio
date: 2020-07-10 00:00:00 +0300
description:  # Add post description (optional)
img: datario.png # Add image post (optional)
tags: [python, dados_abertos, open_data] # add tag
---
---

### Limpando e estruturando os dados disponibilizados pelo município do Rio de Janeiro através do data.rio. 

---

A prefeitura do Rio de Janeiro disponibiliza uma vasta quantidade de dados em seu portal de transparência de dados, [data.rio](http://www.data.rio/). Muitas dessas bases de dados, principalmente as relacionadas a economia fluminense, são disponibilizadas seguindo as divisões de Áreas de Planejamento (AP), Regiões de Planejamento (RP), Regiões Admininistrativas (RA) e Bairros. Apesar de estarem em formato de planilha (xls), os dados são dividios em diferentes abas, uma para cada ano, e em uma estrutura que não permite análises básicas como evolução de série histórica. 



A base de dado que vamos usar pode ser encontra aqui -> [Número de empregados por atividade econômica segundo as Áreas de Planejamento (AP), Regiões Administrativas (RA) e Bairros](http://www.data.rio/datasets/n%C3%BAmero-de-empregados-por-atividade-econ%C3%B4mica-segundo-as-%C3%A1reas-de-planejamento-ap-regi%C3%B5es-administrativas-ra-e-bairros-no-munic%C3%ADpio-do-rio-de-janeiro-em-2005-2018) 

Esses dados me chamaram atenção para entender melhor a regionalização econõmica da cidade. 

Podemos notar que a estrutura é bastante poluída, com linhas sem relevância para análises, com a divisória horizontal entre APs, RPs, RAs e bairro e com os anos separados em abas. Com isso, precisamos limpar os dados e transformá-los em uma estrutura onde seja possível realizar análises estatísticas.  


![title]({{site.baseurl}}/assets/img/datario/data-xls.png)


### Limpando somente uma aba / ano

Primeiro, vamos limpar somente um aba, o ano de 2005, para depois aplicar os mesmo procedimentos em todas as outras. 

Essa primeira etapa demandou um pequeno trabalho manual de contar as linhas que e colunas que desejava manter e quais deseja excluir. Como todos os dados disponibilizados nessa divisão regionaliza apresentam esse mesmo padrão, pelo menos foi um trabalho de uma vez só, podendo ser aplicado em outras bases. 


```python
#import pandas
import pandas as pd

#select files
file_path_empregados =  "raw_data/atividades_econ_rj_ap.xls"

#trying cleaning for one 2005 sheet
dataframe_2005 = pd.read_excel(file_path_empregados, sheet_name="2005", header=3,  nrows=224, usecols="A:Z")

#delete uncessary rows
dataframe_2005.drop([0, 1, 2, 3, 4, 5, 10, 12, 17, 22, 24, 26, 27, 28, 29, 38, 41, 49, 51, 52, 56, 61, 62, 63, 64,
        69, 71, 72, 89, 91, 92, 99, 113, 114, 121, 123, 124, 128, 133, 134, 139, 146, 147, 163, 164, 165, 
         166, 177, 179, 180, 189, 190, 191, 192, 197, 204, 205, 211, 212, 216, 217, 221], inplace=True)
```

Agora já temos os dados estruturados por bairros e todas as diversas atividades econômicas. 


```python
dataframe_2005.head()
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
      <th>Bairros</th>
      <th>Extrativa mineral</th>
      <th>Minerais não-metálicos</th>
      <th>Indústria metalúrgica</th>
      <th>Indústria mecânica</th>
      <th>Indústria do material elétrico e de comunicações</th>
      <th>Indústria de material de transporte</th>
      <th>Indústria da madeira e do mobiliário</th>
      <th>Indústria do papel, papelão, editorial e gráfica</th>
      <th>Indústria da borracha, fumo, couros, peles, similares e diversas</th>
      <th>...</th>
      <th>Comércio varejista</th>
      <th>Comércio atacadista</th>
      <th>Instituições de crédito, seguros e capitalização</th>
      <th>Comércio e administração de imóveis, valores mobiliários, serviços técnicos</th>
      <th>Transportes e comunicações</th>
      <th>Serviços de alojamento, alimentação, reparação, manutenção, redação...</th>
      <th>Serviços médicos, odontológicos e veterinários</th>
      <th>Ensino</th>
      <th>Administração pública direta e autárquica</th>
      <th>Agricultura, silvicultura, criação de animais, extrativismo vegetal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>Saúde</td>
      <td>-</td>
      <td>5</td>
      <td>2</td>
      <td>9</td>
      <td>11</td>
      <td>3</td>
      <td>-</td>
      <td>13</td>
      <td>12</td>
      <td>...</td>
      <td>698</td>
      <td>74</td>
      <td>30</td>
      <td>696</td>
      <td>797</td>
      <td>154</td>
      <td>50</td>
      <td>39</td>
      <td>385</td>
      <td>-</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Gamboa</td>
      <td>-</td>
      <td>8</td>
      <td>30</td>
      <td>8</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>103</td>
      <td>-</td>
      <td>...</td>
      <td>380</td>
      <td>99</td>
      <td>29</td>
      <td>550</td>
      <td>932</td>
      <td>507</td>
      <td>7</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Santo Cristo</td>
      <td>-</td>
      <td>-</td>
      <td>39</td>
      <td>83</td>
      <td>47</td>
      <td>-</td>
      <td>48</td>
      <td>157</td>
      <td>132</td>
      <td>...</td>
      <td>571</td>
      <td>349</td>
      <td>24</td>
      <td>825</td>
      <td>6540</td>
      <td>571</td>
      <td>178</td>
      <td>-</td>
      <td>-</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Caju</td>
      <td>14</td>
      <td>-</td>
      <td>132</td>
      <td>-</td>
      <td>-</td>
      <td>742</td>
      <td>3</td>
      <td>742</td>
      <td>-</td>
      <td>...</td>
      <td>122</td>
      <td>1</td>
      <td>24</td>
      <td>158</td>
      <td>481</td>
      <td>280</td>
      <td>6</td>
      <td>11</td>
      <td>64</td>
      <td>3</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Centro</td>
      <td>659</td>
      <td>384</td>
      <td>301</td>
      <td>1172</td>
      <td>133</td>
      <td>161</td>
      <td>34</td>
      <td>4557</td>
      <td>849</td>
      <td>...</td>
      <td>24884</td>
      <td>5259</td>
      <td>26649</td>
      <td>95081</td>
      <td>19762</td>
      <td>41974</td>
      <td>12590</td>
      <td>5005</td>
      <td>243676</td>
      <td>645</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 26 columns</p>
</div>



Como ainda queremos conseguir adicionar Áreas de Planejamento (AP), Regiões de Planejamento (RP), Regiões Admininistrativas (RA) como colunas ao lado de barrios, precisamos re-estruturar os dados de forma que as atividades econômicas virem uma coluna de atributo e a quantidade de empregos uma coluna de valor. 

Para isso, vamos o método [melt](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.melt.html)

![title]({{site.baseurl}}/assets/img/datario/melt.png)




```python
#check all columns to be melted
dataframe_2005.columns
```




    Index(['Bairros', 'Extrativa mineral', 'Minerais não-metálicos',
           'Indústria metalúrgica', 'Indústria mecânica',
           'Indústria do material elétrico e de comunicações',
           'Indústria de material de transporte',
           'Indústria da madeira e do mobiliário',
           'Indústria do papel, papelão, editorial e gráfica',
           'Indústria da borracha, fumo, couros, peles, similares e diversas',
           'Indústria química de produtos farmacêuticos, veterinários, perfumaria...',
           'Indústria têxtil do vestuário e artefatos de tecidos',
           'Indústria de calçados',
           'Indústria de produtos alimentícios, bebidas e álcool etílico',
           'Serviços industriais de utilidade pública ', 'Construção civil',
           'Comércio varejista', 'Comércio atacadista',
           'Instituições de crédito, seguros e capitalização',
           'Comércio e administração de imóveis, valores mobiliários, serviços técnicos',
           'Transportes e comunicações',
           'Serviços de alojamento, alimentação, reparação, manutenção, redação...',
           'Serviços médicos, odontológicos e veterinários', 'Ensino',
           'Administração pública direta e autárquica',
           'Agricultura, silvicultura, criação de animais, extrativismo vegetal'],
          dtype='object')




```python
#melt columns 
df_melted_2005 = pd.melt(dataframe_2005, id_vars =['Bairros'], value_vars =['Extrativa mineral', 'Minerais não-metálicos',
       'Indústria metalúrgica', 'Indústria mecânica',
       'Indústria do material elétrico e de comunicações',
       'Indústria de material de transporte',
       'Indústria da madeira e do mobiliário',
       'Indústria do papel, papelão, editorial e gráfica',
       'Indústria da borracha, fumo, couros, peles, similares e diversas',
       'Indústria química de produtos farmacêuticos, veterinários, perfumaria...',
       'Indústria têxtil do vestuário e artefatos de tecidos',
       'Indústria de calçados',
       'Indústria de produtos alimentícios, bebidas e álcool etílico',
       'Serviços industriais de utilidade pública ', 'Construção civil',
       'Comércio varejista', 'Comércio atacadista',
       'Instituições de crédito, seguros e capitalização',
       'Comércio e administração de imóveis, valores mobiliários, serviços técnicos',
       'Transportes e comunicações',
       'Serviços de alojamento, alimentação, reparação, manutenção, redação...',
       'Serviços médicos, odontológicos e veterinários', 'Ensino',
       'Administração pública direta e autárquica',
       'Agricultura, silvicultura, criação de animais, extrativismo vegetal'], 
        var_name ='Atividade-Economica', value_name = "Numero-Empregos") 

#find and replace "-" to 0 
df_melted_2005['Numero-Empregos'].replace('-', 0, inplace=True)
```

Pronto, agora temos os dados estruturados por bairros e atividade econômica. 


```python
#check df new structure order by bairros
df_melted_2005.sort_values(by=['Bairros'])
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
      <th>Bairros</th>
      <th>Atividade-Economica</th>
      <th>Numero-Empregos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2003</th>
      <td>Abolição</td>
      <td>Indústria de produtos alimentícios, bebidas e ...</td>
      <td>14</td>
    </tr>
    <tr>
      <th>383</th>
      <td>Abolição</td>
      <td>Indústria metalúrgica</td>
      <td>17</td>
    </tr>
    <tr>
      <th>2813</th>
      <td>Abolição</td>
      <td>Instituições de crédito, seguros e capitalização</td>
      <td>88</td>
    </tr>
    <tr>
      <th>2489</th>
      <td>Abolição</td>
      <td>Comércio varejista</td>
      <td>1451</td>
    </tr>
    <tr>
      <th>1031</th>
      <td>Abolição</td>
      <td>Indústria da madeira e do mobiliário</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>866</th>
      <td>Água Santa</td>
      <td>Indústria de material de transporte</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3296</th>
      <td>Água Santa</td>
      <td>Serviços de alojamento, alimentação, reparação...</td>
      <td>9</td>
    </tr>
    <tr>
      <th>218</th>
      <td>Água Santa</td>
      <td>Minerais não-metálicos</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2000</th>
      <td>Água Santa</td>
      <td>Indústria de produtos alimentícios, bebidas e ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3134</th>
      <td>Água Santa</td>
      <td>Transportes e comunicações</td>
      <td>287</td>
    </tr>
  </tbody>
</table>
<p>4050 rows × 3 columns</p>
</div>



Agora precisamos adicionar as colunas referentes as Áreas de Planejamento (AP), Regiões de Planejamento (RP), Regiões Admininistrativas (RA). Para isso também tive que fazer um trabalho um pouco manual estruturando elas em um csv separado. 


```python
#import bairros, areas de planejamento, regiao de planejamento e regiao administrativa
file_path_bairros = "raw_data/AP-RP-RA-Bairros.csv"
bairros =  pd.read_csv(file_path_bairros)
```


```python
#check
bairros.head(3)
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
      <th>AP</th>
      <th>RP</th>
      <th>RA</th>
      <th>Bairros</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Gamboa</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Santo Cristo</td>
    </tr>
  </tbody>
</table>
</div>



As duas tabelas que temos a nossa disposição possuem a coluna Bairros, nos pemitindo juntá-las usando essa coluna como indexador. Além disso, precisamos adicionar o ano referente a aba que estamos trabalhando


```python
#merge empregos with bairros on column Bairros
complete_2005 = pd.merge(df_melted_2005, bairros, on='Bairros')
```


```python
complete_2005.head()
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
      <th>Bairros</th>
      <th>Atividade-Economica</th>
      <th>Numero-Empregos</th>
      <th>AP</th>
      <th>RP</th>
      <th>RA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Saúde</td>
      <td>Extrativa mineral</td>
      <td>0</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Saúde</td>
      <td>Minerais não-metálicos</td>
      <td>5</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saúde</td>
      <td>Indústria metalúrgica</td>
      <td>2</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Saúde</td>
      <td>Indústria mecânica</td>
      <td>9</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Saúde</td>
      <td>Indústria do material elétrico e de comunicações</td>
      <td>11</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
    </tr>
  </tbody>
</table>
</div>




```python
#add year to dataframe and reorder columns 
complete_2005['Ano'] = '2005'
complete_2005 = complete_2005[['Ano', 'AP', 'RP', 'RA', 'Bairros', 'Atividade-Economica', 'Numero-Empregos']]
```

Pronto! Agora temos os mesmo dados da planilha disponibilizada pela prefeitura em uma estrutura mais amigável para análises. 


```python
#check
complete_2005
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
      <th>Ano</th>
      <th>AP</th>
      <th>RP</th>
      <th>RA</th>
      <th>Bairros</th>
      <th>Atividade-Economica</th>
      <th>Numero-Empregos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Extrativa mineral</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Minerais não-metálicos</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria metalúrgica</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria mecânica</td>
      <td>9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria do material elétrico e de comunicações</td>
      <td>11</td>
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
    </tr>
    <tr>
      <th>3995</th>
      <td>2005</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Serviços de alojamento, alimentação, reparação...</td>
      <td>120</td>
    </tr>
    <tr>
      <th>3996</th>
      <td>2005</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Serviços médicos, odontológicos e veterinários</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3997</th>
      <td>2005</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Ensino</td>
      <td>50</td>
    </tr>
    <tr>
      <th>3998</th>
      <td>2005</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Administração pública direta e autárquica</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3999</th>
      <td>2005</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Agricultura, silvicultura, criação de animais,...</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>4000 rows × 7 columns</p>
</div>



### Iterando o mesmo processo em para todos os anos / abas

Uma vez válidada as etapas de transformação / limpeza dos dados para uma aba precisamos iterar os mesmo processos para todos os outros anos / abas. 



```python
#loop all years in every sheet
#create empty dataframe
empregados = pd.DataFrame()

#every sheets name related to its year
anos =  ['2005', '2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015', '2016', '2017', '2018']

#repeat all the steps looping every year

for ano in anos:
    df = pd.read_excel(file_path_empregados, sheet_name=ano, header=3,  nrows=224, usecols="A:Z")
    df.drop([0, 1, 2, 3, 4, 5, 10, 12, 17, 22, 24, 26, 27, 28, 29, 38, 41, 49, 51, 52, 56, 61, 62, 63, 64,
        69, 71, 72, 89, 91, 92, 99, 113, 114, 121, 123, 124, 128, 133, 134, 139, 146, 147, 163, 164, 165, 
         166, 177, 179, 180, 189, 190, 191, 192, 197, 204, 205, 211, 212, 216, 217, 221], inplace=True)
    df_melt = pd.melt(df, id_vars =['Bairros'], value_vars =['Extrativa mineral', 'Minerais não-metálicos',
       'Indústria metalúrgica', 'Indústria mecânica',
       'Indústria do material elétrico e de comunicações',
       'Indústria de material de transporte',
       'Indústria da madeira e do mobiliário',
       'Indústria do papel, papelão, editorial e gráfica',
       'Indústria da borracha, fumo, couros, peles, similares e diversas',
       'Indústria química de produtos farmacêuticos, veterinários, perfumaria...',
       'Indústria têxtil do vestuário e artefatos de tecidos',
       'Indústria de calçados',
       'Indústria de produtos alimentícios, bebidas e álcool etílico',
       'Serviços industriais de utilidade pública ', 'Construção civil',
       'Comércio varejista', 'Comércio atacadista',
       'Instituições de crédito, seguros e capitalização',
       'Comércio e administração de imóveis, valores mobiliários, serviços técnicos',
       'Transportes e comunicações',
       'Serviços de alojamento, alimentação, reparação, manutenção, redação...',
       'Serviços médicos, odontológicos e veterinários', 'Ensino',
       'Administração pública direta e autárquica',
       'Agricultura, silvicultura, criação de animais, extrativismo vegetal'], 
        var_name ='Atividade-Economica', value_name = "Numero-Empregos")
    df_melt['Numero-Empregos'].replace('-', 0, inplace=True)
    df_merged = pd.merge(df_melt, bairros, on='Bairros')
    df_merged['Ano'] = ano
    df_merged = df_merged[['Ano', 'AP', 'RP', 'RA', 'Bairros', 'Atividade-Economica', 'Numero-Empregos']]
    empregados = empregados.append(df_merged,ignore_index=True)
```

Feito! Todos as planilhas em uma única base de dados com os dados divídios por Ano, AP, RP, RA, Bairro, Atividade Econômica e Número de empregos. 
Podemos salvar em csv para futuras análises.


```python
empregados
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
      <th>Ano</th>
      <th>AP</th>
      <th>RP</th>
      <th>RA</th>
      <th>Bairros</th>
      <th>Atividade-Economica</th>
      <th>Numero-Empregos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Extrativa mineral</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Minerais não-metálicos</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria metalúrgica</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria mecânica</td>
      <td>9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria do material elétrico e de comunicações</td>
      <td>11</td>
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
    </tr>
    <tr>
      <th>55995</th>
      <td>2018</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Serviços de alojamento, alimentação, reparação...</td>
      <td>118</td>
    </tr>
    <tr>
      <th>55996</th>
      <td>2018</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Serviços médicos, odontológicos e veterinários</td>
      <td>12</td>
    </tr>
    <tr>
      <th>55997</th>
      <td>2018</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Ensino</td>
      <td>201</td>
    </tr>
    <tr>
      <th>55998</th>
      <td>2018</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Administração pública direta e autárquica</td>
      <td>0</td>
    </tr>
    <tr>
      <th>55999</th>
      <td>2018</td>
      <td>Área de Planejamento 5</td>
      <td>Região de Planejamento 5.4</td>
      <td>XXVI Guaratiba</td>
      <td>Pedra de Guaratiba</td>
      <td>Agricultura, silvicultura, criação de animais,...</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>56000 rows × 7 columns</p>
</div>




```python
#save to df to csv
empregados.to_csv(r'/home/blackmamba/data_analysis/atividades_econ_rio/clean_data/atividades_econ_rj_ap_bairros_clean.csv', index = False)
```

### Breve análise

Essa texto foi focado na limpeza dos dados e não em sua análise. Entretando, me proponho a fazer uma breve investigação.

A cidade do Rio de Janeiro possui um problema grave de planejamento urbano onde o centro da cidade concentra maior parte dos empregos ofertados porém baixíssima oportunidade de moradia. Uma breve análise que me despertou curiosidade foi justamente analisar essa hipótese, cruzando os dados de população residente com oferta de número de empregos por Região Administrativa. 

Chamo de breve análise pois 1) os dados disponibilizados são frágeis (i.e população residente é uma estimativa, Rocinha aparece com 0 empregos), 2) setor informal é bastante presente no Rio de Janeiro e 3) o cruzamento de dados é apenar um primeiro indicador para testar uma hipóteses.  



```python
#data path
file_path_empregos = "/home/blackmamba/data_analysis/atividades_econ_rio/clean_data/atividades_econ_rj_ap_bairros_clean.csv"
file_path_populacao = "/home/blackmamba/data_analysis/atividades_econ_rio/clean_data/População Residente e Estimada.csv"

#read csv as dataframe
df_empregos = pd.read_csv(file_path_empregos)
df_populacao = pd.read_csv(file_path_populacao)
```


```python
#check
df_empregos.head()
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
      <th>Ano</th>
      <th>AP</th>
      <th>RP</th>
      <th>RA</th>
      <th>Bairros</th>
      <th>Atividade-Economica</th>
      <th>Numero-Empregos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Extrativa mineral</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Minerais não-metálicos</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria metalúrgica</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria mecânica</td>
      <td>9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>Área de Planejamento 1</td>
      <td>Região de Planejamento 1.1</td>
      <td>I Portuária</td>
      <td>Saúde</td>
      <td>Indústria do material elétrico e de comunicações</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check
df_populacao.head()
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
      <th>RAS</th>
      <th>2000</th>
      <th>2010</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2020</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>I Portuária</td>
      <td>39.97</td>
      <td>48.66</td>
      <td>51.41</td>
      <td>52.00</td>
      <td>52.55</td>
      <td>53.09</td>
      <td>55.07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>II Centro</td>
      <td>39.14</td>
      <td>41.14</td>
      <td>41.78</td>
      <td>41.91</td>
      <td>42.04</td>
      <td>42.16</td>
      <td>42.62</td>
    </tr>
    <tr>
      <th>2</th>
      <td>III Rio Comprido</td>
      <td>73.66</td>
      <td>78.98</td>
      <td>80.66</td>
      <td>81.01</td>
      <td>81.35</td>
      <td>81.68</td>
      <td>82.89</td>
    </tr>
    <tr>
      <th>3</th>
      <td>IV Botafogo</td>
      <td>238.90</td>
      <td>239.73</td>
      <td>239.99</td>
      <td>240.05</td>
      <td>240.10</td>
      <td>240.15</td>
      <td>240.34</td>
    </tr>
    <tr>
      <th>4</th>
      <td>IX Vila Isabel</td>
      <td>186.01</td>
      <td>189.31</td>
      <td>190.35</td>
      <td>190.57</td>
      <td>190.79</td>
      <td>190.99</td>
      <td>191.74</td>
    </tr>
  </tbody>
</table>
</div>




```python
#filter 2016
df_pop_2016 = df_populacao[['RAS', '2016']]
df_emp_2016 = df_empregos[df_empregos['Ano'] == 2016]

df_emp_2016 = df_emp_2016[['RA', 'Numero-Empregos']]

df_pop_2016.columns = ['RAS', 'Populacao']
df_emp_2016.columns = ['RAS', 'Numero-Empregos']
```


```python
#groupby 2016
df_emp_2016 = df_emp_2016.groupby('RAS').sum().reset_index()
```


```python
#merge on RAs
df_complete = pd.merge(df_pop_2016, df_emp_2016, on='RAS')
```


```python
#add proportion columns
df_complete['Pop %'] = (df_complete.Populacao / df_complete.Populacao.sum()) * 100
df_complete['Emprego %'] = (df_complete['Numero-Empregos'] / df_complete['Numero-Empregos'].sum()) * 100
```


```python
#check
df_complete.head()
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
      <th>RAS</th>
      <th>Populacao</th>
      <th>Numero-Empregos</th>
      <th>Pop %</th>
      <th>Emprego %</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>I Portuária</td>
      <td>53.09</td>
      <td>108607</td>
      <td>0.809789</td>
      <td>4.783663</td>
    </tr>
    <tr>
      <th>1</th>
      <td>II Centro</td>
      <td>42.16</td>
      <td>500209</td>
      <td>0.643072</td>
      <td>22.032019</td>
    </tr>
    <tr>
      <th>2</th>
      <td>III Rio Comprido</td>
      <td>81.68</td>
      <td>140159</td>
      <td>1.245876</td>
      <td>6.173391</td>
    </tr>
    <tr>
      <th>3</th>
      <td>IV Botafogo</td>
      <td>240.15</td>
      <td>148052</td>
      <td>3.663040</td>
      <td>6.521043</td>
    </tr>
    <tr>
      <th>4</th>
      <td>IX Vila Isabel</td>
      <td>190.99</td>
      <td>51739</td>
      <td>2.913196</td>
      <td>2.278877</td>
    </tr>
  </tbody>
</table>
</div>




```python
import matplotlib.pyplot as plt
plt.style.use('fivethirtyeight')
plt.figure(figsize=(16,8)) 
plt.xticks(rotation=80)

plt.plot(df_complete['RAS'], df_complete['Pop %'], label='População %')
plt.plot(df_complete['RAS'], df_complete['Emprego %'], label='Emprego %')

plt.legend()
plt.show()
```


![grafico]({{site.baseurl}}/assets/img/datario/ra-pop-emp.png)


Em uma primeira leitura dos dados a hipótese previamente discutida parece ser verdadeira!

Link para [github](https://github.com/pbragamiranda/atividade-economica-rj) 