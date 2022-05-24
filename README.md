# fatpak

FatMemo for Delphi

## From FatMemo.txt

[FILES IN PACKAGE]

- FatMemo.txt
- FatThings.pas
- FatThings.dcr
- IRCTags.pas

[ALWAYS KEEP THEM TOGETHER]



{ ******************************************************************************

  TFatMemo component by Gasper Kozak (gasper.kozak@email.si)
  Ideas and bug reports are most welcome!

  LEGAL INFORMATION.
    The component is FREE FOR USE in ANY kind of projects. You are allowed
    to modify the source at will under condition you leave (this) original
    comment intact.

  FEATURES.
    - Color coding, embedded bitmaps, links
    - Automatic copy-to-clipboard when user presses and drags left mouse button
    - Easy to irc color coding interface

  DOES NOT FEATURE.
    - Editing the text in the window. This feature will never be implemented.



  VERSIONS.
    THIS
      0.95, November 23rd
        NEW THINGS
          colored selection
          vertical scrollbar. Horizontal scrollbar will probably be added some
          time in the future, but it's currently too buggy to be functional

        NEW PROPERTIES
          ScrollBarVert : boolean
            these two properties enable/disable scrollbar in window
          StickText : (stBottom, stNone, stTop)
            replaces FixedBottomLine; if set to stBottom, the text is always
            shown last-line-at-bottom, if it is set to stTop, the test is always
            shown first-line-at-top, otherwise its shown TopIndex-line-at-top

        NEW EVENTS
          OnScrollHoriz, OnScrollVert
            these two events are triggered when corresponding scrollbar is moved

    PREVIOUS
      0.91, May 15th


  KNOWN ISSUES.
    
  - Slow paint on large window and lots of text

  -STATEMENT.
    I do not promise any support or new versions of this component.
    But check www.delphipages.com anyway :)

****************************************************************************** }




## Basics
  // note: not all properties/methods/events of objects are listed, just the basic ones
  // for further understanding or deeper development you'll just have to dig in the source;
  // that's why i added it

Ok, here we go. The first thing you need to know is how is `TFatMemo` built.
It has many properties which we'll duscuss later, but now let's look at
the main stuff. `TFatMemo` provides you with a Lines: TFatLines property.
It is the object that holds an array of Lines: `TFatLine` property.
Each `TFatLine` owns another array of Parts: `TFatPart`.
The `TFatPart` is the smallest whole part of this memo.

### Basic types

  - `TPartType = (ptNone, ptText, ptColor, ptStyle, ptBitmap);`

## TFatPart
### PROPERTIES
  - `Index: Integer;` 
    The index of the part in the owning TFatLine

  - `PartType: TPartType;`
    Identifies the part.

  - `FontColor, BackColor: TColor;` 
    Colors of the text in following parts of the same line.
    When set, the PartType automatically becomes ptColor. This part cannot hold
    any other information, it's only purpose is, that FOLLOWING parts with text
    in the same line are drawn in specified colors.

  - `Text: String;`
    PartType = ptText. Use just for text :). This part can only hold text.

  - `Bitmap: TBitmap;`
    Image. When set, PartType is set to ptBitmap. Note: Bitmap is not copied or
    assigned, so you will lose it if you destroy it somewhere else.
    This part can hold two things: text (must be set priorly to bitmap) and bitmap.
    When setting this property, the text wouldn't be deleted, but would also not
    be drawn. It is rather used for copying to clipboard.

  - `Link: String;`
    When link is set, the part becomes a link, regardless of what it was before.
    This part can hold text and/or a bitmap. In both cases, it becomes a clickable link.
    OnLinkClick event of the owning TFatMemo will occur, passing the Link: string value.

  - `Style: TFontStyle;`
    Set this when you want the following text to be of different style.
    Like in FontColor property, this part cannot hold

### METHODS
  - `Create(ALine: TFatLine);`
  - `Destroy;`
  - `Assign(Source: TFatPart);`
  - `Delete;` Deletes the part from the owning TFatLine;
  - `IsLink: Boolean;` is it a link?
  - `RemoveLink(ALink: String);` If the part is a link, and it's Link property is equal
    to the passed ALink: string, then the link is removed and it becomes usual text.
    If you pass an empty string, the original link is removed without comparing.
    Usability (in relation to mIRC): when a user is in the channel, his(her) nick
    is clickable, after (s)he leaves, the link is removed, it becomes usual text


