# **Apriori Algorithm**
---

The Apriori algorithm,  introduced by R. Agrawal and R. Srikant in 1994, is a classic algorithm in data mining used for mining frequent itemsets and discovering association rules in a database. This page aims to provide an overview of the Apriori algorithm, its method, key parameters, advantages, limitations, and a basic implementation guide using Python.

**${\color{black}\textsf{METHOD}}$**
---
The Apriori algorithm operates by identifying frequent itemsets in a dataset and then extending them to larger itemsets, provided they appear sufficiently frequently in the database. It explores the itemset space in a breadth-first manner. Initially, it identifies frequent itemsets of size 1, then uses these to generate candidate itemsets of size 2, and so on, progressively increasing the size of itemsets and checking their support.

>[!Note]
>A key principle of Apriori is that **${\color{blue}\textsf{a subset of a frequent itemset must also be frequent}}$**. For instance, if the itemset {A, B, C} is frequent, it implies that all of its subsets, such as {A, B}, {A, C}, {B, C}, {A}, {B}, and {C}, must also be frequent. This is because any transaction containing {A, B, C} also contains all its subsets. This principle efficiently reduces the number of support calculations needed and speeds up the discovery of association rules in large datasets.

![image](https://github.com/andytoh78/market-basket-analysis/assets/139482827/4cb58796-2919-4ff9-902d-a5f82857f301)

&nbsp;

**${\color{black}\textsf{KEY PARAMETERS}}$**
---
| **Parameter**             | **Description**                                                               |
|:--------------------------|:------------------------------------------------------------------------------|
| `min_support`             | This is the user-defined threshold (a decimal between 0 and 1) that determines the minimum frequency at which an itemset must be present in the dataset to be considered 'frequent'.|
| `confidence_threshold`    | This is the user-defined minimum level of confidence that an association rule must exceed to be considered significant.
| `lift_threshold`          | This is the user-defined minimum lift value (typically >1) focusing on rules that have a positive relationship between the antecedent and consequent.|

&nbsp;

**${\color{black}\textsf{PROS | CONS}}$**
---
| **Pros**                                          | **Cons**                                                    |
|:--------------------------------------------------|:------------------------------------------------------------|
| Simple to understand and easy to implement.      | Inefficient for large datasets due to multiple database scans and generation of numerous candidate sets.
| Effective for small to medium-sized datasets.      |                                                             |

&nbsp;

**${\color{black}\textsf{IMPLEMENTATION STEPS}}$**
---
We will use [Market_Basket_Optimisation](https://github.com/andytoh78/market-basket-analysis/blob/main/Market_Basket_Optimisation.csv) to demonstrate the application of Apriori algorithm in Market Basket Analysis

```python
# Install machine learning extensions (mlxtend) library
pip install mlxtend
```
```python
# Load dataset and view first 5 rows
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings("ignore")
warnings.filterwarnings("ignore", category=DeprecationWarning)
warnings.filterwarnings("ignore", category=FutureWarning, module="seaborn")
warnings.filterwarnings("ignore", category=Warning)
pd.set_option("display.max_columns", None)
pd.set_option("display.max_colwidth", None)

df = pd.read_csv("/kaggle/input/market-basket-optimisation/Market_Basket_Optimisation.csv", header=None, index_col=None, names=[f"Item_{i}" for i in range(1, 21)])
df.head()
```
Below shows the first five rows of the dataset, where each row represents a basket or collection of items that are purchased together in a single transaction.<br><br>
![image](https://github.com/andytoh78/market-basket-analysis/assets/139482827/edeccbd7-a940-4841-b189-2dd55f2397c9)

```python
# View shape of the dataset
print(f"The dataset contains {df.shape[0]} transactions and each transaction has a maximum of {df.shape[1]} items or less.")
```
The dataset contains 7501 transactions and each transaction has a maximum of 20 items or less.

```python
# Generate transaction lists
txns = df.fillna("").values.tolist()
txns = [[item for item in txn if item != ''] for txn in txns]
txns = [[item.strip() for item in txn] for txn in txns]

# Create a list of unique ids for the transactions
ids = [i + 1 for i in range(len(txns))]

# Initialize an empty list
data =[]
# Iterate through transactions and add them to the DataFrame with IDs
for i, txn in enumerate(txns):
    data.extend([{'TID': ids[i], 'Item': item} for item in txn])

df_txn = pd.DataFrame(data)
df_txn.head(25)
```
> | TID  | Item              | 
> |:---  |:----------------- | 
> | 1    | shrimp            |
> | 1    | almonds           |
> | 1    | avocado           |
> | 1    | vegetables mix    |
> | 1    | green grapes      |
> | 1    | whole weat flour  |
> | 1    | yams              |
> | 1    | cottage cheese    |
> | 1    | energy drink      |
> | 1    | tomato juice      |
> | 1    | low fat yogurt    |
> | 1    | green tea         |
> | 1    | honey             |
> | 1    | salad             |
> | 1    | mineral water     |
> | 1    | salmon            |
> | 1    | antioxydant juice |
> | 1    | frozen smoothie   |
> | 1    | spinach           |
> | 1    | olive oil         |
> | 2    | burgers           |
> | 2    | meatballs         |
> | 2    | eggs              |
> | 3    | chutney           |
> | 4    | turkey            |

```python
# Perform one hot encoding using TransactionEncoder
from mlxtend.preprocessing import TransactionEncoder

# Create a TransactionEncoder
te = TransactionEncoder()

# Fit and transform the transaction data
te_array = te.fit(txns).transform(txns)

# Extract the column names
te_columns = te.columns_

# Create a DataFrame from the one-hot encoded array
df1 = pd.DataFrame(te_array, columns=te.columns_)

# Display the results
df1.head()
```
![image](https://github.com/andytoh78/market-basket-analysis/assets/139482827/01e3cc0c-b119-4823-9edb-a6fc8d21a0df)

The output of the above code snippet provides a binary representation of the transaction data. Each row represents a transaction, and each column corresponds to an item. If an item is present in a transaction, the corresponding cell value will be indicated as `True`; otherwise, it will be `False`. This **${\color{blue}\textsf{one-hot encoded}}$** format is commonly used for various data mining tasks, including association rule mining. This is also the required format when using the Apriori alogrithm in identifying frequent itemsets and association rules.

Before applying the Apriori algorithm, let's first take a look at the 30 most commonly purchased items in the dataset

```python
# Find the top most frequent items
top_items = df_txn['Item'].value_counts().reset_index()

# Convert the top items into DataFrame and sort by item count in descending order
df_top_items = pd.DataFrame(top_items)
df_top_items.columns = ['Item', 'Count']

# Calculate the percentage of transactions for each item
total_transactions = len(df)
df_top_items['% Count'] = (df_top_items['Count']*100 / total_transactions).round(2)

# Display the results
df_top_items.style.background_gradient(cmap='Blues')
```
> | Item             | Count | % Count |
> |:-----------------|:------|:--------|
> | mineral water    | 1788  | 23.84   |
> | eggs             | 1348  | 17.97   |
> | spaghetti        | 1306  | 17.41   |
> | french fries     | 1282  | 17.09   |
> | chocolate        | 1230  | 16.40   |
> | green tea        | 991   | 13.21   |
> | milk             | 972   | 12.96   |
> | ground beef      | 737   | 9.83    |
> | frozen vegetables| 715   | 9.53    |
> | pancakes         | 713   | 9.51    |
> | burgers          | 654   | 8.72    |
> | cake             | 608   | 8.11    |
> | cookies          | 603   | 8.04    |
> | escalope         | 595   | 7.93    |
> | low fat yogurt   | 574   | 7.65    |
> | shrimp           | 536   | 7.15    |
> | tomatoes         | 513   | 6.84    |
> | olive oil        | 494   | 6.59    |
> | frozen smoothie  | 475   | 6.33    |
> | turkey           | 469   | 6.25    |
> | chicken          | 450   | 6.00    |
> | whole wheat rice | 439   | 5.85    |
> | grated cheese    | 393   | 5.24    |
> | cooking oil      | 383   | 5.11    |
> | soup             | 379   | 5.05    |
> | herb & pepper    | 371   | 4.95    |
> | honey            | 356   | 4.75    |
> | champagne        | 351   | 4.68    |
> | fresh bread      | 323   | 4.31    |
> | salmon           | 319   | 4.25    |

Mineral water is the most frequently purchased item, and it appears in 1788 (~24%) transactions. Several food items like eggs, spaghetti, french fries, chocolate, and green tea also have high purchase counts. We can also visualize the frequent items using bar charts, heatmaps, pie charts, tree maps, word cloud to better understand their distribution within the dataset.


### **${\color{black}\textsf{Bar Plot}}$**
```python
# Create the countplot to display top 30 most frequent items
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(18, 6))
plt.bar(data=df_top_items.head(30), x="Item", height="Count", edgecolor="black", color="skyblue")
ax.bar_label(ax.containers[0], fontsize=10, fontweight=600)
plt.title("Top 30 Most Frequent Items\n", fontsize=14, fontweight=700)
plt.xlabel("\nItem", fontsize=12, fontweight=700)
plt.ylabel("Quantity\n", fontsize=12, fontweight=700)
plt.xticks(rotation=90)
plt.gca().grid(False)
plt.gca().spines[["left", "bottom"]].set_color("black")
plt.gca().spines[["right", "top"]].set_visible(False)
plt.gca().set_facecolor("white")
plt.tight_layout()
plt.show()
```
![image](https://github.com/andytoh78/market-basket-analysis/assets/139482827/64d9f897-98e6-4310-b569-8c2c85105dbf)

### **${\color{black}\textsf{Word Cloud}}$**
A word cloud can be an engaging way to visualize frequent items where the size of each item's name is proportional to its frequency. This can be useful for a quick view and identification of the most common items, although it is less precise in terms of quantification.
```python
# Create the word cloud to visualize frequent items
from wordcloud import WordCloud

# Combine Item values into a string with space separator
all_values = [item for txn in txns for item in txn]
all_values_list = ' '.join(all_values)

# Create a word cloud object
wordcloud = WordCloud(width=300, height=300, background_color="white", min_font_size=8, colormap='viridis').generate(all_values_list)

# Display the word cloud
plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.tight_layout(pad=0)
plt.show()
```
<img src="https://github.com/andytoh78/market-basket-analysis/assets/139482827/83be14e4-893d-42c8-932b-0aa34ea84e98" width="400" height="400">

### **${\color{black}\textsf{Tree Map}}$**
Tree maps display each item as a rectangle, with the size corresponding to the frequency of the item. This can be a visually appealing way to represent hierarchical data and show relative patterns at a glance.
```python
# Create the tree map to display top 30 most frequent items
import seaborn as sns
sns.set_style("whitegrid")
import squarify

# Convert 'Count' column to integers
df_top_items['Count'] = df_top_items['Count'].astype(int)

# Create a figure and axis
fig, ax = plt.subplots(figsize=(12, 6))

# Top 30 items
df_top_30items = df_top_items.head(30)

# Calculate treemap sizes
sizes = df_top_30items['Count']

# Calculate treemap labels
labels = [f"{item}\n({count})" for item, count in zip(df_top_30items['Item'], df_top_30items['Count'])]

# Plot the treemap
squarify.plot(sizes=sizes, label=labels, alpha=0.8, color=sns.color_palette("Spectral", len(df_top_30items)), text_kwargs={'fontsize':10}, ax=ax)

plt.axis('off')
plt.title("Tree Map for Top 30 Most Frequent Items\n", fontsize=14, fontweight=700)
plt.show()
```
<img src="https://github.com/andytoh78/market-basket-analysis/assets/139482827/32b713fd-38f6-4dec-ab21-f481cf8f3122" width="700" height="400">

<br>We will now apply the Apriori algorithm to find the frequent itemsets and the association rules that are deemed as significant or interesting. 
```python
# Generate frequent itemsets
from mlxtend.frequent_patterns import apriori

# Applying Apriori algorithm assuming an item has to appear in at least 4% of the total transaction to be considered as frequent
min_support_threshold = 0.04
frequent_itemsets = apriori(df1, min_support=min_support_threshold, use_colnames=True)
frequent_itemsets
```
> |   support | itemsets                     |
> |----------:|:-----------------------------|
> |  0.087188 | (burgers)                    |
> |  0.081056 | (cake)                       |
> |  0.046794 | (champagne)                  |
> |  0.059992 | (chicken)                    |
> |  0.163845 | (chocolate)                  |
> |  0.080389 | (cookies)                    |
> |  0.051060 | (cooking oil)                |
> |  0.179709 | (eggs)                       |
> |  0.079323 | (escalope)                   |
> |  0.170911 | (french fries)               |
> |  0.043061 | (fresh bread)                |
> |  0.063325 | (frozen smoothie)            |
> |  0.095321 | (frozen vegetables)          |
> |  0.052393 | (grated cheese)              |
> |  0.132116 | (green tea)                  |
> |  0.098254 | (ground beef)                |
> |  0.049460 | (herb & pepper)              |
> |  0.047460 | (honey)                      |
> |  0.076523 | (low fat yogurt)             |
> |  0.129583 | (milk)                       |
> |  0.238368 | (mineral water)              |
> |  0.065858 | (olive oil)                  |
> |  0.095054 | (pancakes)                   |
> |  0.042528 | (salmon)                     |
> |  0.071457 | (shrimp)                     |
> |  0.050527 | (soup)                       |
> |  0.174110 | (spaghetti)                  |
> |  0.068391 | (tomatoes)                   |
> |  0.062525 | (turkey)                     |
> |  0.058526 | (whole wheat rice)           |
> |  0.052660 | (chocolate, mineral water)   |
> |  0.050927 | (eggs, mineral water)        |
> |  0.040928 | (ground beef, mineral water) |
> |  0.047994 | (milk, mineral water)        |
> |  0.059725 | (mineral water, spaghetti)   |

The most frequent individual items are mineral water (23.84%), eggs (17.97%), and spaghetti (17.41%). There are also five frequent itemsets with two items e.g. (chocolate, mineral water) with a support of 5.27%, indicating that these items often appear together in transactions.

```python
# Create the countplot for top 30 most frequent itemsets
import matplotlib.pyplot as plt

# Convert frozensets to strings and remove 'frozenset' from the representation
frequent_itemsets['itemset'] = frequent_itemsets['itemsets'].apply(lambda x: ', '.join(list(x)))

# Sort and filter top 15 most frequent itemsets
freq_sorted = frequent_itemsets.sort_values(by="support", ascending=False)
freq_sorted_top30 = freq_sorted.head(30)
freq_sorted_top30["support %"] = (freq_sorted_top30["support"]*100).round(2)

fig, ax = plt.subplots(figsize=(18, 6))
plt.bar(data=freq_sorted_top30, x="itemset", height="support %", edgecolor="black", color="skyblue")
ax.bar_label(ax.containers[0], fontsize=10, fontweight=600)
plt.title("Top 30 Most Frequent Items\n", fontsize=14, fontweight=700)
plt.xlabel("\nItem", fontsize=12, fontweight=700)
plt.ylabel("Quantity\n", fontsize=12, fontweight=700)
plt.xticks(rotation=90)
plt.gca().grid(False)
plt.gca().spines[["left", "bottom"]].set_color("black")
plt.gca().spines[["right", "top"]].set_visible(False)
plt.gca().set_facecolor("white")
plt.tight_layout()
plt.show()
```

![image](https://github.com/andytoh78/market-basket-analysis/assets/139482827/21c32e37-17ad-4521-9358-99df573669b2)


```python
# Generate association rules from the frequent itemsets assuming the likelihood of purchasing the antecedent, followed by the consequent has to be at least 30% to be considered significant or interesting 
from mlxtend.frequent_patterns import association_rules
confidence_threshold = 0.3 
rules = association_rules(frequent_itemsets, metric="confidence", min_threshold=confidence_threshold)

# Sorting rules by confidence, support, and lift
sorted_rules = rules.sort_values(['confidence', 'support', 'lift'], ascending=[False, False, False])
sorted_rules
```
> | antecedents   | consequents     | antecedent support | consequent support | support   | confidence | lift     | leverage  | conviction | zhangs_metric |
> |:--------------|:----------------|-------------------:|-------------------:|----------:|-----------:|---------:|----------:|-----------:|--------------:|
> | (ground beef) | (mineral water) |           0.098254 |           0.238368 |  0.040928 |    0.416554 | 1.747522 | 0.017507  |    1.305401 |     0.474369 |
> | (milk)        | (mineral water) |           0.129583 |           0.238368 |  0.047994 |    0.370370 | 1.553774 | 0.017105  |    1.209650 |     0.409465 |
> | (spaghetti)   | (mineral water) |           0.174110 |           0.238368 |  0.059725 |    0.343032 | 1.439085 | 0.018223  |    1.159314 |     0.369437 |
> | (chocolate)   | (mineral water) |           0.163845 |           0.238368 |  0.052660 |    0.321400 | 1.348332 | 0.013604  |    1.122357 |     0.308965 |

Customers who purchase ground beef are 41.66% likely to also purchase mineral water and this association is supported by a lift value of 1.75, which signifies that these items are frequently bought together. Similiar observations were found for some other items - milk, spaghetti, and chocolate, in association with mineral water. To visualize association rules, we can leverage the scatter plots, network graphs, heatmaps, and bar charts to understand their distribution within the dataset and their strength of association.

### **${\color{black}\textsf{Scatter Plot}}$**
```python
# Create a scatterplot to visualize the relationship between support and confidence in association rules
fig, ax = plt.subplots(figsize=(10, 8))
sns.scatterplot(data=sorted_rules, x=sorted_rules["support"], y=sorted_rules["confidence"], size="confidence", sizes=(0, 1000), legend=False)

# Annotate the points with labels
for i, row in sorted_rules.iterrows():
    # Convert frozenset to list, then to string and remove the frozenset and other unwanted characters
    antecedents = ', '.join(list(row['antecedents']))
    consequents = ', '.join(list(row['consequents']))
    label = f"{antecedents}\n→ {consequents}" # \nSupport: {row['support']:.3f}, Confidence: {row['confidence']:.3f}"
    plt.annotate(label, (row['support'], row['confidence']), textcoords="offset points", xytext=(15, 0), ha='left', fontsize=10)
plt.title("Association Rules Support vs Confidence\n", fontsize=14, fontweight=700)
plt.xlabel("\nSupport", fontsize=12, fontweight=700)
plt.ylabel("Confidence\n", fontsize=12, fontweight=700)
plt.gca().grid(False)
plt.gca().spines[["left", "bottom"]].set_color("black")
plt.gca().spines[["right", "top"]].set_visible(False)
plt.gca().set_facecolor("white")
plt.show()
```
<img src="https://github.com/andytoh78/market-basket-analysis/assets/139482827/3c5049ca-18f4-421f-b93c-fb522560fb5b" width="600" height="400">

### **${\color{black}\textsf{Network Graph}}$**
We can also visualize how items are associated with one another using a network graph, which represent items as nodes and the association rules as edges and colour intensity. 
```python
# Create a network graph to visualize the characteristics of the association rules
import networkx as nx
import matplotlib.cm as cm
from matplotlib.cm import get_cmap
import matplotlib.colors as mcolors
import warnings
warnings.filterwarnings("ignore", category=Warning)

# Create a graph
G = nx.Graph()

for _, row in rules.iterrows():
    # Convert frozenset to string
    antecedents_str = ', '.join(list(row['antecedents']))
    consequents_str = ', '.join(list(row['consequents']))
    
    # Add nodes and edges with the string labels
    G.add_node(antecedents_str)
    G.add_node(consequents_str)
    G.add_edge(antecedents_str, consequents_str, weight=row['lift'])

colormap = get_cmap('viridis')
lift_values = [data["weight"] for _, _, data in G.edges(data=True)]
lift_min = min(lift_values)
lift_max = max(lift_values)
lift_norm = mcolors.Normalize(vmin=lift_min, vmax=lift_max)

# Define edge colors and widths based on lift values
edge_colors = lift_values
edge_widths = 3

# Plot the graph
fig, ax = plt.subplots(figsize=(14, 8))
pos = nx.spring_layout(G, k=0.15, iterations=20)
nx.draw_networkx_nodes(G, pos, node_color="azure", node_size=50)
nx.draw_networkx_labels(G, pos, font_size=8)
nx.draw(G, pos, edge_color=edge_colors, width=edge_widths, edge_cmap=colormap, edge_vmin=lift_min, edge_vmax=lift_max)
cbar = plt.colorbar(plt.cm.ScalarMappable(cmap=colormap, norm=lift_norm), ax=ax, shrink=0.4)
cbar.set_label("Lift Value")

plt.axis("off")
plt.show()
```
<img src="https://github.com/andytoh78/market-basket-analysis/assets/139482827/a4d6dc6f-a022-437e-87a3-c6104a843c29" width="650" height="400">

### **${\color{black}\textsf{Heat Maps}}$**
```python
# Create a heatmap to visualize the relationships between the antecedents and consequents
# Convert frozensets to strings and remove 'frozenset' from the representation
rules['antecedents'] = rules['antecedents'].apply(lambda x: ', '.join(list(x)))
rules['consequents'] = rules['consequents'].apply(lambda x: ', '.join(list(x)))

# Show frequency or strength of item associations
pivot_table = rules.pivot(index="consequents", columns="antecedents", values="confidence")
plt.figure(figsize=(14, 2))
sns.heatmap(pivot_table, annot=True, cmap="viridis")
plt.xlabel("\nantecedents", fontsize=12, fontweight=600)
plt.ylabel("consequents\n\n", fontsize=12, fontweight=600)
plt.gca().set_facecolor("white")
plt.show()
```
<img src="https://github.com/andytoh78/market-basket-analysis/assets/139482827/bd2b95de-5d5a-4609-b735-8cdecb38e754" width="700" height="150">

## **Summary**
- Mineral water is a frequent consequent, suggesting it is commonly bought with many different items.<p>
- The strongest association rule based on confidence is (ground beef) -> (water) with a confidence of about 0.42, meaning that when ground beef is bought, water is also likely to be bought in 42% of the cases.<p>
- Milk, chocolate, and spaghetti also have strong associations with mineral water, with confidence levels above 32%.

