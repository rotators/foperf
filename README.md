foperf
======

FOnline script allowing you to compare performance of snippets of code.

How to use
----------

You can check examples in the tests directory of the repository. Below there is
an explanation step by step how to create your own test:

1. First define FOPERF_TESTCASE - a list of pointers to snippet functions:

   ```#define FOPERF_TESTCASE @test1, @test2```

2. Then include the foperf.fos script:

   ```#include "foperf.fos"```

3. Write the snippet functions (functions with the snippets of the code which
   you want to test). For example to test string::toFloat and StrToFloat:
   ```
   string test1()
   {
      string s = "3.1415";
      float f = 0;
      FOPERF_START;
      f = s.toFloat();
      FOPERF_STOP;
      return "string::toFloat";
   }
   
    string test2()
   {
      string s = "3.1415";
      float f = 0;
      FOPERF_START;
      StrToFloat(s, f);
      FOPERF_STOP;
      return "StrToFloat";
   }
   ```
Inside the snippet functions, place the tested code between the macros
FOPERF_START and FOPERF_STOP. You should use the macros only once per snippet
function. If you need some preparation code, put it before FOPERF_START. Finally
return a string with the name of the snippet (the name can be anything, it's
used to display the results in a friendly manner).

4. Use ASCompiler with `-run foperf::main` option to run the test.
   For example let's say the testcase script is saved as "tofloat.fos":
   
   ```ASCompiler tofloat.fos -run foperf::main```
   
   Here is the result part of the script output:
   
   ```
   Result:
   string::toFloat: 940 (BEST)
   StrToFloat: 858 (91%)
   ```
   The higher score the better.
   According to foperf, StrToFloat is slower by *at least* 9%.
 
Accuracy
--------

The script is supposed to tell you which code is faster, and as long as the speed
difference is more than 2-3%, it should tell the truth most of the time.

For snippets which take very little time to execute (under 2ms) the test is not
very accurate, because within the macros there is a hidden loop code that is
also measured. This generally benefits slower snippets by making them look less
slow compared to the fastest one. For example if you compare following snippet
functions:

```
string test3()
{
  FOPERF_START;
  int a = 1;
  FOPERF_STOP;
  return "one int";
}

string test4()
{
  FOPERF_START;
  int a = 1;
  int b = 2;
  FOPERF_STOP;
  return "two ints";
}
```

You may be surprised to see that the second snippet is only 5% slower, and the
logic tells us that it should be around 50%. That's because the hidden loop code
distorts the results for very small snippets. However, the result is still
useful - it tells which code is the fastest. For code that takes more than 2ms
to execute, the distortion of the result should be negligible.

