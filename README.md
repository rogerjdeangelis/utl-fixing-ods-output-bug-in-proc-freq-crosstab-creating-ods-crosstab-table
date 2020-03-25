# utl-fixing-ods-output-bug-in-proc-freq-crosstab-creating-ods-crosstab-table
Fixing ods output bug with proc freq crosstab creating ods output table
    Fixing ods output bug with proc freq crosstab creating ods output table

    github
    https://tinyurl.com/w7qxsyx
    https://github.com/rogerjdeangelis/utl-fixing-ods-output-bug-in-proc-freq-crosstab-creating-ods-crosstab-table

    I don't think it will work in EG (sever), but thats the story of my programming life?

    This solutu=on is base on a post

     https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;aef8c93c.2003d

     by

      Bartosz Jablonski
      yabwon@gmail.com

    The solution only works for 2 dimensional 'proc freq crosstabs with and without missings

      proc freql
           table row * col / missing;   /* missing is the only option supported */
      run;quit;

    I don't think it supoorts very wide crosstabs or more that about 80,000 row levels.

    Macros on end of message and

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data class;
      set sashelp.class;
     if name in ('John','Alice') then call missing(sex,age);
     if name='Joyce') then call missing(sex);
    run;quit;

    Up to 40 obs WORK.CLASS total obs=19

    Obs    NAME       SEX    AGE    HEIGHT    WEIGHT

      1    Joyce       F      11     51.3       50.5
      2    Louise      F      12     56.3       77.0
      3    Alice               .     56.5       84.0
      4    James       M      12     57.3       83.0
      5    Thomas      M      11     57.5       85.0
      6    John                .     59.0       99.5
      7    Jane        F      12     59.8       84.5
      8    Janet       F      15     62.5      112.5

    *            _               _
      ___  _   _| |_ _ __  _   _| |_ ___
     / _ \| | | | __| '_ \| | | | __/ __|
    | (_) | |_| | |_| |_) | |_| | |_\__ \
     \___/ \__,_|\__| .__/ \__,_|\__|___/
                    |_|
    ;


    WORK.AGESEX total obs=28

      ROWNAM     LEVEL      VAR2      F           M    TOTAL

      COUNT       .         2.00     0.00      0.00     2.00
      PERCENT     .        10.53     0.00      0.00    10.53
      ROW PCT     .       100.00     0.00      0.00      .
      COL PCT     .       100.00     0.00      0.00      .
      COUNT       11        0.00     1.00      1.00     2.00
      PERCENT     11        0.00     5.26      5.26    10.53
      ROW PCT     11        0.00    50.00     50.00      .
      COL PCT     11        0.00    12.50     11.11      .
      COUNT       12        0.00     2.00      2.00     4.00
      PERCENT     12        0.00    10.53     10.53    21.05
      ROW PCT     12        0.00    50.00     50.00      .
      COL PCT     12        0.00    25.00     22.22      .
    '''

    WORK.AGESSEXCNT total obs=7

       ROWNAM    LEVEL    VAR2    F    M    TOTAL

       COUNT      .         2     0    0      2
       COUNT      11        0     1    1      2
       COUNT      12        0     2    2      4
       COUNT      13        0     1    1      2
       COUNT      14        0     2    2      4
       COUNT      15        0     2    2      4
       COUNT      16        0     0    1      1

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    %utl_odsFrq(setup);

    proc freq data=class;
      tables age*sex/missing;
    run;quit;

    %utl_odsFrq(outdsn=crosstab);

    proc print data=agesex;
    run;quit;

    data agesSexCnt;
      set crosstab(where=(rownam="COUNT"));
    run;quit;

    *
     _ __ ___   __ _  ___ _ __ ___
    | '_ ` _ \ / _` |/ __| '__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    ;

    %macro utl_odsfrq(outdsn);

       %if %qupcase(&outdsn)=SETUP %then %do;

            filename _tmp1_ clear;  * just in case;

            %utlfkil(%sysfunc(pathname(work))/_tmp1_.txt);

            filename _tmp1_ "%sysfunc(pathname(work))/_tmp1_.txt";

            %let _ps_= %sysfunc(getoption(ps));
            %let _fc_= %sysfunc(getoption(formchar));

            OPTIONS ls=max ps=32756  FORMCHAR='|'  nodate nocenter;

            title; footnote;

            proc printto print=_tmp1_;
            run;quit;

       %end;
       %else %do;

            proc printto;
            run;quit;

            filename _tmp2_ clear;

            %utlfkil(%sysfunc(pathname(work))/_tmp2_.txt);

            filename _tmp2_ "%sysfunc(pathname(work))/_tmp2_.txt";

            proc datasets lib=work nolist;  *just in case;
             delete &outdsn;
            run;quit;

            data _null_;
              infile _tmp1_ length=l;
              input lyn $varying32756. l;
              if index(lyn,'Col Pct')>0 then substr(lyn,1,7)='LEVELN   ';
              lyn=compbl(lyn);
              if countc(lyn,'|')>1;
              putlog lyn;
              file _tmp2_;
              put lyn;
            run;quit;

            proc import
               datafile=_tmp2_
               dbms=dlm
               out=_temp_
               replace;
               delimiter='|';
               getnames=yes;
            run;quit;

            data &outdsn(rename=(_total=TOTAL));
              length rowNam $8 level $64;
              retain rowNam level ;
              set _temp_;
              select (mod(_n_-1,4));
                when (0) do; level=cats(leveln); rowNam="COUNT";end;
                when (1) rowNam="PERCENT";
                when (2) rowNam="ROW PCT";
                when (3) rowNam="COL PCT";
              end;
              drop leveln;
            run;quit;

            filename _tmp1_ clear;
            filename _tmp2_ clear;

            %utlfkil(%sysfunc(pathname(work))/_tmp1_.txt);
            %utlfkil(%sysfunc(pathname(work))/_tmp2_.txt);

       %end;

    %mend utl_odsfrq;





