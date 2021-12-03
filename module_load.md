## Module load

These snippets show how to run a command in Python in a Jupyter notebook that require a system module to be loaded.
This is particular relevant given that the data on the tiering table is not VEP annotated!

Doing `os.system('module load vep')` will obviously not work: once the module-load command is run, the next `os.system`
call will be a different subprocess and have no clue about it.

Unfortunately the `modulecmd` for Python is misconfigured so some extras environment variables need to be declared:

```python3
import os, re, subprocess


def module(*args: str) -> None:
    """
    Use like ``module('load', 'vep')``

    :param args:
    :return:
    """
    # ## correct module load
    # fix version
    if 'MODULE_VERSION' not in os.environ:
        os.environ['MODULE_VERSION_STACK'] = '3.2.10'
        os.environ['MODULE_VERSION'] = '3.2.10'
    else:
        os.environ['MODULE_VERSION_STACK'] = os.environ['MODULE_VERSION']
    # fix path
    os.environ['MODULESHOME'] = '/tools/envmodules/modules-3.2.10'
    # fix path
    if 'MODULEPATH' not in os.environ:
        f = open(os.environ['MODULESHOME'] + "/init/.modulespath", "r")
        path = []
        for line in f.readlines():
            line = re.sub("#.*$", '', line)
            if line is not '':
                path.append(line)
        os.environ['MODULEPATH'] = ':'.join(path)
    # fix loded
    if 'LOADEDMODULES' not in os.environ:
        os.environ['LOADEDMODULES'] = ''
    # ## correct input
    if isinstance(args[0], list):
        args = args[0]
    else:
        args = list(args)
    # run
    (output, error) = subprocess.Popen(['/tools/envmodules/modules-3.2.10/modulecmd', 'python'] +
                                       args, stdout=subprocess.PIPE).communicate()
    exec(output)  # -> None
```

Once this is done one can run:

```python3
infilename = 'filename.vcf'
outfilename = 'filename.vep.vcf'
cmd = (f'vep --input_file {infilename} --output_file {outfilename} ' +
       '--vcf --offline --cache --dir_cache /tools/apps/vep/92/ensembl-vep/.vep --cache_version 93 ' +
       '--assembly GRCh38 --force --no_stats --everything --use_given_ref --per_gene --pick --canonical')
import subprocess

result = subprocess.run(cmd.split(), capture_output=True, text=True)
assert result.returncode == 0, result.stderr
print(result.stdout)
```

> NB. This is intended to be for the annotation of a variant or two. Do not be a BMW driver and run it on a large VCF:
> the node of the virtual desktop is shared with other users, so it's best to run such large tasks in Helix.
> Btw, if someone is being a pain on the shared node as determined by `htop`,
> one could illicitly use the `wall` command to yell at them, but I am not sure the admins of GEL 
> would be happy with the road rage...


