---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.4
  kernelspec:
    display_name: SALAR
    language: python
    name: salar
---

### **Specific Filtering**


<div style="text-align: justify"> Each keyword search might need a different final treatment: (1) erasing specific papers, (2) assigning publication years missing </div>

```python
df = pd.read_csv('GWABM.csv')
df[['TI','DI']]
# Main DB cleanup. Here we erase the 8 papers with No GW or No Human Behavior.
df_eliminated = df.loc[df['DI'] == "10.1016/J.ECOLMODEL.2019.04.015"]
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.3390/W10091184"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1007/S40710-015-0082-6"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1080/14649357.2015.1045015"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1007/S10980-014-0145-5"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1016/J.ENVSOFT.2013.03.001"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1659/MRD-JOURNAL-D-11-00038.S1"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1007/S10113-011-0238-5"])
 

# Now we generate a list with all indexes to erase and erase the papers
list_to_erase = df_eliminated.index.tolist()

print("Initial Database shape is:", df.shape)
df = df.drop(list_to_erase)
df = df.reset_index(drop=True)
print("Final Database shape is:", df.shape)
```

```python
# Here we erase specific papers by adding them to the Pandas Dataframe "df_eliminated"
df_eliminated = df.loc[df['DI'] == "10.1016/S1251-8050(99)80113-X"]
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1046/J.1365-3091.2001.00419.X"])
df_eliminated = df_eliminated.append(df.loc[df['DI'] == "10.1016/J.GEOMORPH.2015.12.018"])

# Now we generate a list with all indexes to erase and erase the papers
list_to_erase = df_eliminated.index.tolist()

print("Initial Database shape is:", df.shape)
df = df.drop(list_to_erase)
df = df.reset_index(drop=True)
print("Final Database shape is:", df.shape)
```

```python
# First we save all NaN values on the Publication Year field to a list (var: "PY_nulls_list") 
PY_nulls_list = df.loc[df['PY'].isnull()].index
PY_nulls_list
# We now print the associated Titles of all these documents without publication year for manual editing
for i in PY_nulls_list:
    print(df.loc[i, ['PY', 'TI']])

# For the manual edit we create a list of years to feed back to the Database and make corresponding changes
PY_new = [2019,...]
j = 0
for i in PY_nulls_list:
    df.loc[i, 'PY'] = PY_new[j]
    j+=1

# Finally we print a revision list
for i in PY_nulls_list:
    print(df.loc[i, ['PY', 'TI']])
```

## **Databases Combination**  
Previously saved cleaned Databases can be combined here, with a further duplicates removal through Titles and DOI identifiers.

```python
# Here we drop a specific row, reset index, check it has been removed and save it to database
GW_ABM.shape
GW_ABM = GW_ABM.drop([31],axis=0)
GW_ABM = GW_ABM.reset_index(drop=True)
GW_ABM.shape
e_path = "Keyword Searches/Main DB (ABM & GW)/Database/Main DB (ABM & GW)2.csv"
GW_ABM.to_csv(e_path, index = False)
```

```python
GW_ABM = pd.read_csv("Keyword Searches/Main DB (ABM & GW)/Database/Main DB (ABM & GW).csv")
SH = pd.read_csv("Keyword Searches/SH/Database/SH.csv")
TC = pd.read_csv("Keyword Searches/TC/Database/TC.csv")
PM = pd.read_csv("Keyword Searches/PM/Database/PM.csv")
EM_2 = pd.read_csv("Keyword Searches/EM2/Database/EM2.csv")
GW_ABM.shape
SH.shape
TC.shape
PM.shape
EM_2.shape
```

```python
list = ["AU","TI","SO","JI","AB","DE","ID","LA","DT","DT2","TC","CR","C1","DI","AR","FU","SN","PN","PP","PU","VL","PY",
        "RP","DB","AU_UN","AU1_UN","AU_UN_NR","SR_FULL","SR","CR_AU","CR_SO","AU_CO","AU1_CO", "BN"]

# First we create an empty dataframe and paste all databases on it
db_combined = pd.DataFrame(columns = list) 
db_combined = db_combined.append(GW_ABM, ignore_index=True)
db_combined = db_combined.append(SH, ignore_index=True)
db_combined = db_combined.append(TC, ignore_index=True)
db_combined = db_combined.append(PM, ignore_index = True)
#db_combined = db_combined.append(EM, ignore_index = True)
db_combined = db_combined.append(EM_2, ignore_index = True)
print("Database combined dimensions are: ", db_combined.shape)

db_combined = db_combined.drop_duplicates(subset = "TI")
print("After removing duplicates from titles, database dimensions are: ", db_combined.shape)
#print(db_combined.isna().sum())

db_combined_nulls = db_combined[db_combined['DI'].isnull()]
print("missing DI values: ", db_combined_nulls.shape)
db_combined_not_nulls = db_combined[db_combined['DI'].notnull()]
print("actual DI: ", db_combined_not_nulls.shape)

# Then we drop duplicates through the DI columns, by filtering only through non-null values
filt = db_combined_not_nulls.drop_duplicates(subset = "DI")
filt = filt.append(db_combined_nulls, ignore_index = True)
print("removing duplicates through DI leaves: ", filt.shape)

db_combined = filt.copy()
print("Cleaned combined database dimensions are: ", db_combined.shape)
```

```python
merged = db_combined.copy()
print('Merged Database dimensions are: ', merged.shape)
e_path = "Keyword Searches/Merged DB (GW-ABM,SH,PM,TC,EM)/Database/Merged DB (GW-ABM,SH,PM,TC,EM).csv"
merged.to_csv(e_path, index = False)
#check = pd.read_csv(e_path)
#check
```
