# Student-Performance-Academic-Analytics
this is a data analytic project in use of python laguage 
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import streamlit as st
import plotly.express as px

st.set_page_config(
    page_title="Student Performance Dashboard",
    page_icon="🎓",
    layout="wide"
)

# Load dataset
data = pd.read_csv("updated_student_performance.csv")


# =========================
# 1. INITIAL INSPECTION
# =========================

print("First 5 rows:")
print(data.head())

print("\nLast 5 rows:")
print(data.tail())

print("\nDataset information:")
data.info()

print("\nDescriptive statistics:")
print(data.describe())


# =========================
# 2. CHECK MISSING VALUES
# =========================

print("\nMissing values:")
print(data.isnull().sum())

print("\nColumns containing missing values:")
print(data.isna().any())


# =========================
# 3. HANDLE MISSING VALUES
# =========================

# Fill numerical missing values with the column mean
data['Attendance'] = data['Attendance'].fillna(data['Attendance'].mean())

data['Sleep_Hours'] = data['Sleep_Hours'].fillna(data['Sleep_Hours'].mean())

data['Programming'] = data['Programming'].fillna(data['Programming'].mean())


# Check again
print("\nMissing values after filling:")
print(data.isnull().sum())


# =========================
# 4. CHECK DUPLICATES
# =========================

print("\nNumber of duplicate rows:")
print(data.duplicated().sum())

print("\nAre there duplicate rows?")
print(data.duplicated().any())


# =========================
# 5. REMOVE DUPLICATES
# =========================

data = data.drop_duplicates()


# Check again
print("\nDuplicate rows after removal:")
print(data.duplicated().sum())

print("\nAre there duplicates after removal?")
print(data.duplicated().any())


# =========================
# 6. FINAL CHECK
# =========================

print("\nFinal dataset shape:")
print(data.shape)

print("\nFinal missing values:")
print(data.isnull().sum())

# Average of all subject 
data["Average"]=data[['Math','Physics',"Programming","English"]].mean(axis=1)
print("average of each student: ", data['Average'])


# pass or fail
# =========================
# PASS / FAIL
# =========================

data["Pass_or_Fail"] = np.where(
    data["Average"] >= 50,
    "Pass",
    "Fail"
)

print("\nPass / Fail:")
print(
    data[
        ["Student_ID", "Average", "Pass_or_Fail"]
    ].head()
)
p_pass = (data["Pass_or_Fail"] == "Pass").mean()
p_fail = (data["Pass_or_Fail"] == "Fail").mean()

# # save the data
# data.to_csv("updated_student_performance.csv",index=False)

# descriptive statitics
print("mean of the data: ",data['Average'].mean())
print("median of the data", data["Average"].median())
print("variance of the data", data["Average"].var())
print("standar diviation of the data",data["Average"].std())

Q1=np.percentile(data['Average'],25)
Q2=np.percentile(data['Average'],50)
Q3=np.percentile(data['Average'],75)

IQR=Q3-Q1
lower=Q1-1.5*IQR
upper=Q3+1.5*IQR

print("Q1=", Q1)
print("Q2=", Q2)
print("Q3=", Q3)
print("iqr=", IQR)
print("lower outlier bound", lower)
print("upper outlier bound", upper)

# aggregation
department_statistics=data.groupby("Department").agg({
    "Math": ["mean", "max", "min"],
    "Physics": ["mean", "max", "min"],
    "Programming": ["mean", "max", "min"],
    "Average": ["mean", "max", "min"]
})

print("department statistics: ", department_statistics)


# univarient analysis
plt.stem(data['Student_ID'],data['Average'],  label='student score')
plt.xlabel("student_id")
plt.ylabel("Average")
plt.title("student_id vs Average")
plt.legend()
plt.grid()
plt.show()

plt.hist(department_statistics)
plt.xlabel(data['Department'])
plt.title("department statistics analyse ")
plt.show()

print(data["Department"].unique())

# univarient analysis
sns.histplot(data['Average'], kde=True,alpha=0.8,color='green')
plt.title("average distribution")
plt.show()

sns.countplot(data=data, x='Department')
plt.title("department distribution")
plt.show()


sns.boxplot(data=data, x=data["Average"], hue='Department')
plt.title("outliers")
plt.show()

sns.ecdfplot(data=data,x='Average', hue='Department')
plt.title("cummulative distribution")
plt.show()


corr=data[['Math','Physics','Programming', 'English']].corr()
plt.imshow(corr)
plt.xticks(
 range(len(corr.columns)),
corr.columns)
plt.show()
print(corr)

plt.figure(figsize=(10, 7))

sns.heatmap(
    corr,
    annot=True,
    cmap='coolwarm'
)

plt.title("Correlation Matrix")
plt.show()

# bivariate analysis
sns.scatterplot(data=data, x=data['Study_Hours'], y=data['Average'],
                 hue='Department', style='Department')
plt.title("study hours vs score")
plt.show()

sns.scatterplot(data=data, x=data['Attendance'],y=data['Average'] ,hue='Department')
plt.title("student attendance vs their score")
plt.show()

# students per department
stedents_per_department= data["Department"].value_counts()
print("students per department", stedents_per_department)



# students their average between 70 to 90
students_averge_90_100= data[data["Average"].between(90, 100)]
print("students their average between 90 to 100:  ", students_averge_90_100)


def grade(score):
    if score>=90:
        return 'A'
    elif score>=80:
        return'B'
    elif score>=70:
        return 'C'
    elif score>=60:
        return 'D'
    else:
        return 'F'
    
data["Grade"] = data["Average"].apply(grade)

print(data[["Student_ID", "Average", "Grade"]].head(5))
# ranking
data["Rank"] = (
    data["Average"]
    .rank(method="min", ascending=False)
    .astype(int)
)
print(
    data[
        ["Student_ID", "Average", "Grade", "Rank"]
    ].sort_values("Rank").head(10)
)

# highest and lowest
print("hishest student:" ,data.loc[data["Average"].idxmax()])

print("lowest students", data.loc[data["Average"].idxmin()])

score=data['Average']
def grade(score):
    if score>=90:
        return 'A'
    elif score>=80:
        return'B'
    elif score>=70:
        return 'C'
    elif score>=60:
        return 'D'
    else:
        return 'F'
    
data["Grade"] = data["Average"].apply(grade)

print(data[["Student_ID", "Average", "Grade"]].head(5))

sns.countplot(data=data, x="Grade")
plt.title("Grade Distribution")
plt.show()


# propability
pss=(data['Pass_or_Fail']=='pass').mean()
print(pss)


fail=(data['Pass_or_Fail']=='fail').mean()
print(fail)

    
# probpability of passing given whith the highest attendance
high_attendance = data[data['Attendance'] >= 80]

p_pass_given_high_attendance = (
    high_attendance['Pass_or_Fail'] == 'Pass'
).mean()

print(
    "P(Pass | Attendance >= 80):",
    p_pass_given_high_attendance
)


