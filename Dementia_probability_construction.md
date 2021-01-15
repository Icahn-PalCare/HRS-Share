# Dementia probability contruction

## Getting TICS results

'''
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
'''


