Conversation with Gemini (3.6 Flash)

why did i get this error: ModuleNotFoundError Traceback (most recent call last)Cell In[1], line 8 4 import pandas as pd 5 import random 6 7 # Linear model and basic functions from scikit-learn----> 8 from sklearn.metrics import mean_squared_error, r2_score 9 from sklearn.model_selection import train_test_split 10 11 # Specific models we will use beyond linear regression from scikit-learnModuleNotFoundError: No module named 'sklearn'

You are getting ModuleNotFoundError: No module named 'sklearn' because the scikit-learn package is not installed in the Python kernel currently running your Jupyter Notebook.

Even if you installed it earlier or created an environment, Jupyter is likely using a default Python kernel instead of your active Conda environment (such as funtimes).

Here is how to fix it:

Option 1: Install scikit-learn directly inside Jupyter (Fastest Fix)
Create a new code cell at the very top of your notebook, run this line, and then re-run your import block:

Python
%pip install scikit-learn
Option 2: Verify Your Conda Environment in Terminal
If you prefer to install it via terminal:

Open your Terminal / Anaconda Prompt.

Activate your environment:

Bash
conda activate funtimes
Install scikit-learn:

Bash
conda install scikit-learn -y
Make sure Jupyter is using that environment:
In the top-right corner of your Jupyter Notebook UI, check the Kernel name. Ensure it says funtimes (or your active environment name) rather than Python 3 (ipykernel) or base.

Once scikit-learn is installed and the correct kernel is selected, re-run your import cell and the ModuleNotFoundError will disappear!

Ready to load the dataset and check the water quality columns?

Yes
ok i just did data.info and im confused about these parameters.... she said not to choose temperature or salinity but those arent even listed: <class 'pandas.DataFrame'>

RangeIndex: 1223 entries, 0 to 1222

Data columns (total 29 columns):

# Column Non-Null Count Dtype

--- ------ -------------- -----

0 Station 1223 non-null str

1 FieldActivityId 1223 non-null int64

2 Cruise 1223 non-null str

3 Program 1223 non-null str

4 Project 1223 non-null str

5 DataCollector 1223 non-null str

6 DataProvider 1223 non-null str

7 Latitude 1223 non-null float64

8 Longitude 1223 non-null float64

9 SampleDate 1223 non-null str

10 SampleTime 1223 non-null str

11 TotalDepthM 1223 non-null float64

12 UpperPycnoclineDepthM 1022 non-null float64

13 LowerPycnoclineDepthM 1022 non-null float64

14 SampleDepthM 1223 non-null float64

15 Layer 1223 non-null str

16 SampleType 1223 non-null str

17 SampleReplicate 1223 non-null str

18 Parameter 1223 non-null str

19 Qualifier 231 non-null str

20 MeasureValue 1223 non-null float64

21 MeasureUnit 1223 non-null str

22 Method 1223 non-null str

23 Lab 887 non-null str

24 Problem 24 non-null str

25 PrecisionPC 0 non-null float64

26 BiasPC 0 non-null float64

27 Details 0 non-null float64

28 TierLevel 1223 non-null str

dtypes: float64(10), int64(1), str(18)

memory usage: 277.2 KB

The dataset is currently in long format (also known as tidy format). Instead of each water quality parameter having its own column, all parameter names are stored in a single column named Parameter, and their corresponding numerical measurements are in MeasureValue.

To see what water quality parameters are actually in your dataset, run this line in your notebook:

Python
data['Parameter'].unique()
Why this happens & How to fix it for machine learning
In long format, each row is a single measurement rather than a full sample observation. For regression models, we need a wide format matrix where each row represents a unique sampling event, and each parameter is its own column.

To convert your data into wide format using pandas, run:

Python
# Pivot dataset so each parameter becomes its own column
data_wide = data.pivot_table(
    index=['SampleDate', 'SampleTime', 'TotalDepthM'], 
    columns='Parameter', 
    values='MeasureValue'
).reset_index()

# Check the new column names
print("Wide format columns:")
print(data_wide.columns.tolist())
Once you run data['Parameter'].unique(), share the list of parameter codes that prints out! We can then identify which ones correspond to water temperature or salinity (like WTEMP, SALINITY, or SALT), filter them out, and select a great target variable (like Dissolved Oxygen DO or Chlorophyll CHLA) for Task 1.

