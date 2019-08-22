# CSS Grid fallbacks for legacy browsers

WP Rig uses CSS grid for global two-dimensional layouts. While `grid` has [widespread support in modern browsers](https://caniuse.com/#search=grid), some older browsers, in particular IE 11, 10, and older do not support the current specification.

If support for layouts in older browsers is needed, the recommendation is to create graceful degradation fallbacks to `flex`. Below you will find some resources on creating such fallbacks:

- [CSS Grid Fallbacks and Overrides](https://rachelandrew.co.uk/archives/2017/03/20/css-grid-fallbacks-and-overrides/) (Rachel Andrew)
- [CSS Grid Layout and Progressive Enhancement](https://rachelandrew.co.uk/archives/2017/03/20/css-grid-fallbacks-and-overrides/) (MDN)
- [Basic grid layout with fallbacks using feature queries](https://www.chenhuijing.com/blog/basic-grid-with-fallbacks/) (Chen Hui Jing)
