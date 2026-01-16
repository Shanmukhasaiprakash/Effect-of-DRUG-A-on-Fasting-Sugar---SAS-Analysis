  Project: Effect of DRUG A on Fasting Sugar – SAS Analysis
  Author: Shanmukha Sai Prakash Jeelakarra
  Description:
  This program merges two study datasets, performs random
  sampling, descriptive statistics, ANOVA/GLM analysis,
  and visualization to evaluate the effect of DRUG A.

/*------------------------------------------------------------
  Step 1: Read Initial Study Dataset
  Contains patient demographics and hospital information
------------------------------------------------------------*/
Filename F1 "C:\Users\shanm\OneDrive\Desktop\Initial_Study_Project3.txt";

Data Initial;
    Infile F1 dlm='09'x dsd missover;
    Input PatientID Age State$ Length_of_Stay Total_Charge;
run;

/* Sort Initial dataset by PatientID for merging */
Proc Sort Data=Initial;
    by PatientID;
run;

/*------------------------------------------------------------
  Step 2: Read Second Study Dataset
  Contains treatment group, site, and fasting sugar levels
------------------------------------------------------------*/
Filename F2 "C:\Users\shanm\OneDrive\Desktop\Second_Study_Project3.txt";

Data Second;
    Infile F2 dlm='09'x dsd missover;
    Input PatientID Site$ Group$ Test_Score;
run;

/* Sort Second dataset by PatientID for merging */
Proc Sort Data=Second;
    by PatientID;
run;

/*------------------------------------------------------------
  Step 3: Merge Both Datasets by PatientID
------------------------------------------------------------*/
Data Merged;
    Merge Initial Second;
    by PatientID;
run;

/* Display merged dataset */
Proc Print Data=Merged;
    Title "Merged Dataset";
run;

/*------------------------------------------------------------
  Step 4: Simple Random Sampling (SRS)
  Sample size = 1000 (no seed as per instructions)
------------------------------------------------------------*/
Proc Surveyselect Data=Merged
    Method=SRS
    Sampsize=1000
    Out=Sampled_Data;
run;

/* Display sampled dataset */
Proc Print Data=Sampled_Data;
    Title "Sampled Dataset";
run;

/*------------------------------------------------------------
  Step 5: Descriptive Statistics
  Mean, SD, Min, Max of fasting sugar by treatment group
------------------------------------------------------------*/
Proc Means Data=Sampled_Data n mean std min max;
    Class Group;
    Var Test_Score;
run;

/*------------------------------------------------------------
  Step 6: Frequency Table
  Distribution of treatment groups across sites
------------------------------------------------------------*/
Proc Freq Data=Sampled_Data;
    Tables Site*Group / norow nocol nopercent;
run;

/*------------------------------------------------------------
  Step 7: Normality Check for Test_Score
------------------------------------------------------------*/
Proc Univariate Data=Sampled_Data normal;
    Var Test_Score;
run;

/*------------------------------------------------------------
  Step 8: Data Cleaning
  Remove missing test scores and invalid group values
------------------------------------------------------------*/
Data Analysis_Data;
    set Sampled_Data;
    if Group ne "n/a" and Test_Score ne .;
run;

/*------------------------------------------------------------
  Step 9: One-Way ANOVA
  Compare mean fasting sugar across treatment groups
------------------------------------------------------------*/
Proc Anova Data=Analysis_Data;
    Class Group;
    Model Test_Score = Group;
    Means Group / Tukey;
run;

/*------------------------------------------------------------
  Step 10: GLM – One-Way Model with Variance Test
  Includes Levene’s test for homogeneity of variances
------------------------------------------------------------*/
Proc Glm Data=Analysis_Data;
    Class Group;
    Model Test_Score = Group;
    Means Group / hovtest=levene;
run;

/*------------------------------------------------------------
  Step 11: Two-Way ANOVA (GLM)
  Evaluate Group, Site, and Group*Site interaction effects
------------------------------------------------------------*/
Proc Glm Data=Analysis_Data;
    Class Group Site;
    Model Test_Score = Group Site Group*Site;
    Means Group;
run;

/*------------------------------------------------------------
  Step 12: Visualization
  Histogram and density plot of fasting sugar by group
------------------------------------------------------------*/
Proc Sgplot Data=Analysis_Data;
    Histogram Test_Score / group=Group transparency=0.5;
    Density Test_Score;
    Title "Histogram of Test Scores by Treatment Group";
run;
