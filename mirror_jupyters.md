## Mirror Jupyter

The virtual desktop nodes may be overly used (a very rare occurence these days)
as a result typing becomes/became laggy. So one way to get around it is
to have a Jupyter notebook running locally and one in GEL.
[Jupyter themes](https://github.com/dunovank/jupyter-themes) is a nice way to not get confused.
It does not work in GEL, but it does locally
—[see here for a screenshot of themes](https://blog.matteoferla.com/2020/11/remote-notebooks-and-jupyter-themes.html).

This also means that one can have a set of snippets that can be shared easily as done here.

One issue is knowing what columns and the expected values are present.
I don't know where in the documentation the details are present,
so for the tiering table I basically transcribed it —not as crazy as it sounds
and I am/was totally thankful for it when the node was laggy.

Here are the columns of the tiering table and what you can expect of them:

```python3
['Key',
 'participant_id',  # int
 'participant_type',  # can be only 'Proband', 'Mother', 'Father', 'Full Sibling' NB. Not borther or sister.
 'participant_phenotypic_sex',  # can be only 'Male' or 'Female'
 'genotype',  # can be only ['heterozygous' 'alternate_homozygous' 'reference_homozygous' 'missing']
 'genomic_feature_hgnc',  # gene name
 'penetrance',  # ['complete' 'incomplete']
 'assembly',  # Do not assume it's all GRCh38! There are some GRCh37!!
 'chromosome',
 'position',
 'reference',
 'alternate',
 'consequence_type',  # loads... see below.
 'segregation_pattern',  # can be one of...
                         # 'SimpleRecessive', 'deNovo', 'CompoundHeterozygous', 'InheritedAutosomalDominant', 
                         # 'UniparentalIsodisomy', 'MitochondrialGenome',
                         # 'XLinkedMonoallelic', 'XLinkedSimpleRecessive', 'XLinkedCompoundHeterozygous'
 'mode_of_inheritance',  # can be one: 'biallelic', 'mitochondrial'
                         # 'monoallelic_not_imprinted', 'monoallelic_maternally_imprinted',
                         # 'xlinked_biallelic', 'xlinked_monoallelic'
 'rare_diseases_family_id',
 'father_affected',  # see booleanise.md
 'mother_affected',  # see booleanise.md
 'full_sisters_affected',  # this is not a no/yes. it's a NA, 0, 1, 2 etc.
 'full_brothers_affected',  # this is not a no/yes. it's a NA, 0, 1, 2 etc.
 'sample_id',
 'interpretation_request_id',
 'tier',  # 'TIER1', 'TIER2', 'TIER3'
 'phenotype',
 'interpretation_cohort_id',
 'so_term',
 'ensembl_id']
```

'consequence_type' can be one of —NB. some typos present in weirdo ones:

```python3
['frameshift_variant',
 'stop_gained',
 'stop_lost',
 'inframe_deletion',
 'missense_variant',
 'splice_donor_variant',
 'non_coding_transcript_exon_variant',
 '3_prime_UTR_variant',
 '5_prime_UTR_variant',
 'NDM_transcript_variant',
 'downstream_gene_variant',
 'intron_variant',
 'splice_region_variant',
 'non_coding_transcript_variant',
 'downstream_variant',
 'upstream_gene_variant',
 '2kb_downstream_variant',
 '2kb_upstream_variant',
 'synonymous_variant']
```

