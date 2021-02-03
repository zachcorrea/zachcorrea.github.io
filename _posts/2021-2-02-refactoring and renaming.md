---
layout: post
title: Refactoring and Renaming
---

After completing another round of the bowling-game kata, I decide to try my hand at refactoring and renaming functions. Though the code is already readable, I wanted it to be a bit more understandable for my own sake. The example src code ended up here:


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

