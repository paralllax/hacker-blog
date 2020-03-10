---
title: Bash Basics &#35;1 - for loops
published: true
---

* * *
This is the first of many short guides where we will go over the basics of bash scripting. The series will follow no particular order. I will try to cover as many topics as I can, but if there is something you would like to learn about feel free to send me a suggestion on my socials under the about page. 

## Index

1. [Loop X number of times](#loopx)
2. [Loop over strings](#strings)
3. [Iterate over command results](#command)
4. [Itterate over an array](#array)

* * *
In bash, the `for` command is a built-in loop command. It executes a sequence of commands for each member in a list of items. The logic itself is fairly readable once you understand the concept. Some of the logic is very similar to what you may see in bash or perl. If you are familiar with either it will be even more easy to pick up. The basic syntax is as follows:

```bash
for <variable_name> [in words...]
do
    <commands>
done
```
In one line this would look like:

`for <variable_name> [in words...]; do <commands>; done`

The `variable_name` can be called anything you prefer. Frequently you will see it as `i`. This is primarily due to it's frequent use as a subscript in math, which propogated into counter's in programming, and is now frequently set as a variable. 

The `words` can be as simple as a string or file, or something more complex like iterating over the result of a command.

## Examples

* * *
### Loop X number of times<a name="loopx"><a/>

Here we loop 5 times, and each time we print the value of `i`, the number in [words...], followed by a new line, so that the results don't stack. We do the same thing in all of the below examples. The last example is probably the most C-like. 

```bash
for i in 1 2 3 4 5; do printf "${i}\n"; done
for i in {1...5}; do printf "${i}\n"; done
for i in $(seq 1 5); do printf "${i}\n"; done
for (( i = 0; i < 5; ++i )); do printf "${i}\n"; done
```

Result:

```bash
$ for (( i = 0; i < 5; ++i )); do printf "${i}\n"; done
1
2
3
4
5
```

As an additional note, what do you think happens when we swap the letters for numbers? It loops 5 times, but the `i` is set to a letter instead. 

```bash
$ for i in a b c d e; do printf "${i}\n"; done
a
b
c
d
e
```

* * *
### Loop over strings<a name="strings"><a/>

Here we loop over the 3 strings, `foo`, `bar`, and `baz`, then print the string. 

```bash
$ for string in foo bar baz; do printf "Here is: %s\n" ${string}; done
Here is: foo
Here is: bar
Here is: baz
```

* * *
### Iterate over result of a command<a name="command"><a/>

A useful and frequent useage is iterating over the result of a command. In this scenario we are pulling ip address information for all of the interfaces. Note that there are better ways to obtain this informatin, this is written just as an example. 

A common pitfall would be to use a for loop to iterate over the lines of a file, something like `for line in $(cat file); do...` - you should not do this and instead use a while read loop. `for` will break apart a file and interpret the whitespace it sees as the IFS. While this can be set to a newline, for performance implications it is better to use a while loop. For a more thorough analysis as to why this is, read [this](https://unix.stackexchange.com/a/24278) explanation.

```bash
$ for device in $(ls /sys/devices/virtual/net/); do ip -o -f inet addr show ${device}; done
14: eth0    inet 192.168.1.8/24 brd 192.168.1.255 scope global dynamic \       valid_lft 85289sec preferred_lft 85289sec
13: eth1    inet 169.254.210.3/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
15: eth2    inet 192.168.56.1/24 brd 192.168.56.255 scope global dynamic \       valid_lft forever preferred_lft forever
3: eth3    inet 169.254.27.44/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
1: lo    inet 127.0.0.1/8 brd 127.255.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
4: wifi0    inet 169.254.245.64/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
7: wifi1    inet 169.254.196.102/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
10: wifi2    inet 169.254.2.252/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
```

* * *
### Iterate over an array<a name="array"><a/>

Here we declare an indexed array called `my_fruit`, we load it with our fruit, then iterate through each item. `[@]` is used to expand positional parameters. In other words, it displays everything in our array. The command, more literally translated reads: `for fruit in apple orange banana...`.

```bash
$ my_fruit=(apple orange banana pear grapes)
$ for fruit in ${my_fruit[@]}; do echo "I will eat (an): ${fruit}"; done
I will eat (an): apple
I will eat (an): orange
I will eat (an): banana
I will eat (an): pear
I will eat (an): grapes
```

* * *

I hope you enjoyed the tutorial! If you want more advanced guides let me know. We'll slowly work our way up. I am excited to cover more things like, while loops, case statements, bolleans, if/elif statements, getopts and more. 
