# Dash

 is one of the least known names when you think about shell family. But [Dash](https://wiki.archlinux.org/title/Dash?ref=linuxhandbook.com) is not meant to replace your current shell and works under the hood.

You must have many questions related to Dash such as what is its use case, how it's different from your regular shell, and so on. So let's dive deep into Dash.

## What is a Dash shell?

Dash stands for Debian Almquist Shell. It is a POSIX-compliant implementation of Bourne Shell. It replaces the /bin/sh in default scripts and provides better execution speed while consuming fewer resources.

If you are using a Debian-based distro, you are already using Dash by default. You'd say, Bash, is my default shell. You're right about your default shell so let me explain this contradictory statement.

Before the release of Ubuntu 6.10, Bash was used when your default scripts wanted to execute /bin/sh as a [symbolic link](https://linuxhandbook.com/symbolic-link-linux/) to provoke bash.

As bash grew bigger over time, the efficiency was compromised and Dash was implemented to replace /bin/sh in default scripts.

Dash (**Debian Almquist Shell**) is less complex and lite than bash. Dash is not meant for interactive sessions and works under the hood for much better efficiency.

Now, let's have a look at the advantages of Dash:

- You get significant performance improvements over bash
- It uses less disk space compared to other shells which is important for wrapper scripts intended for cleanups when the underlying problem exists.
- Dash only relies on libc (the core system library) whereas bash needs to have terminal support libraries and without them, you can't even run a script. This means, Dash can work quite better on broken systems!

### Performance Comparison between Dash and Bash

As I mentioned above, the dash is meant to be efficient but what are the exact numbers or how many times it is faster can only be derived from the test.

So I'll be doing a comparison between bash and dash so you can have a better idea about its performance.

#### Testing Start-up times

I'm going to use a simple shell script that will track the exact time to open the shell 1000 times without any operation performed.

So let's start with bash.

```none
#!/bin/bash
for i in $(seq 1 1000);
do bash -c ":" ;
done
```

Copy

To track time, I used the time utility. It gave me results as given below:

![Running benchmark test for bash](https://linuxhandbook.com/content/images/2022/07/Running-bash-start-up-test.png)Running benchmark test for bash

The used script will invoke bash to run a shell with no operation 1000 times and it took around **3 seconds.**

Now, let's do the same for the dash. With the few changes over the same script as above and it'll be ready for being tested for the dash.

```dash
#!/bin/dash
for i in $(seq 1 1000);
do dash -c ":" ;
done
```

When I executed the above script, it gave me the following result:

![Running benchmark test for bash](https://linuxhandbook.com/content/images/2022/07/Running-dash-start-up-test.png)Running benchmark test for dash

The same script when executed with the dash, it only took **1.1 seconds** which is less than half of what it took with bash.

#### Testing Performance Using ShellBench Script

[ShellBench](https://github.com/shellspec/shellbench?ref=linuxhandbook.com) is a benchmark utility for POSIX shells and avails you of various tests through which you can test shells on different parameters.

ShellBench runs a given set of commands in an infinity loop for 1-2 seconds and then it returns numbers of executions/second.

As I'm only testing between dash and bash, my command will be as follows:  

```
./shellbench -s bash,dash sample/*
```

![Using ShellBench to compare dash and bash](https://linuxhandbook.com/content/images/2022/07/Running-ShellBench-1.png)

Using ShellBench to compare dash and bash

As you can clearly see, dash is far superior in terms of performance.

## Final Words

Dash outstands bash in terms of performance but can not be used as it is not made for interaction. Ubuntu still uses bash as a [login shell](https://linuxhandbook.com/login-shell/) and relies heavily upon it as dash still lacks some features required to replace bash completely.