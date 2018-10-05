---
layout: page
title: Homeworks
---

Homeworks will be mainly based around gem5.  

* [Homework 0]({{site.baseurl}}/hws/hw0) -- Learning Gem5 (not turned in)
* Homework 1
* Homework 2
* Homework 3
* Homework 4

### Development Machine

gem5 is absolutely brutal to the compiler ... requiring >8 GB RAM to compile
the x86 version.  I believe the main issue is code size coming from the
instruction decoder (think thousand-item switch case).  If you don't have a
fast machine with a large RAM that you like to use, then you are welcome to use
my 24-core server ```tetracosa.cs.ucla.edu```.  This machine can only be accessed
from campus (though you could ssh through ```lion.cs.ucla.edu``` for remote access).

To get an account, follow these instructions:

```
ssh account@tetracosa.cs.ucla.edu 
account@tetracosa.cs.ucla.edu's password: 
rain,rain,rain #give the previous line as password, then answer the following: 
Personal name? (first last): Tess Testa 
Username? (small letters or digits only): tt 
Password? (12 or more characters): 
Repeat Password: 
Email? ttesta@ucla.edu 
Repeat Email: ttesta@ucla.edu 
```

after that you will receive this response: 
```
Account configured. It will be activated within one workday.
``` 

* Note 1: It doesn't have a large hard-drive at the moment, so I may need to upgrade if many students are using it and we run out.

* Note 2: Not that you would use it necessarily, but it also has a Volta GPU. No bitcoin mining please. ; )
