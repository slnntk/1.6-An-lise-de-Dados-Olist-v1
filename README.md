# Análise de Dados Olist - Documentação Detalhada

Este documento fornece uma explicação minuciosa das análises realizadas no conjunto de dados da Olist, focando especificamente em duas perguntas de negócio fundamentais. O objetivo é explicar de forma clara e detalhada todas as decisões técnicas e metodológicas tomadas para alguém que não possui conhecimento prévio sobre a análise.

## Índice
1. [Visão Geral do Dataset](#visão-geral-do-dataset)
2. [Análise 1: Relação entre Tempo de Entrega e Avaliação do Cliente](#análise-1-relação-entre-tempo-de-entrega-e-avaliação-do-cliente)
3. [Análise 2: Top 5 Categorias de Produtos Mais Vendidas](#análise-2-top-5-categorias-de-produtos-mais-vendidas)
4. [Estrutura do Código](#estrutura-do-código)
5. [Conclusões e Insights de Negócio](#conclusões-e-insights-de-negócio)

## Visão Geral do Dataset

O dataset da Olist contém informações reais de e-commerce no Brasil, incluindo:
- **Pedidos**: Informações sobre compras, datas, status
- **Itens de Pedidos**: Produtos comprados, preços, quantidades
- **Produtos**: Categorias, dimensões, pesos
- **Avaliações**: Notas e comentários dos clientes
- **Pagamentos**: Métodos e valores de pagamento

---

## Análise 1: Relação entre Tempo de Entrega e Avaliação do Cliente

### 🎯 Pergunta de Negócio
**"Qual é a relação entre o tempo de entrega e a nota de avaliação do cliente?"**

### 📍 Localização no Código
- **Células 15-17** do notebook `Analise_Dados1.ipynb`
- **Célula 15**: Introdução da pergunta (markdown)
- **Célula 16**: Processamento dos dados e cálculos estatísticos
- **Célula 17**: Criação das visualizações

### 🔍 Metodologia Detalhada

#### Passo 1: Carregamento e Preparação dos Dados
```python
# Localização no código: Célula 16, linhas 6-7
orders = pd.read_csv(os.path.join(path, 'olist_orders_dataset.csv'))
reviews = pd.read_csv(os.path.join(path, 'olist_order_reviews_dataset.csv'))
```

**Por que foram escolhidos esses datasets?**
- `olist_orders_dataset.csv`: Contém informações sobre datas de compra e entrega
- `olist_order_reviews_dataset.csv`: Contém as notas de avaliação dos clientes

#### Passo 2: Conversão de Datas
```python
# Localização no código: Célula 16, linhas 9-10
orders['order_purchase_timestamp'] = pd.to_datetime(orders['order_purchase_timestamp'])
orders['order_delivered_customer_date'] = pd.to_datetime(orders['order_delivered_customer_date'])
```

**Decisão técnica explicada:**
- Convertemos strings para objetos datetime do pandas para permitir cálculos temporais
- Isso é essencial para calcular a diferença entre datas de forma precisa

#### Passo 3: Junção dos Dados (Merge)
```python
# Localização no código: Célula 16, linha 12
orders_reviews = orders.merge(reviews, on='order_id', how='inner')
```

**Por que usar 'inner join'?**
- Queremos apenas pedidos que têm tanto informação de entrega quanto avaliação
- Isso garante que temos dados completos para a análise
- Evita valores nulos que poderiam distorcer os resultados

#### Passo 4: Filtros de Qualidade dos Dados
```python
# Localização no código: Célula 16, linhas 15-20
delivered_reviews = orders_reviews[
    (orders_reviews['order_status'] == 'delivered') &
    (orders_reviews['order_delivered_customer_date'].notna()) &
    (orders_reviews['order_purchase_timestamp'].notna()) &
    (orders_reviews['review_score'].notna())
].copy()
```

**Critérios de filtro explicados:**
1. **order_status == 'delivered'**: Só analisamos pedidos efetivamente entregues
2. **notna() nas datas**: Removemos registros com datas faltantes
3. **review_score.notna()**: Só consideramos avaliações válidas

**Por que esses filtros são importantes?**
- Garantem a qualidade e confiabilidade da análise
- Evitam conclusões baseadas em dados incompletos
- Focam apenas em transações completas

#### Passo 5: Cálculo do Tempo de Entrega
```python
# Localização no código: Célula 16, linhas 25-28
delivered_reviews['tempo_entrega_dias'] = (
    delivered_reviews['order_delivered_customer_date'] -
    delivered_reviews['order_purchase_timestamp']
).dt.days
```

**Explicação do cálculo:**
- Subtraímos a data de compra da data de entrega
- `.dt.days` converte o resultado para número de dias inteiros
- Criamos uma nova coluna com essa métrica de tempo

#### Passo 6: Tratamento de Outliers
```python
# Localização no código: Célula 16, linhas 30-34
delivered_reviews = delivered_reviews[
    (delivered_reviews['tempo_entrega_dias'] >= 0) &
    (delivered_reviews['tempo_entrega_dias'] <= 100)
]
```

**Por que remover outliers?**
- **Tempo negativo**: Indica erro nos dados (entrega antes da compra)
- **Tempo > 100 dias**: Valores extremos que podem distorcer a análise
- Melhora a qualidade das conclusões estatísticas

#### Passo 7: Cálculos Estatísticos
```python
# Localização no código: Célula 16, linhas 36-42
stats_by_score = delivered_reviews.groupby('review_score')['tempo_entrega_dias'].agg([
    'count', 'mean', 'median', 'std'
]).round(2)

correlation = delivered_reviews['tempo_entrega_dias'].corr(delivered_reviews['review_score'])
```

**Métricas calculadas:**
- **count**: Quantidade de avaliações por nota
- **mean**: Tempo médio de entrega por nota
- **median**: Valor central (menos sensível a outliers)
- **std**: Variabilidade do tempo de entrega
- **correlation**: Força da relação linear entre as variáveis

### 📊 Visualizações Criadas

#### Localização no Código: Célula 17

1. **Scatter Plot** (Gráfico de Dispersão)
   - **Onde**: Célula 17, linhas 5-11
   - **Objetivo**: Mostrar a relação individual entre cada entrega e sua avaliação

2. **Box Plot** (Gráfico de Caixa)
   - **Onde**: Célula 17, linhas 13-19
   - **Objetivo**: Comparar a distribuição do tempo de entrega por nota

3. **Gráfico de Barras** (Tempo Médio)
   - **Onde**: Célula 17, linhas 21-32
   - **Objetivo**: Mostrar claramente o tempo médio por nota de avaliação

4. **Histograma**
   - **Onde**: Célula 17, linhas 34-39
   - **Objetivo**: Entender a distribuição geral dos tempos de entrega

5. **Mapa de Calor** (Heatmap)
   - **Onde**: Célula 17, linhas 41-45
   - **Objetivo**: Visualizar a correlação de forma gráfica

---

## Análise 2: Top 5 Categorias de Produtos Mais Vendidas

### 🎯 Pergunta de Negócio
**"Quais são as 5 categorias de produtos mais vendidas e qual a receita total gerada por cada uma?"**

### 📍 Localização no Código
- **Células 18-20** do notebook `Analise_Dados1.ipynb`
- **Célula 18**: Introdução da pergunta (markdown)
- **Célula 19**: Processamento dos dados e cálculos
- **Célula 20**: Criação das visualizações

### 🔍 Metodologia Detalhada

#### Passo 1: Carregamento dos Datasets
```python
# Localização no código: Célula 19, linhas 6-7
order_items = pd.read_csv(os.path.join(path, 'olist_order_items_dataset.csv'))
products = pd.read_csv(os.path.join(path, 'olist_products_dataset.csv'))
```

**Por que esses datasets?**
- `olist_order_items_dataset.csv`: Contém itens vendidos e preços
- `olist_products_dataset.csv`: Contém as categorias dos produtos

#### Passo 2: Junção dos Dados
```python
# Localização no código: Célula 19, linha 9
items_products = order_items.merge(products, on='product_id', how='left')
```

**Decisão de usar 'left join':**
- Preservamos todos os itens vendidos (tabela da esquerda)
- Alguns produtos podem não ter categoria, mas queremos manter a venda
- Depois removemos apenas os que realmente não têm categoria

#### Passo 3: Limpeza dos Dados
```python
# Localização no código: Célula 19, linha 11
items_products = items_products.dropna(subset=['product_category_name'])
```

**Por que remover categorias nulas?**
- Não podemos analisar categorias que não existem
- Garante que nossa análise seja baseada em dados válidos
- Evita distorções nos resultados

#### Passo 4: Agregação por Categoria
```python
# Localização no código: Célula 19, linhas 13-18
category_stats = items_products.groupby('product_category_name').agg({
    'order_id': 'count',  # Quantidade vendida
    'price': 'sum'        # Receita total
}).round(2)
```

**Explicação das métricas:**
- **'order_id': 'count'**: Conta quantos itens foram vendidos por categoria
- **'price': 'sum'**: Soma o valor total vendido por categoria
- **round(2)**: Arredonda para 2 casas decimais para melhor apresentação

#### Passo 5: Seleção do Top 5
```python
# Localização no código: Célula 19, linhas 22-23
category_stats = category_stats.sort_values('quantidade_vendida', ascending=False)
top_5_categories = category_stats.head(5)
```

**Critério de ordenação:**
- Ordenamos por **quantidade vendida** (não por receita)
- Isso responde especificamente "mais vendidas" em volume
- `.head(5)` seleciona apenas as 5 primeiras

#### Passo 6: Tentativa de Tradução
```python
# Localização no código: Célula 19, linhas 25-33
try:
    category_translation = pd.read_csv(os.path.join(path, 'product_category_name_translation.csv'))
    # ... código de tradução ...
except:
    translation_dict = {}
    # ... usar nomes originais ...
```

**Por que tentar traduzir?**
- As categorias estão em português
- A tradução ajuda na compreensão internacional
- O `try/except` garante que o código funciona mesmo sem o arquivo de tradução

### 📊 Visualizações Criadas

#### Localização no Código: Célula 20

1. **Gráfico de Barras - Quantidade**
   - **Onde**: Célula 20, linhas 4-15
   - **Objetivo**: Comparar visualmente as quantidades vendidas

2. **Gráfico de Barras - Receita**
   - **Onde**: Célula 20, linhas 17-28
   - **Objetivo**: Comparar a receita gerada por categoria

3. **Gráfico de Pizza - Quantidade**
   - **Onde**: Célula 20, linhas 30-35
   - **Objetivo**: Mostrar a proporção de vendas

4. **Gráfico de Pizza - Receita**
   - **Onde**: Célula 20, linhas 37-42
   - **Objetivo**: Mostrar a proporção da receita

---

## Estrutura do Código

### 📁 Arquivos do Repositório
- `Analise_Dados1.ipynb`: Notebook principal com todas as análises
- `README.md`: Este documento de documentação

### 🏗️ Organização das Análises no Notebook

| Célula | Tipo | Conteúdo | Análise |
|--------|------|----------|---------|
| 15 | Markdown | Pergunta sobre tempo de entrega vs avaliação | Análise 1 |
| 16 | Código | Processamento de dados para tempo/avaliação | Análise 1 |
| 17 | Código | Visualizações para tempo/avaliação | Análise 1 |
| 18 | Markdown | Pergunta sobre top 5 categorias | Análise 2 |
| 19 | Código | Processamento de dados para categorias | Análise 2 |
| 20 | Código | Visualizações para categorias | Análise 2 |

### 🔗 Dependências do Código
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import os
```

---

## Conclusões e Insights de Negócio

### 💡 Insights da Análise 1 (Tempo de Entrega vs Avaliação)

**Descobertas esperadas:**
- Entregas mais rápidas tendem a receber melhores avaliações
- A correlação negativa indica que mais tempo = pior nota
- Isso confirma a importância da logística para satisfação do cliente

### 💡 Insights da Análise 2 (Top 5 Categorias)

**Valor para o negócio:**
- Identifica produtos de maior volume de vendas
- Compara volume vs receita por categoria
- Orienta decisões de estoque e marketing
- Ajuda a priorizar categorias mais importantes

### 🎯 Aplicações Práticas

1. **Melhoria da Logística**: Use os insights de tempo de entrega para otimizar prazos
2. **Gestão de Estoque**: Foque nas categorias de maior volume
3. **Estratégia de Marketing**: Promova categorias com melhor retorno
4. **Expectativas do Cliente**: Comunique prazos realistas baseados nos dados

---

## Como Executar a Análise

1. **Pré-requisitos**: Python 3.x, pandas, matplotlib, seaborn
2. **Dados**: Datasets da Olist no diretório correto
3. **Execução**: Execute as células do notebook em ordem
4. **Interpretação**: Use este documento para entender cada resultado

---

*Este documento foi criado para explicar detalhadamente cada decisão técnica e metodológica das análises. Para dúvidas específicas sobre o código, consulte as referências de células indicadas em cada seção.*