# portfolio

A collection of my work, built with GitHub Pages and the [Minima](https://github.com/jekyll/minima) theme.

Live at https://asharbutt.github.io/portfolio/

## Pages

- `index.md`: home page
- `code.md`: C++ and Python pricing code (also in [LibraryCode](https://github.com/asharbutt/LibraryCode))
- `streamlit.md`: Streamlit dashboards
- `theory.md` and `theory/`: maths notes, rendered with MathJax

## Adding code

Add a `####` heading under the right section in `code.md`, then paste the code in a fenced block wrapped in `{% raw %}` and `{% endraw %}`, so braces in the code aren't read as template tags.

## Adding a theory note

Copy any file in `theory/`, change the `title` at the top and replace the body. Then add a link to it in `theory.md`. Keep `mathjax: true` so equations render.

## Running locally

```
bundle install
bundle exec jekyll serve
```
