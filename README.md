select * from (select
f.cod_prod "Product Code",f.nam_product "Product Name",
decode(f.cod_cr_int_tier,1,'1-Cummulative',2,'2-Incremental',f.cod_cr_int_tier) as "Tiering Type",
a.cod_plan "Int Plan Code",a.cod_plan_desc "IP Desc",b.dat_effective "Eff Date Int Plan",b.cod_int_type "Int Type",
b.bal_min_to_comp "Min Bal to Comp",
c.cod_tier_no "Tier No",c.bal_int_tier "Balance",c.cod_int_indx "Indx Code",e.dat_eff_int_indx "Eff date Indx",
e.rat_indx "Index Rate",c.rat_int_var "Var",
e.rat_indx+c.rat_int_var "Net Rate"
from
ch_int_plan_mast a,
ch_int_plan b,
ch_int_rate_tier_plan c,
(select cod_plan,cod_int_type,max(dat_effective)eff_date from ch_int_rate_tier_plan where cod_int_type='CR' and flg_mnt_status = 'A' group by cod_plan,cod_int_type) d,
(select g.cod_int_indx,g.dat_eff_int_indx,g.rat_indx
from ba_int_indx_rate g,
(select cod_int_indx,max(dat_eff_int_indx)max_eff_date from ba_int_indx_rate
group by cod_int_indx)h
where
g.cod_int_indx=h.cod_int_indx
and g.dat_eff_int_indx = h.max_eff_date) e,
(select cod_prod,nam_product,cod_cr_int_tier,COD_INT_RATE_TIER_PLAN from ch_prod_mast
where flg_rd = 'N'and cod_prod_type ='C') f
where f.cod_prod = '403' 

AND ROWNUM <= 2000000)
