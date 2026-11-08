gloopy
======

A html5 liquidwar

* `gloopy.html` – the original, September 2012. Untouched.
* `gloopy2026.html` – a restart of the 2012 version: same game, same look, rebuilt from scratch.

What changed in the restart:

* Goo follows a real distance field around the walls (Dial's algorithm, 5/7 chamfer
  costs), so blobs pour around obstacles instead of piling up against them.
* Flat typed arrays for the map and points; point hand-over between players is O(1).
* Hue mixing goes around the colour wheel properly, and a capture happens when a
  point looks more like the attacker than its owner (works for the grey player too,
  so all 13 opponents are playable).
* Fights take several hits; points next to friends heal back toward their team colour.
* CPU brains run inside the animation frame instead of separate `setInterval`s and
  steer smoothly instead of bang-bang.
* Win/lose shows in the menu rather than an `alert`; Esc quits a round; HUD bar
  shows everyone's share of the goo.
* Crisp pixel scaling, phone layout and touch controls (drag left half to move and
  strafe, right half to move and turn).
* The cut `Arena2` map is back.