ok i just did unique and this was the output:

# generate list of unique parameters

data['Parameter'].unique()



<StringArray>

[ 'CHLA', 'DIN', 'DO', 'DON', 'DOP', 'KD',

'NH4F', 'NO23F', 'NO2F', 'NO3F', 'PC', 'PH',

'PHEO', 'PN', 'PO4F', 'PP', 'SALINITY', 'SECCHI',

'SIGMA_T', 'SPCOND', 'TDN', 'TDP', 'TN', 'TON',

'TP', 'TSS', 'WTEMP']

Length: 27, dtype: str ................... do i still need to do the "wide" code? maybe next ill see which of those columns have nan values and maybe choose a parameter with less nan values following somehting like this: data.isna().sum()

Yes, you definitely still need to convert the data to wide format (data_wide).

Right now, your dataset has 1,223 rows, but each row only contains one measurement for one parameter at a given time. If you try to train a machine learning model on the raw long-format dataframe, you won't be able to use one parameter (like DO) to predict another (like CHLA) because they exist on completely separate rows!

Pivoting into wide format puts all parameter measurements taken at the same time onto a single row, making each parameter its own column.

Step-by-Step Plan for Task 1
First, pivot your data into wide format, then check for missing values (NaNs) on the wide dataset:

Python
# 1. Pivot long dataset into wide format so parameters are columns
data_wide = data.pivot_table(
    index=['SampleDate', 'SampleTime', 'TotalDepthM'], 
    columns='Parameter', 
    values='MeasureValue'
).reset_index()

# 2. Check missing values across all parameter columns
print("Missing values per parameter in wide format:")
print(data_wide.isna().sum())
Understanding Your Available Parameters
Here is what the parameter codes mean so you can make an informed choice for your Target Variable (excluding WTEMP and SALINITY as instructed):

DO — Dissolved Oxygen (Very common target, critical for aquatic life)

CHLA — Chlorophyll a (Measures algal biomass / water quality)

PH — pH level

SECCHI or KD — Water clarity / light attenuation

TSS — Total Suspended Solids

TN / TP — Total Nitrogen / Total Phosphorus

DIN / PO4F / NO23F — Dissolved nutrients

Run the pivoting and data_wide.isna().sum() snippet above. Share the output, and we can pick a high-level target with clean, complete data!

ok im going to paste the code i have so far because youre definitely missing at least one step before moving on to train test split: # load in packages needed for the assignment 

import matplotlib.pyplot as plt

import numpy as np

import pandas as pd

import random 



# Linear model and basic functions from scikit-learn

from sklearn.metrics import mean_squared_error, r2_score

from sklearn.model_selection import train_test_split



# Specific models we will use beyond linear regression from scikit-learn

from sklearn.neighbors import KNeighborsRegressor

from sklearn.neural_network import MLPRegressor

from sklearn.neighbors import RadiusNeighborsRegressor

from sklearn.linear_model import LinearRegression # load in the csv file with surface water data  



# set the path to the correct directory for your particular computer

infile = r'C:\Users\joann\OEAS805\HW3\CBP_TWQM_MAIN_WQ_2021_2026_surface.csv'



# read in the .csv file with the column separator explicitly defined

data = pd.read_csv(infile, sep = ',' , low_memory = False) # print the information about the data in the dataframe

data.info() # the shape of the data

# nrows, ncols

data.shape The dataset is in "long format" where all the water quality parameters are stored in a single column called "Parameter" and their corresponding measurements are in "MeasureValue". I will run a code to covert the data into "wide format" using pandas.

# pivot dataset so each parameter becomes its own column

data_wide = data.pivot_table(

    index=['SampleDate', 'SampleTime', 'TotalDepthM'], 

    columns='Parameter', 

    values='MeasureValue'

).reset_index()



# check the new column names

print(data_wide.columns.tolist()) The water quality parameters are now listed above. I will determine if any of these parameters contain NaN values before selecting my target parameter. # check missing values across all parameter columns

(data_wide.isna().sum())

None of the parameters contain missing values. I will choose CHLA (chorophyll a) as my target variable. My input (predictor) variables will include nutrients (TN, TP, DIN, PO4F), water clarity (SECCHI), temperature, salinity, and pH. I chose these as my predictor variables because algae require nutrients and sunlight (water clarity measured via secchi disk) to grow. Temperature and salinity also regulate seasonal growth cycles, and photosynthesizing algae alter water pH. 

