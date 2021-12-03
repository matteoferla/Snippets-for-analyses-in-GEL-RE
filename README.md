# Snippets for analyses in Genomics England Research environment
This is a collection of notes and Python 3 snippets to help analyse rare disease data 
in the Genomics England Research environment. I.e. by copy-pasting them into the airlock and 
using them in the other side.
This repo absolutely does not contain any patient data: 
Genomics England Research environment is rightfully very caution about its data.
As a side effect many snippets are transcribed so there may be typos.

## Index

* [Module load for VEP in Python](module_load.md)
* [Add VEP annotations to a pandas dataframe](add_vep_to_pandas.md)

## See also

* [my unofficial PanelApp API client](https://github.com/matteoferla/PanelApp_Python3_client_API)


## Note on using a Jupyter notebook

First, the best Python module last I checked (2020) was 3.8.1
(which they kindly added for me with pandas, jupyter and labkey and few extras), so:

    source /etc/profile.d/modules.sh
    module load python/3.8.1

This needs its kernel installed into jupyter though:

    python3 -m ipykernel install --user
    export XDG_RUNTIME_DIR=""

Note. Autosave, at least for me does not run every 2 minutes, 
so I make sure to add `%autosave 120` at the top of the notebook.