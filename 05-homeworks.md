---
layout: page
title: Homeworks
---

* [Homework 0]({{site.baseurl}}/hws/hw0) -- Learning Gem5 (not turned in)
* [Homework 1]({{site.baseurl}}/hws/hw1) -- Gem5 Intro  ([prof's answers]({{site.baseurl}}/hws/hw1-sol))
* Homework 2
* Homework 3

### Late Policy

You may submit up to 1 day late with a 20% penalty.

### Development Machine / Environment

I strongly suggest using Linux for gem5.  Limit your pain to a minimum if possible.

Also, gem5 is absolutely brutal to the compiler ... requiring >8 GB RAM to compile
the x86 version.  I believe the main issue is code size coming from the
instruction decoder (think thousand-item switch case).   

*I originally suggested using the lnxsrv machines.  lnxsrv10 has
all the packages we need, I think, but we currently don't have enough disk allocation
to use it, so for now the only other option is below.*

If you don't have another
fast machine with a large RAM that you like to use, then you are welcome to use
my 24-core server ```tetracosa.cs.ucla.edu```.  This machine can only be accessed
from campus (though you could ssh through ```lion.cs.ucla.edu``` for remote access).
It now has scons and should work for compiling gem5.

To get an account, follow these instructions:

```
ssh account@tetracosa.cs.ucla.edu 
account@tetracosa.cs.ucla.edu's password: 
snow,snow,hail #give the previous line as password, then answer the following: 
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
