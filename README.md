foperf
======

FOnline script allowing you to compare performance of other scripts.

How to use
----------

1. First define FOPERF_TESTCASE - a list of pointers to snippet functions:

   ```#define FOPERF_TESTCASE @test1, @test2```

2. Include foperf.fos script:

   ```#include "foperf.fos"```

3. Write the snippet functions (functions with the snippets of the code which
   you want to test). In my case I will be checking what is faster - StrToFloat
   or string::toFloat
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
Inisde a snippet function, use macros FOPERF_START and FOPERF_STOP to start and
stop the clock. You should use the macros only once per snippet function. If you
need some preparation code, put it before FOPERF_START. Finally return a string
with the name of the snippet (the name can be anything, it's used to display the
results in a friendly manner).

4. Use ASCompiler with `-run foperf::main` option to run the test. Let's say I
   saved the testcase script as "example.fos":
   
   ```ASCompiler example.fos -run foperf::main```
   
   Here is the script output:
   
   ```
   Result:
   string::toFloat: 940 (BEST)
   StrToFloat: 858 (91%)
   ```
   The higher score the better. According to foperf, StrToFloat is slower by *at
   least* 9%.
 
Accuracy
--------

For snippets which take very little time to execute (under 1ms) the test is not
very accurate, because within the macros there is a hidden loop code that is
also measured. For example if you compare following snippet functions:

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

You may be suprised to see that the second snippet is only 5% slower, and the
logic tells us that it should be closer to 50%. That's because the hidden loop
code distorts the results for very small snippets. However, the result is still
useful - it tells which code is the fastest. For code that takes more than 2ms
to execute, the distortion of the result should be neglible.
