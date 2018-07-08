# Build a simple launch file synthesis tool for ROS using PLY(python-lex-yacc)

During summer vacation 2018, I tried to read some materials about compilers, namely *Introduction to Automata Theory* and *Compilers Principles, Techniques & Tools*. I thought about what kind of thing I can do in order to consolidate the knowledge. During the Robocon project in the last 8 monthes, I found maintaining launch files of ROS based softwares tedious. So, I decided to write a little tool automatically synthesis the roslaunch file(in .xml form)

# Use lex to build a scanner

It's a nightmare for compilers to directly extract useful and hierachial information from strings. Instead, compilers let scanners to do the dirty job. Scanners are usually the first machine in the whole compiler's pipeline, hey are also called lexical analyser. What scanners do is taking a string of characters, either entered through your keyboard or read from a file, and extract tokens from the string. That is to say, the scanner read an array of charatcers, then turn them into an array of tokens. So what is a token? I'm quite sure you are familiar with tokens if you've ever written some code. For example, a simple line of C code:

    var_x = var_x + y

havs 5 tokens in it: 'var_x','=','var_x','+','y'. They are the lowest-level entities that compilers going to manipulate, like atoms in a computer program. By extract these symbols from the string, the compiler get rid of the inconvenience brought by the 1-dimensional input code. in the following stages, parser transform symbols into a tree structre which is proper for interpretation. 

In general, there are many ways to build a scanner(or lexical analyser). You can manully code one, like the early day hackers, or you can automatically generate one with some kind of tool. The most famous one in these tools is **lex**, which is used since the 1970s to build tools in the Unix operating system, typically combined with another tool called **yacc**. The configuration code for lex typically contains three parts:
* Tokens (tokens you want to preserved for the compiler's usage, like commands, operators, etc.)
* regular expressions (how do you find these tokens in string)
* additional C code for each regular expression (what do you want the compiler to do after finding each kind of token)

In Python, lex module in PLY(python-lex-yacc) provide the same functionality as lex, with highly similar configuration rules. 