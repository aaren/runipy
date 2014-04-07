Allow stdin and stdout as input / output.

### Why

This is more unixy. It lets runipy sit in a pipeline, chaining with
other things that ingest or spit out notebook JSON.

My use case is with [notedown], a tool that creates notebooks from
markdown. It reads markdown from a file or stdin and outputs JSON to
stdout.

This PR would allow us to create and execute a notebook in a single
command:

    notedown some_file.md | runipy --stdout > some_notebook.ipynb


[notedown]: https://github.com/aaren/notedown


### Implementation

How do we tell runipy that we are using stdin / stdout? Should it be
magic or do we use a `-` placeholder or `--stdout` switch?

I've gone with the `--stdin` and `--stdout` flags, with `-`
available as a special input filename that forces stdin. If there is
no input file we default to stdin, but exit if it is empty.


### Usage

These are equivalent:

    runipy < notebook.ipynb

    runipy - < notebook.ipynb

    runipy --stdin < notebook.ipynb

i.e. we default to stdin. However, an empty stdin without `-` or
`--stdin` set will cause runipy to print help and exit.


These are equivalent:

    cat old_notebook.ipynb | runipy --stdout > new_notebook.ipynb

    cat old_notebook.ipynb | runipy - new_notebook.ipynb

    runipy --stdout < old_notebook.ipynb > new_notebook.ipynb

i.e. if you want to use stdin and output to a file you need to use
'-' in place of the input file.


These are equivalent:

    cat old_notebook.ipynb | runipy - -
    cat old_notebook.ipynb | runipy - --stdout
    cat old_notebook.ipynb | runipy --stdin --stdout


This will both print the notebook to stdout and save to file:

    cat old_notebook.ipynb | runipy - new_notebook.ipynb --stdout


This will both overwrite and print to stdout:

    runipy old_notebook.ipynb --overwrite --stdout



### Compatibility

All previous usage of runipy is unaffected, including calls to the
module, with the addition that you can now send a file object
instead of a file name.

### Tests

I think tests are unaffected, although they were failing for me
before. They fail in the same way at least. The failure I was
getting before was because two outputs weren't equal, including the
strings

> u'\x1b[0;31mZeroDivisionError

and

> u'\x1b[1;31mZeroDivisionError
