# abc2svg interface 1

This document describes the interface of the abc2svg core version `1`
(abc2svg-1.js).

## Initialization

The abc2svg core may be included in html/xhtml documents
    by:

    <script src="http://moinejf.free.fr/js/abc2svg-1.js" type="text/javascript"></script>

Before any first or new ABC to SVG translation, an instance of the class
`Abc` must be created:

``` 
   var abc = new Abc(user)
```

The `user` argument of this function is a javascript Object which must
or may contain:

  - **img\_out** (optional):  
    Callback function which called when a new SVG image has been
    generated.
    
    This function receives one argument, a string, which is usually a
    SVG image. This function may be absent when no graphic generation is
    needed as, for example, for playing only.

  - **errbld** (mandatory if no errmsg):  
    Callback function which is called when some error has been found
    during the ABC parsing or the SVG generation
    
    This callback function which is called when some error has been
    found during the ABC parsing or the SVG generation.
    
    This function receives 5 arguments:
    
      - *severity\_level* (integer)  
        level of the message
        
          - It may be:  
            `0`: warning
            `1`: error
            other value: fatal error
    
      - *message* (String)  
        Text of the error.
    
      - *file\_name* (String):  
        The name of the abc file, used for reporting. The file name may
        be `undefined`.
    
      - *line\_number and column\_number* (integer):  
        Position of the error in the ABC source.
        
        This information may not be known in which case line\_number is
        `undefined`.
        
        The line and column numbers start from 0.

  - **errmsg** (mandatory if no errbld)  
    Callback function which is called when some error has been found
    during the ABC parsing or the SVG generation
    
    This function receives 3 arguments:
    
      - *message*  
        Text of the error.
        
        Generally, this text includes a reference of the error in the
        ABC source
            format:
        
            file_name ":" line_number ":" column_number " " error_message
    
      - *line\_number* and *column\_number*  
        Position of the error in the ABC source, same as the
        corresponding argument in `errbld()`.

  - **read\_file** (mandatory)  
    Callback function which is called to read a file.
    
    This function receives one argument, the *name* of the file as a
    string.
    
    It must return the file content as a string.
    
    It is called when a `%%abc-include` command has been found in the
    ABC source.

  - **anno\_start** (optional)  
    Callback function for setting ABC references in the SVG images.
    
    This function is called just before the generation of a music
    element.
    
    It receives 7 arguments:
    
      - *music\_type*  
        It is one of `annot`, `bar`, `clef`, `gchord`, `grace`, `key`,
        `meter`, `note`, `part`, `rest`, `tempo`.
      - *start\_offset*  
        Offset of the music element in the ABC source.
      - *stop\_offset*  
        Offset of the end of music element in the ABC source.
      - *x*, *y*, *w*, *h*  
        Coordinates of a rectangle which covers the music element.

  - **anno\_stop** (optional)  
    Callback function for setting ABC references in the SVG images. This
    function is called just after the generation of a music element. It
    receives the same 7 arguments as the callback function
    **anno\_start**.

  - **get\_abcmodel** (optional)  
    Callback function to get the internal representation of the music
    just before SVG generation. This function receives 4 arguments:
    
      - *tsfirst* (object)  
        First musical symbol in the time sequence.  
        The symbols are double-linked by time by `ts_next` /
        `ts_prev`.  
        The start of a new sequence is marked by `seqst`.
      - *voice\_tb* (array of objects)  
        Voice table.  
        The first symbol of a voice is `sym`.  
        The symbols are double-linked in a voice by `next` / `prev`.
      - *music\_types* (array of strings)  
        Array giving the symbol type from integer value of the symbol
        attribute `type`.
      - *info* (object)  
        Text of the information fields (T:, M:, Q:, P:...).  
        A newline ('\\n') separates the appended values.
    
    The returned representation is destroyed by the generation, so,
    either
    
      - use it immediately before returning from the function, or
      - unset the `user.img_out` function.

  - **imagesize** (string)  
    Define the SVG image size.
    
    When `imagesize` is not set, the size of the SVG images is the one
    of a sheet, as defined by %%pagewidth.
    
    When `imagesize` is defined, it must contain `width="image_width"
    height="image_height"`, *image\_width* and *image\_height* being any
    value accepted in the `<svg>` tag.

  - **page\_format** (boolean)  
    Group the SVG images into non-page-breakable blocks.
    
    When `page_format` is not set, only the SVG images are generated.
    With XHTML output, the images go to the right of the previous ones
    (inlined images).
    
    When `page_format` is set, the images are grouped in `<div>`
    containers with a class "nobrk", and a class "newpage" in case of
    %%newpage.

## ABC to SVG translation

ABC to SVG translation is done calling the Abc method `tosvg`:

``` javascript
abc.tosvg(file_name, ABC_source, start_offset, end_offset)
```

  - *file\_name*  
    is the name of the ABC source. It is used for information only in
    error messages.

  - *ABC\_source*  
    is the ABC source as a (UTF-16) string with the `'\n' (\u000a)` as
    the end of line marker.

  - *start\_offset*, *end\_offset*  
    are the character offsets of the ABC sequence in the ABC\_source.
    
    They are optional and their default values are `0` and
    `ABC_source.length`

`abc.tosvg` may be called many times in which case the values set by
previous calls are kept.

The ABC source must contain full tunes, i.e. a same tune cannot be
continued by a second call to `tosvg`.

## Exported attributes and methods of the Abc class

  - **blk\_flush**: function to terminate a block of images.  
    In case of fatal error during the SVG generation, this function may
    be called in the error callback function to output the last SVG
    elements.
    
    `bkl_flush` takes no argument.

  - **out\_svg**: function to add text in the current SVG image  
    It may be used only in the callback functions.
    
    This function expects one mandatory argument:
    
      - *text* (string)  
        Text to be added.

  - **sx**: function to convert the x coordinate  
    This function expects one mandatory argument:
    
      - *x* (number)  
        Local x.
    
    `sx` returns the `x` coordinate in the current SVG container.

  - **sy**: function to convert the y coordinate  
    This function expects one mandatory argument:
    
      - *y*: (number)  
        Local y.
    
    `sy` returns the `y` coordinate in the current SVG container.

  - **out\_sxsy**: function to output x,y coordinates  
    It may be used only in the callback functions.
    
    This function expects 3 mandatory arguments:
    
      - *x* (number)  
        x offset  
      - *separator* (string)  
        text between x and y in the SVG element
      - *y* (number)  
        y offset

  - **sh**: function to convert a height  
    This function expects one mandatory argument:
    
      - *h* (number)  
        Local height.
    
    `sh` returns the height in the current SVG container.

  - **ax**, **ay**, **ah**: function to convert the x coordinate, y
    coordinate, height  
    These functions return values relative to the global SVG container.
    
    They may be used in place of `sx`, `sy` and `sh` when elements must
    be globally added (by `out_svg`).
    
    When `%%pagescale` is 1 (default), there is no global container (the
    coordinates are the ones of the SVG image).
    
    When this is not the case, a global \<g\> container is added, with
    the class "g".

## Global variables

The global variable `abc2svg` is an object which contains:

  - **version**  
    current version of abc2svg
  - **vdate**  
    date of the distribution of current version

Version: v1.16.4-17-gbcf2672
