# Common lisp

## Pattern matching

_Pattern matching_ arguably improves code readability.
It achieves that by replacing code manipulating a structure with an explicit representation,
which, in turn, makes it easier to grasp how data actually looks like.

Common Lisp has a plethora of pattern matching libraries.
In general it's worth to pick a widely used one, however for small programs it might be an overkill.
```lisp
(defmacro if-matches (pattern value then else)
  (cond
    ((eq '_ pattern) then) ; wild card pattern matches everything
    ((and pattern (symbolp pattern)) ; note how '() is still a symbol
     `(let ((,pattern ,value)) ,then))
    ((or (atom pattern) (eq 'quote (first pattern))) ; match atoms and quotes lists as is
     `(if (equal ,pattern ,value) ,then ,else))
    ((consp pattern)
     (let ((ev (gensym)) (elsef (gensym)))
       `(let ((,ev ,value)) ; avoid double evaluation
          (flet ((,elsef () ,else))
            (if (consp ,ev)
              (if-matches ,(car pattern) (car ,ev)
                (if-matches ,(cdr pattern) (cdr ,ev)
                  ,then
                  (,elsef))
                (,elsef))
              (,elsef))))))))
```

One could chain `if-matches` macros into a more conventional `match` macro:
```lisp
(defmacro match (value &rest patterns)
  (if (not patterns)
    `(error "no matching clause for ~a" ,value)
    (destructuring-bind ((pat &rest body) &rest pats) patterns
      (if (eq :else pat)
        `(progn ,@body)
        (let ((ev (gensym)))
          `(let ((,ev ,value)) ; avoid double evaluation
             (if-matches ,pat ,ev
               (progn ,@body)
               (match ,ev ,@pats))))))))
```

Another useful macro: structure type case with slots being automatically bound to their names:
```lisp
(defmacro match-struct (instance &rest clauses)
  (let ((i (gensym)) (c (gensym)))
    `(let* ((,i ,instance) (,c (type-of ,i)))
      (cond
        ,@(mapcar
            (lambda (clause)
              (destructuring-bind ((type &rest slots) &rest body) clause
                `((subtypep ,c (quote ,type)) (with-slots (,@slots) ,i ,@body))))
            clauses)
        (:else
          (error "no matching clause for type ~a" ,c))))))
```

Example use:
```lisp
(defstruct vec2d x y)
(defstruct vec3d x y z)
(defun norm (v)
  (match-struct v
    ((vec2d x y)
     (sqrt (expt x 2) (expt y 2)))
    ((vec3d x y z)
     (sqrt (expt x 2) (expt y 2) (expt z 3)))))
```
