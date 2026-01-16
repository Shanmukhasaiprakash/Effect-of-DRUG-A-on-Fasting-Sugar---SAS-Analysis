/*-------------------------------------------------------------
Project: Effect of DRUG A on Fasting Sugar Levels
Author: Shanmukha Sai Prakash Jeelakarra
Description: SAS analysis of 3400 patients in a randomized trial 
to evaluate whether DRUG A reduces fasting sugar. The workflow 
includes merging datasets, sampling, descriptive statistics, 
ANOVA/GLM analysis, and visualization.
-------------------------------------------------------------*/

/* Step 1: Import the Initial Study dataset */
Filename F1 "C:\Users\shanm\OneDrive\Desktop\Initial_Study_Project3.txt";

Data Initial;
    Infile F1 dlm='09'x dsd missover; /* Tab-delimited data, missing values handled */
    Input PatientID Age State$ Length_of_Stay Total_Charge; /* Define variables */
run;

/* Step 2: Sort the Initial dataset by PatientID for merging */
Proc Sort Data=Initial;
    by PatientID;
run;

/* Step 3: Import the Second Study dataset */
Filename F2 "C:\Users\shanm\OneDrive\Desktop\Second_Study_Project3.txt";

Data Second;
    Infile F2 dlm='09'x dsd missover;
    Input PatientID Site $ Group $ Test_Score; /* Group: Low/High/Placebo, Test_Score: fasting sugar */
run;

/* Step 4: Sort the Second dataset by PatientID for merging */
Proc Sort Data=Second;
    by PatientID;
run;

/* Step 5: Merge Initial and Second datasets by PatientID */
Data Merged;
    Merge Initial Second;
    by PatientID;
run;

/* Step 6: Print merged dataset for verification */
Proc Print Data=Merged;
    Title "Merged Dataset";
run;

/* Step 7: Simple Random Sampling (SRS) of 1000 patients */
Proc SurveySelect Data=Merged Method=SRS Sampsize=1000 Out=Sampled_Data;
run;

/* Step 8: Print sampled dataset */
Proc Print Data=Sampled_Data;
