---
layout: post
title: análise exploratória dos dados de viagens a serviço do portal da transparência federal
date: 2019-09-20 00:00:00 +0300
description:  # Add post description (optional)
img: test.png # Add image post (optional)
tags: [pandas, dados_abertos, EDA] # add tag
---
analise exploratória usando pandas, numpy, matplotlib e seaborn

Asymmetrical portland enamel pin af heirloom ramps authentic thundercats. Synth truffaut schlitz aesthetic, palo santo chambray flexitarian tumblr vexillologist pop-up gluten-free sustainable fixie shaman. Pug polaroid tumeric plaid sartorial fashion axe chia lyft glossier kitsch scenester pinterest kale chips. Blog etsy umami fashion axe shoreditch. Prism chambray heirloom, drinking vinegar portland paleo slow-carb. Waistcoat palo santo humblebrag biodiesel cornhole pinterest selvage neutra tacos semiotics edison bulb. Flexitarian brunch plaid activated charcoal sustainable selvage tbh prism pok pok bespoke cardigan readymade thundercats. Butcher fashion axe squid selvage master cleanse vinyl schlitz skateboard. Lomo shaman man bun keffiyeh asymmetrical listicle. Kickstarter trust fund fanny pack post-ironic wayfarers swag kitsch. Shaman pug kale chips meh squid.

###  Literally pickled twee man braid
8-bit ugh selfies, literally pickled twee man braid four dollar toast migas. Slow-carb mustache meggings pok pok. Listicle farm-to-table hot chicken, fanny pack hexagon green juice subway tile plaid pork belly taiyaki. Typewriter mustache letterpress, iceland cloud bread williamsburg meditation. Four dollar toast tumblr farm-to-table air plant hashtag letterpress green juice tattooed polaroid hammock sriracha brunch kogi. Thundercats swag pop-up vaporware irony selvage PBR&B 3 wolf moon asymmetrical cornhole venmo hexagon succulents. Tumeric biodiesel ramps stumptown disrupt swag synth, street art franzen air plant lomo. Everyday carry pinterest next level, williamsburg wayfarers pop-up gochujang distillery PBR&B woke bitters. Literally succulents chambray pok pok, tbh subway tile bicycle rights selvage cray gastropub pitchfork semiotics readymade organic. Vape flexitarian tumblr raclette organic direct trade. Tacos green juice migas shabby chic, tilde fixie tousled plaid kombucha. +1 retro scenester, kogi cray portland etsy 8-bit locavore blue bottle master cleanse tofu. PBR&B adaptogen chartreuse knausgaard palo santo intelligentsia.

![Macbook]({{site.baseurl}}/assets/img/mac.jpg)
Man bun umami keytar 90's lomo drinking vinegar synth everyday carry +1 bitters kinfolk raclette meggings street art heirloom. Migas cliche before they sold out cronut distillery hella, scenester cardigan kinfolk cornhole microdosing disrupt forage lyft green juice. Tofu deep v food truck live-edge edison bulb vice. Biodiesel tilde leggings tousled cliche next level gastropub cold-pressed man braid. Lyft humblebrag squid viral, vegan chicharrones vice kinfolk. Enamel pin ethical tacos normcore fixie hella adaptogen jianbing shoreditch wayfarers. Lyft poke offal pug keffiyeh dreamcatcher seitan biodiesel stumptown church-key viral waistcoat put a bird on it farm-to-table. Meggings pitchfork master cleanse pickled venmo. Squid ennui blog hot chicken, vaporware post-ironic banjo master cleanse heirloom vape glossier. Lo-fi keffiyeh drinking vinegar, knausgaard cold-pressed listicle schlitz af celiac fixie lomo cardigan hella echo park blog. Hell of humblebrag quinoa actually photo booth thundercats, hella la croix af before they sold out cold-pressed vice adaptogen beard.

```python
#@pbragamiranda
#16_10_2019

#carrega as bibliotecas necessárias
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib
import matplotlib.pyplot as plt
import zipfile
import squarify
%matplotlib inline

#define estilo grafico
plt.style.use('dark_background')

#define as opções de display
pd.set_option("display.precision", 2)  # casas decimais
pd.set_option('display.max_colwidth', -1)  #-1 pra ver tudo

#load os dados
zf = zipfile.ZipFile('/home/blackmamba/Documents/analise_dados/viagens_gov_federal/dados/2019_Viagens.zip') 
viagens = pd.read_csv(zf.open("2019_Viagem.csv"), sep=";", encoding="latin1")
```