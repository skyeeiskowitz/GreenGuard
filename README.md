<p align="center">
<img width=15% src="https://dai.lids.mit.edu/wp-content/uploads/2018/06/Logo_DAI_highres.png" alt="DAI" />
</p>

<p align="center">
<i>A project from Data to AI Lab at MIT.</i>
</p>

<p align="center">
<i> ------------------------------------</i>
</p>

<p align="center">
<img width=20% src="https://dai.lids.mit.edu/wp-content/uploads/2019/03/GreenGuard.png" alt="GreenGuard" />
</p>

<p align="center">
<i>GreenGuard is a machine learning library built for data generated by wind turbines and solar panel installations.</i>
</p>

[![PyPI Shield](https://img.shields.io/pypi/v/greenguard.svg)](https://pypi.python.org/pypi/greenguard)
[![Travis CI Shield](https://travis-ci.org/D3-AI/GreenGuard.svg?branch=master)](https://travis-ci.org/D3-AI/GreenGuard)

# GreenGuard

- Documentation: https://D3-AI.github.io/GreenGuard
- Homepage: https://github.com/D3-AI/GreenGuard

# Overview

The GreenGuard project is a collection of end-to-end solutions for machine learning tasks commonly
found in monitoring wind energy production systems. Most tasks utilize sensor data
emanating from monitoring systems. We utilize the foundational innovations developed for
automation of machine Learning at Data to AI Lab at MIT.

The salient aspects of this customized project are:
* A set of ready to use, well tested pipelines for different machine learning tasks. These are
  vetted through testing across multiple publicly available datasets for the same task.
* An easy interface to specify the task, pipeline, and generate results and summarize them.
* A production ready, deployable pipeline.
* An easy interface to ``tune`` pipelines using Bayesian Tuning and Bandits library.
* A community oriented infrastructure to incorporate new pipelines.
* A robust continuous integration and testing infrastructure.
* A ``learning database`` recording all past outcomes --> tasks, pipelines, outcomes.

## Data Format

**GreenGuard Pipelines** work on time Series formatted as follows:

* A **Turbines** table that contains:
  * `turbine_id`: column with the unique id of each turbine.
  * A number of additional columns with information about each turbine.
* A **Signals** table that contains:
  * `signal_id`: column with the unique id of each signal.
  * A number of additional columns with information about each signal.
* A **Readings** table that contains:
  * `reading_id`: Unique identifier of this reading.
  * `turbine_id`: Unique identifier of the turbine which this reading comes from.
  * `signal_id`: Unique identifier of the signal which this reading comes from.
  * `timestamp`: Time where the reading took place, as an ISO formatted datetime.
  * `value`: Numeric value of this reading.
* A **Targets** table that contains:
  * `target_id`: Unique identifier of the turbine which this label corresponds to.
  * `turbine_id`: Unique identifier of the turbine which this label corresponds to.
  * `timestamp`: Time associated with this target
  * `target - optional`: The value that we want to predict. This can either be a numerical
     value or a categorical label. This column can also be skipped when preparing data that
     will be used only to make predictions and not to fit any pipeline.

#### Demo Dataset

For development and demonstration purposes, we include a dataset with data from several telemetry
signals associated with one wind energy production turbine.

This data, which has been already formatted as expected by the GreenGuard Pipelines, can be
browsed and downloaded directly from the
[d3-ai-greenguard AWS S3 Bucket](https://d3-ai-greenguard.s3.amazonaws.com/index.html).

This dataset is adapted from the one used in the project by Cohen, Elliot J.,
"Wind Analysis." Joint Initiative of the ECOWAS Centre for Renewable Energy and Energy Efficiency (ECREEE), The United Nations Industrial Development Organization (UNIDO) and the Sustainable Engineering Lab (SEL). Columbia University, 22 Aug. 2014.
[Available online here](https://github.com/Ecohen4/ECREEE)

The complete list of manipulations performed on the original dataset to convert it into the
demo one that we are using here is exhaustively shown and explained in the
[Green Guard Demo Data notebook](notebooks/Green Guard Demo Data.ipynb).

## Concepts

Before diving into the software usage, we briefly explain some concepts and terminology.

### Primitive

We call the smallest computational blocks used in a Machine Learning process
**primitives**, which:

* Can be either classes or functions.
* Have some initialization arguments, which MLBlocks calls `init_params`.
* Have some tunable hyperparameters, which have types and a list or range of valid values.

### Template

Primitives can be combined to form what we call **Templates**, which:

* Have a list of primitives.
* Have some initialization arguments, which correspond to the initialization arguments
  of their primitives.
* Have some tunable hyperparameters, which correspond to the tunable hyperparameters
  of their primitives.

### Pipeline

Templates can be used to build **Pipelines** by taking and fixing a set of valid
hyperparameters for a Template. Hence, Pipelines:

* Have a list of primitives, which corresponds to the list of primitives of their template.
* Have some initialization arguments, which correspond to the initialization arguments
  of their template.
* Have some hyperparameter values, which fall within the ranges of valid tunable
  hyperparameters of their template.

A pipeline can be fitted and evaluated using the MLPipeline API in MLBlocks.


## Current tasks and pipelines

In our current phase, we are addressing two tasks - time series classification and time series
regression. To provide solutions for these two tasks we have two components.

### GreenGuardPipeline

This class is the one in charge of learning from the data and making predictions by building
[MLBlocks](https://hdi-project.github.io/MLBlocks) and later on tuning them using
[BTB](https://hdi-project.github.io/BTB/)

### GreenGuardLoader

A class responsible for loading the time series data from CSV files, and return it in the
format ready to be used by the **GreenGuardPipeline**.

### Tuning

We call tuning the process of, given a dataset and a template, find the pipeline derived from the
given template that gets the best possible score on the given dataset.

This process usually involves fitting and evaluating multiple pipelines with different hyperparameter
values on the same data while using optimization algorithms to deduce which hyperparameters are more
likely to get the best results in the next iterations.

We call each one of these tries a **tuning iteration**.


# Getting Started

## Requirements

### Python

**GreenGuard** has been developed and runs on Python 3.5, 3.6 and 3.7.

Also, although it is not strictly required, the usage of a [virtualenv](https://virtualenv.pypa.io/en/latest/)
is highly recommended in order to avoid interfering with other software installed in the system
where you are trying to run **GreenGuard**.

## Installation

The simplest and recommended way to install **GreenGuard** is using pip:

```bash
pip install greenguard
```

For development, you can also clone the repository and install it from sources

```bash
git clone git@github.com:D3-AI/GreenGuard.git
cd GreenGuard
make install-develop
```

## Quickstart

In this example we will load some demo data using the **GreenGuardLoader** and fetch it to the
**GreenGuardPipeline** for it to find the best possible pipeline, fit it using the given data
and then make predictions from it.

**NOTE**: All the examples of this tutorial are run in an [IPython Shell](https://ipython.org/),
which you can install by running the following commands inside your *virtualenv*:

```
pip install ipython
ipython
```

### 1. Load and explore the data

The first step is to load the demo data.

For this, we will import and call the `orion.loader.load_demo` function without any arguments:

```python
In [1]: from greenguard.loader import load_demo

In [2]: X, y, tables = load_demo()
```

The returned objects are:

`X`: A `pandas.DataFrame` with the `targets` table data without the `target` column.

```python
In [3]: X.head()
Out[3]:
   target_id  turbine_id  timestamp
0          1           1 2013-01-01
1          2           1 2013-01-02
2          3           1 2013-01-03
3          4           1 2013-01-04
4          5           1 2013-01-05
```

`y`: A `pandas.Series` with the `target` column from the `targets` table.

```python
In [4]: y.head()
Out[4]:
0    0.0
1    0.0
2    0.0
3    0.0
4    0.0
Name: target, dtype: float64
```

`tables`: A dictionary containing the `readings`, `turbines` and `signals` tables.

```python
In [5]: tables.keys()
Out[5]: dict_keys(['readings', 'signals', 'turbines'])

In [6]: tables['turbines']
Out[6]:
   turbine_id       name
0           1  Turbine 1

In [7]: tables['signals'].head()
Out[7]:
   signal_id                                          name
0          1  WTG01_Grid Production PossiblePower Avg. (1)
1          2  WTG02_Grid Production PossiblePower Avg. (2)
2          3  WTG03_Grid Production PossiblePower Avg. (3)
3          4  WTG04_Grid Production PossiblePower Avg. (4)
4          5  WTG05_Grid Production PossiblePower Avg. (5)

In [8]: tables['readings'].head()
Out[8]:
   reading_id  turbine_id  signal_id  timestamp  value
0           1           1          1 2013-01-01  817.0
1           2           1          2 2013-01-01  805.0
2           3           1          3 2013-01-01  786.0
3           4           1          4 2013-01-01  809.0
4           5           1          5 2013-01-01  755.0
```

### 2. Split the data

If we want to split the data in train and test subsets, we can do so by splitting the
`X` and `y` variables with any suitable tool.

In this case, we will do it using the [train_test_split function from scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html).

```python
In [9]: from sklearn.model_selection import train_test_split

In [10]: X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=0)
```

### 3. Finding the best Pipeline

Once we have loaded the data, we create a **GreenGuardPipeline** instance by passing:

* `template (string)`: the name of a template or the path to a template json file.
* `metric (string or function)`: The name of the metric to use or a metric function to use.
* `cost (bool)`: Whether the metric is a cost function to be minimized or a score to be maximized.

Optionally, we can also pass defails about the cross validation configuration:

* `stratify`
* `cv_splits`
* `shuffle`
* `random_state`

In this case, we will be loading the `greenguard_classification` pipeline, using
the `accuracy` metric, and using only 2 cross validation splits:

```python
In [11]: from greenguard.pipeline import GreenGuardPipeline

In [12]: pipeline = GreenGuardPipeline('greenguard_classification', 'accuracy', cv_splits=2)
Using TensorFlow backend.
```

Once we have created the pipeline, we can call its `tune` method to find the best possible
hyperparameters for our data, passing the `X`, `y`, and `tables` variables returned by the loader,
as well as an indication of the number of tuning iterations that we want to perform.

```python
In [13]: pipeline.tune(X_train, y_train, tables, iterations=10)
```

After the tuning process has finished, the hyperparameters have been already set in the classifier.

We can see the found hyperparameters by calling the `get_hyperparameters` method.

```python
In [14]: import json

In [15]: print(json.dumps(pipeline.get_hyperparameters(), indent=4))
{
    "pandas.DataFrame.resample#1": {
        "rule": "1D",
        "time_index": "timestamp",
        "groupby": [
            "turbine_id",
            "signal_id"
        ],
        "aggregation": "mean"
    },
    "pandas.DataFrame.unstack#1": {
        "level": "signal_id",
        "reset_index": true
    },
    ...
```

as well as the obtained cross validation score by looking at the `score` attribute of the
`pipeline` object:


```python
In [16]: pipeline.score
Out[16]: 0.6447509660798626
```

**NOTE**: If the score is not good enough, we can call the `tune` method again as many times
as needed and the pipeline will continue its tuning process every time based on the previous
results!

### 4. Fitting the pipeline

Once we are satisfied with the obtained cross validation score, we can proceed to call
the `fit` method passing again the same data elements.

This will fit the pipeline with all the training data available using the best hyperparameters
found during the tuning process:

```python
In [17]: pipeline.fit(X_train, y_train, tables)
```

### 5. Use the fitted pipeline

After fitting the pipeline, we are ready to make predictions on new data:

```python
In [18]: predictions = pipeline.predict(X_test, tables)

In [19]: predictions[0:5]
Out[19]: array([1., 0., 0., 0., 0.])
```

And evaluate its prediction performance:

```python
In [20]: from sklearn.metrics import accuracy_score

In [21]: f1_score(y_test, predictions)
Out[21]: 0.6413043478260869
```

### 6. Save and load the pipeline

Since the tuning and fitting process takes time to execute and requires a lot of data, you
will probably want to save a fitted instance and load it later to analyze new signals
instead of fitting pipelines over and over again.

This can be done by using the `save` and `load` methods from the `GreenGuardPipeline`.

In order to save an instance, call its `save` method passing it the path and filename
where the model should be saved.

```
In [22]: path = 'my_pipeline.pkl'

In [22]: pipeline.save(path)
```

Once the pipeline is saved, it can be loaded back as a new `GreenGuardPipeline` by using the
`GreenGuardPipeline.load` method:

```
In [23]: new_pipeline = GreenGuardPipeline.load(path)
```

Once loaded, it can be directly used to make predictions on new data.

```
In [24]: new_pipeline.predict(X_test, tables)
Out[24]: array([1., 0., 0., 0., 0.])
```


## Use your own Dataset

Once you are familiar with the **GreenGuardPipeline** usage, you will probably want to run it
on your own dataset.

Here are the necessary steps:

### 1. Prepare the data

Firt of all, you will need to prepare your data as 4 CSV files like the ones described in the
[data format](#data-format) section above.

### 2. Create a GreenGuardLoader

Once you have the CSV files ready, you will need to import the `orion.loader.GreenGuardLoader`
class and create an instance passing:

* `path - str`: The path to the folder where the 4 CSV files are
* `target - str, optional`: The name of the target table. Defaults to `targets`.
* `target_column - str, optional`: The name of the target column. Defaults to `target`.
* `readings - str, optional`: The name of the readings table. Defaults to `readings`.
* `turbines - str, optional`: The name of the turbines table. Defaults to `turbines`.
* `signals - str, optional`: The name of the signals table. Defaults to `signals`.
* `readings - str, optional`: The name of the readings table. Defaults to `readings`.
* `gzip - bool, optional`: Set to True if the CSV files are gzipped. Defaults to False.

For example, here we will be loading a custom dataset which has been sorted in gzip format
inside the `my_dataset` folder, and for which the target table has a different name:

```python
In [25]: import os

In [26]: os.listdir('my_dataset')
Out[26]: ['readings.csv.gz', 'turbines.csv.gz', 'signals.csv.gz', 'labels.csv.gz']

In [27]: from greenguard.loader import GreenGuardLoader

In [28]: loader = GreenGuardLoader('my_dataset', target='labels', gzip=True)
```

### 3. Call the loader.load method.

Once the `loader` instance has been created, we can call its `load` method:

```python
In [29]: X, y, tables = loader.load()
```

Optionally, if the dataset contains only data to make predictions and the `target` column
does not exist, we can pass it the argument `False` to skip it:

```python
In [29]: X, tables = loader.load(target=False)
```
