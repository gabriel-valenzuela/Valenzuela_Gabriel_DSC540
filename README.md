# Pokemon Video Game SQL Lite Database

## Data Preparation Project

### Objective

The goal of this data preparation project is to collect several different forms of data types including a csv, a website, and an API to combine and input into a SQL Lite Database. The data that is being collected and merged focuses on around the famous video game of Pokemon that emerged in the 90s that contained about only 150 pokemon in the beginning with their own characteristics. As the years progressed and more games were released containing new and exciting creatures, the information surrounding grew and included more characterists and unique combinations. Once the database is created, I created a Microsoft Power BI report to show to provide summaries of different aspects of the creatures. 

### Environment

Python and instances of SQL were utilized within various jupyter notebooks to complete the database. 

### Data Sources

CSV:

Website:

API:

### Methods

For each of the data sources, the goal was to clean the data once collected and then export them as CSV files to be added to the master Pokemon database as data frames. 

#### CSV Preparation
```python
df = pd.read_csv("C:\\Users\\Gabe\\Documents\\Bellevue University\\Data Preparation\\Final Project\\Data\\pokemon.csv")
df.head(10)

# Context:

# Type 2:
# Not all Pokemon will have a secondary type, but there are many Pokemon
# that do. Therefore, you cannot remove ones that only have one or 
# remove the column containing the secondary type because it gives more
# details about the creature. 

# Percentage Male:
# Many pokemon have the chance of either being male or female. However, not
# all pokemon can be classified within a gender of male or female. 
# Therefore, I will give all of these special cases a 50% chance of being 
# either male or female


print("Sum of Null Values")
print(df.isnull().sum())

print("Sum of Empty Values")
print(df.isnull().sum().sum())

# Type 2
# Replace the blank values with "No Secondary Type"
df['type2'].fillna("No Secondary Type", inplace=True)

# Percentage Male
# Replace the blank values with 50
df['percentage_male'].fillna(50, inplace=True)

# Remove hieght and weight of Pokemon since blank values and no need for 
# Japanese name
df.drop(["height_m", "weight_kg", "japanese_name"], axis=1, inplace=True)

# Remove duplicates
df['classification']=df['classification'].str.replace('Pokémon', "")
df.head()

df.to_csv(r'C:\Users\Gabe\Documents\Bellevue University\Data Preparation\Final Project\Data for Pokemon Database\PokemonCSV.csv', index = False)
```


#### Website Preparation
```python
import requests
import lxml.html as lh
import pandas as pd

url='http://pokemondb.net/pokedex/all'
page = requests.get(url)
doc = lh.fromstring(page.content)
tr_elements = doc.xpath('//tr')

tr_elements = doc.xpath('//tr')

#Create empty list
col=[]
i=0

#For each row, store each first element (header) and an empty list
for t in tr_elements[0]:
    i+=1
    name=t.text_content()
    print('%d:"%s"'%(i,name))
    col.append((name,[]))

#Since out first row is the header, data is stored on the second row onwards
for j in range(1,len(tr_elements)):
    T=tr_elements[j]
    
    #If row is not of size 10, the //tr data is not from our table 
    if len(T)!=10:
        break
    
    #i is the index of our column
    i=0
    
    #Iterate through each element of the row
    for t in T.iterchildren():
        data=t.text_content() 
        #Check if row is empty
        if i>0:
        #Convert any numerical value to integers
            try:
                data=int(data)
            except:
                pass
        #Append the data to the empty list of the i'th column
        col[i][1].append(data)
        #Increment i for the next column
        i+=1
        
Dict={title:column for (title,column) in col}
df=pd.DataFrame(Dict)

df.head()

# Reducing the number of pokemon to match the CVS Source's number
print(df[0:932])
df = df[0:932]
print(df.tail())

# Remove duplicates of Pokemon having the same number ID
df = df[~df.name.str.contains("Mega")]

print(df)

df.to_csv(r'C:\Users\Gabe\Documents\Bellevue University\Data Preparation\Final Project\Data for Pokemon Database\PokemonWebsite.csv', index = False)

```
#### API Preparation

With the API preparation, it only allowed for one pokemon to be pulled at a time so I decided with one that I was familar with through my own experiences. Therefore, I created an input system to allow the user to request a Pokemon of their choice.
```python
import requests
import pandas as pd
import json
from pandas.io.json import json_normalize
import warnings
warnings.filterwarnings('ignore')
print("Enter a Pokemon!")
print("Example Pokemon to choose from: krabby, charizard, diglett, zubat")
pokemon = input('Pokemon:')
response_certain_pokemon = requests.get("https://pokeapi.co/api/v2/pokemon/" + pokemon).json()

df = pd.DataFrame.from_dict(json_normalize(response_certain_pokemon), orient="columns")
df.head()

# Change column heirarchy - moves
length_types = len(df['types'][0])
i_types = 0

while i_types < length_types:
    del df['types'][0][i_types]['slot']
    del df['types'][0][i_types]['type']['url']
    i_types +=1
    
df['types'][0] = ' '.join([str(elem) for elem in df['types'][0]])
df['types'][0] = (df['types'][0]).replace("{'type': {'name': '", "")
df['types'][0] = (df['types'][0]).replace("'", "")
print(df['types'][0])

# Split types to account for second types
#if "}}" in df['types'][0]:
df['types'][0] = (df['types'][0]).replace("}}", "")
if " " in df['types'][0]:
    df[['type1', 'type2']] = df.types.str.split(" ",expand=True)
else:
    print(df['types'][0])
# Fill in empty values of secondary type with 'No Secondary Type'
    df['types'][0] = (df['types'][0])+"PokemonNo Secondary Type"
    print(df['types'][0])
    df[['type1', 'type2']] = df.types.str.split("Pokemon",expand=True)

df.head()

df.to_csv(r'C:\Users\Gabe\Documents\Bellevue University\Data Preparation\Final Project\Data for Pokemon Database\PokemonAPI.csv', index = False)
```

#### Database Development
```python
import sqlite3
conn = sqlite3.connect('PokemonDB.db')
c = conn.cursor()

import pandas as pd
csv_df = pd.read_csv("C:\\Users\\Gabe\\Documents\\Bellevue University\\Data Preparation\\Final Project\\Data for Pokemon Database\\PokemonCSV.csv")
website_df = pd.read_csv("C:\\Users\\Gabe\\Documents\\Bellevue University\\Data Preparation\\Final Project\\Data for Pokemon Database\\PokemonWebsite.csv")
api_df = pd.read_csv("C:\\Users\\Gabe\\Documents\\Bellevue University\\Data Preparation\\Final Project\\Data for Pokemon Database\\PokemonAPI.csv")

csv_df.to_sql("CSV", conn, if_exists='replace', index=False)
pd.read_sql('select * from CSV', conn)

website_df.to_sql("WEBSITE", conn, if_exists='replace', index=False)
pd.read_sql('select * from WEBSITE', conn)

api_df.to_sql("API", conn, if_exists='replace', index=False)
pd.read_sql('select * from API', conn)

csv_website_df = csv_df.merge(website_df, on=['pokedex_number', 'name', 'type1', 'type2', 'hp', 'attack', 'defense', 'sp_attack', 'sp_defense', 'speed'], how='left')
csv_website_df.info()

```

### Visualizations
The visualization are located within the repository as a PDF.
