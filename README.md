# ccred

**ccred_coq8.6.patch** is to add an "incomplete" tactc `ccred` to Coq-8.6. `ccred` attempts to run a syntactic reduction with *case-of-case trasformation* without eliminating the inductive variable. Case-of-case trasformation is
mentioned by **Compiling without conitinuation** (Luke Maurer, Paul Downen, Zena M. Ariola, and Simon Peyton Jones, Submitted 2016/11/18). https://www.microsoft.com/en-us/research/wp-content/uploads/2016/11/join-points.pdf. It is to transform nested `match`-expressions, for example 

```
match (match aa with
        | [] => None
        | p::_ => Some p
       end) with
 | None => true
 | Some _ => false
end
```

into 

```
match aa with
 | [] => match None with
          | None => true
          | Some _ => false
         end
 | _::_ => match Some _ with
            | None => true
            | Some _ => false
           end
end
```

which is reduced to 

```
match aa with
 | [] => true
 | _::_ => false
end
```
by iota-reductions. 

## How to use it

Download it to the Coq8.6 directory and type  
> patch -p0 -i ./ccred_coq8.6.patch 

The patch, in addition to modifies the Coq source, adds a folder **ccred_test**, which includes **cwc.v** and **ccred_error.v** which are the examples where `ccred` works and does not work respectively:

* ./ccred_test
  * _CoqProject
  * mk.sh
  * dbgflg_on.ml4
  * ccrev_error.v
  * cwc.v

where **mk.sh** is to make a Makefile with coq_makefile. At the Coq8.6 home directory, 

> make clean; ./configure -local; make 

to build. After settig your PATH to **./bin**, you can check these test files with your emacs and ProofGeneral. **cwc.v** includes the small examples from the above mentioned paper **Compiling without conitinuation**. To avoid irrelevant conflicts in the library compilation, I use a flag `ccred` to open up the designated reduction branch in **kernel/cClosure.ml**, which is turned on by loading **dbgflg_on.ml4**. **ccrev_error.v** is to show an example where `ccred` fails. It fails when Coq runs type-checking at `Qed`, while it succesfully completes the proof in the proof interactions. In specifc it casuse the following error: 

```
Error: In pattern-matching on term "x" the branch for constructor 
"@cons" has type
 "forall (b : bool) (x : list bool),
  (if b then gen0 x else 0) = (if b then gen0 x else 0)" which should be
 "forall (H : bool) (H0 : list bool),
  (if H then gen0 x else 0) = (if H then gen0 H0 else 0)".
```

### Steps 
1. download Coq-8.6 (https://coq.inria.fr/distrib/V8.6/files/coq-8.6.tar.gz) and `tar xvfz` it
1. download the patch (ccred_coq8.6.patch) to the Coq-8.6 home 
1. run `patch -p0 -i ccred_coq8.6.patch` in the Coq-8.6 home
1. compile the Coq (make clean; ./configure -local; make)
1. set ./bin to your PATH 
1. under **ccred_test** directory, run **mk.sh** to create the Makefile 
1. `make` it
1. open cwc.v with emacs with ProofGeneral to see `ccred` works (or check a successful compilation with `coqc cwc.v`) 
1. open ccred_error.v with emacs with ProofGeneral (or by `coqc ccred_error.v`) to observe the above error
