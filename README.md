# Linaria

Fast zero-runtime CSS in JS library.


## Features

1. CSS is extracted out to real CSS files
1. Familiar CSS syntax
1. Class names are stay recognizable to what you wrote (with babel plugin)
1. SCSS like shorthand and nesting
1. Zero runtime in production
1. Server rendering for critical CSS


## Usage

CSS rule declarations use tagged template litreals which produce a class name for use. In any JS file:

```js
import React from 'react';
import { css, compose } from 'library';
import fonts from './fonts.js';
import colors from './colors.js';

const title = css`
  text-transform: uppercase;
`

const container = css`
  height: 3rem;
`

const header = css`
  font-family: ${fonts.heading};
  font-size: 3rem;
  margin-bottom: .5rem;

  [data-theme=dark] & { color: ${colors.white} }

  [data-theme=light] & { color: ${colors.black} }

  @media (max-width: 320px) {
    font-size: 2rem;
  }
`;

export default function Header({ className }) {
  return (
    <div className={compose(container, className)}>
      <h1 className={header} />
    </div>
  );
}

export function Block() {
  return <div className={container} />;
}

export function App() {
  return <Header className={title} />;
}
```

After being transpiled, the code will output following CSS:


```css
.container__jdh5rtz.title__jt5ry4 {
  text-transform: uppercase;
}

.container__jdh5rtz {
  height: 3rem;
}

.header__xy4ertz {
  font-family: Helvetica, sans-serif; /* constants are automatically inlined */
  font-size: 3rem;
  margin-bottom: .5rem;
}

@media (max-width: 320px) {
  .header__xy4ertz {
    font-size: 2rem;
  }
}

[data-theme=dark] .header__xy4ertz {
  color: #fff;
}

[data-theme=light] .header__xy4ertz {
  color: #222;
}
```

And the following JavaScipt:

```js
import React from 'react';

export default function Header({ className }) {
  return (
    <div className={'container__jdh5rtz' + ' ' + className)}>
      <h1 className="header__xy4ertz" />
    </div>
  );
}

export function Block() {
  return <div className="container__jdh5rtz" />;
}

export function App() {
  return <Header className="title__jt5ry4" />;
}
```


## Animations

You could declare CSS animation like so:

```js
const box = css`
  animation: rotate 1s linear infinite;

  @keyframes rotate {
    { from: 0deg }
    { to: 360deg }
  }
`
```

The animation name is always scoped to the selector.


## Server rendering

Even with fully static CSS, we have an opportunity to improve the initial page load by inlining critical CSS.

```js
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import { collect, slugify } from 'library';
import App from './App';

const cache = {};

const css = fs.readFileSync('./dist/styles.css').toString();
const app = express();

app.get('/', (req, res) => {
  const html = ReactDOMServer.renderToString(<App />);
  const { critical, other }  = collect(html, css);
  const slug = slugify(other);

  cache[slug] = other;

  res.end(`
    <html lang='en'>
      <head>
        <title>App</title>
        <style type='text/css'>${critical}</style>
      </head>
      <body>
        <div id='root'>${html}</div>
        <link rel='css' href='/styles/${slug}' />
      </body>
    </html>
  `);
});

app.get('/styles/:slug', (req, res) =>
  res.end(cache[req.params.slug])
);

app.listen(3242);
```

You probably should write these CSS chunks to disk and serve them with correct headers for caching.


## TODO

1. When the content of two rules are the same, use the same rule instead of adding duplicates
1. Composing classnames should ensure correct specificity so that class name towards end has higher specificity. We can increase the specificity manually to achieve this, e.g. - `compose('header', 'title')` might produce `.title__gf63rt, .title__gf63rt.header__gyt654` and `.header__gyt654` instead of `.title__gf63rt` and `.header__gyt654`
1. Babel plugin to inline constants and integrate with libs like `polished`, `color`, `polychrome` and `lodash`
1. Babel plugin to replace `const header = css` with `const header = css.named('header__ghg54t')`
1. Webpack plugin to extract the CSS to a separate file
1. ESLint plugin to lint styles
1. Utilities to help with server rendering:
    - Given some HTML and CSS, we should be able to extract the CSS that's actually used, useful for critical CSS
1. Add ability to use JS objects instead of tagged template literals for people who prefer that


## Challenges to solve

1. Theming should have a nicer API, the idea is to specify set of theme names, generate set of rules for each theme automatically and then change an attribute at application root to switch themes
1. It'll be nicer to figure out common CSS for all pages in server rendering and then ship page specific CSS inline