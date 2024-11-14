# Useful Shortcuts

## Emmet

Emmet is a very powerful tool to quickly build blocks of html and css. Check the [Emmet cheat sheet](https://docs.emmet.io/cheat-sheet/) for more info.

Emmet example:
```txt
ul#my-id.my-class[some-attribute=some-value]>li#li-id$.class1.class2{some content}*3
```
> Note: Where your cursor is when you hit `enter` matters. Make sure the cursor is at the end of the line.

Emmet example result:
```html
<ul id="my-id" class="my-class" some-attribute="some-value">
    <li id="li-id1" class="class1 class2">some content</li>
    <li id="li-id2" class="class1 class2">some content</li>
    <li id="li-id3" class="class1 class2">some content</li>
</ul>
```

To wrap an html block in another tag (or any emmet-defined syntax):
1. `ctrl+shift+p`
2. Select `Emmet: Wrap with Abbreviation`
3. Enter a tag, e.g. `div` (Note: continuing the emmet chain without specifying a tag defaults to a `div`, e.g. `.wrapper>p`)
4. Hit `enter`

## Other VS Code Shortcuts

- Auto format html: `shift+alt+f`
  - Note: this seems to auto rename your favicon file reference to `favicon.ico`.
- Go to file: `ctrl+p`
- Delete line without copying: `ctrl+shift+k`
