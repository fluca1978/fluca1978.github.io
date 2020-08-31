---
layout: post
title:  "Perl Weekly Challenge 76: my last PWC (for a while)"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 76: my last PWC (for a while)

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 76](https://perlweeklychallenge.org/blog/perl-weekly-challenge-076/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



# My last PWC (for a while)

On the very next monday, September 7th, I will be in hospital for a new surgery on my (last) working right eye.
In particular, I will be doing a [trabeculectomy](https://en.wikipedia.org/wiki/Trabeculectomy){:target="_blank"}, and I'm pretty sure I will not be able to see anything for a couple of weeks, may be more.
<br/>
I hope my brain is not going to fail in the meantime!

<a name="task1"></a>
## PWC 76 - Task 1

The first task was about finding the minimum sequence of prime numbers (excluding 1) that provides a sum exactly equal to the script argument. My solution involved the following steps:
1) generate all the primes (excluding 1) up to the specified `$N` value, because values greater than `$N` cannot be summed to provide `$N` itself;
2) use a `@sums` array to store all the sequences that provide the right sum, and start with at least two numbers to sum (because a single number cannot represent a sum), so that I'm going to iterate on the minimum set of numbers up to the maximum;
3) execute a `.permutations` of all the `@primes` numbers, so to get all possible combinations of sequences of numbers;
4) apply `.rotor` on every permutation, so to get groups of numbers with the `$how-many` number of elements;
5) compute the sum (with the reduction `[+]` operator) and store it into the array of `@sums` if that sequence has not been seen before. Since every sequence is sorted, things like `2 + 7` and `7 + 2` are naturally mutually exclusive and equivalent;
6) the last and easiest part is about printing the result.

The code therefore looks like the following one:

<br/>
<br/>
```raku
sub MAIN( Int:D $N where { $N > 1 } ) {

    # get primes excluding 1
    my @primes = ( 1 ^..^ $N ).grep( *.is-prime ).sort;

    my @sums;
    my $how-many = 1;
    while ( @sums.elems == 0 ) {
        $how-many++; # start with summing two numbers
        for @primes.permutations -> @checking {
            for @checking.rotor( $how-many )  {
                my @current-numbers = $_.sort;
                my $sum = [+] @current-numbers;
                @sums.push: @current-numbers if ( $sum == $N && ! @sums.grep( * ~~ @current-numbers ) );
            }
        }
    }


    # print the result
    "$N minimum sum is made by: ".say;
    .join( ' + ' ).say for @sums ;
}

```
<br/>
<br/>


The final result is something like:

<br/>
<br/>
```shell
% raku ch-1.p6 9
9 minimum sum is made by: 
2 + 7

```
<br/>
<br/>


<a name="task2"></a>
## PWC 76 - Task 2

The second task was a lot harder: given a grid of letters, you need to search for words in horizontal, vertical and diagonal paths both forward or backward.
After a first attempt, I decided to start from the grid, decompose it into an array of array (so every line of the grid was an array, that in turn was an array of letters), transforming every letter lowercase.
<br/>
Now, the array of array needs to be composed into an array of strings to, in turn, apply regular expressions against.
Horizontal lines are simple: I can join letters and add the result and its `flip` reverse string to an `@horizontals` array of strings.
Vertical lines are simple if you remember that the `[Z]` zip operator can compose lists of lists.
Diagonals are a lot harder, at least to me, so I decided to implement them as a function that can move up to down, left to right or viceversa:


<br/>
<br/>
```raku
sub diagonal-words( @grid-chars,  $up-to-down = True, $left-to-right = True ) {
    my @diagonals;
    my ( $row, $column ) = $up-to-down ?? 0 !! @grid-chars.elems - 1,
                           $left-to-right ?? 0 !! @grid-chars[ 0 ].elems - 1;

    my ( $row-increment, $column-increment ) = $up-to-down    ?? 1 !! -1,
                                               $left-to-right ?? 1 !! -1;

    my ( $last-row, $last-column ) = $row, $column;
    my @word;
    while ( $last-row ~~ 0 ..^ @grid-chars.elems
            && $last-column ~~ 0 ..^ @grid-chars[0].elems ) {

        ( $last-row, $last-column ) = $row, $column;
        while ( $last-column ~~ 0 ..^ @grid-chars[0].elems ) {
            @word = ();
            while ( $row ~~ 0 ..^ @grid-chars.elems
                    && $column ~~ 0 ..^ @grid-chars[0].elems ) {
                @word.push: @grid-chars[ $row ][ $column ];
                $row += $row-increment;
                $column += $column-increment;
            }

            @diagonals.push: @word.join, @word.join.flip;
            $last-column += $column-increment;
            ($row, $column) = $last-row, $last-column;
        }

    }

    @diagonals.grep( *.chars > 2 );

}
```
<br/>
<br/>

