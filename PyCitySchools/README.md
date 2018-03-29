

```python
#import dependencies 
import pandas as pd
import numpy as np
import os
#import school CSV files
schools_csv= "../raw_data/schools_complete.csv"
#read school CSV files
schools_df= pd.read_csv(schools_csv)
#rename "name column to "school" to match other data set 
schools_df= schools_df.rename(columns={"name": "school"})
#import student CSV files
student_csv= "../raw_data/students_complete.csv"
#read student CSV files
student_df= pd.read_csv(student_csv)

#District Summary
#Create a high level snapshot (in table form) of the district's key metrics:

#Total Schools
TotalSchools= len(schools_df)
#Total Students
TotalStudents= len(student_df)
#Total Budget
TotalBudget = schools_df["budget"].sum()
#Average Math Score
AvgMathscore= student_df["math_score"].mean()
#Average Reading Score
AvgReadingscore= student_df["reading_score"].mean()
# % Passing Math-passing is grade >=70 
passing_math= student_df[student_df["math_score"] >= 70]
count_passing_math= len(passing_math)
passingmath_percentage= (count_passing_math/TotalStudents) * 100
#% Passing Reading-passing is grade >=70 
passing_reading= student_df[student_df["reading_score"] >= 70]
count_passing_reading= len(passing_reading)
passingreading_percentage= (count_passing_reading/TotalStudents) *100
#Overall Passing Rate (passing both math and reading)
passing_both= student_df[(student_df.math_score >= 70) & (student_df.reading_score >= 70)]
count_passing_both= len(passing_both)
passing_both_percentage=(count_passing_both/TotalStudents)*100

#District Summary
District_Summary_df= pd.DataFrame({"Total Schools": [TotalSchools], 
                                   "Total Students": [TotalStudents],
                                   "Total Budget": [TotalBudget],
                                   "Average Math Score": [AvgMathscore],
                                   "Average Reading Score": [AvgReadingscore],
                                   "% Passing Math": [passingmath_percentage],
                                   "% Passing Reading": [passingreading_percentage],
                                   "% Overall Passing Rate": [passing_both_percentage]})
District_Summary_df= District_Summary_df[["Total Schools", 
                                   "Total Students",
                                   "Total Budget",
                                   "Average Math Score",
                                   "Average Reading Score",
                                   "% Passing Math",
                                   "% Passing Reading",
                                   "% Overall Passing Rate"]]

# Format table
District_Summary_df["Total Students"] = District_Summary_df["Total Students"].map("{0:,.0f}".format)
District_Summary_df["Total Budget"] = District_Summary_df["Total Budget"].map("{0:,.0f}".format)
District_Summary_df["Average Math Score"] = District_Summary_df["Average Math Score"].map("{0:,.2f}".format)
District_Summary_df["Average Reading Score"] = District_Summary_df["Average Reading Score"].map("{0:,.2f}".format)
District_Summary_df["% Passing Math"] = District_Summary_df["% Passing Math"].map("{0:,.2f}%".format)
District_Summary_df["% Passing Reading"] = District_Summary_df["% Passing Reading"].map("{0:,.2f}%".format)
District_Summary_df["% Overall Passing Rate"] = District_Summary_df["% Overall Passing Rate"].map("{0:,.2f}%".format)

District_Summary_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>65.17%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#import school CSV files
schools_csv= "../raw_data/schools_complete.csv"
#read school CSV files
schools_df= pd.read_csv(schools_csv, index_col="name")
#rename "name column to "school" to match other data set 
schools_df= schools_df.rename(columns={"name": "school"})
#calculating Per Student Budget
schools_df["Per Student Budget"]= schools_df["budget"]/schools_df["size"]

#import student CSV files
student_csv= "../raw_data/students_complete.csv"
#read student CSV files
student_df= pd.read_csv(student_csv)

#grouping the student csv by school for calculations
SG_df= student_df.groupby("school")
#Total Students for calculations
total_students = SG_df["Student ID"].count()
#Average Math Score
avg_math_score = SG_df["math_score"].mean()
#Average Reading Score
avg_read_score = SG_df["reading_score"].mean()
# % Passing Math-passing is grade >=70 
school1_df= student_df.loc[student_df["math_score"] >= 70,:]
school1_df_group = school1_df.groupby("school")
perc_math = (school1_df_group["math_score"].count()/total_students)*100
#% Passing Reading-passing is grade >=70 
school2_df= student_df.loc[student_df["reading_score"] >= 70,:]
school2_df_group = school1_df.groupby("school")
perc_read = (school2_df_group["reading_score"].count()/total_students)*100
#Overall Passing Rate (passing both math and reading)
passing_both= (perc_read + perc_math)/2


#creating a datafram of calculations to merge with school data
calc_df= pd.DataFrame({"Average Math Score": avg_math_score,"Average Reading Score":avg_read_score,"% Passing Math": perc_math,
                        "% Passing Reading": perc_read,"% Overall Passing Rate":passing_both})

#merging both dataframes together
mergeall=pd.merge(schools_df, calc_df, left_index=True, right_index=True)
#formating new dataframe
school_summary=mergeall.rename(columns={"type":"School Type","size":"Total Students","budget": "Total Budget",
                                        "Per Student Budget":"Per Student Budget", "Average Math Score": "Average Math Score",     
                                   "Average Reading Score":"Average Reading Score","% Passing Math":"% Passing Math",
                                        "% Passing Reading":"% Passing Reading",
                                        "% Overall Passing Rate":"% Overall Passing Rate"})
school_summary=school_summary[["School Type","Total Students","Total Budget", "Per Student Budget", "Average Math Score",     
                                   "Average Reading Score","% Passing Math","% Passing Reading","% Overall Passing Rate"]]



# Format table
#school_summary["Total Students"] = school_summary["Total Students"].map("{0:,.0f}".format)
#school_summary["Total Budget"] = school_summary["Total Budget"].map("{0:,.0f}".format)
#school_summary["Average Math Score"] = school_summary["Average Math Score"].map("{0:,.2f}".format)
#school_summary["Average Reading Score"] = school_summary["Average Reading Score"].map("{0:,.2f}".format)
#school_summary["% Passing Math"] = school_summary["% Passing Math"].map("{0:,.2f}".format)
#school_summary["% Passing Reading"] = school_summary["% Passing Reading"].map("{0:,.2f}".format)
#school_summary["% Overall Passing Rate"] = school_summary["% Overall Passing Rate"].map("{0:,.2f}".format)


school_summary
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>65.683922</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>65.988471</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>1056600</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>93.867121</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>3022020</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.752967</td>
      <td>66.752967</td>
      <td>66.752967</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>93.392371</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>93.867718</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>94.133477</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>3124928</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>66.680064</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>248087</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.505855</td>
      <td>92.505855</td>
      <td>92.505855</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>94.594595</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>1049400</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.333333</td>
      <td>93.333333</td>
      <td>93.333333</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>66.366592</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>66.057551</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>1763916</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.309602</td>
      <td>68.309602</td>
      <td>68.309602</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>1043130</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.272171</td>
      <td>93.272171</td>
      <td>93.272171</td>
    </tr>
  </tbody>
</table>
</div>




```python
#top performing schools by passrate
top_performers = school_summary.sort_values("% Overall Passing Rate", ascending=False)
top_performers= top_performers.iloc[0:5,:]
top_performers
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>94.594595</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>94.133477</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>93.867718</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>1056600</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>93.867121</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>93.392371</td>
    </tr>
  </tbody>
