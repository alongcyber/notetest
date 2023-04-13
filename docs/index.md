# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
### code
examples fot Codeblocks
Some `code`
----
A plain Codeblock

``` c title="a title for the codeblock"
    int main(){
        std::cout<<"fuck off";
    }
```

seconda CodeBlock
``` py linenums="1"
    def bublle_sort(items):
        for i in range(len(iyems)):
            for j in range(len(items)-1-i):
                if items[j]>items[j+1]:
                    items[j],items[j+1]=items[j+1],items[j]
```