# Nutrition_Panel_data
This repository contains my work on nutrition data sets where I have used STATA for data cleaning and analysis

* setting up the working Directory
cd "D:\da"


* Importing Latest data set of Central Asia Stunting Initiative(CASI) Shared by DHRC
 import excel "D:\da\child.xlsx", sheet("Sheet1") firstrow

 * Cleaning Gender variable
 replace gender = "Boy" if gender == "Male"
replace gender = "Girl" if gender == "Female"
replace gender = "male" if gender == "Boy"
replace gender = "female" if gender == "Girl"


* Creating and Labelling Sex Variable
gen sex =.
replace sex = 1 if gender == "male"
replace sex = 2 if gender == "female"
label define xenhi 1 "male" 2 "female"
label val sex xenhi


* Converting string to numerical variable the anthro data 
 
  gen cht = real( height)
  format cht %9.2f
gen chw = real(weight)
format chw %9.2f
gen muac2 = real(muac)
format muac2 %9.2f
* Formatting date of assessment as it is in string format
 
generate date2 = date(doa, "DMY")

format %td date2

* Formatting dob as it is in string format
 

gen dob_date = date(dob, "YMDhms")
format %td dob_date

* formatting dob in different formats
generate dob_date = date(substr(dob, 1, strpos(dob, " ") - 1), "MDY")
format dob_date %td


* Generating age in months variable 
gen age_m = floor(date2 - dob) / 30.4375
generate age_rounded = round(age_m)
replace age_m = round(age_m)

* Making categories of age 
 
 gen agcat = .
 replace agcat = 1 if age_m >= 0 & age_m <= 5
 replace agcat = 2 if age_m >= 6 & age_m <= 23
 replace agcat = 3 if age_m >= 24 & age_m <= 59
 replace agcat = 4 if age_m >= 60
 
 label define acs 1 "0-5" 2 "6-23" 3 "24-59" 4 "overage"
 label val agcat acs
 
 * labelling Districts
 gen district2 = .
 replace district2 = 1 if district == "Upper Chitral"
 replace district2 = 2 if district == "Lower Chitral"
 label define denn 1 "Upper Chitral" 2 "Lower Chitral"
 label val district2 denn
 
 
  * Generating visit Numbers and assigning values 
  rename member_uid m_uid

bys m_uid: gen vn = _n

recode vn (0/1=1) (2/1000= 2), gen (viscategory)

label define assign 1 `""Registration""' 2 `""Followup""'

label values viscategory assign

* Generatig total visits for each of the child/woman
 bys m_uid: gen total_visits = _N
 
 * Generating different date types

* Monthly data
gen mdate=mofd(date2)
format mdate %tm

* Getting the month number 
gen month = real(substr( doa , 4, 2))


* Quarterly Data 

gen qdate=qofd(date2)
format qdate %tq


* Yealry Data 
gen ydate=year(date2)
format ydate %ty

* Generating Z scores for height weight and MUac 
  zscore06, a(age_m) h(height) w(weight) s(sex) male(1) female(2)
  
  
   * Making categories of flagged cases 
  
  gen whzflag = 0
replace whzflag = 1 if whz06 < -5 | whz06 > 5
label define llwhzflag 0 "0 - not flagged" 1 "1 - whz <-5 or Whz > 5"
label val whzflag llwhzflag
 
 gen wazflag = 0
replace wazflag = 1 if waz06 < -6 | waz06 > 5
label define aawazflag 0 "0 -not flagged" 1 "1 - waz < -6 or waz > 5"
label val wazflag aawazflag
  
 
 gen hazflag = 0
 replace hazflag = 1 if haz06 < -6 | haz06 > 6
 label define lahazflag 0 "0 - not flagged" 1 "1 -haz < -6 or haz > 6"
 label val hazflag lahazflag
 
 gen bmiflag = 0
 replace bmiflag = 1 if bmiz06 < -5 | bmiz06 > 5
 label define labmi 0 "0 - not flagged" 1 "1 -bmiz06 < -5 or bmiz06 > 5"
 label val bmiflag labmi
 
  * categorization of stunting and Growth falter

gen consec_count = cond(haz06 >= -2 & haz06 <= -1, 1, 0)
by m_uid: egen consec_group = sum(consec_count)
gen category = "Normal"
replace category = "Stunted" if haz06 < -2
replace category = "Growth Falter" if consec_group >= 3
drop consec_count consec_group
rename category hfa
 
 gen hfa2 = .
 replace hfa2 = 1 if hfa == "Stunted"
 replace hfa2 = 2 if hfa == "Growth Falter"
 replace hfa2 = 3 if hfa == "Normal"
 label define lhff 1 "Stunted" 2 "Growth Falter" 3 "Normal"
label val hfa2 lhff

 
 * Categorization of weight for age
 
 gen wfa = .
 replace wfa = 1 if waz06 >=-2
 replace wfa = 2 if waz06 < -2
 label define lwen 1 "normal" 2 "under weight"
 label val wfa lwen
 
 *Categorizaton of weight for height 
 gen wfh = .
 replace wfh = 1 if whz06 <= -2 & whz06 >= -3
 replace wfh = 2 if whz06 < -3
 replace wfh = 3 if whz06 > -2
 label define lwhen 1 "MAM" 2 "SAM" 3 "Normal"
 label val wfh lwhen








