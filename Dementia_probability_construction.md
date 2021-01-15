# Dementia probability contruction

This code uses both SAS and Stata. 

## Getting TICS results

```
libname cogn 'D:\HRS\Shared\raw\HRS\hrs_public_2016\HRS_Cog';



data cog;
set cogn.COGIMP9216A_R;
run;

%macro gettics(w=,year=);

data tics1_&w;
set cog;
keep hhid pn core_year prediction_year
r&w.cogtot r&w.status r&w.bwc20 r&w.vp
 r&w.ser7  r&w.scis r&w.cact r&w.pres r&w.mo r&w.dy r&w.yr r&w.dw
 r&w.imrc r&w.dlrc r&w.vocab r&w.mstot 
 r&w.fbwc20 r&w.fvp
 r&w.fser7  r&w.fscis r&w.fcact r&w.fpres r&w.fmo r&w.fdy r&w.fyr r&w.fdw
 r&w.fimrc r&w.fdlrc r&w.fvocab;
core_year=&year;
prediction_year=&year+1;
run;

data tics2_&w;
set tics1_&w;
rename r&w.cogtot=TICS_tot;
rename r&w.status=imp_elig_status;
rename r&w.mo=mo;
rename r&w.dy=dy;
rename r&w.yr=yr;
rename r&w.dw=dw;
rename r&w.bwc20=bwc20;
rename r&w.ser7=ser7 ;
rename r&w.scis=scis;
rename r&w.cact=cact;
rename r&w.pres=pres;
rename r&w.imrc=imrc;
rename r&w.dlrc=dlrc;
rename r&w.vp=vp;
rename r&w.vocab=vocab;
rename r&w.mstot=mstot;

rename r&w.fmo=mo_imp;
rename r&w.fdy=dy_imp;
rename r&w.fyr=yr_imp;
rename r&w.fdw=dw_imp;
rename r&w.fbwc20=bwc20_imp;
rename r&w.fser7=ser7_imp;
rename r&w.fscis=scis_imp;
rename r&w.fcact=cact_imp;
rename r&w.fpres=pres_imp;
rename r&w.fimrc=imrc_imp;
rename r&w.fdlrc=dlrc_imp;
rename r&w.fvp=vp_imp;
rename r&w.fvocab=vocab_imp;




%mend;

%gettics(w=4,year=1998);
%gettics(w=5,year=2000);
%gettics(w=6,year=2002);
%gettics(w=7,year=2004);
%gettics(w=8,year=2006);
%gettics(w=9,year=2008);
%gettics(w=10,year=2010);
%gettics(w=11,year=2012);
%gettics(w=12,year=2014);
%gettics(w=13,year=2016);

data tics_9816;
set tics2_4 tics2_5 tics2_6 tics2_7 tics2_8 tics2_9 tics2_10 tics2_11 tics2_12 tics2_13;
label TICS_tot='TICS Cognition Score, scale 0-35';
run;

proc sort nodupkey; by hhid pn core_year ; run;

/*convert to stata*/
proc export data=tics_9816 
outfile="D:\HRS\Shared\base_data\hrs_cleaned\working\tics_9816.dta"
replace;
run;
```

## Proxies

```
libname raw 'D:\HRS\Shared\raw\HRS\hrs_public_2016\core';

proc import datafile="D:\HRS\Shared\raw\HRS\hrs_public_2014\core\H98PC_R.DTA"  out=h98pc_r replace;
run;

data core1998a;
set h98pc_r;
keep hhid pn f1389-f1460;
run;

data core1998;
set core1998a;
core_year=1998;
run;

data core2000;
set raw.core2000;
keep core_year hhid pn g1543-g1614;
core_year=2000;
run;

data core2002;
set raw.core2002;
keep core_year hhid pn hd506-hd554;
core_year=2002;
run;

data core2004;
set raw.core2004;
keep core_year hhid pn jd506-jd554;
core_year=2004;
run;

data core2006;
set raw.core2006;
keep core_year hhid pn kd506-kd554;
core_year=2006;
run;

data core2008;
set raw.core2008;
keep core_year hhid pn ld506-ld554;
core_year=2008;
run;

data core2010;
set raw.core2010;
keep core_year hhid pn md506-md554;
core_year=2010;
run;

data core2012;
set raw.core2012;
keep core_year hhid pn nd506-nd554;
core_year=2012;
run;


data core2014;
set raw.core2014;
keep core_year hhid pn od506-od554;
core_year=2014;
run;

data core2016;
set raw.core2016;
keep core_year hhid pn pd506-pd554;
core_year=2014;
run;

data proxy;
set core1998 core2000 core2002 core2004 core2006 core2008 core2010 core2012 core2014 core2016;
run;

proc export data=proxy outfile="D:\HRS\Shared\base_data\hrs_cleaned\working\\proxy.dta" replace; run;
```

