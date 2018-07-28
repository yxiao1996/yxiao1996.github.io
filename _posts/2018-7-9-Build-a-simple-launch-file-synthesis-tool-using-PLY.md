# Build a simple launch file synthesis tool for ROS using PLY(python-lex-yacc)

During summer vacation 2018, I tried to read some materials about compilers, namely *Introduction to Automata Theory* and *Compilers Principles, Techniques & Tools*. I thought about what kind of thing I can do in order to consolidate the knowledge. During the Robocon project in the last 8 monthes, I found maintaining launch files of ROS based softwares tedious. So, I decided to write a little tool automatically synthesis the roslaunch file(in .xml form)

# Use lex to build a scanner

It's a nightmare for compilers to directly extract useful and hierachial information from strings. Instead, compilers let scanners to do the dirty job. Scanners are usually the first machine in the whole compiler's pipeline, hey are also called lexical analyser. What scanners do is taking a string of characters, either entered through your keyboard or read from a file, and extract tokens from the string. That is to say, the scanner read an array of charatcers, then turn them into an array of tokens. So what is a token? I'm quite sure you are familiar with tokens if you've ever written some code. For example, a simple line of C code:

    var_x = var_x + y

havs 5 tokens in it: 'var_x','EQULS','var_x','PLUS','y'. They are the lowest-level entities that compilers going to manipulate, like atoms in a computer program. By extract these symbols from the string, the compiler get rid of the inconvenience brought by the 1-dimensional input code. in the following stages, parser transform symbols into a tree structre which is proper for interpretation. 

In general, there are many ways to build a scanner(or lexical analyser). You can manully code one, like the early day hackers, or you can automatically generate one with some kind of tool. The most famous one in these tools is **lex**, which is used since the 1970s to build tools in the Unix operating system, typically combined with another tool called **yacc**. The configuration code for lex typically contains three parts:
* Tokens (tokens you want to preserved for the compiler's usage, like commands, operators, etc.)
* regular expressions (how do you find these tokens in string)
* additional C code for each regular expression (what do you want the compiler to do after finding each kind of token)

In Python, lex module in PLY(python-lex-yacc) provide the same functionality as lex, with highly similar configuration rules. 

Although PLY is a Python module, it is a bit different to other packages when using. When using classes in most of other Python packages, like numpy, OpenCV, or panda, etc. You do initializations through the __init__() functions built in the class. However, PLY does not applied this convention, but mimic what you have to do when using lex. So, when using PLY, you usually have to set up a configuration including several variables, and probably many functions in your python environment. Then you can call the lex funcion from PLY, and PLY will give you a scanner. 

Now, let me briefly scatch out a piece of code using PLY to generate a scanner:

    from ply import lex
    
    tokens = [TOKEN_0, TOKEN_1, ...]

    def t_TOKEN_0(t):
        r'(regular expression of TOKEN_0)'
        # code about TOKEN_0
        ...

    def t_TOKEN_1(t):
        r'(regular expression of TOKEN_1)'
        # code about TOKEN_1
        ...
    
    ...

    lex

Needless to mention, we have to import the module first, with:

    from ply import lex

Next, we have to set all the tokens' name in a list called **tokens**, may be we want a 'PLUS' operator, an 'EQUALS' operator, or some other kinds of symbols. Here, we put their names in a list:

    tokens = [TOKEN_0, TOKEN_1, ...]

After assigning names for each token, we have to define their behavior. To define this, we have to provide two parts of information for each token. We have to answer two questions for each token: How can we find the token in string? What do you want to do after finding the token? To do this, we define one function for each token. For example:

    def t_TOKEN_0(t):
        r'(regular expression of TOKEN_0)'
        # code about TOKEN_0

Each of these functions contains two part, first part is a regular expression, second part is the additional code. Scanner use the regular expression to find tokens in string, then run the additional code. 

## Use yacc to build a parser

Obviously, only picking tokens from strings is not sufficient for building a compiler. You have to further extract structural information from the source tokens arrays. A parser(or syntax analyser) is a component in the compiler pipeline, which take token array as input, and reform the 1 dimensional data in a more structural way, according to the pre-defined syntax.

One famous tool for automatically generating parser is called YACC(Yet Another Compiler Compiler). In a similar way as lex, PLY mimic yacc configuration in Python.

Let's continue the 'PLUS' operator's example:

    var_x = var_x + y

Using scanner generated by lex, we can obtain the following token array:

    (ID, 'var_x'), (EQUALS), (ID, 'var_x'), (PLUS), (ID, 'var_y')

To interpret these tokens, we have to define some kind of syntax to build a tree structrual take each of these tokens as leafs. There maybe infinitly many of those syntaxes, but I try to adopt mathematical conventions. As the following context-free grammers.

    variable : ID EUALS variable
             | ID PULS variable
             | ID