Given the `@grid-chars` array of array, the function moves one row at a time, on a diagonal depending on the direction. There is some index machinery, but nothing particularly hard to implement, rather pretty error prone.
Having the list of strings in all directions, it is possible to apply a regular expression and see what words can be found. The final code of `MAIN` is therefore:


<br/>
<br/>
```raku
sub MAIN( $grid-file-name = 'grid.txt',
          $word-file-name = '/usr/share/dict/words',
          $min-length = 3 ) {
    say "Searching words from $word-file-name into grid $grid-file-name";
    my @found-words;


    # get all the lines in the grid lowercase
    my @grid-chars = $grid-file-name.IO.lines.map( *.lc.split( /\s/, :skip-empty ).Array ).Array;
    say @grid-chars;

    my ( @horizontals, @verticals, @diagonals );
    for @grid-chars {
        @horizontals.push: .join, .join.flip;
    }

    for ( [Z] @grid-chars ) {
        @verticals.push: .join, join.flip;
    }


    @diagonals.push: diagonal-words( @grid-chars, True, True );
    @diagonals.push: diagonal-words( @grid-chars, True, False );
    @diagonals.push: diagonal-words( @grid-chars, False, True );
    @diagonals.push: diagonal-words( @grid-chars, False, False );


    for $word-file-name.IO.lines  {
        next if .chars < $min-length;
        my $current-word = $_.lc;
        @found-words.push: $current-word if ( @diagonals.grep( * ~~ / $current-word / )
                                              || @horizontals.grep( * ~~ / $current-word / )
                                              || @verticals.grep( * ~~ / $current-word / ) );
    }

    say "Found { @found-words.elems }  words: { @found-words.join( ',' ) }";
}

```
<br/>
<br/>

The script takes several minutes to execute using a minimum word length of `3`. The end result is something like:


<br/>
<br/>
```shell
% raku ch-2.p6  
Searching words from /usr/share/dict/words into grid grid.txt

Found 296  words: aol,ada,ali,ara,aral,art,ashe,ats,aug,ben,blu,constitution,coy,dec,dee,dis,dot,ebro,eco,eli,eve,fdr,gaul,gil,gila,goa,gus,han,hay,hays,ing,iso,ian,ila,ines,ira,ito,lara,las,lea,lee,len,lie,liz,los,lot,mali,mel,mia,mir,nsa,nan,oct,ola,ora,pat,patna,rca,rae,rose,saab,sal,sara,set,sid,tao,ted,tia,tide,tod,togo,tom,traci,tracie,tut,ute,uzi,visa,wifi,ace,act,aim,aimed,air,align,all,ani,ant,ante,antes,any,are,arm,arose,art,ash,ashed,asp,ass,aye,baa,baas,ban,bans,bid,bide,blunt,blunts,broad,buff,bur,buries,but,cad,cod,cold,con,cons,constitution,constitutions,cot,coy,croon,cub,cube,cue,cues,depart,departed,die,died,dim,dis,dot,doth,dud,dun,duo,dust,ear,edit,eel,egg,emo,ems,enter,era,euro,eve,eves,fed,fey,fie,filch,for,fore,gal,gals,garlic,gin,goat,goats,goo,got,gram,grieve,grieves,grit,gym,gyms,has,hat,hay,hays,hazard,heed,hem,hie,hit,hod,ios,ice,ices,ids,ion,ions,its,lag,lam,lath,lea,lee,lie,lien,liens,lose,lot,luau,lug,malign,malignant,mall,malls,meh,mes,mid,midst,mod,moo,nab,nay,nit,not,oat,oats,ode,ohm,ohms,old,orb,orc,ore,ought,out,ova,ovary,par,part,parted,pas,pat,pee,pudgiest,qua,quash,quashed,rag,ram,rare,rim,road,roe,rose,rub,rue,rug,run,ruse,ruses,sac,sad,say,see,set,sets,she,shed,shrine,shrines,sic,sin,slag,slug,sob,social,socializing,sol,sot,sow,sows,soy,soya,spa,spas,spasm,spasmodic,succor,succors,tall,tap,tat,tea,teas,tee,tees,the,theorem,theorems,tic,tide,tin,tit,tits,tog,tom,tome,ton,too,tsar,tub,ugh,use,uses,vary,vie,virus,viruses,visa,vow,wig,wigged,yon,yup,zing

```

### Small Optimizations

There are a few little considerations to make about the above script:
1) I suppose that each line in the grid has the very same length, so that testing the first row for its length is fine for all the other lines;
2) it is possible to skip all names and words with a single tick in the dictionary with the following addition into the main loop:

<br/>
```raku
    for $word-file-name.IO.lines  {
        next if .chars < $min-length;
        next if / \' / ;
        next if / ^ <[A .. Z]> <[a .. z]>+ $ /;
        ...
```
<br/>

Moreover, I suppose that the `$min-length` is by default set to `3` meaning that all words less than three characters long are not considered at all, and this can speed up a little the comparison.
