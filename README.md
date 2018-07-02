# utl_correct_gender_for_a_group_when_gender_only_appears_on_the_last_record_of_the_group
Correct gender for a group when gender only appears on the last record of the group.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Correct gender for a group when gender only appears on the last record of the group

    I am  not sexist but this was the way it was in my public school.

       Four Solutions  ( I made chages to all except hash - see original)

           Same results in WPS and SAS for all these methods

           1. DOW loop
           2. Merge
           3. Proc SQL
           4. Hash
            
           see additional solutions on end by Bart Jablonski

    github
    https://tinyurl.com/y7jm94wm
    https://github.com/rogerjdeangelis/utl_correct_gender_for_a_group_when_gender_only_appears_on_the_last_record_of_the_group

    see
    https://tinyurl.com/y9hx9vwh
    https://communities.sas.com/t5/Base-SAS-Programming/Populating-columns-with-missing-variables/m-p/474702

      1. PG Stats     https://tinyurl.com/y9hx9vwh
      2. M. Keintz    https://communities.sas.com/t5/user/viewprofilepage/user-id/31461
      3. Novinosrin   https://communities.sas.com/t5/user/viewprofilepage/user-id/138205
      4. Novinosrin   https://communities.sas.com/t5/user/viewprofilepage/user-id/138205

    INPUT
    =====

     WORK.HAVE total obs=19

       COURSE     NAME       SEX

       HOME_EC    Alice
       HOME_EC    Barbara
       HOME_EC    Carol
       HOME_EC    Jane
       HOME_EC    Janet
       HOME_EC    Joyce
       HOME_EC    Judy
       HOME_EC    Louise
       HOME_EC    Mary        F
       SHOP       Alfred
       SHOP       Henry
       SHOP       James
       SHOP       Jeffrey
       SHOP       John
       SHOP       Philip
       SHOP       Robert
       SHOP       Ronald
       SHOP       Thomas
       SHOP       William     M



    PROCESS
    =======

    1. DOW loop
    -----------

       data want(rename=cat=sex);
         do until(last.course);
             set have;
             by course notsorted;
             if not missing(sex) then cat=sex;
         end;
         do until(last.course);
             set have; by course notsorted;
             output;
         end;
         drop sex;
       run;quit;
       * made some simplifications;

    2. Merge
    --------

       data want;
         merge have (keep=course name)
               have (where=(sex^=' '));
         by course;
       run;quit;
       * made some simplifications?

    3. Proc SQL
    -----------

       proc sql;
         create
            table want as
         select
            max(sex) as sex
            ,*
         from
            have
         group
           by course
         order
           by course
       ;quit;

       * made simplifications. works but issues a warning?;

    4. Hash
    -------

       data want;
       set have;
       if (_n_ = 1) then do;
       if 0 then set have(keep=course sex);
           declare hash h(dataset: "have(keep=course sex where=(not missing(sex))", duplicate: "r");
           h.definekey('course');
           h.definedata('sex');
           h.definedone();
        end;
        rc=h.find();
        drop rc;
        run;



    OUTPUT
    ======

     WORK.WANT total obs=19

        COURSE     NAME       SEX

        HOME_EC    Alice       F
        HOME_EC    Barbara     F
        HOME_EC    Carol       F
        HOME_EC    Jane        F
        HOME_EC    Janet       F
        HOME_EC    Joyce       F
        HOME_EC    Judy        F
        HOME_EC    Louise      F
        HOME_EC    Mary        F

        SHOP       Alfred      M
        SHOP       Henry       M
        SHOP       James       M
        SHOP       Jeffrey     M
        SHOP       John        M
        SHOP       Philip      M
        SHOP       Robert      M
        SHOP       Ronald      M
        SHOP       Thomas      M
        SHOP       William     M

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data have;
     input course$ name$ sex$;
    cards4;
     HOME_EC Alice .
     HOME_EC Barbara .
     HOME_EC Carol .
     HOME_EC Jane .
     HOME_EC Janet .
     HOME_EC Joyce .
     HOME_EC Judy .
     HOME_EC Louise .
     HOME_EC Mary F
     SHOP Alfred .
     SHOP Henry .
     SHOP James .
     SHOP Jeffrey .
     SHOP John .
     SHOP Philip .
     SHOP Robert .
     SHOP Ronald .
     SHOP Thomas .
     SHOP William M
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    * see process for SAS solutions

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
       data want(rename=cat=sex);
         do until(last.course);
             set wrk.have;
             by course notsorted;
             if not missing(sex) then cat=sex;
         end;
         do until(last.course);
             set wrk.have; by course notsorted;
             output;
         end;
         drop sex;
       run;quit;
       proc print;run;quit;
       data want;
         merge wrk.have (keep=course name)
               wrk.have (where=(sex^=" "));
         by course;
       run;quit;
       proc print;run;quit;
       proc sql;
         create
            table want as
         select
            max(sex) as sex
            ,*
         from
            wrk.have
         group
           by course
         order
           by course
       ;quit;
       proc print;run;quit;
       data want;
       set wrk.have;
       if (_n_ = 1) then do;
       if 0 then set have(keep=course sex);
           declare hash h(dataset: "wrk.have(keep=course sex where=(not missing(sex))", duplicate: "r");
           h.definekey("course");
           h.definedata("sex");
           h.definedone();
        end;
        rc=h.find();
        drop rc;
        run;
       proc print;run;quit;
       run;quit;
    ');


    *____             _
    | __ )  __ _ _ __| |_
    |  _ \ / _` | '__| __|
    | |_) | (_| | |  | |_
    |____/ \__,_|_|   \__|

    ;

    data want(rename=(s=sex));

    do point = nobs to 1 by -1;
        set have nobs = nobs point = point;
        retain s " ";
        s = coalescec(sex, s);
        drop sex;
        output;
    end;

    stop;


    Paul,

    one more try, but a little bit more "bullet proof" now :-)

    all the best
    Bart

    /**/
    data have;
    infile cards missover;
    input COURSE  $   NAME  $     SEX $;
    cards;
    HOME_EC    Alice
    HOME_EC    Barbara
    HOME_EC    Carol
    HOME_EC    Jane
    HOME_EC    Janet F
    HOME_EC    Joyce
    HOME_EC    Judy
    HOME_EC    Louise
    HOME_EC    Mary
    SHOP       Alfred
    SHOP       Henry
    SHOP       James
    SHOP       Jeffrey M
    SHOP       John
    SHOP       Philip
    SHOP       Robert
    SHOP       Ronald
    SHOP       Thomas
    SHOP       William
    ;
    run;


    data want(rename=(s=sex));
    if 0 then set have;
    declare hash class(dataset: "have", multidata: "yes", ordered: "d");
    _rc_ = class.DefineKey("COURSE", "SEX");
    _rc_ = class.DefineData(ALL: "yes");
    _rc_ = class.DefineDone();
    declare hiter iclass('class');
    drop _rc_;

    retain s " ";
    drop sex;

    _rc_ = iclass.first();
    do while(_rc_ = 0);
        s = coalescec(sex, s);
        output;
        _rc_ = iclass.next();
    end;

    stop;
    run;


