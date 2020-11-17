---
layout: archive
title: "XLKitLearn User Manual"
permalink: /xlkitlearn_manual/
author_profile: true
---

This is a user manual.

* TOC
{:toc}


# Introduction
This manual assumes a working understanding of linear and logistic regressions, k-fold cross validation, boosted trees among others. External links are included for reference. Please share your feedback with us at ideas@xlkitlearn.com.

This excel add-in makes it easy to run causal and predictive analytics like linear/logistic regressions, decision trees and boosted forests using tools like K-fold cross validation, lasso penalties. The general steps to using this tool are:
  1. Select data
  2. Specify model and parameters
  3. Evaluate outputs

Windows users have two advanced options below the setting at the bottom of the Add-in tab to give the user more control over early termination and processsing speed. Mac users have ______________

# The Predictive Analytics Add-in
While this is a Python-based add-in, no Python knowledge is needed nor are any downloads besides the XLKit Learn excel file.

An email needs to be entered before before running any analyses. Email addresses are used to track bugs by identifying the models a user ran. Neither email nor analyses will be shared with any outside party.

The "Edit Settings" button allows the user to enter all the settings for the analysis like selecting the data, specifying the model, derterming the formula(s) and training criteria. Every time the model is run, a new tab is added with the outputs.

## Selecting data
If the data can fit in an excel sheet, add the data as a new tab. "Click to Select" the range of independent and dependent data including column headers.
> Column headers should not include spaces. Always add data to the XLKitLearn; don't move the add-in to another workbook.

When working with very large datasets, save the data as a csv in the same folder as XLKitLearn, and enter the data file name including ```.csv``` to select the data.

## Specifying a Predictive Model

XLKitLearn predictive models comprise two main ingredients:
  1. The predictive model to be used, and any parameters required to fit this model.
  2. The variable that needs to be predicted (the dependent variable) as well as the variables that should be used to predict it (the independent variables).

