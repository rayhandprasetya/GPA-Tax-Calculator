SELECT *
FROM (
  SELECT t.*, ROWNUM AS rn
  FROM (
    -- Kode utama kamu di sini
    SELECT f.cod_prod "Product Code", f.nam_product "Product Name", 
           DECODE(f.cod_cr_int_tier,1,'1-Cummulative',2,'2-Incremental',f.cod_cr_int_tier) AS "Tiering Type", 
           a.cod_plan "Int Plan Code", a.cod_plan_desc "IP Desc", b.dat_effective "Eff Date Int Plan",
           b.cod_int_type "Int Type", b.bal_min_to_comp "Min Bal to Comp", c.cod_tier_no "Tier No",
           c.bal_int_tier "Balance", c.cod_int_indx "Indx Code", e.dat_eff_int_indx "Eff date Indx", 
           e.rat_indx "Index Rate", c.rat_int_var "Var", e.rat_indx + c.rat_int_var "Net Rate"
    FROM ch_int_plan_mast a, ch_int_plan b, ch_int_rate_tier_plan c, 
         (SELECT cod_plan, cod_int_type, MAX(dat_effective) eff_date
          FROM ch_int_rate_tier_plan 
          WHERE cod_int_type = 'CR' AND flg_mnt_status = 'A' 
          GROUP BY cod_plan, cod_int_type) d, 
         (SELECT g.cod_int_indx, g.dat_eff_int_indx, g.rat_indx
          FROM ba_int_indx_rate g, 
               (SELECT cod_int_indx, MAX(dat_eff_int_indx) max_eff_date 
                FROM ba_int_indx_rate 
                GROUP BY cod_int_indx) h 
          WHERE g.cod_int_indx = h.cod_int_indx AND g.dat_eff_int_indx = h.max_eff_date) e, 
         (SELECT cod_prod, nam_product, cod_cr_int_tier, COD_INT_RATE_TIER_PLAN 
          FROM ch_prod_mast 
          WHERE flg_rd = 'N' AND cod_prod_type = 'C') f 
    WHERE f.cod_prod = '403'
  ) t
  WHERE ROWNUM <= 2000000
)
WHERE rn >= 1000000;
