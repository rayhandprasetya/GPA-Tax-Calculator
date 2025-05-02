WITH result AS (
  SELECT 
    f.cod_prod AS "Product Code",
    f.nam_product AS "Product Name",
    DECODE(f.cod_cr_int_tier, 1, '1-Cummulative', 2, '2-Incremental', f.cod_cr_int_tier) AS "Tiering Type",
    a.cod_plan AS "Int Plan Code",
    a.cod_plan_desc AS "IP Desc",
    b.dat_effective AS "Eff Date Int Plan",
    b.cod_int_type AS "Int Type",
    b.bal_min_to_comp AS "Min Bal to Comp",
    c.cod_tier_no AS "Tier No",
    c.bal_int_tier AS "Balance",
    c.cod_int_indx AS "Indx Code",
    e.dat_eff_int_indx AS "Eff date Indx",
    e.rat_indx AS "Index Rate",
    c.rat_int_var AS "Var",
    e.rat_indx + c.rat_int_var AS "Net Rate",
    ROW_NUMBER() OVER (ORDER BY f.cod_prod) AS rn
  FROM ch_int_plan_mast a
  JOIN ch_int_plan b ON a.cod_plan = b.cod_plan
  JOIN ch_int_rate_tier_plan c ON b.cod_plan = c.cod_plan AND b.cod_int_type = c.cod_int_type
  JOIN (
    SELECT cod_plan, cod_int_type, MAX(dat_effective) eff_date
    FROM ch_int_rate_tier_plan
    WHERE cod_int_type = 'CR' AND flg_mnt_status = 'A'
    GROUP BY cod_plan, cod_int_type
  ) d ON b.cod_plan = d.cod_plan AND b.cod_int_type = d.cod_int_type AND b.dat_effective = d.eff_date
  JOIN (
    SELECT g.cod_int_indx, g.dat_eff_int_indx, g.rat_indx
    FROM ba_int_indx_rate g
    JOIN (
      SELECT cod_int_indx, MAX(dat_eff_int_indx) max_eff_date
      FROM ba_int_indx_rate
      GROUP BY cod_int_indx
    ) h ON g.cod_int_indx = h.cod_int_indx AND g.dat_eff_int_indx = h.max_eff_date
  ) e ON c.cod_int_indx = e.cod_int_indx
  JOIN (
    SELECT cod_prod, nam_product, cod_cr_int_tier, COD_INT_RATE_TIER_PLAN
    FROM ch_prod_mast
    WHERE flg_rd = 'N' AND cod_prod_type = 'C'
  ) f ON f.cod_prod = '403'
)
SELECT *
FROM result
WHERE rn BETWEEN 1000000 AND 2000000;
