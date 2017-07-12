---
layout: post
title: "Python package licences"
description: "How to determine the licence of your pip installed packages"
---

Sometimes it's handy to know the licences of your project's dependancies. You
could check each entry in `requirements.txt` one by one, but it'd be too easy
to miss out a nested dependancy. Also, if a project has a lot of dependancies,
it quickly becomes laborious. Don't worry though, it's easy to script, and I've
already done it for you:

```python
from __future__ import print_function
from collections import defaultdict
from os.path import join

# The `pkg_resources` module isn't in the standard library,
# but it's frequently available anywhere the `setuptools`
# module is (it typically comes with `pip`).
import pkg_resources

licences = defaultdict(list)

# We go over each installed package:
for package in pkg_resources.working_set:
    # Find the directory where the metadata is stored.
    info_dir = package._provider.egg_info
    if info_dir is None:
        print('No data for {}'.format(package))
        continue
    # Determine if the package is a wheel...
    is_wheel = info_dir.endswith('.dist-info')
    # ...and chose correct file.
    filename = 'METADATA' if is_wheel else 'PKG-INFO'
    filepath = join(info_dir, filename)

    # We then loop over the lines in the file...
    with open(filepath, 'r') as lines:
        for line in sorted(lines):
            # ...find the line that stores the license...
            if (line.startswith('Classifier: License :: ') or
                    line.startswith('License: ')):
                # ... and add the package to the list for it.
                license = line.rsplit(': ')[-1].strip()
                licences[license].append(package)
                break
        else:
            print('No data for {}'.format(package))

# Then all we have to do is print it out:
for licence, packages in licences.items():
    print(licence)
    for package in packages:
        print('\t', package)
```

Note that this only works for packages where the correct information has been
[added to the dependancy's
`setup.py`](https://packaging.python.org/distributing/#license).
