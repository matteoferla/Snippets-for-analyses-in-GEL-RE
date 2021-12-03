## To vcf monkey patching

Given the function that write a pd.DataFrame to a vcf file:

```python3
import pandas as pd
from typing import *

def to_vcf(self: pd.DataFrame, filename: Optional[str] = 'output.vcf'):
    """
    Write a table to a vcf
    
    :param self: 
    :param filename: 
    :return: 
    """
    with open(filename, 'w') as fh:
        fh.write('#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\n')
        for i, variant in self.iterrows():
            fh.write('{CHROM}\t{POS}\t{ID}\t{REF}\t{ALT}\t{QUAL}\t{FILTER}\tfilename={filename}\n'
                     .format(**variant))
```
It can be added to the class

    pd.DataFrame.to_vcf = to_vcf

Or to an instance

    import types
    dataframe.to_vcf = types.MethodType(to_vcf, dataframe)

## VEP

Assuming module vep is loaded as described in [Module load for VEP in Python](module_load.md),
one can convert a bunch of nucleotide mutations to AA.

One could write out the unique mutations in a tiering table DataFrame to a vcf using the first method:

    patients.drop_duplicates(['position', 'alternate']).to_vcf(infilename)

And then run VEP if small enough (i.e. don't be an arse by doing large VEP annotations on the shared virtual node):

```python3
import pandas as pd

def run_vep(infile:str='in.vcf', outfilename:str='out.vep.vcf') -> pd.DataFrame:
    module('load', 'vep')
    cmd = (f'vep --input_file {infilename} --output_file {outfilename} ' +
           '--vcf --offline --cache --dir_cache /tools/apps/vep/92/ensembl-vep/.vep --cache_version 93 ' +
           '--assembly GRCh38 --force --no_stats --everything --use_given_ref --per_gene --pick --canonical')
    assert os.system(cmd)
    return read_vep(outfilename, cleaned=False)

vep = run_vep(infilename)
```

## Reading VEP file

Here is a function that given a VCF filename returns a VEP annotated pandas DataFrame.

NB. This function is rather janky and may require changes to csq header.

```python3
import pandas as pd
import os

def read_vep(filename:str, cleaned=True) -> pd.DataFrame:
    # cleaned removed empties
    head = 'CHROM	POS	ID	REF	ALT	QUAL	FILTER'.split()
    csqhead = ('Info|Consequence|IMPACT|SYMBOL|Gene|Feature_type|Feature|BIOTYPE|EXON|INTRON|HGVSc|HGVSp|',
               'cDNA_position|CDS_position|Protein_position|Amino_acids|Codons|Existing_variation|DISTANCE|',
               'STRAND|FLAGS|VARIANT_CLASS|SYMBOL_SOURCE|HGNC_ID|CANONICAL|TSL|APPRIS|CCDS|ENSP|SWISSPROT|',
               'TREMBL|UNIPARC|GENE_PHENO|SIFT|PolyPhen|DOMAINS|miRNA|AF|AFR_AF|AMR_AF|EAS_AF|EUR_AF|SAS_AF|',
               'AA_AF|EA_AF|gnomAD_AF|gnomAD_AFR_AF|gnomAD_AMR_AF|gnomAD_ASJ_AF|gnomAD_EAS_AF|gnomAD_FIN_AF|',
               'gnomAD_NFE_AF|gnomAD_OTH_AF|gnomAD_SAS_AF|MAX_AF|MAX_AF_POPS|CLIN_SIG|SOMATIC|PHENO|PUBMED|',
               'MOTIF_NAME|MOTIF_POS|HIGH_INF_POS|MOTIF_SCORE_CHANGE').split('|')
    data = []
    with open(filename) as fh:
        for row in fh:
            if row[0] == '#':
                continue
            row_data = row.split()
            if cleaned:
                new = {k: v for k, v in zip(csqhead,row_data[7].split('|')) if v}
            else:
                new = dict(zip(csqhead,row_data[7].split('|')))
                for kv in new['Info'].split(';'):
                    if kv.find('=') > 0:
                        (k,v) = kv.split('=')
                        new[k] = v
                for i in range(7):
                    new[head[i]] = row_data[i]
                data.append(new)
    return pd.DataFrame.from_records(data)
```

PyVCF is by far a better solution to the above,
but this was installed only recently in the python 3.8 kernel in GEL. It's usage would be something like
(disjointed code from another project of mine)

    vcf_reader = vcf.Reader(filename=vcf_filename, compressed=True)
    self.vep_headers = re.search(r'Format\:\s+(.*)', vcf_reader.infos['vep'].desc).group(1).split('|')
    vep_data = [dict(zip(self.vep_headers, entry.split('|'))) for entry in vep_entries if entry]
    canonicals = [datum for datum in filter(lambda e: len(e) > 1, vep_data) if 'CANONICAL' in datum and \
                  datum['CANONICAL'] == 'YES' and \
                  datum['Consequence'] in ('missense_variant', 'stop_gained', 'stop_lost', 'frameshift_variant')
                  ]

### Merging

```python3
import pandas as pd

def merge_vep_data(df: pd.DataFrame, vep_df: pd.DataFrame):
    for column in ['SYMBOL', 'gnomAD_AF', 'Consequence', 'Amino_acids', 'Protein_position']:
        mapping = {str(row.POS) + row.ALT: row[column] for i, row in vep.iterrows()}
        df[column] = (df.position.astype(str) + df.alternate.astype(str)).map(mapping)

merge_vep_data(patients, vep)
```
### Notes

In the above workflow I am going with canonicals rather than worse, so the consequence may differ from the tiering table.