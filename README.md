# ccred

**ccred_coq8.6.patch** is to add an "incomplete" tactic `ccred` to Coq-8.6. `ccred` attempts to run a syntactic reduction with *case-of-case transformation* without eliminating the inductive variable. Case-of-case transformation is
mentioned by **Compiling without continuation** (Luke Maurer, Paul Downen, Zena M. Ariola, and Simon Peyton Jones, Submitted 2016/11/18). https://www.microsoft.com/en-us/research/wp-content/uploads/2016/11/join-points.pdf. It is to transform nested `match`-expressions, for example 

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

Download it to the Coq-8.6 directory and type  
> patch -p0 -i ./ccred_coq8.6.patch 

The patch, in addition to modifies the Coq source, adds a folder **ccred_test**, which includes **cwc.v** and **ccred_error.v** which are the examples where `ccred` works and does not work respectively:

* ./ccred_test
  * _CoqProject
  * mk.sh
  * dbgflg_on.ml4
  * ccrev_error.v
  * cwc.v

where **mk.sh** is to make a Makefile with coq_makefile. At the Coq-8.6 home directory, 

> make clean; ./configure -local; make 

to build. After setting your PATH to **\`pwd\`/bin**, you can check these test files with your emacs and ProofGeneral (or with 'coqc'). **cwc.v** includes the small examples from the above mentioned paper **Compiling without continuation**. To avoid irrelevant conflicts in the library compilation, I use a flag `ccred` to open up the designated reduction branch in **kernel/cClosure.ml**, which is turned on by loading **dbgflg_on.ml4**. **ccred_error.v** is to show an example where `ccred` fails. It fails when Coq runs type-checking at `Qed`, while it successfully completes the proof in the proof interactions. In specific it causes the following error: 

```
Error: In pattern-matching on term "x" the branch for constructor 
"@cons" has type
 "forall (b : bool) (x : list bool),
  (if b then gen0 x else 0) = (if b then gen0 x else 0)" which should be
 "forall (H : bool) (H0 : list bool),
  (if H then gen0 x else 0) = (if H then gen0 H0 else 0)".
```

### Steps 
1. download Coq-8.6 (https://coq.inria.fr/distrib/V8.6/files/coq-8.6.tar.gz), `tar xvfz` it and setup environment needed to compile Coq-8.6
1. download the patch (ccred_coq8.6.patch) to the Coq-8.6 home 
1. run `patch -p0 -i ccred_coq8.6.patch` in the Coq-8.6 home
1. compile the Coq (make clean; ./configure -local; make)
1. set 'pwd'/bin to your PATH 
1. under **ccred_test** directory, run **mk.sh** to create the Makefile 
1. `make` it (there may be three identical warnings that *findlib: [WARNING] cannot read directory /dev/null: Not a directory*)
1. open cwc.v with emacs with ProofGeneral to see `ccred` works (or check a successful compilation with `coqc cwc.v`) 
1. open ccred_error.v with emacs with ProofGeneral (or by `coqc ccred_error.v`) to observe the above error (in the second type, somehow ‘H0’ has been replaced with ‘x’ in the left hand side of the equation, as if the De Bruijn index has gotten confused).
