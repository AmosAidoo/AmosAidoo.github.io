---
layout: post
category: rust, db-internals
title: "kleindb: Learning database internals by reading SQLite's source code and porting to Rust"
---
I started a new project about two weeks ago called [kleindb](https://github.com/AmosAidoo/kleindb). I had two goals before starting this project. The first one was to get a deeper understanding of the internals of databases and the second was to get better at coding in Rust. My idea for this particular blog post is for it to be the first of multiple posts, documenting everything I am learning whilst working on this project.

## Warm Up: The Shell

> Decades of effort have gone into optimizing SQLite, both for small size and high performance. And optimizations tend to result in complex code. So there is a lot of complexity in the current SQLite implementation. It will not be the easiest library in the world to hack.
>
> -- the README

The SQLite comes with an interactive command line tool that allows one to connect to a SQLite database and allows users to run SQL statements against the database. In my mind, having a tool like this whilst porting the project would be very useful. Also, since the shell uses the public interfaces, reading it would help me understand how those interfaces are used and then I can dive deeper from there.
The source code for the shell is in `shell.c.in`. The source code is fairly complex so whilst reading and porting, I try to filter out those I feel are not immediately useful which I can always comeback to when the need be.

Source code for C binaries have an `int main()` so that's what I looked for first and I found it at line **13020**.
```c
#if SQLITE_SHELL_IS_UTF8
int SQLITE_CDECL main(int argc, char **argv){
#else
int SQLITE_CDECL wmain(int argc, wchar_t **wargv){
  char **argv;
#endif
```

Next there is an object in the main function on line **13033**.
```c
ShellState data;
```
Doing a quick search for `struct ShellState {` leads you to line **1437** with a comment on top stating exactly the purpose of this struct.
```c
/*
** State information about the database connection is contained in an
** instance of the following structure.
*/
typedef struct ShellState ShellState;
struct ShellState {
  sqlite3 *db;           /* The database */
...
  int lineno;
...
  FILE *in;              /* Read commands from this stream */
  FILE *out;             /* Write results here */
...
};
```
`*db` is a pointer to the database connection object. The `sqlite3` struct is fully defined in `sqliteInt.h`. `*in` is a pointer to the file where SQL commands would be read from and `*out` is a pointer to the file where results would be written to.

Reading further, there is a lot of code that handle all the possible command line arguments that can be passed to the shell which I skipped over. I would add those later when it becomes necessary.

There is a start of an `if..else` statement on line **13572**:
```c
if( !readStdin ){
  /* Run all arguments that do not begin with '-' as if they were separate
    ** command-line inputs, except for the argToSkip argument which contains
    ** the database filename.
    */
  ...
} else {
  /* Run commands received from standard input
    */
  ...
}
```
I am interested in the else part as it runs commands received from standard input.
There is another `if..else` statement on a variable `stdin_is_interactive` which basically tells us whether standard input is an interactive input or not. Also, that is what I am interested in:
```c
if( stdin_is_interactive ){
  ...
  sqlite3_fprintf(stdout,
        "SQLite version %s %.19s\n" /*extra-version-info*/
        "Enter \".help\" for usage hints.\n",
        sqlite3_libversion(), sqlite3_sourceid());
  if( warnInmemoryDb ){
    sputz(stdout, "Connected to a ");
    printBold("transient in-memory database");
    sputz(stdout, ".\nUse \".open FILENAME\" to reopen on a"
          " persistent database.\n");
  }
...
  data.in = 0;
  rc = process_input(&data);
...
}else{
  data.in = stdin;
  rc = process_input(&data);
}
```
Line 14 in the snippet above points `*in` to standard input(its [file descriptor](https://en.wikipedia.org/wiki/File_descriptor) is 0) and line 15 calls `process_input` function which would read SQL commands from `*in`.

The `process_input` function can be found at line **12581** to line **12689**. The first few lines initalize variables that would be used throughout the function. Then there is a while loop:
```c
...
while( errCnt==0 || !bail_on_error || (p->in==0 && stdin_is_interactive) ){
  ...
}
...
```
Over here, this while loop keeps running until the user ends the interactive shell(the condition I am interested in). All this while loop does is it takes a line of input from the user and if it is a complete SQL statement(ends in a semi-colon), it tries to run the statement otherwise it will prompt the user to type the rest of the SQL statement by displaying the continuation prompt, `...>`.

This line handles taking a single line of input from the user and displaying the prompt:
```c
zLine = one_input_line(p->in, zLine, nSql>0);
```

The line below scans the current line of input from the user and updates `QuickScanState`. I didn't spend a lot of time here but I did read the enum definition and `#define` directives related to it and my guess is that it is an efficient way of keeping track of the state of a line of input from a user so that certain inputs like comment-only lines or whitespace-only lines won't we processed. It is also fast way of checking if the line ended with a semi-colon.
```c
qss = quickscan(zLine, qss, CONTINUE_PROMPT_PSTATE);
```

The final interesting part for me in this while loop are these lines:
```c
if( ... && sqlite3_complete(zSql) ){
  ...
  errCnt += runOneSqlLine(p, zSql, p->in, startline);
  ...
}
```
`sqlite3_complete` implements a state machine which checks whether a given SQL statement is complete or not. It's complete definition is in `complete.c`.
`runOneSqlLine` runs a single line of SQL and returns the number of errors.

There is one line in `runOneSqlLine` I am interested in and that is:
```c
rc = shell_exec(p, zSql, &zErrMsg);
```

Then looking further into `shell_exec` starting at line **4599**, there are these lines:
```c
static int shell_exec(
  ShellState *pArg,                         /* Pointer to ShellState */
  const char *zSql,                         /* SQL to be evaluated */
  char **pzErrMsg                           /* Error msg written here */
){
  sqlite3_stmt *pStmt = NULL;
  ...
  rc = sqlite3_prepare_v2(db, zSql, -1, &pStmt, &zLeftover);
  ...
  while( zSql[0] && (SQLITE_OK == rc) ){
    ...
    if( SQLITE_OK != rc ){
    ...
    } else {
      ...
      exec_prepared_stmt(pArg, pStmt);
      ...
    }
    ...
  }
}
```

Finally looking into `exec_prepared_stmt`, the most important lines for me are those that have this:
```c
rc = sqlite3_step(pStmt);
```

## A step into SQLite internals

`sqlite3_prepare_v2` and `sqlite3_step` are part of the public API of the SQLite library. In his beautiful [lecture](https://youtu.be/gpxnbly9bz4?list=PL9K3Drwh2MNAVTr1g92Wl46P1dhAXGqu3&t=860) from **The Databaseology Lectures - CMU Fall 2015**, Dr Richard Hipp summarizes SQLite into these two functions.
SQLite consists of
- Compiler to translate SQL into the byte code
- Virtual Machine to evaluate the byte code
![alt text](/images/sqlite-summary.png)

For the remainder of the project, I would dig deeper into the source code with these two functions being the starting points.

## Conclusion

The shell has being a very good starting point into SQLite's source code and has gotten me comfortable with the codebase. As and when needed, I would comeback to it to add more features. Translating these pieces into Rust was quite straight forward. The code is in [main.rs](https://github.com/AmosAidoo/kleindb/blob/main/src/main.rs).