## Exit questionnaire

The exit questionnaire table reports on which proband-type participants are marked as resolved.

The absence of an entry or the presence of an entry with `case_solved_family` of 'no',
does not mean that the gene causing a rare disease is not known by the GMC.

The GMC may have been sent a report and assessed it and may have acted upon it, but the paperwork marking it
as concluded is not done or it will be concluded on the next data release.

However, the presence of a 'yes', 'partial' or even 'unknown' is good way to remove participants from an analysis,
as these will have a `gene_name` which may not match the one of interest.

In the case of contacted clinicians, the requests with participant_ids _were_ visible to all within GEL RE,
but are not any more.

### Pandas
In general, in pandas if you want to add a column with data from a column from another table based on a shared key,
the fastest way is alter the index to the shared key and merge the dataframes,
but it's way easier to using the `.map` method of Series.
Say you have a 'donor_df', 'acceptor_df' both with a 'shared_key' and the former with a 'wanted_field',
you can do:

```python3
mapping: dict = donor_df.set_index(shared_key).to_dict()[wanted_field]
acceptor_df[wanted_field] = acceptor_df.shared_key.map(mapping, na_action='ignore')
```

In the case of the `case_solved_family` column of `gmc_exit_questionnarie`,
the map call would be followed by `.fillna('no')`
or a conversion to boolean as discussed in [booleanise](booleanise.md).