
### Learning Objectives

You will learn how to:

  - setup a database connection to WoS-KB with R
  - use SQL statements in R Markdown
  - assign the results of the SQL query to an R object as a data frame
  - store results into your KB table space

### Prerequisites

#### Required packages and dependencies

Unfortunately, the KB documentation about how to access the WoS-KB is a
bit outdated. Here, we follow the advice from the DZHW Office and
connect to the WoS-KB using the following packages

``` r
install.packages(c("RJDBC", "rJava", "DBI"))
```

In order to connect to the database, you’ll need a database driver.
You’ll find it here:

<https://github.com/wiho-fdm/KB-RMarkdown/blob/master/inst/jdbc_driver/ojdbc8.jar>

Please download the file `ojdbc8.jar` and put it in the folder
`inst/jdbc_driver`

*Some users report frustrations with rJava. An alternative approach
would be using an ODBC driver. Here’s a brief documentation
<https://db.rstudio.com/databases/oracle/>*

We will also use dplyr for analysis

``` r
install.packages("dplyr")
```

and, of course, the `rmarkdown` package.

``` r
install.packages("rmarkdown")
```

#### VPN tunnel to FIZ Karlsruhe

Start your VPN connection. Without it, no access to the WoS-KB is
possible.

#### Database credentials

Keep your database credentials safe by storing it in an `.Renviron` file

``` r
file.edit("~/.Renviron")
```

and add the following two lines.

    kb_user=your_user_name
    kb_pwd=your_password

Restart R.

### Connect to WOS-KB

``` r
require(dplyr)
require(RJDBC)
require(rJava)
.jinit()
jdbcDriver <-
  JDBC(driverClass = "oracle.jdbc.OracleDriver", classPath = "inst/jdbc_driver/ojdbc8.jar")
jdbcConnection <-
  dbConnect(
    jdbcDriver,
    "jdbc:oracle:thin:@//biblio-p-db01:1521/bibliodb01.fiz.karlsruhe",
    Sys.getenv("kb_user"),
    Sys.getenv("kb_pwd")
  ) 
```

### Query WOS-KB with SQL

``` sql
SELECT *
FROM wos_b_2019.items
WHERE wos_b_2019.items.ut_eid IN ('000389110200022', '000372645900002', '000400754000138')
```

<div class="knitsql-table">

|   PK\_ITEMS | FK\_ISSUES | FK\_SOURCES | UT\_EID         | T9\_SGR | DOI                          | PII | ARTICLE\_TITLE                                                                       | ARTICLE\_TITLE\_EN | FIRSTPAGE | LASTPAGE | PAGE\_CNT | PUBYEAR | PUBTYPE | DOCTYPE | D\_AUTHOR\_CNT | D\_REF\_CNT | D\_SOURCE\_REF\_CNT | D\_COUNTRY\_CNT | D\_INST\_FULL\_CNT | ETAL | D\_ORGA1\_CNT |
| ----------: | ---------: | ----------: | :-------------- | :------ | :--------------------------- | :-- | :----------------------------------------------------------------------------------- | :----------------- | :-------- | :------- | :-------- | ------: | :------ | :------ | -------------: | ----------: | ------------------: | --------------: | -----------------: | :--- | ------------: |
| 20618653399 |      92097 |      109001 | 000372645900002 | NA      | 10.1109/TIE.2015.2499252     | NA  | Application of Calorimetric Method for Loss Measurement of a SynRM Drive System      | NA                 | 2005      | 2015     | 11        |    2016 | Journal | Article |              7 |          33 |                  19 |               2 |                  4 | NA   |             4 |
| 15594996565 |    2443136 |       50261 | 000389110200022 | NA      | 10.6018/analesps.33.1.256911 | NA  | Adaptation into European Spanish of the Automated Working Memory Test Battery (AWMA) | NA                 | 188       | 195      | 8         |    2017 | Journal | Article |              8 |          40 |                  26 |               2 |                  4 | NA   |             3 |
| 10388050883 |    1561580 |       53961 | 000400754000138 | NA      | 10.1051/0004-6361/201630260  | NA  | Surface-effect corrections for oscillation frequencies of evolved stars              | NA                 | NA        | NA       | 13        |    2017 | Journal | Article |              2 |          54 |                  46 |               3 |                  4 | NA   |             4 |

3 records

</div>

### Store query results as R object

Using knitr chunk option `output.var`

``` sql
select
        wos_b_2019.items.pubyear,
        wos_b_2019.databasecollection.edition_value,
        count(distinct(ut_eid)) as pubs                         
    from
        wos_b_2019.items                                            
    inner join
        wos_b_2019.databasecollection                                                                                          
            on wos_b_2019.databasecollection.fk_items =  wos_b_2019.items.pk_items    
    where
    wos_b_2019.items.pubyear in (
            2014, 2015, 2016, 2017, 2018                                                   
        )                                                  
    group by
        wos_b_2019.items.pubyear,
        wos_b_2019.databasecollection.edition_value
```

``` r
yearly_collections
#>    PUBYEAR EDITION_VALUE    PUBS
#> 1     2018       WOS.SCI 2054149
#> 2     2018      WOS.ISTP  440967
#> 3     2016      WOS.ISTP  571407
#> 4     2018     WOS.ISSHP   21160
#> 5     2014      WOS.ISTP  515827
#> 6     2018      WOS.BSCI    4039
#> 7     2016      WOS.ESCI  101820
#> 8     2017      WOS.SSCI  331083
#> 9     2016       WOS.SCI 1966278
#> 10    2017      WOS.ISTP  587904
#> 11    2017       WOS.SCI 2007069
#> 12    2014      WOS.SSCI  281326
#> 13    2017      WOS.BSCI    6239
#> 14    2015       WOS.CCR    8338
#> 15    2016      WOS.BSCI   16744
#> 16    2018      WOS.SSCI  352397
#> 17    2018       WOS.CCR    8627
#> 18    2014       WOS.CCR    8486
#> 19    2015        WOS.IC   20586
#> 20    2017      WOS.ESCI   66070
#> 21    2016     WOS.ISSHP   57583
#> 22    2016        WOS.IC   20772
#> 23    2016       WOS.CCR    8619
#> 24    2015      WOS.ISTP  532546
#> 25    2017      WOS.BHCI    2592
#> 26    2017     WOS.ISSHP   49173
#> 27    2018        WOS.IC   20061
#> 28    2017      WOS.AHCI  121615
#> 29    2015      WOS.ESCI   20948
#> 30    2018      WOS.ESCI      35
#> 31    2014       WOS.SCI 1830906
#> 32    2016      WOS.SSCI  318864
#> 33    2015     WOS.ISSHP   51599
#> 34    2015      WOS.AHCI  124350
#> 35    2015      WOS.SSCI  294228
#> 36    2016      WOS.AHCI  122364
#> 37    2016      WOS.BHCI   15297
#> 38    2014     WOS.ISSHP   43251
#> 39    2014      WOS.BSCI    7818
#> 40    2014      WOS.BHCI    7352
#> 41    2017       WOS.CCR    8735
#> 42    2018      WOS.AHCI  110112
#> 43    2017        WOS.IC   20558
#> 44    2015      WOS.BHCI   13015
#> 45    2015      WOS.BSCI    8593
#> 46    2015       WOS.SCI 1885625
#> 47    2018      WOS.BHCI     575
#> 48    2014        WOS.IC   20623
#> 49    2014      WOS.AHCI  124155
#> 50    2014      WOS.ESCI     347
```

### Recommended readings

  - R Markdown: The Definitive Guide.
    <https://bookdown.org/yihui/rmarkdown/>
