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
1. 























                                                 
     
