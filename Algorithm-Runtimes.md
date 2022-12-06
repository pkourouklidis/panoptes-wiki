## Introduction
As seen in the [Core Concepts section](Core Concepts), algorithm runtimes provide a mechanism for data scientists to implement their own dataset shift-detecting algorithms in multiple programming languages/frameworks.

To implement an algorithm in a way that a specific algorithm runtime can understand, users should follow the specifications provided by the developer of the runtime. Below are the specifications for the currently deployed runtimes.

## Python Function Algorithm Runtime
The specifications for this _Algorithm Runtime_ are quite simple. In [this](https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo) repository you can find an example algorithm implemented for this runtime.

All the user needs to do is implement a function called `detector` like so:
```python
from scipy import stats

#two-sample Kolmogorov-Smirnov test
def detector(trainSet, liveSet, parameters):
    firstFeature = trainSet.axes[1][0]
    pValue = stats.ks_2samp(trainSet[firstFeature].to_list(), liveSet[firstFeature].to_list())[1]
    threshold = float (parameters.get("pValue", 0.05))
    return int(pValue < threshold), pValue
```
As you can see the function has three arguments. The first two, `trainSet` and `liveSet` are Pandas dataframes and will contain the feature/predictions/labels specified in an [Algorithm Execution](Core Concepts#algorithm-execution) that uses our _Algorithm_. The third argument, `parameters`, is a dictionary that will contain the parameter key-value pairs specified in the aforementioned _Algorithm Execution_.

If your detector makes use of external python packages, you must include a requirements.txt file with the needed packages in the git repository of your _Algorithm_.

For the development of a new _Algorithm_, you can use the Panoptes CLI to test your python function locally before you use it in production.
To install the Panoptes CLI first clone the following git repository:

`git clone https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/panoptescli`

Then install the python package included in the cloned repo:

`pip install .`

Now you can use the Panoptes CLI to test your algorithms locally.

`panoptes --live ./datasets/live.csv --historical ./datasets/historical.csv --detector ./detector.py --parameter pValue=0.05`

You need to pass in paths to csv files containing the live and historical datasets, the path to the python file that contains the `detector` function, and key-value pairs for any parameters that your detector needs.

