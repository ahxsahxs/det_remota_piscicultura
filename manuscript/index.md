# Aquisição e Pré-processamento, Treinamento do Modelo e Pos processamento

## Aquisição das Imagens

A obtenção de dados para este estudo baseou-se em imagens do satélite SENTINEL 2 L2A, adquiridas por meio da Interface de Serviço de Mapas Web (API WMS) do Sentinel Hub. Optou-se pela resolução espacial de 10 metros, permitindo um detalhamento adequado para a identificação de tanques de piscicultura na região estudada. O conjunto de dados incluiu composições em Cor Verdadeira (bandas B04, B03, B02), Falsa Cor (bandas B12, B11, B04), além dos índices espectrais NDMI (Índice de Umidade por Diferença Normalizada), MSAVI2 (Índice de Vegetação Ajustado ao Solo Modificado), NDWI (Índice de Água por Diferença Normalizada) e NDVI (Índice de Vegetação por Diferença Normalizada).

Devido à limitação no tamanho máximo da imagem fornecida pelo serviço, foi necessário subdividir a área de estudo em uma grade quadrada com células de 20 quilômetros de lado. Esta fragmentação resultou em um conjunto de 21 imagens para cada período analisado, cobrindo integralmente o município de Ariquemes e seu entorno. Para cada célula desta grade, foram obtidas todas as composições e índices espectrais anteriormente mencionados, abrangendo o mês de agosto entre os anos de 2017 e 2024. A escolha do mês de agosto justifica-se por corresponder ao período de menor precipitação pluviométrica na região amazônica, o que proporciona imagens com menor cobertura de nuvens, fator crítico para a qualidade dos dados em análises por detecção remota na região.

![Figura 1: Divisão da área de estudo em grade de células com 20km de lado, mostrando a cobertura das imagens SENTINEL 2 utilizadas na análise. A imagem apresenta os limites do município de Ariquemes destacados e a grade sobreposta à composição colorida verdadeira.](images/fig_1_grid.png)

O período temporal analisado (2017-2024) foi selecionado para possibilitar a identificação de tendências na expansão da piscicultura na região, aproveitando a disponibilidade contínua de imagens do SENTINEL 2 com características radiométricas e espaciais constantes, o que permite comparações interanuais consistentes.

## Pré-processamento

A etapa de pré-processamento das imagens visou principalmente uniformizar os dados, corrigir variações atmosféricas e preparar o material para as análises subsequentes. Para cada célula da grade, as diferentes composições e índices espectrais foram agrupados em um único arquivo raster contendo 10 bandas, facilitando o manuseio e análise integrada dos dados.

O pré-processamento foi estruturado em duas fases principais: padronização intra-anual, buscando uniformidade entre células do mesmo ano, e padronização interanual, visando comparabilidade entre anos diferentes. Os procedimentos detalhados são descritos a seguir.

Para a padronização intra-anual, em cada ano analisado, aplicou-se o seguinte protocolo: primeiramente, realizou-se uma equalização do tipo "contrast stretching" nas bandas 1, 2 e 3 (composição colorida verdadeira) e nas bandas 4, 5 e 6 (composição falsa cor), separadamente. Os valores digitais foram reajustados para o intervalo compreendido entre o percentil 2 e o percentil 98 de cada célula, removendo valores extremos que poderiam comprometer a visualização. As bandas correspondentes aos índices espectrais (7, 8, 9 e 10 - NDMI, MSAVI2, NDWI e NDVI, respectivamente) não foram submetidas a ajustes, preservando seus valores originais, uma vez que a modificação destes poderia comprometer as análises estatísticas zonais subsequentes, essenciais para caracterização da vegetação circundante aos tanques de piscicultura.

Para minimizar efeitos atmosféricos e variações radiométricas inerentes ao sensor do satélite, especialmente considerando que múltiplas cenas são necessárias para cobrir a área de estudo, implementou-se um procedimento de correspondência de histogramas ("histogram matching"). Uma célula foi selecionada aleatoriamente como referência, e as demais células tiveram suas distribuições de níveis digitais ajustadas para corresponder à distribuição da célula de referência. Este procedimento proporcionou uniformidade radiométrica entre as diferentes células que compõem a área de estudo dentro de um mesmo ano.

