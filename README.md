# progrium/bashstyle

Bash is like the JavaScript of systems programming. Although in some cases it's better to use a systems language like C or Go, Bash is actually an ideal systems language for many smaller POSIX-oriented or command line tasks. Here's three quick reasons why:

 * It's everywhere. Like JavaScript for the web, Bash is already there ready for systems programming.
 * It's neutral. Unlike Ruby, Python, JavaScript, or PHP, Bash offends equally across all communities. ;)
 * It's made to be glue. Write complex parts in C or Go (or whatever!), and glue them together with Bash.

The same way people hated JavaScript before people got serious about it, Bash just needs to be used in a way that makes it feel like a real programming language. Part of this is establishing a consistent style and best practices.

This document is how I write Bash and how I'd like collaborators to write Bash with me in my open source projects. It's based on a lot of experience and time collecting best practices. Most of them come from these [two](http://wiki.bash-hackers.org/scripting/obsolete) [articles](http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/), but here integrated, slightly modified, and focusing on the most bang for buck items. Plus some new stuff!

## Big Rules

 * Always double quote variables, including subshells. No naked `$` signs
   * This rule gets you pretty far. Read http://mywiki.wooledge.org/Quotes for details
 * All code goes in a function. Even if it's one function, `main`. 
   * Unless a library script, you can do global script settings and call `main`. That's it.
   * Avoid global variables. Though when defining constants use `readonly`
 * Always have a `main` function for runnable scripts, called with `main` or `main "$@"`
   * If script is also usable as library, call it using `[[ "$0" == "$BASH_SOURCE" ]] && main "$@"`
 * Always use `local` when setting variables, unless there is reason to use `declare`
   * Exception being rare cases when you are intentionally setting a variable in an outer scope.
 * Always use `set -eo pipefail`. Fail fast and be aware of exit codes. 
   * Use `|| true` on programs that you intentionally let exit non-zero.
 * Never use deprecated style. Most notably:
   * Define functions as `myfunc() { ... }`, not `function myfunc { ... }`
   * Always use `[[` instead of `[` or `test`
   * Never use backticks, use `$( ... )`
   * See http://wiki.bash-hackers.org/scripting/obsolete for more
 * Always use `declare` and name variable arguments at the top of functions that are more than 2-lines
   * Example: `declare arg1="$1" arg2="$2"`. You'll write/see this a lot now.
   * The exception is when defining variadic functions. See below.

If you know what you're doing, you can bend or break some of these rules, but generally they will be right and be extremely helpful.

## Best Practices and Tips

 * Use Bash variable substitution if possible before awk/sed
 * Generally use double quotes unless it makes more sense to use single quotes
 * For simple conditionals, try using `&&` and `||`.
 * Put `then`, `do`, etc on same line, not newline
 * Use `.sh` or `.bash` extension if file is meant to be included/sourced. Never on executable script.
 * Put complex one-liners of `sed`, `perl`, etc in a standalone function with a descriptive name
 * Good idea to include `[[ "$TRACE" ]] && set -x`
 * Avoid flag arguments and parsing, instead use optional environment instead
 * In large systems or for any CLI commands, add a description to functions 
   * Use `declare desc="description"` at the top of functions, even above argument declaration
   * This can be queried/extracted with a simple function using reflection
 * No hard tabs?
 
## Good References and Help

 * http://wiki.bash-hackers.org/scripting/start
   * Especially http://wiki.bash-hackers.org/scripting/newbie_traps
 * http://tldp.org/LDP/abs/html/
 * Tips for interactive Bash: http://samrowe.com/wordpress/advancing-in-the-bash-shell/

## Examples

### Regular function with named arguments
Defining functions with arguments
```
regular_func() {
	declare arg1="$1" arg2="$2" arg3="$3"

	# ...
}
```

### Variadic functions
Defining functions with a final variadic argument
```
variadic_func() {
	local arg1="$1"; shift
	local arg2="$1"; shift
	local rest="$@"

	# ...
}
```

### More todo