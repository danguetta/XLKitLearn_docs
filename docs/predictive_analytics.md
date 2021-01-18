# Predictive Analytics


## Getting started
While this is a Python-based add-in, no Python knowledge is needed nor are any downloads besides the XLKit Learn excel file.

An email needs to be entered before running any analyses. Email addresses are used to track bugs. Neither email nor analyses will be shared with any outside party.

The `Edit Settings` button allows the user to design analysis like selecting the data, specifying the model, determining the formulas and training criteria. Every time the model is run, the outputs are added as a new tab.


---

## Selecting data

The first step in any predictive analytic workflow is to specify the data that will be used in the model. The data should be in the same workbook; click on the ["Click to Select"](MATSON static shot of the settings with a red box around the button) button under "Training" in the add-in dialogue, and select the entire dataset you will be using, including column headers. Click [here](Select_Small_Data.gif) for a demo.

!!! warning "Adding Sheets Containing Data"
    As discussed in the [introduction](index.md), you should never move the add-in sheet *to* another workbook. If you data sits in another Excel workbook, bring the sheet containing your data *into* the add-in Excel workbook.
    
!!! warning "Column Names"
    The first row of your data should contain column names. There are a number of rules these column names must satisfy. They must contain only letters, numbers, and underscores - no spaces - and cannot start with a number. They are also case-sensitive.
    
---

### Reading data from files

If your file is very large, opening it in Excel may slow down your computer (and indeed, if your file is very large, it might not even be possible to open it in Excel).

For those situations, XLKitLearn allows you to read data directly from a source file, without having to open it in Excel. After clicking on "Click to select", simply type the name of the file directly into the dialogue box. Click [here](Select_Large_Data.gif) for a demo. The first row of the file should contain the column names (the same column name rules apply).

XLKit learn can read .csv files as well as .xlsx files; make sure to include the file extension (.csv or .xslx) when you specify the file name.

!!! warning "File Existence"
    On a Windows machine, XLKitLearn will automatically check whether the file exists when you click "OK"; you will not be able to select a file that doesn't exist. Because of Mac security settings, XLKitLearn won't be able to check this immediately on a Mac - you'll need to press "run" and launch Python before finding out whether the file can be loaded.

---

## Specifying a Formula

The linchpin of any predictive analytic workflow in XLKitLearn is the formula, which specifies the *outcome* variable to be predicted (the dependent or 'y' variable) and the features that will be used to predict this variable (the independent or 'x' variables). The formula should be input into the [formula box](MATSON static shot of the settings with a red box around the box).

XLKitLearn formulas are always provided in the following format

`dependent variable ~ independent variable 1 + independent variable 2 + ...`

Variables are referred to using the columns names in the data.

For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using the crime per capita and the average number of rooms per dwelling. The formula would then be

`median_property_value ~ crime_per_capita + av_rooms_per_dwelling`

### The Formula Editor

You can always type the formula into the [formula box](MATSON static shot of the settings with a red box around the box) manually. To make things easier for you, XLKitLearn makes a formula editor available, which will prevent you from typing the name of every variable manually.

To launch the formula editor, click on the [three dots](MATSON gif of clicking on the three dots and loading the formula editor) next to the formula box.

The formula editor contains a [list at the left](MATSON static shot of the formula editor with a red box around the list) listing all the variables in the selected data (whether in Excel or in an external file). It also contains a [larger textbox](MATSON static shot of the formula editor with a red box around the larger textbox) in which to type your formula.

The formula editor also supports auto-complete for quicker formula entry. As variable names are typed in the formula entry box, the list on the left is filtered down to variables that begin with those letters, and the first such variable is automatically suggested. Pressing the Tab key will complete the name of that variable. Click [here](Formula_Editor.gif) for a demo.

!!! warning "Variables Names in External Files"
    In the following circumstances, XLKitLearn will not be able to load variable names from the file and display them in the formula editor. (1) If you are loading a .xlsx file, XLKitLearn will not be able to open the file and read column names. (2) If you are on a Mac and loading an external file, Mac security settings will stop XLKitLearn from opening the file and reading column names.

If the formula you entered contains an error, the formula input box will turn red, and an error message will be displayed in the [bottom part](MATSON static shot of the formula editor with a red box around the error area) of the formula editor.

### Advanced XLKitLearn Formulas

The formula language used by XLKitLearn also allows you to seamlessly transform variables in your data in a number of useful ways:

