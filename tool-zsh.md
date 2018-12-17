
## Themes
* [avit](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)
* [powerlever9k](https://github.com/bhilburn/powerlevel9k)

## Fonts
* [nerd-fonts](https://github.com/ryanoasis/nerd-fonts)

## Plugins
* [z](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/z)
* [extract](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/extract)
* [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
* [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

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

### Blogs
- [zsh oh-my-zsh 插件推荐](https://hufangyun.com/2017/zsh-plugin/)
- [Ubuntu 下 Oh My Zsh 的最佳实践「安装及配置」](https://segmentfault.com/a/1190000015283092)