</table>
</div>




```python
#bottom performing schools by passrate
least_performers= school_summary.sort_values("% Overall Passing Rate", ascending=True)
least_performers= least_performers.iloc[0:5,:]
least_performers
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>65.683922</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>65.988471</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>66.057551</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>66.366592</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>3124928</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>66.680064</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Math Scores by grade-initial work needed due to reformatting previous dataframes

#import school CSV files
schools_csv= "../raw_data/schools_complete.csv"
schools_df= pd.read_csv(schools_csv)
#rename "name column to "school" to match other data set 
schools_df= schools_df.rename(columns={"name": "school"})
#merging dataframes on school
mergeall_df= pd.merge(schools_df, student_df, on="school")

#Math scores by grade
math_scores_grade = pd.pivot_table(mergeall_df, index=["school"],
                                   values = ("math_score"), 
                                   columns = ["grade"], aggfunc=np.mean)
# Clean up header and reorder columns
math_final = pd.DataFrame(math_scores_grade.to_records())
math_final = math_final[["school", "9th", "10th", "11th", "12th"]]

math_final
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Reading Score by Grade
reading_scores_grade = pd.pivot_table(mergeall_df, index = ["school"],
                                   values = ("reading_score"), 
                                   columns = ["grade"], aggfunc=np.mean)

# Clean up header and reorder columns
reading_final = pd.DataFrame(reading_scores_grade.to_records())
reading_final = reading_final[["school", "9th", "10th", "11th", "12th"]]

reading_final
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
spending_stats=pd.cut(school_summary["Per Student Budget"], [-float("inf"),594,616,638,float("inf")], 
        labels=["1. Low funding $0-594","2. Med funding $594-616","3. High funding $616-638","4.Top funding $638+"],
        include_lowest=True)
spending_stats_df=pd.DataFrame(spending_stats)
spending_scores=school_summary
spending_scores["Spending Level"]=spending_stats_df
spending_scores_df=pd.DataFrame(spending_scores)

#spending_summary= spending_scores_df.reset_index(drop=False)
#spending_scores.reset_index(inplace=True)
spending_scores=spending_scores.rename(columns={'index':"school"})
#spending_scores
spending_summary=spending_scores.groupby(["Spending Level","school"]).mean()
#del spending_summary['level_0']
spending_summary

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Level</th>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">1. Low funding $0-594</th>
      <th>Cabrera High School</th>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>94.133477</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>427</td>
      <td>248087</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.505855</td>
      <td>92.505855</td>
      <td>92.505855</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>93.867718</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>1800</td>
      <td>1049400</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.333333</td>
      <td>93.333333</td>
      <td>93.333333</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2. Med funding $594-616</th>
      <th>Pena High School</th>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>94.594595</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>1761</td>
      <td>1056600</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>93.867121</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3. High funding $616-638</th>
      <th>Bailey High School</th>
      <td>4976</td>
      <td>3124928</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>66.680064</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>93.392371</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>66.366592</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>1635</td>
      <td>1043130</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.272171</td>
      <td>93.272171</td>
      <td>93.272171</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">4.Top funding $638+</th>
      <th>Figueroa High School</th>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>65.988471</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>2739</td>
      <td>1763916</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.309602</td>
      <td>68.309602</td>
      <td>68.309602</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>4635</td>
      <td>3022020</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.752967</td>
      <td>66.752967</td>
      <td>66.752967</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>65.683922</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>66.057551</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_summary
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
      <th>Spending Level</th>
      <th>size category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Huang High School</td>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>4.Top funding $638+</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Figueroa High School</td>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>4.Top funding $638+</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Shelton High School</td>
      <td>1761</td>
      <td>1056600</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>2. Med funding $594-616</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hernandez High School</td>
      <td>4635</td>
      <td>3022020</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.752967</td>
      <td>66.752967</td>
      <td>66.752967</td>
      <td>4.Top funding $638+</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>3. High funding $616-638</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Wilson High School</td>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>1. Low funding $0-594</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Cabrera High School</td>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>1. Low funding $0-594</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Bailey High School</td>
      <td>4976</td>
      <td>3124928</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>3. High funding $616-638</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Holden High School</td>
      <td>427</td>
      <td>248087</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.505855</td>
      <td>92.505855</td>
      <td>92.505855</td>
      <td>1. Low funding $0-594</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>2. Med funding $594-616</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Wright High School</td>
      <td>1800</td>
      <td>1049400</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.333333</td>
      <td>93.333333</td>
      <td>93.333333</td>
      <td>1. Low funding $0-594</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Rodriguez High School</td>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>3. High funding $616-638</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Johnson High School</td>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>4.Top funding $638+</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ford High School</td>
      <td>2739</td>
      <td>1763916</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.309602</td>
      <td>68.309602</td>
      <td>68.309602</td>
      <td>4.Top funding $638+</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Thomas High School</td>
      <td>1635</td>
      <td>1043130</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.272171</td>
      <td>93.272171</td>
      <td>93.272171</td>
      <td>3. High funding $616-638</td>
      <td>Small</td>
    </tr>
  </tbody>
</table>
</div>




```python
size_stats=pd.cut(school_summary["Total Students"], 3,labels=["Small","Medium","Large"], include_lowest=True)
size_stats_df=pd.DataFrame(size_stats)
size_scores=school_summary
size_scores["size category"]=size_stats_df
size_scores_df=pd.DataFrame(size_scores)
size_scores_df=size_scores.rename(columns={'index':"school"})

