
## Themes
* [avit](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)

## Plugins
* [z](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/z)

NAME

    z - jump around

SYNOPSIS

    z [-chlrtx] [regex1 regex2 ... regexn]

OPTIONS

    -c     restrict matches to subdirectories of the current directory

    -h     show a brief help message

    -l     list only

    -r     match by rank only

    -t     match by recent access only

    -x     remove the current directory from the datafile

EXAMPLES

    z foo         cd to most frecent dir matching foo

    z foo bar     cd to most frecent dir matching foo, then bar

    z -r foo      cd to highest ranked dir matching foo

    z -t foo      cd to most recently accessed dir matching foo

    z -l foo      list all dirs matching foo (by frecency)
