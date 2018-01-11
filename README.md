# Microsoft.Practices.EnterpriseLibrary.Common.Standard
**Microsoft.Practices.EnterpriseLibrary.Common updated for 2018.**

[![rolosoft_public_packages MyGet Build Status](https://www.myget.org/BuildSource/Badge/rolosoft_public_packages?identifier=76485b11-81e3-46e9-a438-e96e7a1e770a)](https://www.myget.org/)

## About
Microsoft Patterns and Practices (p&p) Enterprise Library came to an unfortunate end (i.e. it was dropped by Microsoft).

Because my software company still heavily uses some of the p&p blocks, I've decided to update some of the old Microsoft packages to work with .Net Standard 2.0.

## Installation
~~~
install-package Rsft.EntLib.Common.Standard
~~~

## Caveats
This update has only recently been feasible with Microsoft releasing .Net Standard 2.x. Even with .Net standard 2.x, there are some missing API's that were in the old .Net framework 4.x.

With the missing API's I had to remove some of the class libraries. However, much of the original core functionality is available.

Where unit test are compatible, they are also included.

## Features
### Compatibility
* .Net4.6.1 or higher
* .Net Standard 2.0 or higher
* .Net Core 2.0 or higher

## Documentation
Please see the original [Microsoft Guidance](https://msdn.microsoft.com/en-us/library/ff648951.aspx).
