## A tool to grasp the scale of the source code made with go

### Install

+ https://github.com/hhatto/gocloc

```sh
go get -u github.com/hhatto/gocloc/cmd/gocloc
```


### Example

```
git clone https://github.com/golang/go.git
cd go
gocloc .
```

```
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Go                            2101          40676          56671         321018
C                              372          20658          22719         148717
Plain Text                      37           2346              0          36805
HTML                            58           4234            283          25475
C Header                        96           1746           3843          13954
Assembly                       134           1510           2144           8132
Yacc                             6            449            434           5174
Perl                            12            718           1269           4442
JavaScript                       6            590            421           2571
Python                           6            577           1885           1762
BASH                            19            291            412           1545
XML                             14             91              9           1299
CSS                              6            123             34            883
LISP                             2            108            147            618
VimL                             9             94            157            456
Bourne Shell                    11             53             83            273
Batch                            4             44              1            193
Makefile                        20             77             69            173
WiX                              1             12             10            142
Awk                              2             12             35            104
YAML                             3             14             12             60
JSON                             1              0              0             17
Ruby                             1              0              0              8
-------------------------------------------------------------------------------
TOTAL                         2921          74423          90638         573821
-------------------------------------------------------------------------------

```

### Example: Generate data for each tag

```sh
git clone https://github.com/golang/go.git
cd go
git tag --sort=taggerdate > list
```

```ruby
File.open('list') do |f|
  f.each_line do |tag|
    tag.chomp!
    puts tag
    system("git checkout -b #{tag} refs/tags/#{tag}")
    system("gocloc --output-type=sloccount . > data/#{tag}")
  end
end
```


