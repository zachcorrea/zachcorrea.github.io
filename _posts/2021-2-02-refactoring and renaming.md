---
layout: post
title: Refactoring and Renaming
---

After completing another round of the bowling-game kata, I decide to try my hand at refactoring and renaming functions. Though the code is already readable, I wanted it to be a bit more understandable for my own sake. The example src code brought me here:


```clojure
(ns bowling-game.core)

(defn- sum [rolls]
  (reduce + rolls))

(defn- spare? [rolls]
  (= 10 (sum (take 2 rolls))))

(defn- strike? [rolls]
  (= 10 (first rolls)))

(defn- rolls-for-frame [rolls]
  (if (or (strike? rolls) (spare? rolls))
    (take 3 rolls)
    (take 2 rolls)))

(defn- rest-rolls [rolls]
  (if (= 10 (first rolls))
    (drop 1 rolls)
    (drop 2 rolls)))

(defn- to-frames [rolls]
  (lazy-seq (cons (rolls-for-frame rolls)
                  (to-frames (rest-rolls rolls)))))
				  
(defn score [rolls]
  (sum (map sum (take 10 (to-frames rolls)))))
```

The first thing I notice is the function `rest-rolls`. This can be refactored with the new `spare?` function that was already created. making it read as the following instead:

```clojure
(defn- rest-rolls [rolls]
	(if (strike? rolls)
	(drop 1 rolls)
	(drop 2 rolls)))
```

That's a little bit cleaner, but I think I can refactor a little more. Looking at this `score` function, I think it's doing two things, when it could really be doing just one.

```clojure
(defn score [rolls]
  (sum (map sum (take 10 (to-frames rolls)))))
```

currently, It is taking a series of 10 frame rolls being passed into it (eg. `[5 5 3] [3 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0]`) and summing the numbers within each vector. This results in a series that looks like this: `[13] [3] [0] [0] [0] [0] [0] [0] [0] [0]` That series then gets passed into another sum function which adds each frame roll together into a singel total score (eg. `[16]`). I want to separate these into two separate functions and name them a bit more distinctly. 

I'll rename the `score` function to `total-score` (eg. `[16]` ) in effort to distinguish it from our refactored function `frame-scores` (eg. `[13] [3] [0] [0] [0] [0] [0] [0] [0] [0]`) which sequences the sum of the rolls for each frame.

```clojure
(defn- frame-scores [rolls]
	(map sum (take 10 (all-frame-rolls rolls))))
						
(defn total-score [rolls]
	(sum (frame-scores rolls)))
```

I wish to rename the `to-frames` function to something that more clearly depicts its job of sequencing a list of the all the rolls for each frame (eg. `[5 5 3] [3 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0] [0 0]` ). let's try `all-frame-rolls`

```clojure
(defn- all-frame-rolls [rolls]
	(lazy-seq (cons (frame-roll rolls)
					(all-frame-rolls (rest-rolls rolls)))))

```
Now I want to rename the `rolls-for-frame` function to create a creater association between it and the `all-frame-rolls`. So I'll name it `frame-rolls`, because it determines how many rolls will be in each frame.

```clojure
(defn- frame-rolls [rolls]
	(if (or (strike? rolls) (spare? rolls))
	(take 3 rolls)
	(take 2 rolls)))
```
Now I feel as though I've done enough damage. I save the file and check to make sure that all my tests still pass...

```clojure
bowling
- gutter game returns a total score of zero
- one-pin game returns a total score of twenty
- spare adds first roll of next frame to frame score
- strike adds first and second rolls of next frame to frame score
- perfect game returns a total score of 300

Finished in 0.00111 seconds
5 examples, 0 failures

```

They do! so now my refactored and renamed src code in its entirety is:

```clojure
(ns bowling-game.core)

(defn- sum [rolls]
	(reduce + rolls))
	
(defn- spare? [rolls]
	(= 10 (sum (take 2 rolls))))
	
(defn- strike? [rolls]
	(= 10 (first rolls)))	

(defn- frame-rolls [rolls]
	(if (or (strike? rolls) (spare? rolls))
	(take 3 rolls)
	(take 2 rolls)))
		
(defn- rest-rolls [rolls]
	(if (strike? rolls)
	(drop 1 rolls)
	(drop 2 rolls)))

(defn- all-frame-rolls [rolls]
	(lazy-seq (cons (frame-rolls rolls)
					(all-frame-rolls (rest-rolls rolls)))))

(defn- frame-scores [rolls]
	(map sum (take 10 (all-frame-rolls rolls))))
						
(defn total-score [rolls]
	(sum (frame-scores rolls)))
```