* **Creating categorical variables**: When a variable needs to be treated as [categorical](https://en.wikipedia.org/wiki/Categorical_variable), XLKitLearn can automatically create [dummy variables](https://en.wikipedia.org/wiki/Dummy_variable_(statistics)) on the fly. Simply surround the variable with `C()`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using the crime per capita and the highway accessibility treated as a categorical. The formula would be

    `median_property_value ~ crime_per_capita + C(highway_accessibility)`
    
    A few notes
      * When a variable is treated as categorical, dummies will always be created for every possible value of that variable. The first dummy (in alphabetical order) will then be [dropped](https://en.wikipedia.org/wiki/Dummy_variable_(statistics)) from the model.
      * Any column containing non-numerical values will automatically be created as a categorical, with or without the `C()` around it.
* **Standardizing variables**: Surrounding any variable with `standardize()` will subtract the mean of that variable form every value in the column, and divide by its standard deviation. This can be particularly useful when comparing the size of coefficients in a linear regression. Note that when using training and test sets, this transformation will be done correctly; the mean of the *training* set will be used to standardize variables in the evaluation set.
* **Removing the intercept**: An intercept will be added to the formula by default. Adding `-1` at the end of the formula will remove the intercept.
* **Applying Python functions**: Any Python function can be used as part of a formula; `numpy` functions can be accessed using `np.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using the crime per capita and the crime per capita squared. The formula would be

    `median_property_value ~ crime_per_capita + np.power(crime_per_capita, 2)`
    
* **Including all variables**: to include every variable in the data as independent variables in your model, simply use `~.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using every other variable in the data. The formula would then be
    `median_property_value ~ .`

!!! warning "Categorical Variables and `~.`"
        As mentioned above, any column containing non-numerical values that is explicitly named in the formula will automatically be treated as a categorical variable. This, however, is not true of variables that are implicitely included using `~.`.
* **Creating a yes/no outcomes**: As described [below](#specifying-a-predictive-model), XLKitLearn will automatically interpret any formula in which the outcome column contains only 0s and 1s as a classification task. In some cases, the outcome column contains other values, but the model still needs to be treated as a classification task. In those circumstances, XLKitLearn allows you to automatically convert the outcome variable to '1' when it is equal to a certain value, and 0 otherwise. This is best illustrated by example. Suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict whether the highway accessibility is equal to 4 or not using every other column in the data. You could use the following formula:
    ```(highway_accessibility = 4) ~ .```
    The model will then interpret all rows in which highway accessibility is 4 as a '1' outcome, and '0' otherwise. Note that this only works with `~.` formula (i.e., those that use every variable in the dataset).

Note that XLKitLearn's formula feature builds on the Python library [Patsy](https://patsy.readthedocs.io/en/latest). Any formula features available in Patsy can also be used in XLKitLearn. Some of the features described above, however, are not directly available in patsy.

---

## Specifying a Predictive Model

XLKitLearn predictive models comprise two main ingredients:

1. The predictive model to be used, and any parameters required to fit this model.
2. The variable that needs to be predicted (the dependent variable) as well as the variables that should be used to predict it (the independent variables).

The model can be selected among [models available](#supported-predictive-models) in the `Model` dropdown, and its parameters can be specified in the `Parameter(s)` box below.

To specify the dependent and independent variables, XLKitLearn leverages the [patsy](https://patsy.readthedocs.io/en/latest/) library to allow the specification of these variables as a _Patsy formula_ (similar but not identical to [R](https://www.r-project.org/)-style formulas). It also adds a number of [features](#advanced-xlkitlearn-formulas) above and beyond those available in Patsy.

The basic structure of each formula is as follows:

`[Dependent Variable] ~ [Independent Variable 1] + [Independent Var. 2] + ...`

Formulas can be typed directly into the formula box of the add-in settings, or using the [formula editor](#the-formula-editor).

!!! note "Specify Variables"
    Each variable should be referred to by the column headers in the first row of the [training data](#selecting-data).

---

### Supported predictive models

The following predictive models can be chosen from the dropdown.

1. **Linear and logistic regression**: XLKitLearn will automatically determine whether a continuous [linear regression](https://en.wikipedia.org/wiki/Linear_regression) model or a binary [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) model should be used. If the dependent variable in the model contains 0s or 1s only (_or_ if the dependent variable is a [logical statement](#logical-dependent-var)), a logistic regression model will be used. Otherwise, a linear regression model will be used. Note that XLKitLearn does not support classification models with more than two possible outcomes. XLKitLearn makes one unique parameter available for these models - the [Lasso penalty](https://en.wikipedia.org/wiki/Lasso_(statistics)). This parameter should either be specified as a number indicating the weight of the Lasso penalty _or_ can be set to ```BS``` to perform best subset selection. Compare multiple lasso penalties by separating entries with an ```&```.

2. **Decision tree**: A simple [decision tree](https://en.wikipedia.org/wiki/Decision_tree_learning) will be fit using CART. `Tree depth` is the only unique parameter that needs to be chosen; multiple depths can be compared by separating depths with an ```&```.

3. **Boosted decision tree**: A boosted decision tree based on the given `Tree depth`, `Max trees` and `Learning rate`. Different tree depths can be compared by using an `&`.
> Only the tree depth should be compared because XLKitLearn will automatically generate trees based on the max number of trees.

4. **Random forest**: Fit a random forest based on `Number of trees` and `Tree depth` which can be compared with an `&`.

---

## Parameter Tuning
Specifying the number of folds in the settings will tell the add-in to run a [k-fold cross validation](https://en.wikipedia.org/wiki/Cross-validation_(statistics)) with the given number of folds. The training and test data will be randomly sorted into folds based upon the [Randomization Seed](https://en.wikipedia.org/wiki/Random_seed) then all regressions will be automatically run. The model with the highest average out-of-sample score for the training data will used on the evaluation dataset if an [evaluation dataset](#Specifying_an_evaluation_set) is specified.

Two or more models can be compared using an ```&``` to separate each model in the [formula editor](#the-formula-editor).

!!! success "Replicating Analysis"
    Using the same `Randomization Seed` can enable two users to generate the same random order. Enter a random list of numbers once the model is finished to truly randomly sort the data.

---

## Training and Test Sets
`No evaluation set` option is selected by default. Reserve data for evaluation by either specifying the proportion of data to be set aside or by selecting a specific evaluation data set range the same way training data is selected.
!!! warning "Crashing"
    Be careful not to generate an evaluation output dataset that is too big for excel or it might crash.

---

## Making predictions on new data
Make predictions on new input data using by adding data to the `Make predictions for new data` section. The model with the highest out-of-sample score is used to make predictions on the new data and the predicted outcomes will be added to the output.

---

## The Randomization Seed

---

## Understanding the Predictive Add-in Output
The model output is dynamic based upon the model and dataset. Depending on model type and parameters not all of the following sections will be returned. The output will be separated into:


1. **Parameter tuning**: A table summarizes the results of the K-Fold Cross-Validation. Each row corresponds to the model the user inputted and shows the In-sample score, Out-of-sample score, Out-of-sample SE, Formula, Lasso penalty and the number of Nonzero coefficients.Below the summary table are two charts to visually compare the models. The first chart shows the out-of-sample score and out-of-sample score SE by the row number / model input. The second chart also shows the out-of-sample score and out-of-sample score SE by the number of non-zero coefficients.
2. **Model**: The model and lasso penalty is shown along with the in-sample score at the top of the model table. If single model is run, that model is used, if multiple models were compared, the model that returned the highest average out-of-sample score on the training set is returned. The model table will list each variable and the associated coefficient. When a single linear regression is run (is this correct  @guetta?) the t-stat, 97.5% confidence interval, p-value and stars indicating levels of statistical significance are also listed.
3. **Model Evaluation**: Lists every predicted and true outcome for the entire evaluation dataset. The AUC cure and the out-of-sample score for the evaluation set is generated when a evaluation set is specified.
   > Very large datasets can generate evaluation datasets that are too big for excel, so be mindful of how large your evaluation dataset when deciding to include the evaluation

4. **Model Prediction**: The predicted outcomes for each input
> True outcome will not be listed because perditions are being made on inputs that don't have outcomes.

5. **Equivalent Python code**: The Python code that was used to run the analysis with descriptions for each section of code.
6. **Technical Details**: The input settings that we're used to run the analysis. This can be copied and pasted directly into the "Settings" on the Add-in tab to run the exact same analysis again. The add-in version and the time for each step of the analysis are also noted.

## Advanced options


Windows users have two advanced options below the setting to give the user more control over early termination and processing speed. Mac users have ______________


!!! Note "Early Termination"
    If the early termination option is selected, a black Python console will pop up. Close the console to stop the analysis.
    
---