## Proxy Stata

```

use "D:\HRS\Shared\base_data\hrs_cleaned\working\proxy.dta", clear


local fvars 1389 1394 1399 1404 1409 1414 1419 1424 1429 1434 ///
 1439 1444 1448 1451 1454 1457

local i=506
foreach f of local fvars {
	gen fd`i'=f`f'
	gen fd`=`i'+1'=f`=`f'+1'
	gen fd`=`i'+2'=f`=`f'+2'
	local i=`i'+3
}

local gvars 1543 1548 1553 1558 1563 1568 1573 1578 1583 1588 1593 ///
1598 1602 1605 1608 1611

local i=506
foreach g of local gvars {
	gen gd`i'=g`g'
	gen gd`=`i'+1'=g`=`g'+1'
	gen gd`=`i'+2'=g`=`g'+2'
	local i=`i'+3
}
keep hhid pn *d* core_year
tokenize f g h j k l m n o p
gen cy2=(core_year-1996)/2
levelsof cy2, local(levels)

local j=1
local k=506
forvalues i=1/16 {
	gen base=.
	gen bet=.
	gen worse=.
	gen pc`i'_notdone=0
	foreach j of local levels {
		qui replace pc`i'_notdone=1 if ``j''d`k'==4 & cy2==`j' 
		qui replace base=``j''d`k' if cy2==`j'
		qui replace bet=``j''d`=`k'+1' if cy2==`j'
		qui replace worse=``j''d`=`k'+2' if cy2==`j'
}
	gen pc`i'=3 if base==2
	qui replace pc`i'=bet if inlist(bet,1,2)
	qui replace pc`i'=worse if inlist(worse,4,5)
	drop base bet worse
	local k=`k'+3
	local j=`j'+1
}

forvalues i=1/16 {
	local pc `pc' pc`i'
}
foreach x in miss total mean {
	egen iq`x'=row`x'(`pc')

}





drop if iqmiss==16
save "D:\HRS\Shared\base_data\hrs_cleaned\working\proxycog.dta", replace

```

## Creating probable dementia for all individuals and exporting dataset

```
use "D:\HRS\Shared\raw\HRS\hrs_tracking_2016\trk2016tr_r.dta", clear


tokenize a b c d e f g h j k l m n o p

local yr=1
foreach x in 1992 1993 1994 1995 1996 1998 2000 2002 2004 2006 2008 2010 2012 2014 2016 {
	rename ``yr''age age`x'
	local yr=`yr'+1
}	
keep hhid pn age* gender
reshape long age, i(hhid pn) j(core_year)
tempfile track
save `track'

use "D:\HRS\Shared\base_data\hrs_cleaned\core_00_to_16.dta", clear
gen prediction_year=core_year+1
merge 1:1 hhid pn core_year using "D:\HRS\Shared\base_data\hrs_cleaned\working\tics_9816.dta" , nogen keep(match master)
merge 1:1 hhid pn prediction_year using  "D:\HRS\Shared\raw\HRS\hrs_public_2016\dementia\pdem_withvarnames.dta", keepusing(prob)  keep(match master) nogen
merge 1:1 hhid pn core_year using "D:\HRS\Shared\raw\HRS\hrs_public_2016\dementia\ADAMS\dementia_dx_adams_wave1_only.dta", nogen
merge m:1 hhid  pn core_year using `track', keepusing(age gender) keep(match master) nogen
merge 1:1 hhid pn core_year using "D:\HRS\Shared\base_data\hrs_cleaned\working\proxycog.dta", nogen keep(match master) keepusing(iq*)

replace female=gender-1 if !missing(gender)

replace c_ivw_date=td(31dec2014) if core_year==2014 & missing(c_ivw_date)

keep if age>=65
gen age_cat=1
replace age_cat=2 if age>=70
replace age_cat=3 if age>=75
replace age_cat=4 if age>=80
replace age_cat=5 if age>=85
replace age_cat=6 if age>=90
tab age_cat, gen(age_cat)
label define age_cat 1 "Age<70" 2 "Age 70-74" 3 "Age 75-79" 4 "Age 80-84" ///
5 "Age 85-89" 6 "Age>=90"
label values age_cat age_cat