## TFatLine
### PROPERTIES
  - `Parts[Index: Integer]: TFatPart;` The array of owned parts.
  - `Index: Integer;` Index of the line in the owning - TFatLines object.
  - `Height`: Height of the line in pixels
  - `AsText: String;` Returns the line in string with no special characters.
  - `AsIrcText: String;` Set this if you have an IRC-colored line. Read this if you want
    a line to be returned with IRC-special charactes.
    Works with tags: `bold`, `underline`, `normal`, `inverse`, `color`
    
    **NOTE**
    You can also use TAG_BEGINLINK and TAG_ENDLINK charactes (which are not
    supported by mIRC) if you want to set a link somewhere in the line.
    The link can be made of different parts, thus meaning it can contain different
    colors, styles and bitmaps.
    Example (tags are marked with [#tag] for clearness)
    <rAa> blah blah [#color]4,1 this is [#beginlink] hey man [#bold] this is cool! [#endlink] :)
    In this example, the line from "hey man" to (and including) "cool!" would become a link.
    The colors and styles would be coded normally.
    Also note: if IRC-style tags are included between the TAG_BEGINLINK and TAG_ENDLINK tag,
    they are removed from the Link: String property that is passed to Memo.OnLinkClick event. 
  
### METHODS
  Create(ALines: TFatLines);
  Destroy;
  Assign(Source: TFatLine);
  Clear; Clears the line; deletes all parts.
  Add: TFatPart; Adds a part at the end of the line;
  Insert(Index: Integer): TFatPart; Inserts a part before the index position
  Delete(Index: Integer); Deletes a particular part
  Count: Integer; Returns number of all parts in a line;
  RemoveLinks(Link: String); Removes any link from any part, if matching.
    See TFatPart.RemoveLink for further details.



## TFatLine
### PROPERTIES
  `Items[Index: Integer]: TFatLine; default;  The lines.`

### METHODS
  - `Create(AMemo: TFatMemo);`
  - `Destroy;`
  - `Assign(Source: TFatLines);`
  - `Clear; Clears the lines;`
  - `Delete(Index: Integer);` deletes the line at index
  - `Count: Integer;` returns the number of lines
  - `AddNew: TFatLine;` Creates a new line, returning it
  - `InsertNew(Index: Integer): TFatLine;` Inserts a line at position Index
  - `Add(S: String);` Adds a line and sets its AsText property. Use this to add
    standard non-formatted text
  - `AddLineWithIrcTags(S: String): Integer;` This method adds a line, formatting
    it from IRC-style. See TFatLine.AsIrcText for details and example.
  - `RemoveLinks(Link: String);` removes any matching link from any part of every line



Ok, this was the non-visual and the core part. Now here is a standard visual
component TFatMemo.

I will only explain the things that are not clearly understandable.

  `TDrawFlag = (dfDontDraw, dfWordWrap, dfStretchImages, dfAlignMiddle, dfAlignBottom);`
  `TDrawFlags = set of TDrawFlag;`


## TFatMemo
### PROPERTIES
  - `DrawFlags`: TDrawFlags;
    - `dfDontDraw`: set it if you don't want the text/ bitmaps to be displayed
    - `dfWordWrap`: enables/disables word wrapping in the memo
    - `dfStretchImages`: if this option is set, the bitmaps are stretched
      vertically to fit the LineHeight property. Bitmaps are (ofcourse) not
      stretched horizontally
    - `dfAlignMiddle`, dfAllignBottom: aligning options, comes handy if you
      use larger LineHeight

  - `FixedBottomLine`: Boolean ... this property is removed
  `StickText`: (`stTop`, `stNone`, `stBottom`)
    Three ways of displaying text:
      the first line on top, TopIndex sets the line on top, last line on bottom

  `LineHeight`: because FatThings include a lot of drawing and formatting,
    the height of a line must be fixed. Specify your own value.
    Usually it is around 15 for a size 8, FixedSys font.
    
  *NOTE*: the component will not allow you to set values below the actual
      height of the font

  `TopIndex`: when `FixedBottomLine` is not in use, this property identifies the
    first line that is displayed at the top.

  `OverLine: TFatLine`; returns the line that mouse cursor is over
  `OverPart: TFatPart`; returns the part that mouse cursor is over

### METHODS
  - CopyToClipboard; TFatMemo does not implement selections, therefore
    this method copies all the text into the clipboard

### TIPS AND TRICKS

1. A simple tutorial of how to make a clickable bitmap (link) in the memo,
   and keep the text inside to be copied in a clipboard.

```pascal 
var FatLine: TFatLine;
  FatPart: TFatPart;
...

// Get, add or insert a line
FatLine := FatMemo1.Lines[Index: Integer];
or
FatLine := FatMemo1.Lines.AddNew;
or
FatMemo1.Lines.InsertNew(Index: Integer);

// Get, add or insert a part to that line
FatPart := FatLine.Parts[Index: Integer];
or
FatPart := FatLine.Add;
or
FatLine.Insert(Index: Integer);

// Now let's change that part
with FatPart do
begin
  Text := 'this text will show in the clipboard';
  // The part is now just text

  Bitmap := 'Image1.Picture.Bitmap;
  // Now the text is not visible anymore, instead a bitmap is drawn.
  // The text is not deleted, but it is used when copying the part to the clipboard.

  Link := 'www.somewhere.com';
  // And finally, that bitmap becomes clickable.
  // FatMemo1.OnLinkClick event will now occure when clicking the bitmap.
end;
```

That's all for now.