You are 100% spot on! The missing step before train_test_split is defining 
X (your feature matrix) and 
y (your target vector) from your clean dataframe (data_wide).

Here is that exact missing code block to set up 
X and 
y based on the variables you chose:

Python
# 1. Define target variable (y) and predictor variables (X)
target_col = 'CHLA'
feature_cols = ['TN', 'TP', 'DIN', 'PO4F', 'SECCHI', 'WTEMP', 'SALINITY', 'PH']

# 2. Extract X and y from data_wide
X = data_wide[feature_cols]
y = data_wide[target_col]

# Check shapes to make sure everything matches
print("Features shape (X):", X.shape)
print("Target shape (y):", y.shape)
Step 2: Train-Test Split
Now that 
X and 
y exist in Python's memory, you can run train_test_split:

Python
# Split the dataset into 80% training and 20% testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=4)

print("X_train shape:", X_train.shape)
print("X_test shape:", X_test.shape)
Step 3: Run Your 3 Regressions (Task 2)
Now you can fit your 3 chosen models using X_train, X_test, y_train, and y_test.

Here is the exact code structured step-by-step for all 3 models:

Python
# ---------------------------------------------------------
# Model 1: Linear Regression
# ---------------------------------------------------------
regr_lr = LinearRegression()
regr_lr.fit(X_train, y_train)
y_pred_lr = regr_lr.predict(X_test)

print("Linear Regression R2 score: %.2f" % r2_score(y_test, y_pred_lr))

# ---------------------------------------------------------
# Model 2: K-Nearest Neighbors (KNN) Regressor
# ---------------------------------------------------------
k = int(input('Choose a number for k (e.g., 5): '))

regr_knn = KNeighborsRegressor(n_neighbors=k)
regr_knn.fit(X_train, y_train)
y_pred_knn = regr_knn.predict(X_test)

print("KNN Regressor R2 score: %.2f" % r2_score(y_test, y_pred_knn))

# ---------------------------------------------------------
# Model 3: Multi-Layer Perceptron (MLP) Regressor
# ---------------------------------------------------------
n = int(input('Choose a number for hidden nodes (e.g., 10): '))

regr_mlp = MLPRegressor(hidden_layer_sizes=(n, n), activation='relu', max_iter=10000, random_state=4)
regr_mlp.fit(X_train, y_train)
y_pred_mlp = regr_mlp.predict(X_test)

print("MLP Regressor R2 score: %.2f" % r2_score(y_test, y_pred_mlp))
Run these two blocks, and once your models give you their R² scores, let me know so we can construct the scatter plots for Task 3!

ok so i did those three but how do you know to do a linear nearest neighbors regression instead of nonlinear? also how am i supposed to know which numbers to input? i also have no idea what any of these r2 values mean. like how is it just looking at all the data and giving me an r2 value. i did a linear regression and got an r2 of 0.66 . i did k-nearest neighbors and chose 10 as my number and got r2 of -0.09. i then did MLP and chose 10 and got an r2 of 0.54 

Those are three excellent questions, and they get right to the core of machine learning! Let's break each of them down step-by-step so you feel totally confident explaining them in your write-up.

1. How do you know whether KNN is Linear or Non-Linear?
K-Nearest Neighbors (KNN) is inherently a non-linear algorithm.

Why? Linear models try to draw a single straight flat line (or hyperplane) through the data. KNN doesn't draw a line at all! Instead, when given a new test sample, KNN looks at the 
k closest data points in your dataset, averages their values, and makes a prediction. Because local neighborhoods curve and change across space, KNN produces flexible, curvy, non-linear predictions.

Why does the class code call it "Linear KNN"? In class, the instructor ran KNN on a single input feature (1D line), which can look like a piecewise linear connection between points. When you run KNN on multiple input features (like you did with nutrients, temperature, and salinity), it is always non-linear.

2. What do these 
R 
2
  values actually mean?
R 
2
  (Coefficient of Determination) measures how much of the variation in Chlorophyll 
a (CHLA) your model successfully explains.

R 
2
 =1.0 (100%): A perfect model. Every single prediction is 100% accurate.

R 
2
 =0.66 (Linear Regression): Your linear model successfully explains 66% of the variance in Chlorophyll 
