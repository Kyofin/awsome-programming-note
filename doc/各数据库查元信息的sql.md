# 各数据库查元信息的sql

## oracle

### 查表字段信息

```sql
select
                a.COLUMN_NAME AS B_NAME,
                a.DATA_TYPE,
                CASE WHEN a.COLUMN_NAME IN
                                    (SELECT cols.column_name
                                        FROM all_constraints  cons,
                                            all_cons_columns cols
                                        WHERE cons.constraint_type = 'P'
                                        AND cons.constraint_name = cols.constraint_name
                                        AND cons.owner = cols.owner
                                        AND cols.COLUMN_NAME=a.COLUMN_NAME
                                        AND cols.TABLE_NAME='TB_CIS_CONSULT_DETAIL'
                                        AND cons.OWNER = 'GZFY') THEN 'PRI'
                                        ELSE NULL END COLUMN_KEY,
                                b.COMMENTS AS remark
                from all_tab_cols a LEFT JOIN all_col_comments b
                on a.TABLE_NAME=b.TABLE_NAME AND a.COLUMN_NAME=b.COLUMN_NAME
                AND a.OWNER=b.OWNER
                where a.TABLE_NAME='TB_CIS_CONSULT_DETAIL'
                AND HIDDEN_COLUMN = 'NO'
                AND a.OWNER='GZFY'
```

### 查表键信息

```SQL
    select
               column_name
            from
               all_ind_columns
            where
               table_owner='GZFY'
            AND table_name = 'TB_CIS_CONSULT_DETAIL'
```

### 查表名视图

```SQL
SELECT DISTINCT
                    view_name AS table_name
                FROM
                    all_views
                WHERE OWNER = 'GZFY'
                UNION
                    (
                        SELECT DISTINCT
                            table_name
                        FROM
                            all_tables
                        WHERE OWNER = 'GZFY'
                    )
                ORDER BY
                    table_name
```

### 预览数据

```SQL
select  "TB_CIS_EMR_FEE"."UPDATE_DATE", "TB_CIS_EMR_FEE"."COMMENTS", "TB_CIS_EMR_FEE"."ID", "TB_CIS_EMR_FEE"."VISITING_SERIAL_NUMBER", "TB_CIS_EMR_FEE"."MEDICAL_INSTITUT_CODE", "TB_CIS_EMR_FEE"."MEDICAL_RECORD_NUMBER", "TB_CIS_EMR_FEE"."INDIVIDUALS_AMOUNT", "TB_CIS_EMR_FEE"."INVOICE_TOTAL_AMOUNT", "TB_CIS_EMR_FEE"."COMMON_MED_SERVICE_FEE", "TB_CIS_EMR_FEE"."COMMON_TREAT_FEE", "TB_CIS_EMR_FEE"."NURSE_FEE", "TB_CIS_EMR_FEE"."SYNTH_MED_SERVICE_FEE", "TB_CIS_EMR_FEE"."PATHO_DIAGNOSIS_FEE", "TB_CIS_EMR_FEE"."LABORATORY_FEE", "TB_CIS_EMR_FEE"."RADIO_FEE", "TB_CIS_EMR_FEE"."CLINIC_DIAGNOSIS_FEE", "TB_CIS_EMR_FEE"."NOOP_TREAT_FE", "TB_CIS_EMR_FEE"."PHY_TREAT_FEE", "TB_CIS_EMR_FEE"."OP_TREAT_FEE", "TB_CIS_EMR_FEE"."ANESTHESIA_FEE", "TB_CIS_EMR_FEE"."OPERATION_FEE", "TB_CIS_EMR_FEE"."RECOVER_FEE", "TB_CIS_EMR_FEE"."CN_DIAGNOSIS_FEE", "TB_CIS_EMR_FEE"."EN_MEDIC_FEE", "TB_CIS_EMR_FEE"."ANTIB_DRUG_FEE", "TB_CIS_EMR_FEE"."CN_MEDIC_FEE", "TB_CIS_EMR_FEE"."CN_HERB_FEE", "TB_CIS_EMR_FEE"."TRANS_FUSION_FEE", "TB_CIS_EMR_FEE"."ALB_PRODUCT_FEE", "TB_CIS_EMR_FEE"."GLB_PRODUCT_FEE", "TB_CIS_EMR_FEE"."BCF_PRODUCT_FEE", "TB_CIS_EMR_FEE"."CKS_PRODUCT_FEE", "TB_CIS_EMR_FEE"."MATERIAL_DIS_CHECK_FEE", "TB_CIS_EMR_FEE"."MATERIAL_DIS_TREAT_FEE", "TB_CIS_EMR_FEE"."MATERIAL_DIS_OP_FEE", "TB_CIS_EMR_FEE"."OTHER_FEE", "TB_CIS_EMR_FEE"."SECRECY_LEVEL", "TB_CIS_EMR_FEE"."REVISE_SIGN", "TB_CIS_EMR_FEE"."RESERVE1", "TB_CIS_EMR_FEE"."RESERVE2", "TB_CIS_EMR_FEE"."ETL_DATE", "TB_CIS_EMR_FEE"."STATUS", "TB_CIS_EMR_FEE"."CREATE_BY", "TB_CIS_EMR_FEE"."CREATE_DATE", "TB_CIS_EMR_FEE"."UPDATE_BY" from "GZFY"."TB_CIS_EMR_FEE" WHERE ROWNUM <= 10
```



