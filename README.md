# utl-renaming-duplicate-excel-column-names-before-importing
Renaming duplicate excel column names before importing,
    Renaming duplicate excel column names before importing                                                 
                                                                                                           
    github                                                                                                 
    https://tinyurl.com/yabaozrt                                                                           
    https://github.com/rogerjdeangelis/utl-renaming-duplicate-excel-column-names-before-importing          
                                                                                                           
    SAS Forum                                                                                              
    https://tinyurl.com/ydylkjpu                                                                           
    https://communities.sas.com/t5/SAS-Enterprise-Guide/Dataset-cut-short-after-proc-import/m-p/530784     
                                                                                                           
    INPUT                                                                                                  
    =====                                                                                                  
                                                                                                           
      d:/xls/class.xlsx  with sheet name class and named range class                                       
                                                                                                           
          Note DUPLICATE column names                                                                      
                                                                                                           
                                | Column names repeat |                                                    
          +---------+------+----+---------+------+----+                                                    
          |     A   |   B  |  C |   D     |   E  |  F |                                                    
          +---------------------+----------------------                                                    
       1  | NAME    |  SEX | AGE| NAME    |  SEX | AGE|                                                    
          +---------+------+----+---------+------+----+                                                    
       2  | ALFRED  |   M  | 99 | ALFRED  |   M  | 99 |                                                    
          +---------+------+----+---------+------+----+                                                    
       3  | BARBARA |   F  | 13 | BARBARA |   F  | 13 |                                                    
          +---------+------+----+---------+------+----+                                                    
           ...                   ...                                                                       
          +---------+------+----+---------+------+----+                                                    
       20 | WILLIAM |   M  | 15 | WILLIAM |   M  | 15 |                                                    
          +---------+------+----+---------+------+----+                                                    
                                                                                                           
       [CLASS]                                                                                             
                                                                                                           
    EXAMPLE OUTPUT SAS TABLE                                                                               
    ------------------------                                                                               
                                                                                                           
    Naming of duplicates is arbitrary, you can decide.                                                     
                                                                                                           
    WORK.WANTPRE total obs=20                                                                              
                                                                                                           
      AGE    AGE_COL6    NAME     NAME_COL4    SEX    SEX_COL5                                             
                                                                                                           
       14       14       Alfr999    Alfred     M        M                                                  
       13       13       Alice      Alice      F        F                                                  
       13       13       Barbara    Barbara    F        F                                                  
                                                                                                           
                                                                                                           
    CREATE INPUT                                                                                           
    ------------                                                                                           
                                                                                                           
    * CREATE TWO SETS OF SASHELP.CLASS SIDE BY SIDE;                                                       
                                                                                                           
    data have(drop=height weight);                                                                         
      merge sashelp.class sashelp.class(                                                                   
         rename =(                                                                                         
            NAME      = DNAME                                                                              
            SEX       = DSEX                                                                               
            AGE       = DAGE                                                                               
         ));                                                                                               
                                                                                                           
        label                                                                                              
           DNAME    =  "NAME  "  /* use labels to make dup columns */                                      
           DSEX     =  "SEX   "                                                                            
           DAGE     =  "AGE   "                                                                            
                                                                                                           
           NAME     =  "NAME  "                                                                            
           SEX      =  "SEX   "                                                                            
           AGE      =  "AGE   "                                                                            
        ;                                                                                                  
    run;quit;                                                                                              
                                                                                                           
    * EXCEL SHEET WITH DUP COLUMNS;                                                                        
                                                                                                           
    %utlfkil("d:/xls/class.xlsx");                                                                         
    ods excel file="d:/xls/class.xlsx" options(sheet_name="class");     ;                                  
    proc print data=have label noobs;                                                                      
    run;quit;                                                                                              
    ods excel close;                                                                                       
                                                                                                           
    *          _       _   _                                                                               
     ___  ___ | |_   _| |_(_) ___  _ __                                                                    
    / __|/ _ \| | | | | __| |/ _ \| '_ \                                                                   
    \__ \ (_) | | |_| | |_| | (_) | | | |                                                                  
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|                                                                  
                                                                                                           
    ;                                                                                                      
                                                                                                           
    * GET NAMES;                                                                                           
                                                                                                           
    libname xel "d:/xls/class.xlsx" scan_text=no header=no;                                                
    data names;                                                                                            
     set xel.'class$A1:Z1'n; * over specify then number of columns;                                        
    run;quit;                                                                                              
    libname xel clear;                                                                                     
                                                                                                           
    /*                                                                                                     
     WORK.NAMES total obs=1                                                                                
                                                                                                           
       F1     F2     F3      F4     F5     F6                                                              
                                                                                                           
      NAME    SEX    AGE    NAME    SEX    AGE                                                             
    */                                                                                                     
                                                                                                           
    proc transpose data=names out=namXpo( drop=_label_ rename=(_name_=col col1=colNam)) ;                  
    var _all_;                                                                                             
    run;quit;                                                                                              
                                                                                                           
    /*                                                                                                     
     WORK.NAMXPO total obs=6                                                                               
                                                                                                           
      COL    COLNAM                                                                                        
                                                                                                           
      F1      NAME                                                                                         
      F2      SEX                                                                                          
      F3      AGE                                                                                          
      F4      NAME                                                                                         
      F5      SEX                                                                                          
      F6      AGE                                                                                          
    */                                                                                                     
                                                                                                           
    proc sort data=namXpo out=namSrt;                                                                      
     by colNam;                                                                                            
    run;quit;                                                                                              
                                                                                                           
    data namFix;                                                                                           
      retain rens;                                                                                         
      length rens $4096;                                                                                   
      length ren $32;                                                                                      
      set namSrt end=dne;                                                                                  
      by colNam;                                                                                           
      if first.colNam then ren=catx(" ",col,"as", colNam);                                                 
      else ren=catx(" ",col,"as",cats(colNam,"_COL",substr(col,2)));                                       
      rens=catx(",",rens,ren);                                                                             
      if dne then call symputx("rens",rens);                                                               
    run;quit;                                                                                              
                                                                                                           
    %put &=rens;                                                                                           
                                                                                                           
     RENS=                                                                                                 
           F1 as NAME                                                                                      
          ,F4 as NAME_COL4  * second column with same name                                                 
          ,F2 as SEX                                                                                       
          ,F5 as SEX_COL5                                                                                  
          ,F3 as AGE,                                                                                      
          ,F6 as AGE_COL6                                                                                  
                                                                                                           
    * apply the rename;                                                                                    
    proc sql dquote=ansi;                                                                                  
       connect to excel (Path="d:\xls\class.xlsx" header=no);                                              
         create                                                                                            
             table wantPre as                                                                              
         select * from connection to Excel                                                                 
             (                                                                                             
              Select                                                                                       
                   &rens                                                                                   
              from                                                                                         
                   [class$]                                                                                
             );                                                                                            
         disconnect from Excel;                                                                            
    quit;                                                                                                  
                                                                                                           
                                                                                                           
    WORK.WANTPRE total obs=20                                                                              
                                                                                                           
      AGE    AGE_COL6    NAME     NAME_COL4    SEX    SEX_COL5                                             
                                                                                                           
        .        .       NAME       NAME       SEX      SEX                                                
       14       14       Alfred     Alfred     M        M                                                  
       13       13       Alice      Alice      F        F                                                  
       13       13       Barbara    Barbara    F        F                                                  
                                                                                                           
                                                                                                           
    * remove row with duplicate name text;                                                                 
                              
                                                                                                       
    data want;                                                                                             
      set wantPre(firstobs=2);                                                                             
    run;quit;                                                                                              
                                                                                                           
    WORK.WANTPRE total obs=20                                                                              
                                                                                                           
      AGE    AGE_COL6    NAME     NAME_COL4    SEX    SEX_COL5                                             
                                                                                                           
       14       14       Alfr999    Alfred     M        M                                                  
       13       13       Alice      Alice      F        F                                                  
       13       13       Barbara    Barbara    F        F                                                  
                                                                                                           
                                                                                                           
                                                                                                           
                                                                                                           
                                                                                                           
                                                                                                           