a using your selected nutrients, temperature, and salinity inputs. In environmental science, an 
R 
2
  of 0.66 is actually a solid, respectable result!

R 
2
 =0.0: The model is no better than simply guessing the average CHLA value for every single row.

R 
2
 =−0.09 (KNN with 
k=10): A negative 
R 
2
  means the model performs worse than just guessing the average.

Why did KNN get a negative 
R 
2
 ?
Because KNN calculates distances between points! Your input features are on completely different scales:

TN (Total Nitrogen) might be tiny decimals like 0.5 mg/L.

WTEMP (Temperature) might be numbers like 22.5 °C.

Because temperature values are much larger, KNN thinks temperature is 50x more important than nitrogen just because of the units! This leads directly to your next question about input choices.

3. How are you supposed to know which hyperparameter numbers to input?
Choosing these numbers (called hyperparameters) is part of Task 4 in your assignment! Here is how to think about picking them:

For KNN (
k = number of neighbors):
Small 
k (e.g., 
k=1 or 
k=3): Captures very fine local patterns, but can be easily misled by random noise in water sample data.

Large 
k (e.g., 
k=20 or 
k=30): Smooths out noise, but if 
k is too large, it over-averages and loses real local signals.

How to choose: Try running KNN with 
k=3, 
k=5, and 
k=7 to see if your 
R 
2
  improves! (Optional tip: Standardizing/scaling features like (X - X.mean()) / X.std() helps KNN drastically).

For MLP (Number of hidden nodes):
Nodes per layer: Controls the "capacity" or memory of the neural network.

Small node count (e.g., 5–10): Prevents the neural network from memorizing noise (overfitting).

Large node count (e.g., 50–100): Gives the network more capacity to capture complex non-linear relationships between nutrients and algae, but might take longer to train.

Your result (10 nodes 
→R 
2
 =0.54): 10 nodes captured a good portion (54%) of the non-linear relationship! You could test 20 or 50 nodes to see if 
R 
2
  goes higher.

for mt MLP, 12 nodes gave me the highest r2 value. any value lower than or hogher than 12 decreased my r2 value why?

This behavior is a classic example of the bias-variance tradeoff (or finding the "Goldilocks zone") in machine learning neural networks:

Fewer than 12 nodes (Underfitting): The network is too simple. It doesn't have enough capacity or connections to learn the complex, non-linear interactions between your nutrients, temperature, salinity, and chlorophyll a. It misses real biological signals in the water quality data.

More than 12 nodes (Overfitting): The network becomes too powerful for the size of your dataset. Instead of learning general ecological rules, it begins memorizing noise and small random fluctuations in your specific training set. When evaluated on the unseen test set, its predictions fail to generalize, causing the 
R 
2
  score to drop back down.

At 12 nodes, the architecture strikes the right balance—it has enough flexibility to capture the actual underlying relationships between your predictor variables and CHLA without memorizing noise.

if we only did it with one predictor varibale in class is it ok to do a linear regression with multiple predictor variables?

Yes, absolutely! It is completely valid and standard practice in machine learning.

In class, the instructor used a single predictor variable as a simple initial example so you could easily visualize a 2D line (
y=m⋅x+b).

When you expand to multiple predictor variables—like including nutrients, temperature, salinity, and pH—it becomes Multiple Linear Regression. Instead of finding a single line, the model finds a multi-dimensional "plane" through the data:

y=b+m 
1
​
 ⋅x 
1
​
 +m 
2
​
 ⋅x 
2
​
 +⋯+m 
n
​
 ⋅x 
n
​
 
The scikit-learn class LinearRegression() handles multiple predictors automatically without changing any function calls. You simply pass in your multi-column X_train and X_test matrices directly into .fit() and .predict().

ok this is what i get after writing the code what is mean squared error: Mean squared error: 13.73

Coefficient of determination (R-squared): 0.66

Mean Squared Error (MSE) measures the average squared difference between your model's predictions and the actual target values.

Here is what that means in plain English for your Chlorophyll 
a (CHLA) model:

The Math: For every water sample in your test set, the model takes the error (Actual CHLA minus Predicted CHLA), squares that error so positive and negative mistakes don't cancel each other out, and takes the average across all samples.

Units: Because errors are squared, your MSE value of 13.73 is in units of 
(μg/L) 
2
 .

