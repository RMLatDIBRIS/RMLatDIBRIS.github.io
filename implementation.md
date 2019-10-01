---
layout: default
title: Implementation
show_downloads: false
---

# Implementation

RML [Fra19] compiles down to an operational, low-level formalism, named
(parametric) trace expressions [AFM16,AFM17,AFFM17]. Trace expressions and their predecessors, [global types](https://ieeexplore.ieee.org/document/8187184?arnumber=8187184) (behavioral types for verification of interaction protocols), have been used to model and verify, among the others (see the [selected publications list](biblio.md)),  multiagent systems [ADM13,BMA15] and IoT systems [ABFM15b,AFDL+17,LAFO+18].

An optimized implementation of trace expressions has been
developed in [SWI-Prolog](https://www.swi-prolog.org/), thanks to its native support for
cyclic terms and coinductive logic programming [SMBG06].

The current RML compiler uses [ANTLR](https://www.antlr.org/), a powerful and flexible parser generator targeting Java. 

The Prolog
translation is implemented in [Kotlin](https://kotlinlang.org/), which runs on the JVM
and is fully interoperable with the generated Java parser.

A Prolog monitor is available, expecting a specification file
as an argument, which then verifies a sequence of events. Both
an offline monitor (reading from a log file) and on online one
(receiving events through an HTTP interface) exist.
Finally, as an example of an instrumentation layer, a Node.js
static instrumentation tool is available on the repository.



## Bibliography

*[ABFM15b]* Ancona D., Briola D., Ferrando A., Mascardi V. (2015) Runtime verification of fail-uncontrolled and ambient intelligence systems: A uniform approach. Intelligenza Artificiale 9(2): 131-148 [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/journals/ia/AnconaBFM15)

*[ADM13]* Ancona D., Drossopoulou S., Mascardi V. (2013) Automatic Generation of Self-monitoring MASs from Multiparty Global Session Types in Jason. In: Baldoni M., Dennis L., Mascardi V., Vasconcelos W. (eds) Declarative Agent Languages and Technologies X. DALT 2012. Lecture Notes in Computer Science, vol 7784. Springer, Berlin, Heidelberg [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/dalt/AnconaDM12)

*[AFDL+17]* Ancona D., Franceschini L., Delzanno G., Leotta M., Ribaudo M., Ricca F. (2017)
Towards Runtime Monitoring of Node.js and Its Application to the Internet of Things. In Proceedings First Workshop on Architectures, Languages and Paradigms for IoT, ALP4IoT@iFM 2017: 27-42 [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/journals/corr/abs-1802-01790)

*[AFM16]* Ancona D., Ferrando A., Mascardi V. (2016) Comparing Trace Expressions and Linear Temporal Logic for Runtime Verification. In: Ábrahám E., Bonsangue M., Johnsen E. (eds) Theory and Practice of Formal Methods. Lecture Notes in Computer Science, vol 9660. Springer, Cham [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/birthday/AnconaFM16)

*[AFFM17]* Ancona D., Ferrando A., Franceschini L., Mascardi V. (2017) Parametric Trace Expressions for Runtime Verification of Java-Like Programs. In Proceedings of the 19th Workshop on Formal Techniques for Java-like Programs (FTFJP'17). ACM, New York, NY, USA, Article 10, 6 pages. DOI: https://doi.org/10.1145/3103111.3104037 [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/ecoop/AnconaFFM17)

*[AFM17]* Ancona D., Ferrando A., Mascardi V. (2017) Parametric Runtime Verification of Multiagent Systems. In Proceedings of the 16th Conference on Autonomous Agents and MultiAgent Systems (AAMAS '17). International Foundation for Autonomous Agents and Multiagent Systems, Richland, SC, 1457-1459 [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/atal/AnconaFM17)

*[BMA15]* Briola D., Mascardi V., Ancona D. (2015) Distributed Runtime Verification of JADE Multiagent Systems. In: Camacho D., Braubach L., Venticinque S., Badica C. (eds) Intelligent Distributed Computing VIII. Studies in Computational Intelligence, vol 570. Springer, Cham [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/idc/BriolaMA14)

*[Fra19]* Franceschini L. (2019) RML: runtime monitoring language: a system-agnostic DSL for runtime verification. In Proceedings of the Conference Companion of the 3rd International Conference on Art, Science, and Engineering of Programming (Programming '19), Stefan Marr and Walter Cazzola (Eds.). ACM, New York, NY, USA, Article 28, 3 pages. DOI: https://doi.org/10.1145/3328433.3328462 [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/programming/Franceschini19)

*[LAFO+18]* Leotta M., Ancona D., Franceschini L., Olianas D., Ribaudo M., Ricca F. (2018) Towards a Runtime Verification Approach for Internet of Things Systems. In: Pautasso C., Sánchez-Figueroa F., Systä K., Murillo Rodríguez J. (eds) Current Trends in Web Engineering. ICWE 2018. Lecture Notes in Computer Science, vol 11153. Springer, Cham [Bibtex from DBLP](https://dblp.uni-trier.de/rec/bibtex/conf/icwe/LeottaAFORR18)

*[SMBG06]* Simon,  L.,  Mallya,  A.,  Bansal,  A.,  and  Gupta,  G. (2006)
Coinductive logic programming. In Logic Programming (Berlin, Heidelberg, 2006),
S. Etalle and M. Truszczyński, Eds., Springer Berlin Heidelberg, pp. 330-345