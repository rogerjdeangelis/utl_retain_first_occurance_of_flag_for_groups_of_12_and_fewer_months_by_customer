# utl_retain_first_occurance_of_flag_for_groups_of_12_and_fewer_months_by_customer
 Retain first occurance of flag equal 1 for groups of 12 or fewer months by customer.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Retain first occurance of flag equal 1 for groups of 12 or fewer months by customer

    This should be simpler but I am burned out?

        Seven Related Solutions

           1. DOW loop
           2. Datatep DOSUBL
           (3 more in github link)
           (2 more in SAS Forum)


    SAS Forum (2 solutions)
    https://tinyurl.com/y72gjdoq
    https://communities.sas.com/t5/SAS-Programming/moving-SUM-of-n-observations-by-group/m-p/492338

    see github
    For three more general solutions(with interpoltion) see
    https://tinyurl.com/yd8fzerc
    https://github.com/rogerjdeangelis/utl_calculating_rolling_3_month_skewness_of_prices_by_stock

    Result below assumes:

     1. Flag has value 0 or 1
     2. Only One flag=1 in consecutive 12 months
     3,  No interpolation needed
     4.  If partial year retain flag=1

     FYI Months can be replaced with mod(_n_-1,12) and reset after last.customer
     Once you form the groups sums abd retains are easy.

    INPUT
    ====

    Ploblem seems to boil down to retaining the flag after the first occurance in the grouos
    below;

    SD1.HAVE total obs=22

      Up to 40 obs from SD1.HAVE total obs=22           RULES
                                                        =====
      Obs    CUSTOMER       DATE       FLAG       |    DESIRED
                                                  |
        1    customer1    31AUG2015      1        |       1
        2    customer1    30SEP2015      0        |       1
        3    customer1    31OCT2015      0        |       1
        4    customer1    30NOV2015      0        |       1
        5    customer1    31DEC2015      0        |       1
        6    customer1    31JAN2016      0        |       1
        7    customer1    29FEB2016      0        |       1
        8    customer1    31MAR2016      0        |       1
        9    customer1    30APR2016      0        |       1
       10    customer1    31MAY2016      0        |       1
       11    customer1    30JUN2016      0        |       1
       12    customer1    31JUL2016      0        |       1  sequence of 12 months
                                                             retain flag
       13    customer1    31AUG2016      0        |       0
       14    customer1    30SEP2016      0        |       0
       15    customer1    31OCT2016      1        |       1
       16    customer1    30NOV2016      0        |       1
       17    customer1    31DEC2016      0        |       1
       18    customer1    31JAN2017      0        |       1 do not have 12
                                                            retain flag

       19    customer2    31AUG2015      1        |       1
       20    customer2    30SEP2015      0        |       1
       21    customer2    31OCT2015      0        |       1
       22    customer2    30NOV2015      0        |       1  do not have 12
                                                             retain flag


    PROCESS
    =======

     1. DOW loop

        data want(keep=customer date flag flg rename=flg=desired);
           retain flg savOb 0 obs obsx 1;
           do until (obs=13 or last.customer);
              set sd1.have;
              by customer;
              if flag=1 then savOb=obs-1;  * make the ob for retain;
              obs=obs+1;
           end;
           obsx=1;
           do until (obsx=13 or last.customer);
              set sd1.have;
              by customer;
              if obsx ge savob then flg=1;
              else flg=0;
              output;
              obsx=obsx+1;
           end;
           obs=2;
       run;quit;

    2. Datatep DOSUBL

       data want;

         if _n_=0 then do;   %let rc=%sysfunc(dosubl('
           * CREATE 12 MONTH AND LESS THAN 12 MONTH GROUPS;
           data havGrp;
              retain obs grp 0 ;
              do until (obs=12 or last.customer);
                 set sd1.have;
                 by customer;
                 output;
                 obs=obs+1;
              end;
              obs=0;
              grp=grp+1;
              drop obs;
           run;quit;'));
         end;

         retain desired 0;
         set havGrp;
           by grp;
           if flag eq 1 then desired=flag;
           output;
           if last.grp then desired=0;

       run;quit;

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input CUSTOMER :$10. DATE :date9. flag ;
    format date date9.;
    cards4;
    customer1 31AUG2015 1
    customer1 30SEP2015 0
    customer1 31OCT2015 0
    customer1 30NOV2015 0
    customer1 31DEC2015 0
    customer1 31JAN2016 0
    customer1 29FEB2016 0
    customer1 31MAR2016 0
    customer1 30APR2016 0
    customer1 31MAY2016 0
    customer1 30JUN2016 0
    customer1 31JUL2016 0
    customer1 31AUG2016 0
    customer1 30SEP2016 0
    customer1 31OCT2016 1
    customer1 30NOV2016 0
    customer1 31DEC2016 0
    customer1 31JAN2017 0
    customer2 31AUG2015 1
    customer2 30SEP2015 0
    customer2 31OCT2015 0
    customer2 30NOV2015 0
    ;;;;
    run;quit;