The model can be selected among [models available](#supported-predictive-models) in the "model" dropdown, and its parameters can be specified in the "Parameter(s)" box below.

To specify the dependent and independent variables, XLKitLearn leverages the [patsy](https://patsy.readthedocs.io/en/latest/) library to allow the specification of these variables as a _Patsy formula_ (similar but not identical to [R](https://www.r-project.org/)-style formulas). It also adds a number of [features](#advanced-xlkitlearn-formulas) above and beyond those available in Patsy.

The basic structure of each formula is as follows:

```[Dependent Variable] ~ [Independent Variable 1] + [Independent Var. 2] + ...```

Each variable should be referred to by the column headers in the first row of the [training data](#selecting-data).

For example, to use the average number of rooms per dwelling and the crime per capita to predict median property values, the appropriate formula would be

```median_property_value ~ av_rooms_per_dwelling + crime_per_capita```

Formulas can be typed directly into the formula box of the add-in settings, or using the [formula editor](#the-formula-editor).

### Supported predictive models

XLKitLearn supports the following predictive models. Note that all parameters in each of these models can be automatically chosen using [K-fold cross-validation](#parameter-tuning-and-k-fold-cross-validation).

Linear and logistic regression
: If this option is selected, XLKitLearn will automatically determine whether a continuous [linear regression](https://en.wikipedia.org/wiki/Linear_regression) model or a binary [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) model should be used. If the dependent variable in the model contains 0s or 1s only (_or_ if the dependent variable is a [logical statement](#logical-dependent-var)), a logistic regression model will be used. Otherwise, a linear regression model will be used. Note that XLKitLearn does not support classification models with more than two possible outcomes.
: XLKitLearn makes one unique parameter available for these models - the [Lasso penalty](https://en.wikipedia.org/wiki/Lasso_(statistics)). This parameter should either be specified as a number indicating the weight of the Lasso penalty _or_ can be set to ```BS``` to perform best subset selection. Use multiple lasso penalties by separating entries with an ```&```.

Decision trees
: if this option is selected, XLKitLearn will fit a simple [decision tree](https://en.wikipedia.org/wiki/Decision_tree_learning) using CART.
: tree depth is the only unique parameter avaiable for decision trees. Multiple depths can be compared by separating multiple entries with an ```&```.

Boosted decision tree
: if this opiton is selected

Random forest
: if this opiton is selected


### The Formula Editor

XLKitLearn's formula editor can be accessed by clicking on the three dots to the right of the formula box in the add-in settings. The formula editor lists all the headers in the training data on the left, and provides a larger area in which to enter a formula on the right.

> When data from an external file (rather than another sheet in the same Excel workbook) is used, Mac security settings do not allow XLKitLearn to open the file and read the headers. Thus, the formula editor will _not_ work when external data is used on a Mac computer.

The formula editor also supports auto-complete for quicker formula entry. As variable names are typed in the formula entry box, the list on the left is filtered down to variables that begin with those letters, and the first such variable is automatically suggested. Pressing the Tab key will complete the name of that variable:

<img src="/files/images/xlkitlearn/formula_editor.gif" width="50%">

As in other parts of the add-in, a red input box indicates an error in the formula; the specific error will be displayed in the area at the bottom of the formula editor.

<a name="logical-dependent-var"></a>

### Advanced XLKitLearn Formulas
The following advanced features are avaiable:
  - Categoricals
    Variables that have numberical values can be changed into categorical veraibles by using the following structure in the f[formula editor](#the-formula-editor) '''C(variable_1) + variable_2....'''. Any variable with text in it is automatically converted into a categorical variable with the base category dropped.
  - Dot formulas
    Include all variables in the model by using the formula '''y~.''' in the f[formula editor](#the-formula-editor)
  - Standardization
    Standardize each variable by using the following structure in the f[formula editor](#the-formula-editor) '''standard(variable_1) + standard(variable_2)....'''
  - Python formulas @guetta I think I missed this one??
  - Removing the intercept
    The intercept can be removed in the f[formula editor](#the-formula-editor) by adding a '''-1''' after the formula

## Parameter Tuning and K-Fold Cross-Validation
Specifying the number of folds in the settings will tell the add-in to run a [k-fold cross validation] https://en.wikipedia.org/wiki/Cross-validation_(statistics) with the stated number of folds. First the training and test data will be randomly sorted into different folds based upon the "Randomization Seed" then all regressions will be automatically run. The model with the highest average out-of-sample score for the training data will used on the evaluation dataset if an [evaluation dataset](#Specifying_an_evaluation_set) is specified.
 > The Randomization Seed [https://en.wikipedia.org/wiki/Random_seed] determines the randomization method and can enable two users to generate the same random list. Enter a random list of numbers once the model is finished to truly randomly sort the data.

Two or more models can be compared using an ```&``` to separate each model in the [formula editor](#the-formula-editor). Include the all combinations of models for comparision with the formula '''y~.'''.
> Models with lots of data might take more than 90 seconds, so select the appropriate advanced options as a Windows user or ________ as a Mac user

## Specifying an evaluation set
The "No evaluation set" option is selected by defult. Reserve data for evaluation by either specifying the proportion of data to be set aside based on the Randomization Seed or by selecting a specific evaluation data set range the same way training data is selected.
> Be careful not to generate an evaluation output dataset that is too big for excel or it might crash.

## Making predictions on new data - @guetta what does this mean? adding new data at a regular interval?

## Understanding the Predictive Add-in Output
The model output is dynamic based upon the model and dataset. Depending on model type and parameters not all of the following sections will be returned. The output will be separated into:
1. Parameter tuning
   A table summarizes the results of the K-Fold Cross-Validation. Each row corresponds to the model the user inputed and shows the In-sample score, Out-of-sample score, Out-of-sample SE, Formula, Lasso penalty and the number of Nonzero coefficients. Below the summary table are two charts to visually compare the models. The first chart shows the out-of-sample score and out-of-sample score SE by the row number / model input. The second chart also shows the out-of-sample score and out-of-sample score SE by the number of non-zero coefficients.
2. Model
   The model and lasso penalty is shown along with the in-sample score at the top of the model table. If single model is run, that model is used, if multiple models were compared, the model that returned the highest average out-of-sample score on the training set is retured. The model table will list each variable and the associated coefficient. When a single linear regression is run (is this correct  @guetta?) the t-stat, 97.5% conficence interval, p-value and stars indicating levels of stistical signicance are also listed.
3. Evaluation
   Lists every predicted and true outcome for the entire evaluation dataset. The AUC cure and the out-of-sample score for the evaluation set is generated when a evaluation set is specified.
   Note: Very large datasets can generate evaluation datasets that are too big for excel, so be mindful of how large your evaluation dataset when deciding to include the evaluation
4. Equivalent Python code
   The Python code that was used to run the analysis with descriptions for each section of code.
5. Technical Details
   The input settings that we're used to run the analysis. This can be copied and pasted directly into the "Settings" on the Add-in tab to run the exact same analysis again. The add-in version and the time for each step of the analysis are also noted.

# The Text Analytics Add-in
