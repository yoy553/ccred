# ccred

This ccred_coq8.6.patch is to add an "incomplete" tactc ccred to Coq-8.6. ccred attempts to run a syntactic reduction, case-of-case, mentioned by Compiling without conitinuation (Luke Maurer, Paul Downen, Zena M. Ariola, and Simon Peyton Jones, Submitted 2016/11/18). https://www.microsoft.com/en-us/research/wp-content/uploads/2016/11/join-points.pdf

Download it to the Coq8.6 directory and type  
> patch -p0 -i ./ccred_coq8.6.patch 
