<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<hr>
<!-- code_chunk_output -->

* [check_alert.py](#check_alertpy)
	* [Main](#main)
		* [read bound,db info,check_interval](#read-bounddb-infocheck_interval)
		* [check alert among various equips](#check-alert-among-various-equips)
		* [generate sms txt](#generate-sms-txt)

<!-- /code_chunk_output -->
<hr>

## check_alert.py
**Locate: pwsv03:/home/cptmain/elsa/gen_sms/**
**Run with check_alert.sh** /every 2 mins


This is written for checking alert among various equipments and creating sms.

I. Include:
1. alert_actual_weather:
HOT,COLD,DRY,PREC_H,PREC_2H,WSPD_10,WSPDX_10,VIS,UV,TIDE,TIDE_F,FTGMS,WAVE,LIGHTNING,RADAR
2.	alert_actual_air:
POLU_HO_IDX,PO_HO(PM10,PM25,SO2,CO,O3,NO2)
3. alert_actual_earth:
GAMMA,GAMMA_LOW
4. alert_special_audio:
audio


II. website
1. [config website](http://tra007.smg.net:9444/auto-s-m-s/config)
2. [show db check_alert](http://tra007.smg.net:9444/auto-s-m-s/read-sms-db)

III. need library:
1. sms_ft.py: include ft for creating sms
2. constant.py: include some needed constants
3. alert_ft.py: include ft for checking alert
4. check_config.csv:
	a. db info
	b. file loc(if exist)
	c. check_interval: the time interval get the equipment data
	d. insert_gap: time frequency for inserting db check_alert
5. config_from_php.json: config from [website](http://tra007.smg.net:9444/auto-s-m-s/configs)


<hr>

###  Main
```flow
st=>start: start
e=>end: end
op1=>operation: read bound,db info,check_interval:>#read-bounddb-infocheck_interval
op2=>operation: read last checked alert 
from db check_alert
op3=>operation: 1. check alert among various equips 
2. create alert_df:>#check-alert-among-various-equips
op4=>operation: insert db check_alert
para1=>parallel: 1. generate sms txt
2. insert db auto_sms :>#generate-sms-txt



st->op1->op2->op3->op4(right)->para1
para1(path1,bottom)->e
para1(path2, top)->op4
```
<hr>

#### read bound,db info,check_interval
```flow
st=>start: start
op1=>operation: get bound,db info,check_interval
e=>end: complete reading config
op1=>operation: read check_config.csv as config_df
op2=>operation: read config.json cover the column['insert_gap'] in config_df
op3=>operation: turn config.json into bound_n
st->op1->op2->op3->e

```
<hr>

#### check alert among various equips
```flow
st=>start: start
op1=>operation: get data from db among various equips
op2=>operation: create alert_dict among this data
e=>end: end
sub1=>subroutine: go def_n_grade:
    1. check if any data out of bound?
    2. import grade,threshold_value in alert_dict
op3=>operation: output: alert_dict
sub2=>subroutine: go set_df2

op4=>operation: insert alert_dict to alert_df
op5=>operation: skip
cond1=>condition: exist to last checked data
in db check_alert?
st->op1->op2->sub1->op3->sub2->cond1
cond1(yes,right)->op5
cond1(no)->op4->e

```
<hr>

#### generate sms txt
```flow
st=>start: with alert_df
op1=>operation: output:type_submsg_dic
key=[type/name],value=[station]
e=>end: end
op2=>operation: type_submsg_dic > filename_msg_dic
key=[filename],vale=[message]
sub1=>subroutine: go gen_type_submsg and
gen_type_submsg_gamma
op3=>operation: filename_msg_dic > 
1. write txt 
2. insert auto sms db
sub2=>subroutine: sms.gen_type_submsg_gamma
op4=>operation: a
cond1=>condition: exist to last checked alert data?

st->sub1->op1->op2->op3->e


```
[Top](#main)