[![Build Status](https://travis-ci.org/bloodstalker/cgrep.svg?branch=master)](https://travis-ci.org/bloodstalker/cgrep)
[![Build status](https://ci.appveyor.com/api/projects/status/caab8oxmgljb87te?svg=true)](https://ci.appveyor.com/project/bloodstalker/cgrep)
[![Language grade: C/C++](https://img.shields.io/lgtm/grade/cpp/g/bloodstalker/cgrep.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/bloodstalker/cgrep/context:cpp)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/726c70d0e3294f149036fa6134998fe8)](https://www.codacy.com/manual/bloodstalker/cgrep?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=bloodstalker/cgrep&amp;utm_campaign=Badge_Grade)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fbloodstalker%2Fcgrep.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fbloodstalker%2Fcgrep?ref=badge_shield)
[![Gitter](https://badges.gitter.im/mutatortools/community.svg)](https://gitter.im/mutatortools/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

# cgrep

cgrep is `grep` for C-family source files.

You can write something like this:

```bash
cgrep --regex [a-z]* --func -A 1 -B 1 myawesomecode.cpp
```

and it will match your regex against all function declarations, and will output the result, plus one line before and after the context.

`cgrep` is implemented using Clang's libtooling libraries.

## Features

* It's basically Clang regexing it's way through your C-family source-code. You have all the context you can ever need.
* Can output whether to print the declaration of a match even if the match itself is not a declaration along with the matched result.
* Can output matches in a script-friendly format which could be used in turn by a secondary script.

### Will cgrep try to implement all of the grep switches?
The answer is no. The main distinction is that `cgrep` is only meant to work on C-family source files not text files. Most of `grep`'s switches don't apply to the usecase or provide almost no benefits at all.<br/>
That being said, I might have missd something so you can always make suggestions in the form of a new issue.

### Will cgrep support a new switch that matches X?
If it makes sense sure, but I want to be careful with what cgrep implements. If everything gets implemented, that is, cgrep implements every possible switch(well, a subset of "all"), we end up with an inferior version of `clang-query` that would be too slow to be of any use to anyone. So please keep in mind that I will have to draw the line somewhere.

## Building

You can get the source files from the [release](https://github.com/bloodstalker/cgrep/releases) page or a release branch usually named `rcx`.<br/>
The Release Candidate branches are the release branches. Master is the dev branch.<br/>
Currently the master branch requires `libloost-filesystem`, which will most probably change in the future.<br/>

### Fedora
`sudo dnf install clang-devel llvm-devel llvm-static`

### Ubuntu
`sudo apt install clang-X llvm-X-dev libclang-common-X-dev libclang-X-dev`<br/>
Replace X with the LLVM version of your choice.

### Debian
`sudo apt install clang-X llvm-X-dev libclang-common-X-dev libclang-X-dev`<br/>
Replace X with the LLVM version of your choice.

### Arch
You can get cgrep from AUR. Thanks to [schra](https://github.com/schra).

### Cygwin
You will need `libclang-devel, libllvm-devel, clang, libiconv-devel`.

Assuming you have the llvm/clang libraries (the build file will read your llvm options using `llvm-config` so make sure it's in path), just run:

```bash
git clone https://github.com/bloodstalker/cgrep
cd cgrep
git submodule init
git submodule update
make
```

After the build is finished you can choose to run `make install`. It will simply symlink cgrep into `/usr/local/bin`.

If you have installed LLVM but don't have `llvm-config`, you are missing the dev package for LLVM.<br/>
`cgrep` supports LLVM 5,6,7,8,9,10 and 11.<br/>
The makefile assumes clang is called `clang` and llvm-config is called `llvm-config`. On some distros, the names might not be the same. In those cases use `CXX` and `LLVM_CONF` to pass the values to the makefile like so:
```bash
make CXX=clang-9 LLVM_CONF=llvm-config-9
```

For windows builds, cygwin builds are supported. Get llvm and clang along with their sources and build like usual. If you run into problems while bulding on cygwin, you can take a look at the `appveyor.yml` file under the repository root.

## Usage

A simple usage example:

```bash
cgrep -A 1 -B 1 --func --declrefexpr --regex n[aA]m --nocolor --nodecl ./myawesomecode.cpp
```

In order for cgrep to work, you need to have a compilation database, tools like `cmake` can generate one for you.<br/>
You can, by all means, run cgrep without a compilation database but whether that works or not really depends on your source file. Can you build your source file with clang without passing it any options?
If the answer to that is yes, then you can just run cgrep without a compilation databse like so:<br/>
```bash
cgrep -A 1 -B 1 --func --declrefexpr --regex n[aA]m --nocolor --nodecl ./myawesomecode.cpp --
```
the `--` at the end is an explicit way of saying that you will not be providing a compialtion database. Newer versions of clang will try to still go through with the compilation even if there is no compilaiton database found.
Otherwise you need a compilation database.<br/>

Please do note that the regex will pass through both C++ and the regex engine, so if you would want to escape `\`, the regex you pass as the command line arg would be `\\\\` instead of the normal `\\`.<br/>
If your build tool doesn't do that, you can just use [bear](https://github.com/rizsotto/Bear) or [scan-build](https://github.com/rizsotto/scan-build).<br/>
You can also skip the compilation database altogether passing cgrep `--` after the input file name which means you have chosen not to pass it anything.<br/>
You can pass the options by hand using `--extra-arg=` since cgrep is a clang instance so it recognizes every option clang has.
As a general rule, if you're not going to pass cgrep a compilation database, it's always better to explicitly let cgrep know using `--`. Not doing so can result in instances when cgrep behaves in a way that you might not expect it.<br/>

cgrep uses ANSI escape sequences for colors so your terminal should support those. In case your terminal does not support ANSI escape sequences or you don't want thos for any other reason, you can silence those using the `--nocolor` option.

By default, cgrep will print out the declaration location for a match. In case you don't want those in the output, you can pass cgrep the `--nodecl` switch.

You can use `--extra-arg=--std=` to tell cgrep which C-family language the source file is supposed to be in.

## Options

Here's an option list, though it's usually not up-to-date.<br/>
For an up-to-date list, you can run `cgrep --help` or look at the man page.

```bash
  -A=<int>                    - Same as grep, how many lines after the matched line to print. Defaults to 0.
  -B=<int>                    - Same as grep, howm many lines before the matched line to print. Defaults to 0.
  --all                       - Turns on all switches other than nameddecl.
  --awk                       - Outputs location in a gawk freidnly format, not meant for human consumption. Defaults to false.
  --call                      - Match function calls.
  --class                     - Match class declrations.
  --cxxcall                   - Match member function calls.
  --declrefexpr               - Matches declrefexpr.
  --dir=<string>              - recursively goes through all the files and directories. assumes compilation databases are present for all source files.
  --extra-arg=<string>        - Additional argument to append to the compiler command line
  --extra-arg-before=<string> - Additional argument to prepend to the compiler command line
  --func                      - Match functions.
  --header                    - Match headers in header inclusions.
  --macro                     - Match macro definitions.
  --mainfile                  - Match identifiers in the main file only. Defaults to true.
  --memfunc                   - Match member functions.
  --memvar                    - Match member variables.
  --nameddecl                 - Matches all named declrations.
  --nocolor                   - For terminals that don't supprt ANSI escape sequences. The default is to false.
  --nodecl                    - For switches that are not declarations, don't print declarations. Defaults to false.
  -p=<string>                 - Build path
  --regex=<string>            - The regex to match against.
  --struct                    - Match structures.
  --syshdr                    - Match identifiers in system header as well. Defaults to false.
  --union                     - Match unions.
  --var                       - Match variables.
```

`cgrep` is a clang tool, so it will accept all valid clang command line options.

## Known Issues
`cgrep`, replaces the clang diagnosticConsumer with a simple one that only tells you there are erros during the compilation. You can get the normal clang output using the `--clangdiag` switch. The decision was made to declutter the output generated by cgrep.

## License

cgrep is licensed under GPL-3.0. Eveything else is licensed under it's own respective license.


[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fbloodstalker%2Fcgrep.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fbloodstalker%2Fcgrep?ref=badge_large)
