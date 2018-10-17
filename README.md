# progrium/bashstyle

Bash is the JavaScript of systems programming. Although in some cases it's better to use a systems language like C or Go, Bash is an ideal systems language for smaller POSIX-oriented or command line tasks. Here's three quick reasons why:

 * It's everywhere. Like JavaScript for the web, Bash is already there ready for systems programming.
 * It's neutral. Unlike Ruby, Python, JavaScript, or PHP, Bash offends equally across all communities. ;)
 * It's made to be glue. Write complex parts in C or Go (or whatever!), and glue them together with Bash.

This document is how I write Bash and how I'd like collaborators to write Bash with me in my open source projects. It's based on a lot of experience and time collecting best practices. Most of them come from these [two](http://wiki.bash-hackers.org/scripting/obsolete) [articles](http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/), but here integrated, slightly modified, and focusing on the most bang for buck items. Plus some new stuff!

Keep in mind this is not for general shell scripting, these are rules specifically for Bash and can take advantage of assumptions around Bash as the interpreter. 

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
 * Variable names should be lowercase unless exported to environment.
 * Always use `set -eo pipefail`. Fail fast and be aware of exit codes. 
   * Use `|| true` on programs that you intentionally let exit non-zero.
 * Never use deprecated style. Most notably:
   * Define functions as `myfunc() { ... }`, not `function myfunc { ... }`
   * Always use `[[` instead of `[` or `test`
   * Never use backticks, use `$( ... )`
   * See http://wiki.bash-hackers.org/scripting/obsolete for more
 * Prefer absolute paths (leverage $PWD), always qualify relative paths with `./`.
 * Always use `declare` and name variable arguments at the top of functions that are more than 2-lines
   * Example: `declare arg1="$1" arg2="$2"`
   * The exception is when defining variadic functions. See below.
 * Use `mktemp` for temporary files, always cleanup with a `trap`.
 * Warnings and errors should go to STDERR, anything parsable should go to STDOUT.
 * Try to localize `shopt` usage and disable option when finished.

If you know what you're doing, you can bend or break some of these rules, but generally they will be right and be extremely helpful.

## Best Practices and Tips

 * Use Bash variable substitution if possible before awk/sed.
 * Generally use double quotes unless it makes more sense to use single quotes.
 * For simple conditionals, try using `&&` and `||`.
 * Don't be afraid of `printf`, it's more powerful than `echo`.
 * Put `then`, `do`, etc on same line, not newline.
 * Skip `[[ ... ]]` in your if-expression if you can test for exit code instead.
 * Use `.sh` or `.bash` extension if file is meant to be included/sourced. Never on executable script.
 * Put complex one-liners of `sed`, `perl`, etc in a standalone function with a descriptive name.
 * Good idea to include `[[ "$TRACE" ]] && set -x`
 * Design for simplicity and obvious usage.
   * Avoid option flags and parsing, try optional environment variables instead.
   * Use subcommands for necessary different "modes".
 * In large systems or for any CLI commands, add a description to functions.
   * Use `declare desc="description"` at the top of functions, even above argument declaration.
   * This can be queried/extracted using reflection. For example:
   ```
   eval $(type FUNCTION_NAME | grep 'declare desc=') && echo "$desc"
   ```
 * Be conscious of the need for portability. Bash to run in a container can make more assumptions than Bash made to run on multiple platforms.
 * When expecting or exporting environment, consider namespacing variables when subshells may be involved. 
 * Use hard tabs. Heredocs ignore leading tabs, allowing better indentation.
 
## Good References and Help

 * http://wiki.bash-hackers.org/scripting/start
   * Especially http://wiki.bash-hackers.org/scripting/newbie_traps
 * http://tldp.org/LDP/abs/html/
 * Tips for interactive Bash: http://samrowe.com/wordpress/advancing-in-the-bash-shell/
 * For reference, [Google's Bash styleguide](https://google.github.io/styleguide/shell.xml)
 * For linting, [shellcheck](https://github.com/koalaman/shellcheck)

## Examples

### Regular function with named arguments
Defining functions with arguments
```bash
regular_func() {
	declare arg1="$1" arg2="$2" arg3="$3"

	# ...
}
```

### Variadic functions
Defining functions with a final variadic argument
```bash
variadic_func() {
	local arg1="$1"; shift
	local arg2="$1"; shift
	local rest="$@"

	# ...
}
```

### Conditionals: Testing for exit code vs output

```bash
# Test for exit code (-q mutes output)
if grep -q 'foo' somefile; then
  ...
fi

# Test for output (-m1 limits to one result)
if [[ "$(grep -m1 'foo' somefile)" ]]; then
  ...
fi
```

### More todo