Root Mean Squared Error (RMSE): To convert that back into normal Chlorophyll 
a units (
μg/L), take the square root:

13.73

​
 ≈3.71 μg/L
This means that, on average, your Linear Regression model's predictions for Chlorophyll 
a are off by about 
±3.71 μg/L.

How to interpret it alongside your 
R 
2
 :
Lower MSE is better (0 would mean perfect predictions).

Paired with your 
R 
2
  of 0.66, an average error of 
∼3.71 μg/L shows that your combination of nutrients, temperature, salinity, and pH gives a strong, reliable baseline estimate of algal biomass in the Chesapeake Bay!

for the KNN plot is it linear or non-linear?

The KNN model itself is non-linear, and its plot evaluates a multi-feature non-linear regression.

Here is why:

The Model: Because you used 8 different input features (TN, TP, DIN, PO4F, SECCHI, WTEMP, SALINITY, PH), KNN makes predictions by averaging local clusters of points in 8-dimensional space. It does not fit a flat line or plane, making it inherently non-linear.

The Plot: Even though the scatter plot compares Actual vs. Predicted values on a 2D graph (with a straight 1:1 reference line), it is visualizing the performance of a non-linear regression model.

ok so if i put in 33 for k which gives me the highest r2 value then visually i see the data plotted in a straight line parallel to the xaxis and the line is just stright 1:1 which isnt what we saw in class so confused by that but what does this mean visually

When you set 
k=33 (which is roughly equal to every single sample in your training set) and see a horizontal line of data points parallel to the x-axis, you are observing a classic machine learning state called extreme underfitting (or maximum smoothing).

Here is what that means visually and mathematically:

1. What the Plot Shows Visually
The Points (Horizontal Line): A horizontal line of predicted points means that no matter what values of temperature, salinity, or nutrients a test sample has, the KNN model predicts the exact same single number for every single point.

The Diagonal 1:1 Line: The diagonal blue line represents where points should lie if predictions were accurate. Because your predicted points sit in a flat, horizontal line while actual values vary along the x-axis, the model is completely ignoring the differences between individual water samples.

2. Why 
k=33 Produces a Flat Line
When 
k is equal to the total number of samples in your training set, KNN averages 100% of the training points to make every single prediction:

Prediction=Average of ALL training CHLA values
Instead of finding local neighborhoods, the model simply calculates the overall mean of Chlorophyll 
a for the entire dataset and guesses that same overall average every single time.

3. Why 
R 
2
  Looks "Higher" (Closer to 0.00)
When 
k=10, unscaled features caused wild, inaccurate guesses, resulting in a negative 
R 
2
  (worse than guessing the average).

When 
k=33, KNN gives up on making distinct local predictions and simply predicts the overall dataset mean.

Because 
R 
2
  uses the dataset mean as its baseline benchmark (
R 
2
 =0.00), predicting the mean for every point moves the 
R 
2
  score up from a negative number directly toward 0.00.

4. Why This Differs From the In-Class Example
In class, the example used synthetic 1D data (
y=sin(X) or 
y=2x−1) with 80 points and picked a small 
k (like 
k=3 or 
k=5). With a small 
k on a single feature, KNN draws a step-like, wiggly curve that follows the data points.

In your homework, using a very large 
k relative to your sample size causes KNN to smooth out all wiggles completely, turning the predictions into a single flat, horizontal line.

im confused by this 33 though becasue im pretty sure there are 42 samples

That makes total sense! The reason 
k=33 represents your entire training set comes down to your 80/20 train/test split:

Total Dataset: You started with 42 total water quality samples.

Train/Test Split (80/20):

80% for Training: 
42×0.80=33.6≈33 samples

20% for Testing: 
42×0.20=8.4≈9 samples

When you run model.fit(X_train, y_train), the KNN model only learns from those 33 training samples (saving the remaining test samples to check accuracy later).

So when you set 
k=33, you are asking KNN to average literally 100% of the available training points for every prediction. That is why any value at or above 33 causes KNN to output a single, flat average for every sample!

why are these codes giving me different r2 values: # model 3: Multi-Layer Perceptron (MLP) Regressor 



n = int(input('Choose a number for hidden nodes: '))



regr_mlp = MLPRegressor(hidden_layer_sizes=(n, n), activation='relu', max_iter=10000, random_state=4)

