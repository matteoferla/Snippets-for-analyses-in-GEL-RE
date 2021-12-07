## GMCs

> Big caveat: clinicians are not allowed to look up their own patients in GEL RE.

However, in doing analyses it is not uncommon to be asked by faculty in what GMC is a participant is from
—no idea why.

This data can be found in the 'participant' table under `handling_gmc_trust`.
Do note some participants have a different value for `registered_at_gmc_trust` than the `handling_gmc_trust`.
—ignore the former.

There is a shortcut to this though.
The `participant_id` is understandably a common key and the first three letters refer to the GMCs.
The 111 to 128 (plus 210?) are for the 'Rare Diseases' `programme`, 211 to 228 are for the 'Cancer' `programme`.
Each of these two dozen values refers to a specific GMC.
Oxford University NHS Foundation Trust is 117 and 217, for example.
The GMCs are described in [the GEL site](https://www.genomicsengland.co.uk/about-genomics-england/the-100000-genomes-project/genomic-medicine-centres/#gmcmap),
which includes this figure
![map](https://www.genomicsengland.co.uk/wp-content/uploads/2018/02/1813_GMC-UK-map22222.jpg)

There are more than 17 trusts because a few codes are taken up multiple times by Guy's and St. Thomas's NHS FT.
Note the figure calls them by a region name as opposed to the NHS FT —I have no idea why.

Different GMCs _seem_ to have different speeds with dealing with the requests to contact the referring clinicians.


