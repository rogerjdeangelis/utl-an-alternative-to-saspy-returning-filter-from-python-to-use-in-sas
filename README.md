# utl-an-alternative-to-saspy-returning-filter-from-python-to-use-in-sas
An alternative to saspy returning a filter from python to use in sas 
    %let pgm=utl-an-alternative-to-saspy-returning-filter-from-python-to-use-in-sas;

    An alternative to saspy returning a filter from python to use in sas

    Subset sashelp.sas based on meta data from Python
    Select the males in sashelp.class

    github
    https://tinyurl.com/2p8rv4v8
    https://github.com/rogerjdeangelis/utl-an-alternative-to-saspy-returning-filter-from-python-to-use-in-sas

    Stackoverflow
    https://tinyurl.com/2p9bzn3j
    https://stackoverflow.com/questions/70900074/sas-python-how-to-make-table-name-dynamic-using-date-parameter

    Same code can be used with R.

    Is this what you are tring to do

    %utl_submit_py64_38('
    import pyperclip;
    sex="M";
    pyperclip.copy(sex);
    ',return=sex);

    %put &=sex;

    data want_&sex;
      set sashelp.class(where=(sex="&sex"));
    run;quit;

    /* Output
    Up to 40 obs from WANT_M total obs=10 01FEB2022:16:04:25

    Obs    NAME       SEX    AGE    HEIGHT    WEIGHT

      1    Alfred      M      14     69.0      112.5
      2    Henry       M      14     63.5      102.5
      3    James       M      12     57.3       83.0
      4    Jeffrey     M      13     62.5       84.0
      5    John        M      12     59.0       99.5
      6    Philip      M      16     72.0      150.0
      7    Robert      M      12     64.8      128.0
      8    Ronald      M      15     67.0      133.0
      9    Thomas      M      11     57.5       85.0
     10    William     M      15     66.5      112.0
    */

    /*
     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    */


    %macro utl_submit_py64_38(
          pgm
         ,return=  /* name for the macro variable from Python */
         )/des="Semi colon separated set of python commands - drop down to python";

      * delete temporary files;
      %utlfkil(%sysfunc(pathname(work))/py_pgm.py);
      %utlfkil(%sysfunc(pathname(work))/stderr.txt);
      %utlfkil(%sysfunc(pathname(work))/stdout.txt);

      filename py_pgm "%sysfunc(pathname(work))/py_pgm.py" lrecl=32766 recfm=v;
      data _null_;
        length pgm  $32755 cmd $1024;
        file py_pgm ;
        pgm=&pgm;
        semi=countc(pgm,";");
          do idx=1 to semi;
            cmd=cats(scan(pgm,idx,";"));
            if cmd=:". " then
               cmd=trim(substr(cmd,2));
             put cmd $char384.;
             putlog cmd $char384.;
          end;
      run;quit;
      %let _loc=%sysfunc(pathname(py_pgm));
      %let _stderr=%sysfunc(pathname(work))/stderr.txt;
      %let _stdout=%sysfunc(pathname(work))/stdout.txt;
      filename rut pipe  "c:\Python38\python.exe &_loc 2> &_stderr";
      data _null_;
        file print;
        infile rut;
        input;
        put _infile_;
      run;
      filename rut clear;
      filename py_pgm clear;

      * use the clipboard to create macro variable;
      %if "&return" ^= "" %then %do;
        filename clp clipbrd ;
        data _null_;
         length txt $200;
         infile clp;
         input;
         putlog "*******  " _infile_;
         call symputx("&return",_infile_,"G");
        run;quit;
      %end;

    %mend utl_submit_py64_38;