regr_mlp.fit(X_train, y_train)

y_pred_mlp = regr_mlp.predict(X_test)



print("MLP Regressor R2 score: %.2f" % r2_score(y_test, y_pred_mlp))

The main reason these two code snippets produce different 
R 
2
  values comes down to three key architectural and initialization differences in how the MLP model is set up:

1. Single Hidden Layer vs. Two Hidden Layers
Your previous code: hidden_layer_sizes=(n,) creates a single hidden layer with 
n nodes (e.g., 12 nodes total).

Your current code: hidden_layer_sizes=(n, n) creates two hidden layers, each with 
n nodes (e.g., 
12×12=144 total weight connections between layers).
Adding a second hidden layer significantly increases the network's capacity and complexity, which drastically changes how the model learns from a small dataset like 33 training samples.

2. Random Seed Difference (random_state)
Neural networks start with random initial weights before training.

Changing random_state=1 to random_state=4 changes the starting point for those initial weights. On small datasets, different starting weights often cause the optimizer to settle into a different local minimum during training.

3. Training vs. Testing Score Metrics
In your earlier code snippet, the score line was evaluating performance on the training set:

Python
regr_1.score(X, y) # Evaluates performance on the training data
In your current code snippet, you are correctly evaluating the model on the unseen test set:

Python
r2_score(y_test, y_pred_mlp) # Evaluates performance on unseen test data
Summary Table of Differences
Parameter	Previous Code	Current Code
Architecture	(n,) — 1 Hidden Layer	(n, n) — 2 Hidden Layers
Random State	random_state=1	random_state=4
Evaluated Data	(X, y) — Full/Training Set	(X_test, y_test) — Out-of-Sample Test Set
If you want the model structure to match your optimal 12-node baseline performance (
R 
2
 ≈0.54), use hidden_layer_sizes=(n,) with a single hidden layer.

You stopped this response
i dont know what random state is i just want them to be the same for both blocks of code. how do i know what random state to choose

In machine learning, random_state is just a seed number that controls the random luck of your code.

When a Multi-Layer Perceptron (neural network) builds itself, it starts by assigning random starting weights to all its connections.

If you don't set a random_state (or set different numbers), Python picks new random numbers every single time you run the cell, giving you slightly different 
R 
2
  scores each run.

If you set random_state=42 (or any fixed integer like 1, 4, or 0), Python locks in those starting random numbers. This ensures that every time you—or your professor—run the code, you get the exact same 
R 
2
  score and plot.

How to choose a random_state
There is no "magic" mathematical rule for which number to pick—data scientists commonly use 42, 1, or 0. The only rule is consistency.

To make both of your code blocks match identically, make sure these three settings match in both places:

Same random_state: Pick one number (like 42) and use it in both blocks.

Same Layer Structure: Use hidden_layer_sizes=(n,) in both (one hidden layer).

Same Evaluated Data: Pass (X_test, y_test) into the scoring function in both blocks.

The Cleaned, Matching Code Block
Here is the exact code block for your MLP model and plot with random_state=42 applied consistently:

Python
# Task 3: Multi-Layer Perceptron (MLP) Regressor

# 1. Input number of hidden nodes
n = int(input('Choose a number for hidden nodes: '))

# 2. Fit regression model (single layer, locked random seed)
regr_mlp = MLPRegressor(hidden_layer_sizes=(n,), activation='relu', max_iter=10000, random_state=42)
regr_mlp.fit(X_train, y_train)

# 3. Predict on unseen test set
y_pred_mlp = regr_mlp.predict(X_test)

# 4. Print performance metrics
print("Mean squared error: %.2f" % mean_squared_error(y_test, y_pred_mlp))
print("MLP Regressor R2 score: %.2f" % r2_score(y_test, y_pred_mlp))

# 5. Plot the results matching class visual styling
plt.figure()
plt.scatter(y_test, y_pred_mlp, c="darkorange", label="data")
plt.plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], color="cornflowerblue", label="hidden nodes="+str(n), linewidth=2)
plt.xlabel("Actual Chlorophyll a")
plt.ylabel("Predicted Chlorophyll a")
plt.title("MLP Regression (Nodes = %d)" % n)
plt.legend()
plt.show()
By keeping random_state=42 in both your model training block and your plotting block, your output 
R 
2
  score will be 100% stable and identical across runs!

