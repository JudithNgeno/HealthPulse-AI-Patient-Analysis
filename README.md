# HealthPulse-AI-Patient-Analysis
HealthPulse AI is a rapid-deployment analytics solution designed to bridge the gap between reactive scheduling and proactive operational excellence. This project addresses a $1.2M revenue loss by leveraging predictive modeling and interactive business intelligence.
## Data Cleaning & Transformation
### Fact_Appointments(The Transaction Table)
1. Swap Column names
   - Rename the current PATIENT_ID column to CLINIC_ID.
   - Rename the current CLINIC_ID column to PATIENT_ID.
(Verification: Patient IDs should start with "P" and Clinic IDs should start with "C").
2. Handled Missing Wait Times
   - Select the WAIT_TIME_MINUTES column.
   - Go to Transform > Replace Values.
Replaced null with 0 (Since these represent "No-Shows," the wait time is effectively zero).
3. Rename the Columns to Appear Clean and for Consistency
4. Correct Data Types
   - Change APPOINTMENT_DATE to Date.
   - Change APPOINTMENT_TIME to Time.
   - Change WAIT_TIME_MINUTES and LEAD_TIME_DAYS to Whole Number.
   - Change IS_NO_SHOW to True/False.
   - Add Status Column
     (Add Column > Conditional Column > Status > If Is No Show = FALSE then 'Completed' > If Is 'No Show' = TRUE then No Show > Else 'Null' and change type to text)


### Dim_Clinics (The Clinic Dimension)
** This file is currently a "flat" file. It should only contain unique clinic information.
1. Remove Unnecessary Columns
   - This file contains transaction data that belongs in the Fact table.
   - Keep only provider_clinic_id, clinic_assignment, clinic_name, city, and hours columns, and delete the rest.
2. Remove Duplicates
   - Right-click the provider_clinic_id column and select Remove Duplicates to be left with a small number of unique clinics.
3. Split Columns
   - Split Hours column to have Start_Time column and End_Time column
4. Add Columns
   - Add Hours column
     (Add Column > Custom Column > Hours > [End_Time] - [Start_Time] > Duration(Hours) > Delete the first Hours column and rename the new column to Hours) type should be whole number
3. Rename the Columns for Consistency
   - Rename provider_clinic_id to Clinic_Id to match your Fact table
   - Rename other columns to appear similar to the ones in Fact Table
4. Set Data Types
   - Ensure all remaining columns are set to Text.


### Dim_Patients (The Patient Dimension)
1. Correct Data Types
   - Set AGE to Whole Number.
   - Set PATIENT_ID and INSURANCE_TYPE to Text.
2. Validation
   - Right-click PATIENT_ID and select Remove Duplicates just to ensure data integrity.
3. Add Columns
   - Add Age_Group_Range column
   - (Add Column tab > Conditional Column > New column name: Age_group
* If Age is less than or equal to 12 Then 0-12
* Else If Age is less than or equal to 18 Then 13-18
* Else If Age is less than or equal to 40 Then 19-40
* Else If Age is less than or equal to 65 Then 41-65
* Else 65+)
   - Add Age_Group column
   - (Add Column tab > Conditional Column > New column name: Age_group
* If Age_Group_Range is equals to 0-12 Then Child
* Else If Age_Group_Range is equal to 13-18 Then Teen
* Else If Age_Group_Range is equal to 19-40 Then Young
* Else If Age_Group_Range is equal to 41-65 Then Middle Aged
* Else Old)
4. Rename the Columns for Consistency
   - Rename the columns to appear similar to the ones in Fact Table.


### Dim_Providers (The Provider Dimension)
1. Correct Data Types
   - Ensure all columns are set to Text.
2. Clean Whitespace
   - Select all columns, right-click, and select Transform > Trim to remove any accidental trailing spaces.
3. Rename the Columns for Consistency
   - Rename the columns to appear similar to the ones in Fact Table.

### Dim_Dates (The Date Dimension)
1. Fix Date Formatting
   - Select APPOINTMENT_DATE and change the type to Date.
2. Year Formatting
   - Change YEAR to Whole Number
3.  Rename the Columns for Consistency
   -  Rename the columns to appear similar to the ones in Fact Table.
* Ensure the "Year" column is set to "Don't Summarize" in the Power BI report view.

## Data Visualization
### Phase 1: The "Power Measures" (DAX)
1. No-Show Rate (%)
   - No-Show Rate = DIVIDE(SUM('Fact_Appointments'[Is_No_Show]), COUNT('Fact_Appointments'[Appointment_Id]))
