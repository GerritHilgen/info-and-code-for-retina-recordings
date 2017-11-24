# How to read stimulus timings

The projector generates two files for each experiment that contain all the required information:
* A ``Trigger`` file (Matlab format), which holds the time stamps of all stmulus presentations
* A ``Report`` file with details about the stimuli shown

Usually (if not, chaos), hdf5 files with spike data and Trigger/Report files have similar names, 
so it should be clear which pairing is correct.

They can be parsed using the following code.

## Step 1 - preliminaries

Required imports

```python
import h5py
from scipy.io import loadmat
import numpy as np
import pandas as pd
```

And choose an experiment:

```python
triggerfile  = 'Trigger_stim1_basic_ctl.mat'
reportfile = 'Report_2017-07-05_13_25_30_ProtocolName_Basic_Exp_stim1.txt'
```
## Step 2 - red the triggers

```python
try:
    mf = h5py.File(triggerfile,'r')
except:
    mf = loadmat(triggerfile)
    
timeStampMatrix = np.array(mf.get('timeStampMatrix'),dtype='int64')
onsetsFrame = np.array(mf.get('onsetsFrame'),dtype='int32')
```

## Step 3 - read info

```python
# parses an integer from a line
n = lambda l: int(l.rpartition(': ')[-1].strip()) 
# parses a float from a line
nf = lambda l: float(l.rpartition(': ')[-1].strip()) 
# get the stimulus name (from directory)
seqname =  lambda l: l.rpartition(' Executing sequence: ')[-1].strip().replace('D:\\Stimuli\\','') 
    
# these are the tags in the report file we need
tags = {'Executing sequence': ((seqname, [], {}),'Name'),
        'Num of stimuli to be displayed': ((n, [], {}), 'Nstim1'),
        'Num of stimuli displayed': ((n, [], {}), 'Nstim2'),
        'Num of refresh per stimulus': ((n, [], {}), 'Nrefresh'),
        'ND': ((nf, [], {}), 'ND')
       }

Stimuli = pd.DataFrame(columns=('Name', 'Nstim1', 'Nstim2', 'Nrefresh', 'Onset'))

with open(prtcfile) as file:
    i = -1
    for line in file:
        tag = line.partition(':')[0].strip()
        fun, args, kwargs = tags[tag][0]
        val = fun(line)
        if tags[tag][1] is 'Name': # make new line
            i += 1
            Stimuli = Stimuli.append({tags[tag][1]:fun(line)},ignore_index=True)
            Stimuli['Onset'][i] = onsetsFrame[i][0]
        elif tags[tag][1] is not 'ND':
            Stimuli[tags[tag][1]][i] = fun(line)
``` 
## Analyse

Now all relevant information is in the following:
* ``timeStampMatrix``: an array with all stimulus time stamps
* ``Stimuli``: a Pandas table with all stimuli

``Stimuli`` looks like this:

Name|Nstim1|Nstim2|Nrefresh|Onset
---|---|---|---|---
Images\\Fullfield|60|60|120|220625
Images\\Gratings\\KarinaContrast\\chirp2|10160|10160|2|1091770
Images\\Gratings\\KarinaContrast\\movingbar\\d0|1430|1430|2|3505340
Images\\Gratings\\KarinaContrast\\movingbar\\d30|1430|1430|2|3863403
Images\\Gratings\\KarinaContrast\\movingbar\\d60|1430|1430|2|4221583
...|...|...|...|...

So to see where the second stimulus (``Images\\Gratings\\KarinaContrast\\chirp2``) begins, and to get all time stamps, use:

```python
stim = 1 # the second stimulus
my_onset = Stimuli['Onset'][stim]
stimtimes = timeStampMatrix[timeStampMatrix>=Stimuli['Onset'][stim]][:Stimuli['Nstim1'][stim]]
```


