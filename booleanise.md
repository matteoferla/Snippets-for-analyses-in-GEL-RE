## Booleanise

There are several faux-boolean entries in the labkey tables.
These will be string with values 'Yes', 'No' and 'NA'/'Unknown'.

This is an obvious thing to fix, but can cause issues if one does not pay attention.

```python3
phenotypes['hpo_present'] = phenotypes.hpo_present.map(dict(Yes=True, No=False, NA=None, Unknown=None))
```

Also, some fields are in lowercase (cf. `gmc_exit_questionaire`) and the value `partial` appears (`case_solved_family`).

Given a whole `pd.DataFrame`, something like the following will do it:

```python3
mapping = dict(Yes=True, No=False, NA=None, Unknown=None)
mapping['partial'] = True  # your call...
mapping = {**mapping, **dict(zip(map(str.lower, mapping.keys()), mapping.values()))}
for col in df.columns:
    if len( set( df[col].unique() ) - set( mapping.keys() )  ) == 0:
        df[col] = df[col].map(mapping)
```

Tripping hazard: it's `len( set( df[col].unique() ) - set( mapping.keys() )  ) == 0`
and not `set( df[col].unique() ) == set( mapping.keys() )` because there may be more values
in the mapping than in the columns!

Also note that pandas when assigning a new column will convert `None` to `float('nan')` if the column is numeric,
but if its string or boolean it will leave it alone. But but... if there is no value, it will be `float('nan')`.
So there's an extra potential issue (I have not seen it in ages so may not be there) of NaNs.
So the last line can be changed to:

```python3
df[col] = df[col].map(mapping, na_action='ignore').fillna(None)
```