Após a equalização das imagens, os rasters de todas as células foram combinados em um mosaico abrangendo toda a área de estudo para cada ano analisado, como ilustrado na figura abaixo.

![Figura 2: Processo de mosaicagem das células após equalização. A imagem mostra a composição colorida verdadeira para o ano de 2017, evidenciando a continuidade visual entre as células após o pré-processamento.](images/fig_2_raw_sentinel_2017.png)

Para a padronização interanual, estabeleceu-se o ano de 2017 como referência temporal para o treinamento do modelo de classificação. Visando reduzir variações temporais decorrentes de efeitos climáticos e fenológicos, aplicou-se novamente o procedimento de correspondência de histogramas, desta vez equalizando a distribuição dos níveis digitais dos anos de 2018 a 2024 com a distribuição de níveis digitais do ano de 2017. Esta etapa foi fundamental para garantir a consistência temporal nas análises, permitindo que o modelo treinado com dados de 2017 fosse aplicado com confiabilidade aos dados dos anos subsequentes.

Todas as operações de pré-processamento foram implementadas utilizando a biblioteca Scikit-Image, componente do ecossistema científico da linguagem de programação Python, reconhecida por sua eficiência no processamento de imagens e ampla utilização em estudos de sensoriamento remoto.

![Figura 3: Comparação visual das imagens antes e após o pré-processamento completo. O painel superior mostra recortes das imagens originais para os anos de 2017, 2020 e 2023 em composição colorida verdadeira, enquanto o painel inferior apresenta as mesmas áreas após os procedimentos de equalização intra-anual e interanual, evidenciando a uniformidade visual alcançada.](images/fig_3_comparison.png)

A metodologia de pré-processamento adotada permitiu obter um conjunto de dados temporalmente e espacialmente consistente, essencial para a aplicação de técnicas de classificação automática visando a identificação de tanques de piscicultura e análise de sua evolução temporal no município de Ariquemes, Rondônia.

Vou desenvolver as seções de treinamento do modelo e pós-processamento para seu artigo acadêmico sobre detecção de tanques de piscicultura em Ariquemes, seguindo as orientações fornecidas.

## Treinamento do Modelo

O processo de classificação supervisionada requer dados de referência para o treinamento do algoritmo. Neste estudo, foram identificados polígonos representativos das diferentes classes de uso e cobertura do solo presentes na área de estudo. A nomenclatura adotada seguiu uma generalização da classificação utilizada pelo projeto MapBiomas, garantindo compatibilidade com outras iniciativas de mapeamento no território brasileiro. 

A seleção dos polígonos de treinamento foi realizada mediante interpretação visual das imagens SENTINEL 2 do ano de 2017, utilizando a composição colorida verdadeira e falsa cor, além dos índices espectrais previamente mencionados. Buscou-se amostrar áreas representativas de cada classe, distribuídas espacialmente em toda a extensão da área de estudo, a fim de capturar a variabilidade espectral existente. A tabela 1 apresenta a distribuição quantitativa dos polígonos de treinamento por classe.

**Tabela 1.** Distribuição dos polígonos de treinamento por classe de uso e cobertura do solo.

| lulc             |   N. Polygons |   Area (ha) |
|:-----------------|--------------:|------------:|
| aquaculture      |          1206 |     618,223 |
| farming          |            19 |     639,784 |
| forest           |             6 |    1247,59  |
| non_vegetated    |            23 |    4990,48  |
| river_lake_ocean |             9 |     286,562 |

![Figura 4: Distribuição espacial dos polígonos de treinamento sobre a área administrativa de Ariquemes e a grade. Os polígonos estão simbolizados por cores distintas para cada classe.](images/fig_4_training_poly.png)

