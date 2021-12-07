## Tiering table

Once one has created the `.netrc` file in the home directory as described in confluence,
one can query labkey with Python.

Genomics England provides a really handy guide to the commands for labkey in [Python labkey API](https://cnfl.extge.co.uk/display/SDKB/Python+LabKey+API)

Additionally, the labkey web browser table has a cool feature in that one can export a snippet to copy,
which is something like this (retyped slightly differently):

```python3
import labkey

server_context = labkey.utils.create_server_context('labkey-embassy.gel.zone', 
                                                    'main-programme/main-programme_v👾👾_👾👾👾-👾👾-👾👾', 
                                                    'labkey',
                                                    use_ssl=True)
filter_array = [
                labkey.query.QueryFilter('genomic_feature_hgnc', '👾👾👾', 'contains'),
               ]
my_results = labkey.query.select_rows(server_context=server_context,
                                        schema_name='lists',
                                        query_name='tiering_data',
                                        filter_array=filter_array
                                    )
```
The results are in 'rows' of this dictionary (as with most REST APIs of a DB query). So

```python3
import pandas as pd
participants = pd.DataFrame(my_results['row'])
print(f'There are {len(participants)} participants')
```
No column name has spaces or is a reserved keyword (like `rank` or `id`):
they can all cannot be accessed as an attribute. (_cf._ `print(participant.columns)`)

For what the columns and what values to expect are see [mirror_jupyters.md](mirror_jupyters.md).

## Multiples

One thing to note is that there are multiple entries per `position` for a `participant_id`.
This is because there may be multiple values for `consequence_type` or `phenotype` or `segragation_pattern`.
That is there were multiple VEP results and different PanelApp were applied and the `tier` differs.

The easiest thing to do would seem:

```python3
participants.drop_duplicates(['participant_id', 'position'])
```

But don't!
The table does not state what is the canonical transcript or what is the amino acid change,
but for most usages, a coding change is preferred. So I'd suggest sorting by that first,
and even then dropping duplicates only for a plot or tally etc.:

```python3
def sorter(value: str) -> int:
    return  ['frameshift_variant',
             'stop_gained',
             'stop_lost',
             'inframe_deletion',
             'missense_variant',
             'splice_donor_variant',
             '2KB_downstream_variant',
             'non_coding_transcript_exon_variant',
             '3_prime_UTR_variant',
             'NMD_transcript_variant',
             'downstream_gene_variant',
             'intron_variant',
             'splice_region_variant',
             'non_coding_transcript_variant',
             'upstream_gene_variant',
             '2KB_upstream_variant',
             '5_prime_UTR_variant',
             'synonymous_variant',
             'downstream_variant'].index(value)

participants['sort'] = participants.consequence_type.apply(sorter)
subset = participants.sort_values('sort').drop_duplicates(['participant_id', 'position'])
```

## InheritedAutosomalDominant

The `segregation_pattern` is one of 'InheritedAutosomalDominant', 'CompoundHeterozygous', 'deNovo', 'SimpleRecessive'.

The `segragation_pattern` "InheritedAutosomalDominant" is totally misleading
as this does not reflect whether the relevant parent is affect.

Therefore, to filter out these the columns `mother_affected` and `father_affected` come in handy.
Do note that a parent will/might have a record even if they lack the variant allele, in which
case their `genotype` will be 'reference_homozygous', i.e. the same as the reference genome.

Do note that whereas the `rare_diseases_family_id` is generally the same as the proband `participant_id`,
it sometimes is not and is some alphanumeric. So no shortcuts, eh!

So one can do the following for the cases of 'InheritedAutosomalDominant' with the correct parent,
assuming the `participants` dataframe was not filtered for `participant_type` 'Proband'.

Probands seem to be the only ones with filled `<parent>_affected` values, 
so the resulting dataframe from the snippet below will be proband only.
Note also that these fields are 'Yes', 'No' and 'NA'/'Unknown' and not boolean
—this is super important to remember for a few fields as I can guarantee everyone trips up on it sooner or later
(see (booleanise.md)[booleanise.md].

```python3
def htz_parent_families(data: pd.DataFrame, parent: str) -> np.ndarray:
    """
    returns list of rare_diseases_family_id
    """
    return (data.loc[(data.participant_type == parent.title()) &
                     (data.genotype != 'reference_homozygous')]
                .rare_diseases_family_id
                .unique()
            )


het_mums: np.ndarray = htz_parent_families(participants, 'Mother')
het_dads: np.ndarray = htz_parent_families(participants, 'Father')
# inherited dominant needs to match
correct_inheritance = (
        ((participants.father_affected == 'Yes') & (participants.rare_diseases_family_id.isin(het_dads))) |
        (participants.mother_affected == 'Yes') & (participants.rare_diseases_family_id.isin(het_mums))
)
inherited_correctly = participants.loc[correct_inheritance & (participants.segregation_pattern == 'InheritedAutosomalDominant')]
```

But that last line is a bit weird for most cases as one 
may want 'CompoundHeterozygous', 'deNovo', 'SimpleRecessive' and not only 'InheritedAutosomalDominant'.
So the last line could be changed to all cases, possibly with some filters against non-coding:

```python3
goi:str = '👾👾👾'  # gene of interest

meaningful_consequences = ['missense_variant', 'splice_region_variant',
                           'splice_donor_variant', 'stop_gained', 'frameshift_variant',
                           'inframe_deletion', 'downstream_variant', 'stop_lost']

subdata = participants.loc[correct_inheritance | (participants.segregation_pattern != 'InheritedAutosomalDominant')]
subdata = subdata.loc[(subdata.genomic_feature_hgnc == goi) \
                      & (subdata.consequence_type.isin(meaningful_consequences))]
```

Now... it should be noted that a parent may be marked as 'No', 
but may be 'Yes' for an `hpo_present` in the 'rare_diseases_participant_phenotype'.
For help navigating the phenotype data see [the GEL manual on phenotype data](https://research-help.genomicsengland.co.uk/display/GERE/Clinical+and+phenotype+data).

## VEP

Now the next step may be find what the amino acid consequence is, so see [add_vep_to_pandas.md](add_vep_to_pandas.md).