gen ed_hs_only=educ==2
gen ed_gt_hs=educ>2
gen n=1
sort id core_year
by id: gen hasn1=!missing(core_year[_n-1])
egen adl_diff_index=rowtotal(adl_diff*)
egen adlmiss=rowmiss(adl_diff*)
local iadl iadl_diff_mp iadl_diff_gr iadl_diff_ph iadl_diff_rx iadl_diff_m 
sum n `iadl'
egen iadl_diff_index=rowtotal(`iadl')
egen iadlmiss=rowmiss(`iadl')
replace adlmiss=1 if adlmiss>1
replace iadlmiss=1 if iadlmiss>1
*replace adl_diff_index=. if missing(adl_index_core)
gen dates=mo+dy+yr+dw
gen dates_imp=mo_imp==1 |dy_imp==1 | yr_imp==1 | dw_imp==1
gen iqmissany=iqmiss>0 if !missing(iqmiss)
gen iqmissgt2=iqmiss>2 if !missing(iqmiss)
local cogvars proxy_core  imrc dlrc ser7 bwc20 dates scis cact pres vp adl_diff_index ///
iadl_diff_index adlmiss iadlmiss iqmean iqmissgt2 iqmissany
sum n `cogvars' if !proxy & !missing(dx_a)
sum n `cogvars' if proxy & !missing(dx_a)


foreach x of local cogvars {
sort id core_year
qui by id: gen prev`x'=`x'[_n-1]
qui gen ch_`x'=`x'-prev`x'
if inlist("`x'","imrc","dlrc","ser7","bwc20","dates","scis","cact","pres","vp") ///
qui gen miss`x'=`x'_imp==1
if !inlist("`x'","imrc","dlrc","ser7","bwc20","dates","scis","cact","pres","vp") ///
qui gen miss`x'=`x'==.
qui gen missprev`x'=prev`x'==.
if inlist("`x'","imrc","dlrc","ser7","bwc20","dates","scis","cact","pres","vp") ///
& proxy[_n-1]==0 qui replace missprev`x'=`x'_imp[_n-1]==1

qui replace ch_`x'=0 if ch_`x'==.
qui replace prev`x'=`x' if prev`x'==.
sum prev`x' if proxy==1
replace prev`x'=0 if missprev`x' & proxy==1 & prevproxy==1 & "`x'" !="proxy_core"
}
gen adliadlmiss=adlmiss | iadlmiss
egen cogmissany=rowmax(missbwc missser7 missscis misscact misspres missimrc missdlrc ///
missdates adliadlmiss)

egen missprev=rowmax(missprevimrc missprevdlrc missprevser7 missprevbwc20 missprevdates ///
missprevscis missprevcact missprevpres missprevvp)

/*note-most missing variables excluded due to collinearity*/
local regvars  age_cat3 age_cat4 age_cat5 age_cat6 ed_hs_only ed_gt_hs female ///
adl_diff_index iadl_diff_index ch_adl_diff ch_iadl_diff  dates bwc20 ///
ser7 scis cact pres imrc dlrc ch_dates ch_bwc20 ch_ser7 ch_scis ch_cact ch_pres ///
ch_imrc ch_dlrc 

local proxyvars  age_cat3 age_cat4 age_cat5 age_cat6 ed_hs_only ed_gt_hs female ///
adl_diff_index iadl_diff_index ch_adl_diff ch_iadl_diff iqmean  ///
prevproxy c.ch_iqmean prevdates prevser7 prevpres previmrc prevdlrc



sum `regvars' if core_year==2012
sum `regvars' if core_year==2014
sum `regvars' if !proxy & !missing(dx_a)
sum `proxyvars' if proxy & !missing(dx_a)


oprobit dx `regvars' if proxy==0
estimate store oprob1
predict pself if proxy==0
predict pself2 if proxy==0, outcome(#2)
predict pself3 if proxy==0, outcome(#3)
oprobit dx `proxyvars' if proxy==1
estimates store oprob2
predict pdem if proxy==1
predict pdem2 if proxy==1, outcome(#2)
predict pdem3 if proxy==1, outcome(#3)

replace pdem=pself if proxy==0
replace pdem2=pself2 if proxy==0
replace pdem3=pself3 if proxy==0
/*/note--this gets most-likely diagnosis, but we more commonly use a cutoff for 
probable dementia of pdem>.5*/
gen ldem=1 if !missing(pdem)
replace ldem=2 if pdem2>pdem
replace ldem=3 if pdem3>pdem2 & pdem3>pdem
gen likely_dem=ldem==1 if !missing(pdem)
gen likely_cind=ldem==2 if !missing(pdem)
gen likely_normal=ldem==3 if !missing(pdem)
preserve
keep id hhid pn proxy pdem* prob_dem core_year
rename prob_dem prob_hurd
save "D:\HRS\Shared\raw\HRS\hrs_public_2016\dementia\pdem_withvarnames_00_16", replace
restore
gen dem=dx_adams==1 if !missing(dx_adams)
*logit dem pdem
*lroc
gen mdem=missing(pdem)
tab mdem
gen mhurd=missing(prob) if core_year<=2006 & age>=70
tab mdem mhurd if !missing(dx_a)

sum `regvars' if !proxy & core_year>=2000
sum `proxyvars' if proxy & core_year>=2000

```

