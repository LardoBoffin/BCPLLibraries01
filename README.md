# BCPLLibraries01
Creation and use of a simple Library for Strings.

In BCPLCopyFile02 we used the standard LIB in order to call VDU and get green text on a black background. In this sample I have created a simple Library file to add a few string handling functions not natively present in BCPL. 

The main purpose was of course to figure out how to create a library which can be described as easy if you know how...

So why bother?

1) Code re-use, obviously

2) Precompiled code - the LIB files are already in CINT code so just need to be added in (via NEEDCIN) which is faster

3) Limited resources - the Beeb only has 32KB of RAM which BCPL eats into very quickly. Not having all the code in one file helps

According to the BCPL manual on a Beeb with no co-processor you can expect to only have a 200 line program to compile before running out of memory. Assuming nothing else is loaded of course. Things are better with a co-processor but still limited.

Anatomy of a Library file: - 

There are two files required, a header and a code file.

HEADER FILE

The header creates global variables to hold the function names, in this example we have: - 
  
    //strhdr
  
    manifest $( ug = firstfreeglobal $)
    
    GLOBAL $(
      LENGTH:ug
      CONCATSIZE:ug+1
      CONCAT:ug+2
      LEFT:ug+3
    $)

Each function that can be called from the library has a name (that matches exactly) set here. I use the 'firstfreeglobal' constant to avoid having to keep track of them.

CODE FILE

The code file contains one or more sections which have code which will be compiled to the library file. In this example we have: -

    SECTION "STRINGS"
    
    //Various string functions
    //
    //27/03/2016 PJ
    //
    //V1.00
    // length(string) - returns a length in bytes
    // concatSize(string1,string2) - returns the required vector size
    // concat(string1,string2) - joins two strings and returns a vector
    // left(string,length) - returns 'length' number of characters
    
    get "LIBHDR"
    get "SYSHDR"
    GET "STRHDR"
    
    let length(S) = VALOF
    $(
       RESULTIS S%0 //size of string in bytes is held in byte 0
    $)
    
    let concatSize(S,T) = VALOF
    $(
       //takes two strings and returns the length divided by 2
       //this is the size of the vector to hold the results
       RESULTIS (S%0 + T%0)/2
    $)
    
    let concat(S,T) = VALOF
    $(
       let size, vector, ScharCount, TcharCount = 0,0,0,0
    
       size := concatSize(S,T)*2  //get the size of the total string (this is a VEC size so *2)
       ScharCount := S%0
       TcharCount := T%0
    
       vector := GETVEC(size)
    
       //loop through the first string and add it to the new vctor
    
       FOR N = 0 TO ScharCount DO
          vector%N := S%N
    
       FOR N = ScharCount+1 TO Size DO
          vector%(N) :=T%(N-ScharCount)
    
       vector%0 := Size
    
       RESULTIS vector
    
    $)
    
    let left(S,L) = VALOF
    $(
       //return the left ## number of characters
       //if ## is greater than string size return the string as is
    
       let size, vector = 0,0
    
       size := S%0
    
       IF size < L THEN L :=size
    
       vector := GETVEC(L/2)
    
       FOR N = 0 TO L DO
          vector%N := S%N
    
       vector%0 :=L
    
       RESULTIS vector
    
    $)
    .

Note that the first line is a 'SECTION' and there is a full stop at the end of the section. Each section must have a full stop (unless the section ends at the end of the file but best to use one anyway).

Note the name in the SECTION is the name to be used in the NEEDS directive in the file that will be calling the library. 

Note there is no START() function as this is a library not a program.

Note that you must include the header file in the GET section (GET "STRHDR") for the function names to be recongised in the library file. This call must also be placed in whichever program wants to use the library.

To compile the library file it is a simple case of the usual: - 

    BCPL stings LIBStr
    *drive 1
    SAVE LIBStr


Note the inclusion of the section name in the compiler output.

Once compiled the file can be used as a library in the normal way.

On drive 0 there is a sample file that uses the library called "STRTEST".

This file has a NEEDS "strings" at the start and includes GET "STRHDR". As mentioned above in order for this file to be compiled the global variables holding the locations of the library functions need to be present.

Sample code to call the library: - 

    NEEDS "strings"
    GET "LIBHDR"
    GET "STRHDR"
    
    LET START() BE
    $(
       LET l = 0
       LET s,t = "Hello there," , " World!*N"
       LET target = GETVEC(concatSize(s,t))
    
       target := concat(s,t)
    
       WRITES(target)
    
       l := LENGTH ("string test take 2")
       WRITEN(l)
    
    $)

Ideally this program would have its own header file which would include the GET functions.

In order to use this file: - 

    BCPL strtest tmp 
    *drive 1 
    SAVE tmp 
    NEEDCIN tmp LIBSTR exe 
    *destroy tmp 


Then type "exe" to run the program. It will show some text and then print out the length of the text "string test take 2".

To follow - addition of more string functions and a look into vectors which are used extensively in string handling. 