2. Avg Wait Time (Minutes)
   - Avg Wait Time = AVERAGE('Fact_Appointments'[Wait_Time_Minutes])
3. Estimated Revenue Loss (Based on the $1.2M figure and ~26k no-shows)
   - Revenue Loss = SUM('Fact_Appointments'[Is_No_Show]) * 45
4. Provider Utilization Index (Total appts per provider vs. average)
   - Provider Load = COUNT('Fact_Appointments'[Appointment_Id])

### Phase 2: Dashboard Layout Design
#### Page 1: Executive Overview
1. KPI Cards (Top Row)
   - Current No-Show Rate. "Conditional Formatting"
   - Select No-Show Rate Card > Format Visual pane > Callout value > Color and click the fx icon > Set up the rules
- Use "Conditional Formatting" (Green if $<12\%$ (Goal Met), Red if $>15\%$ (Critical - $1.2M Loss)).

   - Avg Wait Time (Target: 15 min)
  - * Create an "Actual Wait Time" Measure
    * Actual Avg Wait Time = CALCULATE(AVERAGE('Fact_Appointments'[Wait_Time_Minutes]), 'Fact_Appointments'[Is_No_Show] = 0)
  - Reason for creating this new measure is;
  - As a result of the cleaning step where we replaced null values with 0
  - (To reduce wait times from 45 minutes to 15 minutes, we are likely referring to the experience of the patients who actually walk in the door. Including No-Shows as "0 minutes" artificially drags our average down and makes the clinics look like they are performing better than they actually are using avg wait time)

   -Total Revenue Loss (A big red number to drive urgency)
  
2. Slicers to allow for granular drill-downs into the 15 clinics' operations

| Slicer | Business Purpose |
| :--- | :--- |
| **Month Name** | Enables time-series analysis and seasonal trend identification. |
| **Specialty** | Allows Department Heads to isolate performance metrics for their specific units. |
**Select the Slicer, go to the Filters Pane, and under "Specialty," uncheck the (Blank) value to filter out the blank.

3. The "Matrix" (Middle)
- Visual-Matrix.
- Rows-Clinic Name
- Columns-Day of Week.
- Values-No-Show Rate.
- * To arrange the Days of the Week in order (instead of alphabetically)
  * Create a Day Number Column
  * Table View > Select Dim_Dates table > New Column > (Day Order = WEEKDAY('Dim_Dates'[Appointment_Date], 2)) > column header for Day_of_Week > Column Tools tab > Sort by Column > Day Order from the dropdown list
    d. Clustered Column Chart
    - X-Axis-Month_Name
    - Y-Axis-No Show Rate
    - Legend-Year
    - * Add a Constant Line to show the industry goal of 12%
      * Select the Chart > Open the Analytics Pane(magnifying glass icon) > Y Axis then Find Constant Line > Add Line and edit text to Industry Goal (12%) > Edit the Properties * Value: This should be set to 0.12 (since your Y-axis is in percentages > Style (Change the dash type to Dotted and color)
      * Data Label: Ensure "Data label" is toggled On to see the "Industry Goal (12%)" text appearing on the chart itself
      
#### Page 2: AI Analysis(No-Show & Flow Analysis)
1. Scatter Chart
   - Lead Time Days (X-axis)
   - * click the small down-arrow next to Lead_time_days > Select "Don't Summarize"
   - No-Show Rate (Y-axis)
2. Wait Time vs. No-Show Correlation (Scatter Chart)
   - Visual-Scatter Chart.
   - X-Axis-Actual Avg Wait Time.
   - Y-Axis-No-Show Rate
   - Values-Clinic_Name
   - * Add two Median lines for no show rate and actual avg time so that lines will automatically recalculate if you filter the data
     * Select the Chart > Open the Analytics Pane(magnifying glass icon) > Median (Because a scatter chart has two numeric axes, the Analytics pane asks which "Series" you want the median for)
     * Line 1 (X-Axis): Add a Median line and set the Series to Actual Avg Wait Time. This creates the vertical line at 21.32.
     * Line 2 (Y-Axis): Clicked "+ Add" again to create "Median line 2" and set that Series to No Show Rate. This creates the horizontal line at 17.65%.
   - NOTE The No Show Rate don't match because, if we have a few very large clinics with no-show rates higher than the rest of the group, our Executive Overview (Mean) will be higher than your Scatter Chart in our AI-Analysis (Median).
   - Essentially, the 17.79% tells us the "Real World" performance of the whole organization, while the 17.65% tells us where the "Typical" clinic sits relative to its peers.
3.  


  























                                                 
     
