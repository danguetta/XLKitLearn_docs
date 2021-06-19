# Predictive Analytics

XLKitLearn's predictive analytics addin allows you to fit a range of predictive models in Excel. There are three main components of predictive analytic workflows in XLKitLearn:

1. [Specify the data](#selecting-data) to be used to train the predictive model; XLKitLearn can read data from the current Excel workbook, or pull data directly from a file.
2. Describe the dependent and independent variables using a [formula](#specifying-a-formula); XLKitLearn's formula language allows you to carry out variable transformations, automatically create dummies for categorical variables, and more.
3. Select the [predictive model](#specifying-a-predictive-model) to be used, together with any parameters required.

The predictive analytic addin also includes full support for [training and test sets](#training-and-evaluation-sets), [parameter tuning](#parameter-tuning) using K-Fold cross-validation, and [inference on new datasets](#making-predictions-on-new-data).

## Selecting data

The simplest way to load data into XLKitLearn is to drag any sheets containing the data into the addin workbook.

!!! warning "Adding Sheets Containing Data"
    You should never move the addin sheet *to* another workbook. If your data sits in another Excel workbook, bring the sheet containing your data *into* the addin Excel workbook.

Once the data is in your workbook, click on the <a id="select-data" class="hover-link" href="javascript:;" title="">"Click to Select"</a> button under "Training" in the addin dialogue, and select the entire dataset you will be using, including column headers. Hover <a id="small-data" class="hover-link" href="javascript:;" title="">here</a> for a demo.
    
!!! note "Column Names"
    The first row of your data should contain column names. There are a number of rules these column names must satisfy. They must contain only letters, numbers, and underscores - no spaces - and cannot start with a number. Column names also cannot be python reserved keywords or the word "intercept".

---

### Reading data from files

When working with a very large dataset, opening it in Excel may slow down your computer. In fact, it may be altogether impossible if the dataset contains more rows than Excel can handle.

For these situations, XLKitLearn allows you to read data directly from a source file, without having to open it in Excel. After clicking on "Click to select", simply type the name of the file directly into the dialogue box. Hover <a id="large-data" class="hover-link" href="javascript:;" title="">here</a> for a demo. The first row of the file should contain the column names (the same column name rules apply).

XLKit learn can read .csv files as well as .xlsx files; make sure to include the file extension (.csv or .xslx) when you specify the file name.

---

## Specifying a Formula

In XLKitLearn, a formula specifies the *outcome* variable to be predicted (the dependent or 'y' variable) and the features that will be used to predict this variable (the independent or 'x' variables). The formula should be input into the <a id="formula-box" class="hover-link" href="javascript:;" title="">formula box</a>.

XLKitLearn formulas are always provided in the following format

`dependent_variable ~ independent_variable_1 + independent_variable_2 + ...`

Variables are referred to using the columns names in the data.

For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median house price using the crime per capita and the average number of rooms per dwelling. The formula would then be

`median_property_value ~ crime_per_capita + av_rooms_per_dwelling`

### The Formula Editor

You can always type the formula into the <a id="formula-box2" class="hover-link" href="javascript:;" title="">formula box</a> manually. To make things easier for you, XLKitLearn also includes a formula editor, which offers a number of helpful features. To launch the formula editor, click on the <a id="formula-editor-gif" class="hover-link" href="javascript:;" title="">three dots</a> next to the formula box. DREW - love the animated gif on hover, but it doesn't reset - in other words, if you hover over and play it, and then hover again, it stays static.

The formula editor contains a <a id="formula-editor-list" class="hover-link" href="javascript:;" title="">list at the left</a> of all the variables in the selected data (whether in Excel or in an external file). It also contains a <a id="formula-editor" class="hover-link" href="javascript:;" title="">larger textbox</a> in which to type your formula.

The formula editor also supports auto-complete for quicker formula entry. As variable names are typed in the formula entry box, the list on the left is filtered down to variables that begin with those letters, and the first such variable is automatically suggested. Pressing the Tab key will complete the name of that variable. Hover <a id="formula-editor-large-gif" class="hover-link" href="javascript:;" title="">here</a> for a demo.

!!! note "Variables Names in External Files"
    In the following circumstances, XLKitLearn will not be able to load variable names from the file and display them in the formula editor. (1) If you are loading a .xlsx file, XLKitLearn will not be able to open the file and read column names. (2) If you are on a Mac and loading an external file, Mac security settings will stop XLKitLearn from opening the file and reading column names.

If the formula you entered contains an error, the formula input box will turn red, and an error message will be displayed in the <a id="formula-editor-error" class="hover-link" href="javascript:;" title="">bottom part</a> of the formula editor.

### Advanced XLKitLearn Formulas

The formula language used by XLKitLearn also allows you to seamlessly transform variables in your data in a number of useful ways:

* **Creating categorical variables**: When a variable needs to be treated as categorical ([wikipedia reference](https://en.wikipedia.org/wiki/Categorical_variable)), XLKitLearn can automatically create dummy variables ([wikipedia reference](https://en.wikipedia.org/wiki/Dummy_variable_(statistics))) on the fly. Simply surround the variable with `C()`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the crime per capita and the highway accessibility treated as a categorical. The formula would be

  `median_property_value ~ crime_per_capita + C(highway_accessibility)`

  A few notes. (a) When a variable is treated as categorical, dummies will always be created for every possible value of that variable. The first dummy (in alphabetical order) will then be dropped from the model. (b) Any column containing non-numerical values will automatically be created as a categorical, with or without the `C()` around it.

* **Standardizing variables**: Surrounding any variable with `standardize()` will subtract the mean of that variable form every value in the column, and divide by its standard deviation. This can be particularly useful when comparing the size of coefficients in a linear regression. Note that when using training and test sets (DREW add link), this transformation will be done correctly; the mean of the *training* set will be used to standardize variables in the evaluation set.

* **Including all variables**: To include every variable in the data as independent variables in your model, simply use `~.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using every other variable in the data. The formula would then be

  `median_property_value ~ .`

!!! warning "Categorical Variables and `~.`"
        As mentioned above under "Creating categorical variables", any column containing non-numerical values that is explicitly named in the formula will automatically be treated as a categorical variable. This, however, is not true of variables that are implicitly included using `~.`. Therefore, if any of your columns contain non-numerical values, using a `~.` will result in an error.

* **Including interaction terms**: It is sometimes necessary to include an interaction term ([wikipedia reference](https://en.wikipedia.org/wiki/Interaction_(statistics))) in a predictive model. This is done using the ` * ` character. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the interaction of crime per capita and highway accessibility. The formula would be 

  `median_property_value ~ crime_per_capita * highway_accessibility ` 

  Note the the ` a*b ` construction is shorthand for ` a + b + a x b`. In the example above, the model output would include both crime per capita and highway accessibility as dependent variables on their own, as well as the interaction term.

* **Removing the intercept**: An intercept will be added to the formula by default. Adding `-1` at the end of the formula will remove the intercept.

* **Applying Python functions**: Any Python function can be used as part of a formula; `numpy` functions can be accessed using `np.`. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value using the crime per capita and the crime per capita squared. The formula would be

  `median_property_value ~ crime_per_capita + np.power(crime_per_capita, 2)`

* **Creating yes/no outcomes**: XLKitLearn will [automatically interpret](#regression-vs-classification) any formula in which the outcome column contains only 0s and 1s as a classification task.

  XLKitLearn also supports one-versus-all classification models, in which an outcome column is converted to a 1 if it is equal to a certain value, and 0 otherwise. This is best illustrated by example. Suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict whether the highway accessibility is equal to 4 or not using every other column in the data. You would use the following formula:
  `(highway_accessibility = 4) ~ .`
  The model will then interpret all rows in which highway accessibility is 4 as a '1' outcome, and '0' otherwise. 

!!! warning "Non-numeric Columns as dependent variables"
        Including a non-numeric column as the dependent variable will return an error unless a yes/no outcome is created in this way.

Note that XLKitLearn's formulas leverage the [Patsy](https://patsy.readthedocs.io/en/latest) Python library, which itself approximates [R](https://www.r-project.org/)-style formulas. Any formula features available in Patsy can also be used in XLKitLearn. Some of the features described above, however, are not directly available in Patsy.

---

## Specifying a Predictive Model

The final component of any predictive analytic workflow is the model that will be trained and used to make predictions. The model can be selected in the <a id="model-dropdown" class="hover-link" href="javascript:;" title="">model dropdown</a>. DREW update this to include knn in the dropdown 

Some predictive models require parameters to be specified. In those cases, the parameters can be specified in the <a id="parameters" class="hover-link" href="javascript:;" title="">parameter section</a>, which will update to list the parameters required for the specific model selected. In addition, any parameters that are required (and cannot be left blank) will be highlighted in red.

### Regression vs Classification

For each of its models, XLKitLearn supports both regression models (with continuous outcomes) and classification models (with 0/1 outcomes). XLKitLearn will automatically determine the type of model required - if the outcome column (as specified in the [formula](#specifying-a-formula)) only contains 0s and 1s, a classification model will be fit. Otherwise, a regression model will be fit.

Note that XLKitLearn does not currently support classification models with more than two possible outcomes. As described above, however, these can be coverted into one-versus-all classifcation models.

### Supported Models

XLKitLearn supports the following models:

1. **Linear and Logistic Regression**: Depending on whether a [regression or classification](#regression-vs-classification) model is required, XLKitLearn will fit a linear regression ([wikepedia reference](https://en.wikipedia.org/wiki/Linear_regression)) model or a logistic regression ([wikipedia reference](https://en.wikipedia.org/wiki/Logistic_regression)) on the data. XLKitLearn makes the following parameter(s) available for this model

   * The Lasso penalty ([wikipedia reference](https://en.wikipedia.org/wiki/Lasso_(statistics))), which can be specified in one of three ways
     - If it is left blank, XLKitLearn will assume it is equal to 0, and fit an unpenalized linear model.
     - It can be set to a numerical value to specify the weight of the Lasso penalty in the model.
     - If set to the special value ```BS```, XLKitLearn will perform best-subset selection, in which every possible combination of the variables provided will be tested against each other using K-fold cross validation. Note that this option is only available when the formula includes fewer than 10 variables.

2. **K-Nearest Neighbors**: A simple k-nearest-neighbors ([wikipedia reference](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)) regressor or classifier will be fit on the data. XLKitLearn makes the following parameter(s) available for this model

   * The number of neighbors, K
   * The weighting to be used when averaging neighboring values. This parameter can take two possible values: 'uniform' (or 'u'), and 'distance' (or 'd'). In the case of uniform weighting, all neighbors will be weighted equally, and in the case of distance weighting, closer neighbors will be weighted more heavily and distant neighbors will be weighted less. If this field is left blank, uniform weighting will be used by default.

3. **Decision Tree**: A simple decision tree ([wikipedia reference](https://en.wikipedia.org/wiki/Decision_tree_learning)) will be fit using CART. The splitting criterion will be chosen based on whether a [regression or classification](#regression-vs-classification) model is required; the mean-squared-error for the former, and the gini impurity for the latter. XLKitLearn makes the following parameter(s) available for this model

   * The tree depth, which determines the maximum depth of the tree

4. **Boosted Decision Tree**: A boosted decision tree ([wikipedia reference](https://en.wikipedia.org/wiki/Gradient_boosting)) fit using gradient boosting. XLKitLearn makes the following parameter(s) available for this model:

   * The tree depth, which determines the maximum depth of each tree in the ensemble
   * The maximum number of trees in the ensemble

   > Boosted trees do not always continue improving as extra trees are added - after a certain point, performance drops. Thus, after fitting the number of trees specified by this parameter, XLKitLearn will automatically determine whether some smaller number of trees performs better on the left-out folds in K-fold cross validation. For this reason, boosted decision trees *always* need a value for K in the [parameter tuning](#parameter-tuning) section of the addin, even when multiple parameters are not being compared against each other.

   * The learning rate for the gradient boosting procedure

5. **Random Forest**: A random forest ([wikipedia reference](https://en.wikipedia.org/wiki/Random_forest)). XLKitLearn makes the following parameter(s) available for this model:

   * The tree depth, which determines the maximum depth of each tree in the ensemble
   * The number of trees in the ensemble

For specific details on how these models are implemented, see the [generated Python code](DREW link to the generated python code in the main intro page) for each model - these generated pieces of code will illustrate how SciKitLearn packages are used to fit each of these models.

---

## Parameter Tuning

XLKitLearn supports K-fold cross validation ([wikipedia reference](https://en.wikipedia.org/wiki/Cross-validation_(statistics))) for parameter tuning. The model that performs *best* on all the folds on average will be selected and fit on the entire data.

In particular, for any of the parameters described [above](#supported-models), multiple values of the parameters can be compared against each other by specifying several values, separated by `&` signs. For example, if you are fitting a decision tree and would like to compare trees of depth 2, 3, and 4, you should enter `2 & 3 & 4` in the "tree depth" parameter for the decision tree.

In addition to comparing multiple parameters against each other, you can also test multiple formulas against each other. For example, suppose you are fitting a model on the Boston housing dataset provided with XLKitLearn, and that you want to predict the median property value. To compare a model that makes this prediction using `crime_per_capita` to one that makes this prediction using `av_rooms_per_dwelling`, you would type the following formula into XLKitLearn:

`median_property_value ~ crime_per_capita & median_property_value ~ av_rooms_per_dwelling`

!!! note "Comparing Two Formulas"
        Note that when comparing two formulas, the dependent variable (`median_property_value` above) needs to be repeated for each formula. Additionally, all formulas being compared must have the same dependent variable.

XLKitLearn allows you to specify the number of folds to use; simply enter the number in the <a id="kfold-box" class="hover-link" href="javascript:;" title="">appropriate box</a>.

!!! note "Required Cross Validation"
        If your model requires cross-validation (i.e., if you are testing multiple parameters or formulas against each other) but you do not specify the number of folds, the "folds" box will turn red to remind you this parameter is required. Remember that when fitting a boosted decision tree, cross-validation is *always* required.

The folds will be split randomly, using the [randomization seed](#the-randomization-seed) provided in the addin.

Note that if multiple parameters are tuned (eg: tree depths of 1 and 2 _and_ learning rates of 0.1 and 0.2 for a boosted tree), every combination of these parameters will be tested (tree depth of 1 and learning rate of 0.1, tree depth of 2 and learning rate of 0.1, tree depth of 1 and learning rate of 0.2, tree depth of 2 and learning rate of 0.2).

---

## Training and Evaluation Sets

XLKitLearn is able to work with separate training and evaluation datasets. Evaluation datasets can be specified in the 'Evaluation' section of the addin (DREW hover image please!) in a number of ways:

    1. **No evaluation set**: the entire training set will be used to train the model, and no evaluation set will be used.
    2. **Automatically generate an evaluation set with __ % of the training data**: the addin will shuffle the data, and randomly set aside a portion of the dataset to be used as an evaluation set on which to test the performance of the model. The randomization seed (DREW: link) will determine the way the dataset is shuffled, so using this option multiple times with the same percentage and randomization seed will yield identical evaluation sets.
    3. **Use a specific evaluation set**: a specific dataset can be selected - using either method above (DREW link to the selecting dataset) - to be used as an evaluation dataset on which the model will be tested. Note that this evaluation dataset must satisfy one of two conditions - it must either (a) have the same number of columns as the original training dataset, and in the same order (b) must contains all the training columns used in the model, with the same column names.

---

## Making Predictions on New Data

The "make predictions on new data" (DREW screenshot) section allows inference to be carried out on new data. If a dataset is specified in this section, the highest performing model will be used to make predictions for the rows in that dataset.

As in the case with selecting an evaluation dataset manually, this dataset must satisfy one of two conditions - it must either (a) have the same number of columns as the original training dataset, and in the same order (b) must contains all the training columns used in the model, with the same column names.

---

## The Randomization Seed

The randomization seed determines the choices XLKitLearn makes whenever randomization is required. This includes

1. The random splitting of training data into training and test datasets
2. The random splitting of training data into K folds for cross validation
3. The randomness involved in fitting models (eg: the random features selected in each tree of a random forest)

If the addin is run twice with the same randomization seed, the results will be *identical*. This can be useful in a classroom setting, to ensure the entire class gets the same answer.

---

## Understanding the Predictive Addin Output

The model output is dynamic based on the selected model and dataset. Depending on model type and parameters, not all of the following sections will be returned.

### Parameter tuning

If the addin was asked to compare multiple parameters to each other using K-fold cross validation, this section will summarize the result of these comparison. Each row corresponds to one combination of parameters that were tested, and the table will contain the following columns:

  - **Row**: the row number
  - **In-sample score**: the in-sample score for the model with the parameters specified in that row. If K-folds are used, K different models will be fit (each on a training dataset comprising all the data minus one fold), the in-sample score for each of these K models will be calculated, and these scores will be averaged and listed in this column.
  - **Out-of-sample score**: the out-of-sample score for the model with the parameters specified in that row. For each of the K models, the performance of these models will be calculated on the left-out fold, and these scores will be averaged and listen in this column.
  - **Out-of-sample SE**: the standard error of the average out-of-sample score. This will be calculated by finding the standard deviation of the K out-of-sample scores, and dividing the results by the square root of K.
  - The remaining columns of the table will contain the formula, and the relevant parameters being tested in that row.
    If a regression model is fit, the scores output will be R-squared scores. If a classification model is fit, the scores output will be AUC scores ([wikipedia reference](https://en.wikipedia.org/wiki/Receiver_operating_characteristic#Area_under_the_curve)).

Note that in two specific instances, there will be two additional columns included in this table:

  - **For boosted decision trees**: an additional column called "Best number of trees". As discussed above, when a boosted decision tree ensemble is fit with a given maximum number of trees, the addin monitors the performance of the model as new trees are added, and stops when the performance starts to decline. This column specifies the number of trees at which the addin stopped, which is then used for the final model.
  - **For Lasso-penalized linear models**: an additional column called "Nonzero coefficients" will list the number of nonzero coefficients in the model when using this particular penalty.

Below this table, the addin will create a bar graph summarizing the results of parameter tuning - each bar will correspond to one row in the table, and display the out-of-sample score with error bars.

In addition, certain models will output additional graphs

  - **Penalized linear models (or best-subset regressions)**: another graph will be created charting the optimal number of nonzero coefficients against the out-of-sample performance.
  - **Tree-based ensemble models**: a graph will be produced charting the out-of-sample performance as additional trees are added to the ensemble. One line will be produced for every row in the validation table.

### Model

This section describes the final model that was fit on the entire training dataset. If multiple models were compared using K-fold cross validation, the model with the highest out-of-sample score will be re-fit on the entire training dataset and displayed here.

The specific details output about each model will depend on the model in question:

  - **Linear models**: a table will list the coefficients of the model. If a simple linear model is fit with no cross-validation or penalty, p-values and confidence intervals will also be provided.
  - **Decision tree**: for decision trees with fewer than four layers, a graphical representation of the tree will be output. For all trees, a textual representation of the tree will be output.
  - **K-nearest neighbors and ensemble models**: for these models, the permutation variable importance ([scikit-learn reference](https://scikit-learn.org/stable/modules/permutation_importance.html)) will be listed for each variable, in decreasing order of importance. In particular, each column will be randomly permuted a number of times, and the resulting drop in model performance will be calculated in each case. The greater the drop, the more important the variable. These values will also be graphed - each red dot represents the drop in performance for a given permutation, and the black star is the average of the red dots.

Note that for models with thousands of variables, this section might be rather large. If your application does not require these details, you can uncheck the "output model" (DREW screenshot) box in the addin settings to omit it.

### Model evaluation

If an evaluation set is selected using any of the methods described above, the best model selected above will be evaluated using this data. This section will contain the following:

  - **The out-of-sample score** of the model, as evaluated on this dataset. If a regression model is fit, this will be an R-squared score. If a classification model is fit, this will be an AUC score ([wikipedia reference](https://en.wikipedia.org/wiki/Receiver_operating_characteristic#Area_under_the_curve)).
  - **An ROC curve** will be output, if a classification model is fit.
  - **A table** listing every row in the evaluation set, together with the row number in the original dataset, the _true_ value of the dependent variable from that original dataset, and the _predicted_ value using our model. If the evaluation set is large, this table will be output to a file in the same directory as the addin, and the name of the file will be listed here. If you application does not require this table, you can uncheck the "output evaluation dataset" (DREW screenshot) box in the addin settings to omit it.

### Model prediction

If the addin was asked to perform inference on a new dataset (DREW link to the above), this section will contain a table listing those predictions. All the caveats listed regarding the table in the previous section apply here too.

### Equivalent Python Code

This section is XLKitLearn's crown jewel. It contains automatically generated Python code that will carry out analyses equivalent to those run by XLKitLearn.

The code in this section is *not* simply the code that the addin uses. It is dynamically generated to ensure maximal pedagogical impact. The simplest scikit learn function will always be used in this code, and plentiful comments will be included.

Note that:

  - If the data was selected from a specific sheet in the Excel workbook, the generated Python code will attempt to pull the data from the same sheet. The Excel workbook has to be open for this to work.
  - If the data was loaded from a file, the generated Python code will attempt to pull the data from the same file. The code must therefore be in the same directory as the file.

### Technical details

The input settings that were used to run the analysis. These can be copied and pasted directly into the "Settings" cell in the addin tab to run the exact same analysis again, and can be useful to students who might want to replicate the analysis in a solutions spreadsheet.

!!! Note "Copying a Settings String"
    The settings string contains the name of the files and/or sheets containing the data used for the run. Thus, if copying the settings string to another device, the addin will only run successfully if those files and/or sheets exist on the new device.
    

---

## Advanced Settings

A number of advanced settings (DREW screenshot) are provided by XLKitLearn at the bottom of the addin sheet. These settings difer on PC and Mac computers.

### PC Advanced Settings

Two settings are provided on a PC

  - **Keep an active Python connection**: when this box in unchecked, XLKitLearn will launch Python every time the "run" button is pressed, and then terminate that Python instance when the run has completed. This means that every time the addin is run, relevant Python packages need to be loaded, which can be time-consuming. When this box is checked, Python will be launched once, and _not_ terminated at the end of the run. This will make future runs much faster.
  - **Run Python visibly to allow early termination**: when this box is unchecked, XLKitLearn will run Python in the background, invisibly. If this box is checked, the the background process running Python will become visible, as a black window. The benefit of this window is that if the addin is taking too long to run or freeing your computer, you can simply close the black window to interrupt the process. If you do this, an error will pop up which you can just click out of to return to the addin.

  Note that if the first option is selected to keep an active Python connection, Python will _always_ be run visibly.

!!! Note "Active Python Connections with Lenghty Runs"
    One key downside of keeping an active Python connection is that if the run takes more than approximately a minute, the run will fail with an incomprehensible error. This arises because of a limitation in the way Excel connects to Python. If this happens, simply click out of the error, uncheck the "keep an active Python connection" box and attempt the run again.

### Mac advanced settings

Mac users will see a button that will allow them to "Attempt to terminate python". Clicking this button will send a kill code to the Python running in the background and terminate it if the addin is taking too long to run. Note that after the button is pressed, it can take a few moments to interrupt the process.

<script>
  $(document).ready(function() {
    $("#select-data").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/select-data.png" />' });
    $("#formula-box").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-box.png" />' });
    $("#formula-box2").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-box.png" />' });
    $("#formula-editor-list").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor-list.png" />' });
    $("#formula-editor").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor.png" />' });
    $("#formula-editor-error").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor-error.png" />' });
    $("#kfold-box").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/kfold-box.png" />' });
    $("#model-dropdown").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/model-dropdown.png" />' });
    $("#parameters").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/parameters.png" />' });
    $("#formula-editor-gif").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/formula-editor-gif.gif" />' });
    $("#small-data").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/Select_Small_Data.gif" />' });
    $("#large-data").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/Select_Large_Data.gif" />' });
    $("#formula-editor-large-gif").tooltip({ content: '<img src="https://danguetta.github.io/XLKitLearn_docs/Formula_Editor.gif" />' });
  })
</script>

