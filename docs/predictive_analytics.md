# Predictive Analytics

XLKitLearn's predictive analytics add-in allows you to fit a range of predictive models in Excel. There are three main components of predictive analytic workflows in XLKitLearn:

1. [Specify the data](#selecting-data) to be used to train the predictive model; XLKitLearn can read data from the current Excel workbook, or pull data directly from a file.
2. Describe the dependent and independent variables using a [formula](#specifying-a-formula); XLKitLearn's formula language allows you to carry out variable transformations, automatically create dummies for categorical variables, and more.
3. Select the [predictive model](#specifying-a-predictive-model) to be used, together with any parameters required.

The predictive analytic add-in also includes full support for [training and test sets](#training-and-evaluation-sets), [parameter tuning](#parameter-tuning) using K-Fold cross-validation, and [inference on new datasets](#making-predictions-on-new-data).

## Selecting data

The simplest way to load data into XLKitLearn is to drag any sheets containing the data into the add-in workbook.

!!! warning "Adding Sheets Containing Data"
    You should never move the add-in sheet *to* another workbook. If your data sits in another Excel workbook, bring the sheet containing your data *into* the add-in Excel workbook.

Once the data is in your workbook, click on the <a id="select-data" href="#" title="">"Click to Select"</a> button under "Training" in the add-in dialogue, and select the entire dataset you will be using, including column headers. Click [here](Select_Small_Data.gif) for a demo.
    
!!! note "Column Names"
    The first row of your data should contain column names. There are a number of rules these column names must satisfy. They must contain only letters, numbers, and underscores - no spaces - and cannot start with a number. Column names also cannot be python reserved keywords or the word "intercept".

---

### Reading data from files

When working with a very large dataset, opening it in Excel may slow down your computer. In fact, it may be altogether impossible.

For these situations, XLKitLearn allows you to read data directly from a source file, without having to open it in Excel. After clicking on "Click to select", simply type the name of the file directly into the dialogue box. Click [here](Select_Large_Data.gif) for a demo. The first row of the file should contain the column names (the same column name rules apply).

XLKit learn can read .csv files as well as .xlsx files; make sure to include the file extension (.csv or .xslx) when you specify the file name.

---

## Specifying a Formula

In XLKitLearn, a formula specifies the *outcome* variable to be predicted (the dependent or 'y' variable) and the features that will be used to predict this variable (the independent or 'x' variables). The formula should be input into the <a id="formula-box" href="#" title="">formula box</a>.

XLKitLearn formulas are always provided in the following format

`dependent variable ~ independent variable 1 + independent variable 2 + ...`

Variables are referred to using the columns names in the data.

For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using the crime per capita and the average number of rooms per dwelling. The formula would then be

`median_property_value ~ crime_per_capita + av_rooms_per_dwelling`

### The Formula Editor

You can always type the formula into the <a id="formula-box2" href="#" title="">formula box</a> manually. To make things easier for you, XLKitLearn also includes a formula editor, which offers a number of helpful features. To launch the formula editor, click on the [three dots](SCREENSHOT gif of clicking on the three dots and loading the formula editor) next to the formula box.

The formula editor contains a <a id="formula-editor-list" href="#" title="">list at the left</a> of all the variables in the selected data (whether in Excel or in an external file). It also contains a <a id="formula-editor" href="#" title="">larger textbox</a> in which to type your formula.

The formula editor also supports auto-complete for quicker formula entry. As variable names are typed in the formula entry box, the list on the left is filtered down to variables that begin with those letters, and the first such variable is automatically suggested. Pressing the Tab key will complete the name of that variable. Click [here](Formula_Editor.gif) for a demo.

!!! note "Variables Names in External Files"
    In the following circumstances, XLKitLearn will not be able to load variable names from the file and display them in the formula editor. (1) If you are loading a .xlsx file, XLKitLearn will not be able to open the file and read column names. (2) If you are on a Mac and loading an external file, Mac security settings will stop XLKitLearn from opening the file and reading column names.

If the formula you entered contains an error, the formula input box will turn red, and an error message will be displayed in the <a id="formula-editor-error" href="#" title="">bottom part</a> of the formula editor.

### Advanced XLKitLearn Formulas

The formula language used by XLKitLearn also allows you to seamlessly transform variables in your data in a number of useful ways:

* **Creating categorical variables**: When a variable needs to be treated as [categorical](https://en.wikipedia.org/wiki/Categorical_variable), XLKitLearn can automatically create [dummy variables](https://en.wikipedia.org/wiki/Dummy_variable_(statistics)) on the fly. Simply surround the variable with `C()`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the crime per capita and the highway accessibility treated as a categorical. The formula would be

    `median_property_value ~ crime_per_capita + C(highway_accessibility)`
    
    A few notes:
      - When a variable is treated as categorical, dummies will always be created for every possible value of that variable. The first dummy (in alphabetical order) will then be [dropped](https://en.wikipedia.org/wiki/Dummy_variable_(statistics)) from the model.
      - Any column containing non-numerical values will automatically be created as a categorical, with or without the `C()` around it.
    
* **Standardizing variables**: Surrounding any variable with `standardize()` will subtract the mean of that variable form every value in the column, and divide by its standard deviation. This can be particularly useful when comparing the size of coefficients in a linear regression. Note that when using training and test sets, this transformation will be done correctly; the mean of the *training* set will be used to standardize variables in the evaluation set.

* **Including all variables**: To include every variable in the data as independent variables in your model, simply use `~.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using every other variable in the data. The formula would then be

    `median_property_value ~ .`

* **Including interaction terms**: It is sometimes necessary to include an [interaction term](https://en.wikipedia.org/wiki/Interaction_(statistics)) in a predictive model. This is done using the ` * ` character. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the interaction of crime per capita and highway accessibility. The formula would be 

    ` median_property_value ~ crime_per_capita * highway_accessibility ` 

    Note the the ` a*b ` construction is shorthand for ` a + b + a x b`. In the example above, the model output would include both crime per capita and highway accessibility as dependent variables on their own, as well as the interaction term.

* **Removing the intercept**: An intercept will be added to the formula by default. Adding `-1` at the end of the formula will remove the intercept.

* **Applying Python functions**: Any Python function can be used as part of a formula; `numpy` functions can be accessed using `np.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the crime per capita and the crime per capita squared. The formula would be

    `median_property_value ~ crime_per_capita + np.power(crime_per_capita, 2)`

!!! warning "Categorical Variables and `~.`"
        As mentioned above, any column containing non-numerical values that is explicitly named in the formula will automatically be treated as a categorical variable. This, however, is not true of variables that are implicitely included using `~.`. Therefore, this will return an error.

* **Creating yes/no outcomes**: XLKitLearn will [automatically interpret](#regression-vs-classification) any formula in which the outcome column contains only 0s and 1s as a classification task. In some cases, the outcome column contains other values, but the model still needs to be treated as a classification task. In those circumstances, XLKitLearn allows you to automatically convert the outcome variable to '1' when it is equal to a certain value, and '0' otherwise. This is best illustrated by example. Suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict whether the highway accessibility is equal to 4 or not using every other column in the data. You would use the following formula:
    ```(highway_accessibility = 4) ~ .```
    The model will then interpret all rows in which highway accessibility is 4 as a '1' outcome, and '0' otherwise. 

!!! warning "Non-numeric Columns"
        Including a non-numeric column as the dependent variable will return an error unless a yes/no outcome is created in this way.

Note that XLKitLearn's formulas leverage the [Patsy](https://patsy.readthedocs.io/en/latest) Python library, which itself approximates [R](https://www.r-project.org/)-style formulas. Any formula features available in Patsy can also be used in XLKitLearn. Some of the features described above, however, are not directly available in Patsy.

---

## Specifying a Predictive Model

The final component of any predictive analytic workflow is the model that will be trained and used to make predictions. The model can be selected in the <a id="model-dropdown" href="#" title="">model dropdown</a>.

Some predictive models require parameters to be specified. In those cases, the parameters can be specified in the <a id="parameters" href="#" title="">parameter section</a>, which will update to list the parameters required for the specific model select. In addition, any parameters that are required (and cannot be left blank) will highlight in red.

### Regression vs Classification

For each of its models, XLKitLearn supports both regression models (with continuous outcomes) and classification models (with 0/1 outcomes). XLKitLearn will automatically determine the type of model required - if the outcome column (as specified in the [formula](#specifying-a-formula)) only contains 0s and 1s, a classification model will be fit. Otherwise, a regression model will be fit.

Note that XLKitLearn does not currently support classification models with more than two possible outcomes.

### Supported Models

XLKitLearn supports the following models:

1. **Linear and Logistic Regression**: Depending on whether a [regression or classification](#regression-vs-classification) model is required, XLKitLearn will fit a [linear regression](https://en.wikipedia.org/wiki/Linear_regression) model or a [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) on the data. XLKitLearn makes the following parameter(s) available for this model
   
    * The [Lasso penalty](https://en.wikipedia.org/wiki/Lasso_(statistics)), which can be specified in one of three ways
        - If it is left blank, XLKitLearn will assume it is equal to 0, and fit an unpenalized linear model
        - It can be set to a single numerical value to specify the weight of the Lasso penalty in the model
        - If set to the special value ```BS```, XLKitLearn will perform best-subset selection, in which every possible combination of the variables provided will be tested against each other using K-fold cross validation. Note that this option is only available when the formula includes fewer than 10 variables        
    
2. **K-Nearest Neighbors**: A simple k-nn regressor or classifier will be fit on the data. XLKitLearn makes the following parameter(s) available for this model
   
    * The number of neighbors, K
    * The weighting to be used when averaging neighboring values. This parameter can take two possible values: 'uniform' (or 'u'), and 'distance' (or 'd'). In the case of uniform weighting, all neighbors will be weighted equally, and in the case of distance weighting, closer neighbors will be weighted more heavily and distant neighbors will be weighted less. If this field is left blank, uniform weighting will be used by default.
    
2. **Decision Tree**: A simple [decision tree](https://en.wikipedia.org/wiki/Decision_tree_learning) will be fit using CART. The splitting criterion will be chosen based on whether a [regression or classification](#regression-vs-classification) model is required; the mean-squared-error for the former, and the gini impurity for the latter. XLKitLearn makes the following parameter(s) available for this model
   
    * The tree depth, which determines the maximum depth of the tree
    
3. **Boosted Decision Tree**: A [boosted decision tree](https://en.wikipedia.org/wiki/Gradient_boosting) fit using gradient boosting. XLKitLearn makes the following parameter(s) available for this model:
   
    * The tree depth, which determines the maximum depth of each tree in the ensemble
    * The maximum number of trees in the ensemble
    > Boosted trees do not always continue improving as extra trees are added - after a certain point, performance drops. Thus, after fitting the number of trees specified by this parameter, XLKitLearn will automatically determine whether some smaller number of trees performs better on the left-out folds in K-fold cross validation. For this reason, boosted decision trees *always* need a value for K in the [parameter tuning](#parameter-tuning) section of the addin, even when multiple parameters are not being compared against each other.
    * The learning rate for the gradient boosting procedure
    
4. **Random Forest**: A [random forest](https://en.wikipedia.org/wiki/Random_forest). XLKitLearn makes the following parameter(s) available for this model:
   
    * The tree depth, which determines the maximum depth of each tree in the ensemble
    * The number of trees in the ensemble

For specific details on how these models are implemented, see the [generated Python code](SCREENSHOT link to the generated python code in the main intro page) for each model - these generated pieces of code will illustrate how SciKitLearn packages are used to fit each of these models.

---

## Parameter Tuning

XLKitLearn supports [K-fold cross validation]((https://en.wikipedia.org/wiki/Cross-validation_(statistics)) for parameter tuning. The model that performs *best* on all the folds on average will be selected and fit on the entire data.

In particular, for any of the parameters described [above](#supported-models), multiple values of the parameters can be compared against each other by specifying several values, separated by `&` signs. For example, if you are fitting a decision tree and would like to compare trees of depth 2, 3, and 4, you should enter `2 & 3 & 4` in the "tree depth" parameter for the decision tree.

In addition to comparing multiple parameters against each other, you can also test multiple formulas against each other. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value. To compare a model that makes this prediction using `crime_per_capita` to one that makes this prediction using `av_rooms_per_dwelling`, you would type the following formula into XLKitLearn:

`median_property_value ~ crime_per_capita & median_property_value ~ av_rooms_per_dwelling`

!!! note "Comparing Two Formulas"
        Note that when comparing two formulas, the dependent variable (`median_property_value` above) needs to be repeated for each formula. Additionally, all formulas being compared must have the same dependent variable.

XLKitLearn allows you to specify the number of folds to use; simply enter the number in the <a id="kfold-box" href="#" title="">appropriate box</a>.

!!! note "Required Cross Validation"
        If your model requires cross-validation (i.e., if you are testing multiple parameters or formulas against each other) but you do not specify the number of folds, the "folds" box will turn red to remind you this parameter is required. Remember that when fitting a boosted decision tree, cross-validation is *always* required.

The folds will be split randomly, using the [randomization seed](#the-randomization-seed) provided in the add-in.

---

## Training and Evaluation Sets
XLKitLearn is able to work with separate training and evaluation datasets. Evaluation datasets are specified in the 'Evaluation' section of the add-in in a number of ways. First, a test dataset can be automatically generated as a percentage of the provided training set. Second, a test dataset can be specified manually in the same way the training data is specified. By default, the `No evaluation set` option is selected.

Each option is explained in depth below:

1. 'Automatically generate an evaluation set with __ % of the training data': When this option is selected, the add-in will require a number (between 0 and 100) be entered in the blank. The add-in will then randomly select a portion of the training data according to the percentage indicated, and set it aside as evaluation data. The randomization seed is used to split the training data, so running the add-in multiple times with the same percentage and randomization seed will yield identical evaluation sets.
2. 'Use a specific evaluation set': With this option, evaluation data can be selected manually, in the same way the training data is selected. Note that if column headers are included in this dataset, they must match headers in the training dataset. If headers do not match, but the number of columns do match, then the headers in the training dataset will be used. If neither headers nor the number of columns match, an error will be returned.
3. 'No evaluation set': In this case, the entire training set will be used to train the model and no evaluation output will appear in the Excel output sheet.

!!! warning "Large evaluation datasets"
    If the evaluation dataset you provide is large, XLKitLearn will output it to a file directly rather than to the output Excel spreadsheet.
    

---

## Making Predictions on New Data

Make predictions on new input data using the `Make predictions for new data` section. The model with the highest out-of-sample score is used to make predictions on the new data and the predicted outcomes will be added to the output.

As in the case with selecting an evaluation dataset manually, column headers, if included, must match headers in the training dataset. If headers do not match, but the number of columns do match, then the headers in the training dataset will be used. If neither headers nor the number of columns match, an error will be returned.

---

## The Randomization Seed

The randomizing seed determines the choices XLKitLearn makes whenever randomizing is required. This includes

1. The random splitting of training data into training and test datasets
2. The random splitting of training data into K folds for cross validation
3. The randomness involved in fitting models (eg: the random features selected in each tree of a random forest)

If the add-in is run twice with the same randomization seed, the results will be *identical*. This can be useful in a classroom setting, to ensure the entire class gets the same answer.

---

## Understanding the Predictive Add-in Output
The model output is dynamic based on the selected model and dataset. Depending on model type and parameters, not all of the following sections will be returned.

### Parameter tuning

First is a table summarizing the results of the k-fold cross-validation. Each row corresponds to one formula/parameter combination and includes the in-sample score, out-of-sample score, out-of-sample SE, formula, and parameter values. Below the table is a chart of the out-of-sample score and out-of-sample score SE by the row number. In the case of linear/logistic regression, out-of-sample score will also be shown against the number of nonzero coefficients, and for any of the decision tree models, out-of-sample score will be shown against number of trees.

### Model

This section describes the model that was fit using the training/evaluation data. If single model is run, that model is described here. If multiple models were compared, the model that returned the highest average out-of-sample score on the training set is described. First, the model formula and parameters are given, along with the in-sample score. Next, if the model is a linear/logistic regression, coefficients will be shown. For other models, variable importance will be provided in both a table and a chart.

Variable importance measures the impact of each variable's contribution to the model. The add-in does this by randomly swapping variable values (up and down a column in Excel, for example) and seeing by how much the out-of-sample score decreases. The variables that cause the greatest drops in out-of-sample score are the most important.

### Model evaluation

First, the out-of-sample score of the model is presented. If evaluating a classfication model (binary output), the ROC curve is output next. Lastly, a table containing the predicted and true outcome, as well as the row number, for the entire evaluation dataset is given. The row number can be used to identify the input variable values for rows in the evaluation dataset.

Note that the out-of-sample score of the model on the evaluation dataset will be, and should be, different than the out-of-sample score of the model on the training set.

### Model prediction

This short section includes a table listing the predicted outcomes for each input in the prediction dataset. True outcomes will not be listed because predictions are being made on data that doesn't include outcomes.

### Equivalent Python Code

This section is XLKitLearn's crown jewel. It contains automatically generated Python code that will carry out analyses equivalent to those run by XLKitLearn.

The code in this section is *not* simply the code that the add-in uses. It is dynamically generated to ensure maximal pedagogical impact. The simplest scikit learn function will always be used in this code, and plentiful comments will be included.

Note that:
  - If the data was selected from a specific sheet in the Excel workbook, the generated Python code will attempt to pull the data from the same sheet. The Excel workbook has to be open for this to work.
  - If the data was loaded from a file, the generated Python code will attempt to pull the data from the same file. The code must therefore be in the same directory as the file.

### Technical details

The input settings that were used to run the analysis. This can be copied and pasted directly into the "Settings" cell in the Add-in tab to run the exact same analysis again. The add-in version and the time for each step of the analysis are also noted.

!!! Note "Copying a Settings String"
    It is sometimes helpful to share the entire settings string in order to ensure an entire class gets the same result. However, the settings string contains the file name and sheet name of the device used to generate the output sheet from which the settings string is copied. Since file names and sheet names vary across devices, any student copying a string from another device must account for this variance.

---

## Advanced Options

Windows users have two advanced options below the settings to give the user more control over early termination and processing speed. 

1. 'Keep an active Python connection': Selecting this option will launch a Python terminal when the add-in is run and will keep the terminal open after the calculations and output have been completed. This is helpful for two reasons:
   - With the terminal open, it is possible to terminate the calculations early by pressing Ctrl+C
   - If running multiple models in succession, keeping the Python connection active will allow all subsequent models to run more quickly, since a Python connection need not be opened at the start of each run
2. 'Run Python visibly to allow early termination': With this option, a Python terminal will be launched to allow for early termination. However, the terminal is closed automatically when the add-in has finished.

Mac users have the option to "Attempt to terminate python," which sends a kill code to Python running in the background. Because this button is pressed while the add-in is running, it can take a few seconds for the Python terminate.

<script>
  $(document).ready(function() {
    $("head").append('<link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">');
</script>
<script>
    $("#select-data").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/select-data.png" />' });
    $("#formula-box").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-box.png" />' });
    $("#formula-box2").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-box.png" />' });
    $("#formula-editor-list").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor-list.png" />' });
    $("#formula-editor").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor.png" />' });
    $("#formula-editor-error").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor-error.png" />' });
    $("#kfold-box").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/kfold-box.png" />' });
    $("#model-dropdown").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/model-dropdown.png" />' });
    $("#parameters").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/parameters.png" />' });
  })
</script>
