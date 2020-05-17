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

Before we begin you may be curious why I am using `printf` over `echo` in most of the examples. `echo` is not standardized and may behave slightly different depending on the system. There is a lot more to it than that, but I don't want to detract from the topic at hand. If you are curious, a really good explanation on why printf is better can be found [here](https://unix.stackexchange.com/questions/65803/why-is-printf-better-than-echo/65819#65819).

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

A useful and frequent useage is iterating over the result of a command. With these however, you have to be careful. We mention this more in depth later, but spaces or special characters in the result can lead to unexpected results. In most cases, it is better and safer to use a while loop, or globbing, depending on the scenario.


```bash
for number in $(echo 1 2 3 4 5); do printf "Here is ${number}\n"; done

```

As a reader pointed out, In this scenario we are pulling ip address information for all of the interfaces. While you may think the below works fine, we could run into the issue mentioned earlier if there are spaces or special characters in the file names. Additionally, as we are not calling ls in a subshell `$()`, we save execution time due to the reduced system calls.

#### THIS IS BAD


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

#### THIS IS GOOD

```
shopt -s nullglob
$ for device in /sys/devices/virtual/net/*; do ip -o -f inet addr show "${device##*/}"; done
14: eth0    inet 192.168.1.8/24 brd 192.168.1.255 scope global dynamic \       valid_lft 85289sec preferred_lft 85289sec
13: eth1    inet 169.254.210.3/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
15: eth2    inet 192.168.56.1/24 brd 192.168.56.255 scope global dynamic \       valid_lft forever preferred_lft forever
3: eth3    inet 169.254.27.44/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
1: lo    inet 127.0.0.1/8 brd 127.255.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
4: wifi0    inet 169.254.245.64/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
7: wifi1    inet 169.254.196.102/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
10: wifi2    inet 169.254.2.252/16 brd 169.254.255.255 scope global dynamic \       valid_lft forever preferred_lft forever
```

Here is an example to show how spaces in the result can cause issues. A way around it could be to change the IFS to match a unique separator, but if it can be avoided through a while loop or globbing, that would be advised. As you can see, it breaks the line apart into 3 separate variables. [Here](http://mywiki.wooledge.org/ParsingLs) is an excellent resource I was provided with for more insight on that and parsing `ls`.

```bash
$ for word in $(echo "Here is Food"); do printf "${word}\n"; done
Here
is
Food
```

Here is an example with a file with spaces in it demonstrating the same concept.

```bash
$ touch 'file with spaces'
$ for file in $(ls); do printf "${file}\n"; done
file
with
spaces
```

Another common pitfall would be to use a for loop to iterate over the lines of a file, something like `for line in $(cat file); do...` - you should not do this and instead use a while read loop. `for` will break apart a file and interpret the whitespace it sees as the IFS. While this can be set to a newline, for performance implications it is better to use a while loop. For a more thorough analysis as to why this is, read [this](https://unix.stackexchange.com/a/24278) explanation.

* * *
### Iterate over an array<a name="array"><a/>

Here we declare an indexed array called `my_fruit`, we load it with our fruit, then iterate through each item. `[@]` is used to expand positional parameters. In other words, it displays everything in our array. The command, more literally translated reads: `for fruit in apple orange banana...`.

As a reader pointed out, you should get in the habbit of quoting your variables. Particularly if your variable contains a space, special characters, or is empty. This can otherwise lead to all sorts of issues, such as breaking the variables apart, or matching more items than you intially wanted (wildcards). If you are worried about your syntax there are great tools such as [shellcheck](https://github.com/koalaman/shellcheck). 

```bash
$ my_fruit=(apple orange banana pear grapes)
$ for fruit in "${my_fruit[@]}"; do printf "I will eat (an): ${fruit}\n"; done
I will eat (an): apple
I will eat (an): orange
I will eat (an): banana
I will eat (an): pear
I will eat (an): grapes
```

Another neat trick is iterating over the number of items in an array, this will follow the syntax from one of our earlier loops. It works by pulling the index number of the array instead of expanding it then iterating over the values.

```bash
$ for (( i=0; i<${#my_fruit[@]}; i++ )); do printf "We are on item # ${i} -- ${my_fruit[i]}\n"; done
We are on item # 0 -- apple
We are on item # 1 -- orange
We are on item # 2 -- banana
We are on item # 3 -- pear
We are on item # 4 -- grapes
```

* * *

I hope you enjoyed the tutorial! If you want more advanced guides let me know. We'll slowly work our way up. I am excited to cover more things like, while loops, case statements, bolleans, if/elif statements, getopts and more. 
