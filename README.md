# terminal.js

A VT100-style terminal emulator for Node.js and the browser. terminal.js parses
terminal escape sequences into an in-memory screen buffer that can be rendered
as plain text, ANSI, HTML, or a live DOM view.

## Features

- Parses commonly used VT100/xterm control and escape sequences
- Works as a Node.js writable stream
- Preserves cursor position, colors, text attributes, terminal modes, and
  alternate screen buffers
- Renders to plain text, ANSI, HTML, a browser DOM element, or a TTY
- Exposes terminal state and diff primitives for custom integrations

## Installation

```sh
npm install terminal.js
```

## Quick start

```js
var Terminal = require('terminal.js');
var terminal = new Terminal({
  columns: 20,
  rows: 4
});

terminal.write('\x1b[31mHello\x1b[0m, terminal!');

console.log(terminal.toString());
```

`Terminal` is a writable stream, so readable streams can also be piped into it:

```js
process.stdin.pipe(terminal);
```

## Output formats

Call `terminal.toString()` without an argument for plain buffer contents, or
pass a format name to select a renderer:

```js
terminal.toString();        // plain buffer contents
terminal.toString('plain'); // fixed-height plain-text screen
terminal.toString('ansi');  // ANSI-colored screen
terminal.toString('html');  // HTML markup
```

## Browser usage

Build the browser bundle, then include `dist/terminal.js`. The bundle exposes
the constructor as `Terminal`.

```sh
npm install
npx grunt
```

```html
<div id="terminal"></div>
<script src="dist/terminal.js"></script>
<script>
  var terminal = new Terminal({ columns: 80, rows: 24 });
  var input = terminal.dom(document.getElementById('terminal'));

  input.on('data', function (data) {
    // Send keyboard input to your shell, PTY, or other data source.
  });

  terminal.write('\x1b[32mReady.\x1b[0m');
</script>
```

`terminal.dom(element)` keeps the element synchronized with the terminal state
and returns a readable stream containing keyboard input.

## API

### `new Terminal(options)`

Creates a terminal with the following options:

| Option | Default | Description |
| --- | --- | --- |
| `columns` | `80` | Number of columns |
| `rows` | `24` | Number of rows |
| `attributes` | `{}` | Initial text attributes such as `fg`, `bg`, `bold`, `italic`, `underline`, `blink`, and `inverse` |

### `terminal.write(data)`

Writes text and escape sequences to the terminal. Because `Terminal` implements
Node.js's writable stream interface, it supports `write()` and `end()` and can
be used as the destination of `readable.pipe(terminal)`.

### `terminal.toString([format])`

Renders the current screen. Supported formats are `plain`, `ansi`, and `html`.
With no format, it returns the plain contents of the terminal buffer.

### `terminal.dom(element[, options])`

Attaches live terminal input and output to a DOM element. Returns a readable
stream that emits encoded keyboard input.

### `terminal.state`

The underlying `TermState` instance. Use `terminal.state.resize()` to resize
the screen:

```js
terminal.state.resize({ columns: 100, rows: 30 });
```

Additional constructors are exported for custom integrations:

```js
var Terminal = require('terminal.js');

Terminal.TermState;
Terminal.TermDiff;
Terminal.output.PlainOutput;
Terminal.output.AnsiOutput;
Terminal.output.TtyOutput;
Terminal.output.HtmlOutput;
Terminal.output.DomOutput;
Terminal.input.DomInput;
Terminal.input.TtyInput;
Terminal.source.EmitterSource;
```

## Development

```sh
npm install
npm test
```

The project uses a legacy Grunt toolchain. If JSHint is incompatible with your
Node.js version, run `npx grunt mochaTest` to execute the behavioral test suite
independently.

Useful Grunt tasks:

- `grunt` builds `dist/terminal.js` and `dist/terminal.min.js`
- `grunt test` runs JSHint and the Node.js test suite
- `grunt test-browser` builds the debug bundle for `test/index.html`
- `grunt doc` generates API documentation in `doc/`

## License

[MIT](LICENSE)
