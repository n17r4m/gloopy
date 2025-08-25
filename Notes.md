this is awesome, Martin. it reads like a 2013 time capsule: hand-rolled raycaster, gloopy cellular blobs, memoized HSL, IE event fallbacks, even a Y-combinator cameo. legit stylish.

here’s a tight code review—celebrating the good bits and flagging the sneaky bugs that somehow survived the 13k wars.

# what slaps

* the **two-canvas trick** (back = 1px strip “3D” via rays, front = minimap/world) is clever and fast for the era.
* **AI via setInterval** with local state captured in closures — cheap & effective.
* **colour system** (rgba ↔ hsla, mixes, “closest hue”) adds real juice to the visuals.
* the **map DSL** (`map.rect`) and little preview generator? chef’s kiss.
* keyboard state + requestAnimationFrame polyfill + IE fallbacks = shipped anywhere.

# a few delightful landmines (aka “how did this even work?”)

* ### `deg` math is wrong

  ```js
  function deg(rad){ return rad / Math.PI/180 } // <- nope
  // should be:
  function deg(rad){ return rad * 180/Math.PI }
  ```

  (maybe unused, which is why nothing exploded.)

* ### `hsla` achromatic path is busted

  ```js
  if(s == 0) r = g = b = l; // you set r,g,b...
  // ...but you *still* return using hue2rgb(p,q,...) which isn't defined here.
  ```

  Fix by returning early:

  ```js
  if (s == 0) return cache(key, new rgba(l, l, l, a));
  ```

* ### cache eviction in `memo(size)` never evicts the right thing

  ```js
  // you push values into `a` but delete by treating them as keys:
  o[k] = i; a.push(i);
  if (a.length > s) delete o[a.shift()];
  ```

  Store keys instead:

  ```js
  o[k] = i; a.push(k);
  if (a.length > s) delete o[a.shift()];
  ```

* ### bitwise tricks: `x|x` is… peculiar

  ```js
  get: function(x, y){ return map[x|x][y|y] }
  ```

  `x|x` coerces to int, sure, but `x|0` is the canonical form (and clearer). Same for `y`.

* ### `atan` alias is misleading

  ```js
  var cos = Math.cos, sin = Math.sin, atan = Math.tan, atan2 = Math.atan2;
  ```

  `atan` points to **tan**, not **atan**. You don’t use `atan`, so it’s harmless—but spooky.

* ### ray sort comparator is brittle

  ```js
  ray.sort(function(r1, r2){
    return (sortX)*(r2.x - r1.x) - (sortY)*(r2.y - r1.y);
  })
  ```

  It “works” for your grid steps, but it’s not a total ordering and can behave oddly at equal x/y. It’s fine for 2013; just worth noting.

* ### perf footgun: full-frame `getImageData`/`putImageData`

  On the front canvas you write every pixel every frame. With 100×(100\*ratio) it’s OK; scale that up and it becomes a cliff. (Your background trick—1px high strip—is the performance hero.)

# nice micro-polish 

* replace `alert` win screens with an overlay div so the main loop doesn’t freeze on some browsers.
* clamp a few ops: `viewDistance = Math.min(viewDistance, size)` so rays don’t step too far when the map is empty.
* in `playerMove`, compute `d90` once outside the function; it’s constant.
* keyboard: use `e.key` with a fallback to `keyCode` if you ever remaster (keeps QWERTZ/Intl keys saner).

# todo if ever “remaster” it (still under tiny budget)

* **gzip-aware**: repetition is your friend. Don’t golf variable names; let gzip crush repeated tokens and strings.
* swap the pixel loops for **`putImageData` line by line** (scanline writes), which tends to be faster than big blocks on some canvases.
* keep the **fast raycaster** and drop the precise one; visual payoff > complexity here.
* store CPU AI intervals with **`requestAnimationFrame`-paced ticks** (derive dt) and ditch multiple `setInterval`s; simpler shutdown and more coherent motion.
* add a faint 1-px outline or shadow on the white blocks so they sit “above” the goo without blowing the palette.
* sprinkle a touch of noise/ordered-dither in the back gradient to hide banding; costs \~10 lines, buys a lot.
* scale the front canvas with `imageSmoothingEnabled = false` so the pixels stay crunchy at any size.
* tiny “heading tick” on the player (you already draw a mini line) + brief trail would sell motion even more.
* precompute a 256-entry color mix LUT to speed the HSL blends and make the wakes even smoother.

But yeah—stylistically bold, readable, and weird in a good way. I’d still play this. 🎨👾