size_summary=size_scores_df.groupby(["size category","school"]).mean()

size_summary=size_summary[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading",
                           "% Overall Passing Rate"]]
size_summary
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>size category</th>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Large</th>
      <th>Bailey High School</th>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.680064</td>
      <td>66.680064</td>
      <td>66.680064</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.752967</td>
      <td>66.752967</td>
      <td>66.752967</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.057551</td>
      <td>66.057551</td>
      <td>66.057551</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.366592</td>
      <td>66.366592</td>
      <td>66.366592</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Medium</th>
      <th>Figueroa High School</th>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.988471</td>
      <td>65.988471</td>
      <td>65.988471</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.309602</td>
      <td>68.309602</td>
      <td>68.309602</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.683922</td>
      <td>65.683922</td>
      <td>65.683922</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.867718</td>
      <td>93.867718</td>
      <td>93.867718</td>
    </tr>
    <tr>
      <th rowspan="7" valign="top">Small</th>
      <th>Cabrera High School</th>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.133477</td>
      <td>94.133477</td>
      <td>94.133477</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.392371</td>
      <td>93.392371</td>
      <td>93.392371</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.505855</td>
      <td>92.505855</td>
      <td>92.505855</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.594595</td>
      <td>94.594595</td>
      <td>94.594595</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.867121</td>
      <td>93.867121</td>
      <td>93.867121</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.272171</td>
      <td>93.272171</td>
      <td>93.272171</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.333333</td>
      <td>93.333333</td>
      <td>93.333333</td>
    </tr>
  </tbody>
</table>
</div>