## postgressql

### 查表字段信息

```SQL
 SELECT col.COLUMN_NAME, col.data_type,
        CASE WHEN col. COLUMN_NAME IN (
        select
          "pg_catalog"."pg_constraint"."conname"
        from "pg_catalog"."pg_constraint"
          join "pg_catalog"."pg_namespace"
          on "pg_catalog"."pg_constraint"."connamespace" = "pg_catalog"."pg_namespace".oid
          join "pg_catalog"."pg_class"
          on "pg_catalog"."pg_constraint"."conrelid" = "pg_catalog"."pg_class".oid
        where
          "pg_catalog"."pg_constraint"."contype" = 'p'
          and "pg_catalog"."pg_namespace"."nspname" = 'public'
          and "pg_catalog"."pg_class"."relname" = 'salaries'
        ) THEN
        'PRI'
        ELSE
        NULL END PRI,
        (SELECT  descr.description
           FROM pg_class As cls
            INNER JOIN pg_attribute As attr ON cls.oid = attr.attrelid
           LEFT JOIN pg_namespace nspace ON nspace.oid = cls.relnamespace
           LEFT JOIN pg_tablespace tspace ON tspace.oid = cls.reltablespace
           LEFT JOIN pg_description As descr ON (descr.objoid = cls.oid AND descr.objsubid = attr.attnum)
           WHERE  cls.relkind IN('r', 'v') AND  nspace.nspname = 'public' AND cls.relname = 'salaries' AND attr.attname=col.COLUMN_NAME
        ) DESCRIPTION
        FROM
        information_schema. COLUMNS col
        WHERE
        table_schema = 'public'
        AND table_catalog = 'employees'
        AND TABLE_NAME = 'salaries'
```

### 预览数据

```SQL
select  "salaries"."emp_no", "salaries"."salary", to_char("salaries"."from_date", 'YYYY-MM-DD HH24:MI:SS'), to_char("salaries"."to_date", 'YYYY-MM-DD HH24:MI:SS') from "employees"."public"."salaries" limit 10
```

### 查表名视图

```SQL
  SELECT table_name
        FROM information_schema.tables
        where table_catalog='employees'
          and table_schema='public'
          and table_type not in('FOREIGN TABLE')
```



## Mysql

### 查表名视图

```SQL
SELECT
                    DISTINCT TABLE_NAME, TABLE_TYPE != 'BASE TABLE'
                FROM
                    information_schema.`TABLES`
                WHERE
                    TABLE_SCHEMA = "test"
```

### 查表字段信息

```SQL
 SELECT `COLUMN_NAME`,`DATA_TYPE`,`COLUMN_KEY`,`COLUMN_COMMENT` FROM information_schema.columns where TABLE_SCHEMA = 'test' AND TABLE_NAME = 'tb_cis_patient_info'
```

### 查表键信息

```SQL
select distinct column_name from `information_schema`.`STATISTICS` where table_schema = 'test' and TABLE_NAME = 'tb_cis_patient_info'
```

### 预览数据