Para a classificação, optou-se pela implementação do modelo de Mistura Gaussiana (Gaussian Mixture Model - GMM), executado por meio do complemento dzetsaka do Sistema de Informações Geográficas QGIS. A escolha deste algoritmo baseou-se em sua capacidade de modelar a distribuição estatística multivariada dos valores espectrais das diferentes classes, sendo particularmente eficiente na discriminação de corpos d'água artificiais, como os tanques de piscicultura.

O conjunto de dados de treinamento foi composto pelos polígonos anteriormente descritos e pelo mosaico de imagens pré-processadas do ano de 2017. Adotou-se uma abordagem de validação cruzada, na qual 50% dos pixels presentes nos polígonos de treinamento foram utilizados para o ajuste do modelo, enquanto os 50% restantes foram reservados para avaliação do desempenho da classificação. Esta abordagem permitiu quantificar a acurácia do modelo sem a necessidade de dados de referência externos.

Os resultados da validação são expressos na matriz de confusão apresentada na tabela 2, onde as colunas representam as classes preditas pelo modelo e as linhas correspondem às classes reais dos pixels de validação.

**Tabela 2.** Matriz de confusão do modelo de Mistura Gaussiana.

|                  |   aquacultura |   rio_lago_oceano |   floresta |   agricultura |   não_vegetado |
|:-----------------|--------------:|-------------------:|---------:|-------------:|----------------:|
| aquacultura      |          5456 |                396 |       15 |            0 |            8021 |
| rio_lago_oceano  |           660 |              13414 |        0 |            0 |            1568 |
| floresta         |             0 |                 23 |    59120 |           86 |            1527 |
| agricultura      |             0 |                  0 |     1186 |        30727 |           36686 |
| não_vegetado     |            92 |                 78 |      238 |          238 |          194539 |

A partir da matriz de confusão, calcularam-se métricas de avaliação do desempenho do modelo, apresentadas no relatório de classificação a seguir:

**Tabela 3.** Relatório de Classificação para o modelo gaussiano.

| Classe          | Precisão | Recall    | Medida-F | Suporte  |
|-----------------|----------|-----------|----------|----------|
| aquacultura     | 0,88     | 0,39      | 0,54     | 13888    |
| rio_lago_oceano | 0,96     | 0,86      | 0,91     | 15642    |
| floresta        | 0,98     | 0,97      | 0,97     | 60756    |
| agricultura     | 0,99     | 0,45      | 0,62     | 68599    |
| não_vegetado    | 0,80     | 1,00      | 0,89     | 195185   |
| média macro     | 0,92     | 0,73      | 0,79     | 354070   |
| média ponderada | 0,88     | 0,86      | 0,84     | 354070   |


A análise das métricas de desempenho revela que o modelo apresentou acurácia global de 86%, considerada satisfatória para os objetivos do estudo. Observa-se que a classe "floresta" obteve os melhores resultados individuais, com precisão e recall superiores a 97%, indicando elevada capacidade de discriminação desta classe. A classe "rio_lago_oceano" também apresentou bom desempenho, particularmente importante por sua semelhança espectral com a classe alvo "aquacultura".

A classe "aquacultura" apresentou alta precisão (88%), porém recall moderada (39%), indicando que o modelo possui baixa taxa de falsos positivos, mas considerável taxa de falsos negativos. Isto sugere uma abordagem conservadora na identificação dos tanques de piscicultura, privilegiando a confiabilidade das áreas identificadas em detrimento da completude do mapeamento.

Foram testados algoritmos alternativos, nomeadamente Florestas Aleatórias (Random Forests) e K-Vizinhos Mais Próximos (K-Nearest Neighbors). Contudo, estes modelos apresentaram elevada sensibilidade ao ruído presente nas imagens, resultando em classificações fragmentadas e inconsistentes. Por este motivo, optou-se pela utilização do modelo de Mistura Gaussiana, que demonstrou maior robustez frente às características dos dados disponíveis.

## Pós-processamento

A etapa de pós-processamento visa refinar os resultados da classificação, corrigindo imperfeições e extraindo informações derivadas relevantes para a análise da distribuição espacial dos tanques de piscicultura. O procedimento foi estruturado em duas vertentes principais: a filtragem e vetorização dos resultados da classificação, e a delimitação das áreas circundantes aos tanques identificados.

Inicialmente, foram definidas áreas de exclusão nas quais o modelo de classificação não deveria ser aplicado. Estas áreas compreendem o perímetro urbano do município de Ariquemes, zonas de mineração ativa e uma faixa marginal de 300 metros ao longo dos cursos d'água naturais. A exclusão destas áreas justifica-se pelo elevado potencial de classificação errônea, seja pela similaridade espectral entre estruturas artificiais urbanas e tanques de piscicultura, seja pela ocorrência de acúmulos de água em áreas de mineração, ou ainda pela presença de áreas alagáveis naturais às margens dos rios.

![Figura 5: Delimitação das áreas de exclusão, destacando em vermelho o perímetro urbano de Ariquemes, em laranja as zonas de mineração e em azul claro o buffer de 300 metros ao longo dos cursos d'água.](images/fig_5_restricted_areas.png)

Para cada ano do período analisado (2017-2024), o procedimento de pós-processamento seguiu a seguinte sequência metodológica: primeiramente, aplicou-se o modelo de Mistura Gaussiana ao mosaico de imagens pré-processadas, gerando um mapa matricial de uso e cobertura do solo com cinco classes. Visando reduzir o ruído característico da classificação pixel a pixel, implementou-se um filtro GDAL Sieve, que substitui áreas classificadas com menos de 20 pixels contíguos (equivalente a 0,2 hectares) pela classe predominante em seu entorno. Esta operação contribuiu significativamente para a homogeneização da classificação, eliminando pixels isolados e pequenos agrupamentos que provavelmente representam erros de classificação.

O raster filtrado foi então convertido para o formato vetorial, preservando-se os atributos de classe de cada polígono. Deste conjunto, foram extraídos exclusivamente os polígonos classificados como "aquacultura" (classe 1). Na sequência, realizou-se uma operação de sobreposição espacial para remover qualquer polígono de aquacultura que intersectasse as áreas de exclusão previamente definidas. O resultado desta etapa constitui o produto parcial denominado "Mapa de Tanques de Piscicultura" para cada ano analisado.

![Figura 6: Fluxograma ilustrativo do processo de pós-processamento, desde a classificação original até a obtenção dos mapas de tanques de piscicultura e áreas circundantes.](images/fig_6_post_process_flow.png)

A segunda vertente do pós-processamento concentrou-se na identificação das áreas circundantes aos tanques de piscicultura, fundamentais para a análise das condições ambientais associadas a estas estruturas. Para cada conjunto de tanques identificados em um determinado ano, foram gerados dois buffers: um com 50 metros e outro com 150 metros de distância a partir do perímetro dos tanques. Cada conjunto de buffers foi submetido a uma operação de dissolução, resultando em um polígono único para cada distância, eliminando-se as sobreposições entre buffers adjacentes.

Mediante operação de diferença entre o buffer de 150 metros e o buffer de 50 metros, obteve-se uma faixa anular com 100 metros de largura circundando as áreas ocupadas por tanques de piscicultura. Este polígono anular foi então fragmentado em áreas disjuntas através da operação "Multipart Polygon to Singlepart Polygons", permitindo a análise individualizada de diferentes setores do entorno dos tanques. O resultado desta etapa constitui o produto parcial denominado "Mapa de Áreas Circundantes aos Tanques de Piscicultura" para cada ano analisado.

![Figura 7: Exemplo de identificação das áreas circundantes aos tanques de piscicultura para o ano de 2021. A imagem ilustra: (A) os tanques identificados em azul; e (B) a faixa anular resultante da diferença entre os buffers, destacando as áreas circundantes aos tanques, em verde escuro.](images/fig_7_riparian_buffers.png)


Os mapas resultantes deste procedimento possibilitam não apenas a quantificação e análise da evolução temporal da piscicultura no município de Ariquemes, mas também a caracterização do contexto ambiental no qual estas atividades se inserem, aspectos essenciais para a compreensão da dinâmica desta importante cadeia produtiva na região amazônica.