```SQL
select  `tb_cis_patient_info`.`card_number`, `tb_cis_patient_info`.`card_type`, `tb_cis_patient_info`.`medical_institut_code`, `tb_cis_patient_info`.`file_number`, `tb_cis_patient_info`.`pub_card_division`, `tb_cis_patient_info`.`id_number`, `tb_cis_patient_info`.`id_type`, `tb_cis_patient_info`.`gender_code`, `tb_cis_patient_info`.`patient_name`, `tb_cis_patient_info`.`patient_type`, cast(`tb_cis_patient_info`.`birth_date` as datetime), `tb_cis_patient_info`.`phone_muber`, `tb_cis_patient_info`.`residence_addr`, cast(`tb_cis_patient_info`.`create_time` as datetime), `tb_cis_patient_info`.`revise_sign`, `tb_cis_patient_info`.`empi_data_type`, `tb_cis_patient_info`.`empi`, `tb_cis_patient_info`.`status`, cast(`tb_cis_patient_info`.`update_date` as datetime), `tb_cis_patient_info`.`id`, cast(`tb_cis_patient_info`.`etl_date` as datetime) from `test`.`tb_cis_patient_info` limit 10
```



## sqlserver

### 查表字段信息

```sql
SELECT
                remarks.column_name,
                columns.DATA_TYPE,
                columns.PRI,
                remarks.remark
            FROM
                (
                    SELECT
                        sc.name [column_name],
                        sep.value [remark]
                    FROM
                        sys.tables st
                    INNER JOIN sys.columns sc ON st.object_id = sc.object_id
                    LEFT JOIN sys.extended_properties sep ON st.object_id = sep.major_id
                    AND sc.column_id = sep.minor_id
                    AND sep.name = 'MS_Description'
                    WHERE
                        st.name = 'CB_COST_ITEM'
                ) remarks
            LEFT JOIN (
                SELECT DISTINCT
                    T1.COLUMN_NAME,
                    T1.DATA_TYPE,
                    T2.PRI
                FROM
                    (
                        SELECT
                            TABLE_NAME,
                            COLUMN_NAME,
                            DATA_TYPE
                        FROM
                            INFORMATION_SCHEMA.COLUMNS
                        WHERE
                            (TABLE_NAME = 'CB_COST_ITEM')
                    ) T1
                LEFT JOIN (
                    SELECT
                        COLUMN_NAME,
                        'PRI' AS PRI
                    FROM
                        INFORMATION_SCHEMA.KEY_COLUMN_USAGE
                    WHERE
                        TABLE_NAME = 'CB_COST_ITEM' AND CONSTRAINT_NAME LIKE 'PK%'
                ) T2 ON T2.COLUMN_NAME = T1.COLUMN_NAME
            ) columns ON columns.COLUMN_NAME = remarks.column_name
```

### 查表键信息

```SQL
 SELECT
                 ColumnName = col.name
            FROM
                 sys.indexes ind
            INNER JOIN
                 sys.index_columns ic ON  ind.object_id = ic.object_id and ind.index_id = ic.index_id
            INNER JOIN
                 sys.columns col ON ic.object_id = col.object_id and ic.column_id = col.column_id
            INNER JOIN
                 sys.tables t ON ind.object_id = t.object_id
            WHERE
                 ind.is_primary_key = 0
                 AND ind.is_unique = 0
                 AND ind.is_unique_constraint = 0
                 AND t.is_ms_shipped = 0
                 AND t.name = 'CB_COST_ITEM'
```

### 预览数据

```SQL
select TOP 10 [CB_COST_ITEM].[PK_ID], [CB_COST_ITEM].[ITEM_CODE], [CB_COST_ITEM].[ITEM_NAME], [CB_COST_ITEM].[ITEM_MNEMONIC], [CB_COST_ITEM].[ITEM_TYPE], [CB_COST_ITEM].[PARENT_ID], [CB_COST_ITEM].[LEVELS], [CB_COST_ITEM].[ORDER_NUM], [CB_COST_ITEM].[STATUS], [CB_COST_ITEM].[CREATE_BY], convert(NVARCHAR(120), [CB_COST_ITEM].[CREATE_DATE], 120), [CB_COST_ITEM].[UPDATE_BY], convert(NVARCHAR(120), [CB_COST_ITEM].[UPDATE_DATE], 120), [CB_COST_ITEM].[COMMENTS], [CB_COST_ITEM].[COLLECTION_SOURCE], [CB_COST_ITEM].[COLLECTION_WAY], [CB_COST_ITEM].[UNIT_ID], [CB_COST_ITEM].[DEPARTMENT_TYPE] from [medicare_ZhuHaiJinWan].[dbo].[CB_COST_ITEM] 
```

### 查表名视图

```SQL
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES ORDER BY TABLE_NAME
```

