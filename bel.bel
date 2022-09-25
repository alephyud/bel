; Bel in Bel. 9 October 2019, 9:14 GMT


(def no (x)
  (id x nil))

(def atom (x)
  (no (id (type x) 'pair)))

(def all (f xs)
  (if (no xs)      t
      (f (car xs)) (all f (cdr xs))
                   nil))

(def some (f xs)
  (if (no xs)      nil
      (f (car xs)) xs
                   (some f (cdr xs))))

(def reduce (f xs)
  (if (no (cdr xs))
      (car xs)
      (f (car xs) (reduce f (cdr xs)))))

(def cons args
  (reduce join args))

(def append args
  (if (no (cdr args)) (car args)
      (no (car args)) (apply append (cdr args))
                      (cons (car (car args))
                            (apply append (cdr (car args))
                                          (cdr args)))))

(def snoc args
  (append (car args) (cdr args)))

(def list args
  (append args nil))

(def map (f . ls)
  (if (no ls)       nil
      (some no ls)  nil
      (no (cdr ls)) (cons (f (car (car ls)))
                          (map f (cdr (car ls))))
                    (cons (apply f (map car ls))
                          (apply map f (map cdr ls)))))

(mac fn (parms . body)
  (if (no (cdr body))
      `(list 'lit 'clo scope ',parms ',(car body))
      `(list 'lit 'clo scope ',parms '(do ,@body))))

(set vmark (join))

(def uvar ()
  (list vmark))

(mac do args
  (reduce (fn (x y)
            (list (list 'fn (uvar) y) x))
          args))

(mac let (parms val . body)
  `((fn (,parms) ,@body) ,val))

(mac macro args
  `(list 'lit 'mac (fn ,@args)))

(mac def (n . rest)
  `(set ,n (fn ,@rest)))

(mac mac (n . rest)
  `(set ,n (macro ,@rest)))

(mac or args
  (if (no args)
      nil
      (let v (uvar)
        `(let ,v ,(car args)
           (if ,v ,v (or ,@(cdr args)))))))

(mac and args
  (reduce (fn es (cons 'if es))
          (or args '(t))))

(def = args
  (if (no (cdr args))  t
      (some atom args) (all [id _ (car args)] (cdr args))
                       (and (apply = (map car args))
                            (apply = (map cdr args)))))

(def symbol (x) (= (type x) 'symbol))

(def pair   (x) (= (type x) 'pair))

(def char   (x) (= (type x) 'char))

(def stream (x) (= (type x) 'stream))

(def proper (x)
  (or (no x)
      (and (pair x) (proper (cdr x)))))

(def string (x)
  (and (proper x) (all char x)))

(def mem (x ys (o f =))
  (some [f _ x] ys))

(def in (x . ys)
  (mem x ys))

(def cadr  (x) (car (cdr x)))

(def cddr  (x) (cdr (cdr x)))

(def caddr (x) (car (cddr x)))

(mac case (expr . args)
  (if (no (cdr args))
      (car args)
      (let v (uvar)
        `(let ,v ,expr
           (if (= ,v ',(car args))
               ,(cadr args)
               (case ,v ,@(cddr args)))))))

(mac iflet (var . args)
  (if (no (cdr args))
      (car args)
      (let v (uvar)
        `(let ,v ,(car args)
           (if ,v
               (let ,var ,v ,(cadr args))
               (iflet ,var ,@(cddr args)))))))

(mac aif args
  `(iflet it ,@args))

(def find (f xs)
  (aif (some f xs) (car it)))

(def begins (xs pat (o f =))
  (if (no pat)               t
      (atom xs)              nil
      (f (car xs) (car pat)) (begins (cdr xs) (cdr pat) f)
                             nil))

(def caris (x y (o f =))
  (begins x (list y) f))

(def hug (xs (o f list))
  (if (no xs)       nil
      (no (cdr xs)) (list (f (car xs)))
                    (cons (f (car xs) (cadr xs))
                          (hug (cddr xs) f))))

(mac with (parms . body)
  (let ps (hug parms)
    `((fn ,(map car ps) ,@body)
      ,@(map cadr ps))))

(def keep (f xs)
  (if (no xs)      nil
      (f (car xs)) (cons (car xs) (keep f (cdr xs)))
                   (keep f (cdr xs))))

(def rem (x ys (o f =))
  (keep [no (f _ x)] ys))

(def get (k kvs (o f =))
  (find [f (car _) k] kvs))

(def put (k v kvs (o f =))
  (cons (cons k v)
        (rem k kvs (fn (x y) (f (car x) y)))))

(def rev (xs)
  (if (no xs)
      nil
      (snoc (rev (cdr xs)) (car xs))))

(def snap (xs ys (o acc))
  (if (no xs)
      (list acc ys)
      (snap (cdr xs) (cdr ys) (snoc acc (car ys)))))

(def udrop (xs ys)
  (cadr (snap xs ys)))

(def idfn (x)
  x)

(def is (x)
  [= _ x])

(mac eif (var (o expr) (o fail) (o ok))
  (with (v (uvar)
         w (uvar)
         c (uvar))
    `(let ,v (join)
       (let ,w (ccc (fn (,c)
                      (dyn err [,c (cons ,v _)] ,expr)))
         (if (caris ,w ,v id)
             (let ,var (cdr ,w) ,fail)
             (let ,var ,w ,ok))))))

(mac onerr (e1 e2)
  (let v (uvar)
    `(eif ,v ,e2 ,e1 ,v)))

(mac safe (expr)
  `(onerr nil ,expr))

(def literal (e)
  (or (in e t nil o apply)
      (in (type e) 'char 'stream)
      (caris e 'lit)
      (string e)))

(def variable (e)
  (if (atom e)
      (no (literal e))
      (id (car e) vmark)))

(def isa (name)
  [begins _ `(lit ,name) id])

(def bel (e (o g globe))
  (ev (list (list e nil))
      nil
      (list nil g)))

(def mev (s r (p g))
  (if (no s)
      (if p
          (sched p g)
          (car r))
      (sched (if (cdr (binding 'lock s))
                 (cons (list s r) p)
                 (snoc p (list s r)))
             g)))

(def sched (((s r) . p) g)
  (ev s r (list p g)))

(def ev (((e a) . s) r m)
  (aif (literal e)            (mev s (cons e r) m)
       (variable e)           (vref e a s r m)
       (no (proper e))        (sigerr 'malformed s r m)
       (get (car e) forms id) ((cdr it) (cdr e) a s r m)
                              (evcall e a s r m)))

(def vref (v a s r m)
  (let g (cadr m)
    (if (inwhere s)
        (aif (or (lookup v a s g)
                 (and (car (inwhere s))
                      (let cell (cons v nil)
                        (xdr g (cons cell (cdr g)))
                        cell)))
             (mev (cdr s) (cons (list it 'd) r) m)
             (sigerr 'unbound s r m))
        (aif (lookup v a s g)
             (mev s (cons (cdr it) r) m)
             (sigerr (list 'unboundb v) s r m)))))

(set smark (join))

(def inwhere (s)
  (let e (car (car s))
    (and (begins e (list smark 'loc))
         (cddr e))))

(def lookup (e a s g)
  (or (binding e s)
      (get e a id)
      (get e g id)
      (case e
        scope (cons e a)
        globe (cons e g))))

(def binding (v s)
  (get v
       (map caddr (keep [begins _ (list smark 'bind) id]
                        (map car s)))
       id))

(def sigerr (msg s r m)
  (aif (binding 'err s)
       (applyf (cdr it) (list msg) nil s r m)
       (err 'no-err)))

(mac fu args
  `(list (list smark 'fut (fn ,@args)) nil))

(def evmark (e a s r m)
  (case (car e)
    fut  ((cadr e) s r m)
    bind (mev s r m)
    loc  (sigerr 'unfindable s r m)
    prot (mev (cons (list (cadr e) a)
                    (fu (s r m) (mev s (cdr r) m))
                    s)
              r
              m)
         (sigerr 'unknown-mark s r m)))

(set forms (list (cons smark evmark)))

(mac form (name parms . body)
  `(set forms (put ',name ,(formfn parms body) forms)))

(def formfn (parms body)
  (with (v  (uvar)
         w  (uvar)
         ps (parameters (car parms)))
    `(fn ,v
       (eif ,w (apply (fn ,(car parms) (list ,@ps))
                      (car ,v))
               (apply sigerr 'bad-form (cddr ,v))
               (let ,ps ,w
                 (let ,(cdr parms) (cdr ,v) ,@body))))))

(def parameters (p)
  (if (no p)           nil
      (variable p)     (list p)
      (atom p)         (err 'bad-parm)
      (in (car p) t o) (parameters (cadr p))
                       (append (parameters (car p))
                               (parameters (cdr p)))))

(form quote ((e) a s r m)
  (mev s (cons e r) m))

(form if (es a s r m)
  (if (no es)
      (mev s (cons nil r) m)
      (mev (cons (list (car es) a)
                 (if (cdr es)
                     (cons (fu (s r m)
                             (if2 (cdr es) a s r m))
                           s)
                     s))
           r
           m)))

(def if2 (es a s r m)
  (mev (cons (list (if (car r)
                       (car es)
                       (cons 'if (cdr es)))
                   a)
             s)
       (cdr r)
       m))

(form where ((e (o new)) a s r m)
  (mev (cons (list e a)
             (list (list smark 'loc new) nil)
             s)
       r
       m))

(form dyn ((v e1 e2) a s r m)
  (if (variable v)
      (mev (cons (list e1 a)
                 (fu (s r m) (dyn2 v e2 a s r m))
                 s)
           r
           m)
      (sigerr 'cannot-bind s r m)))

(def dyn2 (v e2 a s r m)
  (mev (cons (list e2 a)
             (list (list smark 'bind (cons v (car r)))
                   nil)
             s)
       (cdr r)
       m))

(form after ((e1 e2) a s r m)
  (mev (cons (list e1 a)
             (list (list smark 'prot e2) a)
             s)
       r
       m))

(form ccc ((f) a s r m)
  (mev (cons (list (list f (list 'lit 'cont s r))
                   a)
             s)
       r
       m))

(form thread ((e) a s r (p g))
  (mev s
       (cons nil r)
       (list (cons (list (list (list e a))
                         nil)
                   p)
             g)))

(def evcall (e a s r m)
  (mev (cons (list (car e) a)
             (fu (s r m)
               (evcall2 (cdr e) a s r m))
             s)
       r
       m))

(def evcall2 (es a s (op . r) m)
  (if ((isa 'mac) op)
      (applym op es a s r m)
      (mev (append (map [list _ a] es)
                   (cons (fu (s r m)
                           (let (args r2) (snap es r)
                             (applyf op (rev args) a s r2 m)))
                         s))
           r
           m)))

(def applym (mac args a s r m)
  (applyf (caddr mac)
          args
          a
          (cons (fu (s r m)
                  (mev (cons (list (car r) a) s)
                       (cdr r)
                       m))
                s)
          r
          m))

(def applyf (f args a s r m)
  (if (= f apply)    (applyf (car args) (reduce join (cdr args)) a s r m)
      (caris f 'lit) (if (proper f)
                         (applylit f args a s r m)
                         (sigerr 'bad-lit s r m))
                     (sigerr 'cannot-apply s r m)))

(def applylit (f args a s r m)
  (aif (and (inwhere s) (find [(car _) f] locfns))
       ((cadr it) f args a s r m)
       (let (tag . rest) (cdr f)
         (case tag
           prim (applyprim (car rest) args s r m)
           clo  (let ((o env) (o parms) (o body) . extra) rest
                  (if (and (okenv env) (okparms parms))
                      (applyclo parms args env body s r m)
                      (sigerr 'bad-clo s r m)))
           mac  (applym f (map [list 'quote _] args) a s r m)
           cont (let ((o s2) (o r2) . extra) rest
                  (if (and (okstack s2) (proper r2))
                      (applycont s2 r2 args s r m)
                      (sigerr 'bad-cont s r m)))
                (aif (get tag virfns)
                     (let e ((cdr it) f (map [list 'quote _] args))
                       (mev (cons (list e a) s) r m))
                     (sigerr 'unapplyable s r m))))))

(set virfns nil)

(mac vir (tag . rest)
  `(set virfns (put ',tag (fn ,@rest) virfns)))

(set locfns nil)

(mac loc (test . rest)
  `(set locfns (cons (list ,test (fn ,@rest)) locfns)))

(loc (is car) (f args a s r m)
  (mev (cdr s) (cons (list (car args) 'a) r) m))

(loc (is cdr) (f args a s r m)
  (mev (cdr s) (cons (list (car args) 'd) r) m))

(def okenv (a)
  (and (proper a) (all pair a)))

(def okstack (s)
  (and (proper s)
       (all [and (proper _) (cdr _) (okenv (cadr _))]
            s)))

(def okparms (p)
  (if (no p)       t
      (variable p) t
      (atom p)     nil
      (caris p t)  (oktoparm p)
                   (and (if (caris (car p) o)
                            (oktoparm (car p))
                            (okparms (car p)))
                        (okparms (cdr p)))))

(def oktoparm ((tag (o var) (o e) . extra))
  (and (okparms var) (or (= tag o) e) (no extra)))

(set prims '((id join xar xdr wrb ops)
             (car cdr type sym nom rdb cls stat sys)
             (coin)))

(def applyprim (f args s r m)
  (aif (some [mem f _] prims)
       (if (udrop (cdr it) args)
           (sigerr 'overargs s r m)
           (with (a (car args)
                  b (cadr args))
             (eif v (case f
                      id   (id a b)
                      join (join a b)
                      car  (car a)
                      cdr  (cdr a)
                      type (type a)
                      xar  (xar a b)
                      xdr  (xdr a b)
                      sym  (sym a)
                      nom  (nom a)
                      wrb  (wrb a b)
                      rdb  (rdb a)
                      ops  (ops a b)
                      cls  (cls a)
                      stat (stat a)
                      coin (coin)
                      sys  (sys a))
                    (sigerr v s r m)
                    (mev s (cons v r) m))))
       (sigerr 'unknown-prim s r m)))

(def applyclo (parms args env body s r m)
  (mev (cons (fu (s r m)
               (pass parms args env s r m))
             (fu (s r m)
               (mev (cons (list body (car r)) s)
                    (cdr r)
                    m))
             s)
       r
       m))

(def pass (pat arg env s r m)
  (let ret [mev s (cons _ r) m]
    (if (no pat)       (if arg
                           (sigerr 'overargs s r m)
                           (ret env))
        (literal pat)  (sigerr 'literal-parm s r m)
        (variable pat) (ret (cons (cons pat arg) env))
        (caris pat t)  (typecheck (cdr pat) arg env s r m)
        (caris pat o)  (pass (cadr pat) arg env s r m)
                       (destructure pat arg env s r m))))

(def typecheck ((var f) arg env s r m)
  (mev (cons (list (list f (list 'quote arg)) env)
             (fu (s r m)
               (if (car r)
                   (pass var arg env s (cdr r) m)
                   (sigerr 'mistype s r m)))
             s)
       r
       m))

(def destructure ((p . ps) arg env s r m)
  (if (no arg)   (if (caris p o)
                     (mev (cons (list (caddr p) env)
                                (fu (s r m)
                                  (pass (cadr p) (car r) env s (cdr r) m))
                                (fu (s r m)
                                  (pass ps nil (car r) s (cdr r) m))
                                s)
                          r
                          m)
                     (sigerr 'underargs s r m))
      (atom arg) (sigerr 'atom-arg s r m)
                 (mev (cons (fu (s r m)
                              (pass p (car arg) env s r m))
                            (fu (s r m)
                              (pass ps (cdr arg) (car r) s (cdr r) m))
                            s)
                      r
                      m)))

(def applycont (s2 r2 args s r m)
  (if (or (no args) (cdr args))
      (sigerr 'wrong-no-args s r m)
      (mev (append (keep [and (protected _) (no (mem _ s2 id))]
                         s)
                   s2)
           (cons (car args) r2)
           m)))

(def protected (x)
  (some [begins (car x) (list smark _) id]
        '(bind prot)))

(def function (x)
  (find [(isa _) x] '(prim clo)))

(def con (x)
  (fn args x))

(def compose fs
  (reduce (fn (f g)
            (fn args (f (apply g args))))
          (or fs (list idfn))))

(def combine (op)
  (fn fs
    (reduce (fn (f g)
              (fn args
                (op (apply f args) (apply g args))))
            (or fs (list (con (op)))))))

(set cand (combine and)
     cor  (combine or))

(def foldl (f base . args)
  (if (or (no args) (some no args))
      base
      (apply foldl f
                   (apply f (snoc (map car args) base))
                   (map cdr args))))

(def foldr (f base . args)
  (if (or (no args) (some no args))
      base
      (apply f (snoc (map car args)
                     (apply foldr f base (map cdr args))))))

(def of (f g)
  (fn args (apply f (map g args))))

(def upon args
  [apply _ args])

(def pairwise (f xs)
  (or (no (cdr xs))
      (and (f (car xs) (cadr xs))
           (pairwise f (cdr xs)))))

(def fuse (f . args)
  (apply append (apply map f args)))

(mac letu (v . body)
  (if ((cor variable atom) v)
      `(let ,v (uvar) ,@body)
      `(with ,(fuse [list _ '(uvar)] v)
         ,@body)))

(mac pcase (expr . args)
  (if (no (cdr args))
      (car args)
      (letu v
        `(let ,v ,expr
           (if (,(car args) ,v)
               ,(cadr args)
               (pcase ,v ,@(cddr args)))))))

(def match (x pat)
  (if (= pat t)                t
      (function pat)           (pat x)
      (or (atom x) (atom pat)) (= x pat)
                               (and (match (car x) (car pat))
                                    (match (cdr x) (cdr pat)))))

(def split (f xs (o acc))
  (if ((cor atom f:car) xs)
      (list acc xs)
      (split f (cdr xs) (snoc acc (car xs)))))

(mac when (expr . body)
  `(if ,expr (do ,@body)))

(mac unless (expr . body)
  `(when (no ,expr) ,@body))

(set i0  nil
     i1  '(t)
     i2  '(t t)
     i10 '(t t t t t t t t t t)
     i16 '(t t t t t t t t t t t t t t t t))

(set i< udrop)

(def i+ args
  (apply append args))

(def i- (x y)
  (if (no x) (list '- y)
      (no y) (list '+ x)
             (i- (cdr x) (cdr y))))

(def i* args
  (foldr (fn (x y) (fuse (con x) y))
         i1
         args))

(def i/ (x y (o q))
  (if (no x)   (list q nil)
      (i< x y) (list q x)
               (i/ (udrop y x) y (i+ q i1))))

(def i^ (x y)
  (foldr i* i1 (map (con x) y)))

(def r+ ((xn xd) (yn yd))
  (list (i+ (i* xn yd) (i* yn xd))
        (i* xd yd)))

(def r- ((xn xd) (yn yd))
  (let (s n) (i- (i* xn yd) (i* yn xd))
    (list s n (i* xd yd))))

(def r* ((xn xd) (yn yd))
  (list (i* xn yn) (i* xd yd)))

(def r/ ((xn xd) (yn yd))
  (list (i* xn yd) (i* xd yn)))

(set srzero (list '+ i0 i1)
     srone  (list '+ i1 i1))

(def sr+ ((xs . xr) (ys . yr))
  (if (= xs '-)
      (if (= ys '-)
          (cons '- (r+ xr yr))
          (r- yr xr))
      (if (= ys '-)
          (r- xr yr)
          (cons '+ (r+ xr yr)))))

(def sr- (x y)
  (sr+ x (srinv y)))

(def srinv ((s n d))
  (list (if (and (= s '+) (~= n i0)) '- '+)
        n
        d))

(def sr* ((xs . xr) (ys . yr))
  (cons (if (= xs '-)
            (case ys - '+ '-)
            ys)
        (r* xr yr)))

(def sr/ (x y)
  (sr* x (srrecip y)))

(def srrecip ((s (t n [~= _ i0]) d))
  (list s d n))

(def sr< ((xs xn xd) (ys yn yd))
  (if (= xs '+)
      (if (= ys '+)
          (i< (i* xn yd) (i* yn xd))
          nil)
      (if (= ys '+)
          (~= xn yn i0)
          (i< (i* yn xd) (i* xn yd)))))

(set srnum cadr
     srden caddr)

(def c+ ((xr xi) (yr yi))
  (list (sr+ xr yr) (sr+ xi yi)))

(def c* ((xr xi) (yr yi))
  (list (sr- (sr* xr yr) (sr* xi yi))
        (sr+ (sr* xi yr) (sr* xr yi))))

(def litnum (r (o i srzero))
  (list 'lit 'num r i))

(def number (x)
  (let r (fn (y)
           (match y (list [in _ '+ '-] proper proper)))
    (match x `(lit num ,r ,r))))

(set numr car:cddr
     numi cadr:cddr)

(set rpart litnum:numr
     ipart litnum:numi)

(def real (x)
  (and (number x) (= (numi x) srzero)))

(def inv (x)
  (litnum (srinv:numr x) (srinv:numi x)))

(def abs (x)
  (litnum (cons '+ (cdr (numr x)))))

(def simplify ((s n d))
  (if (= n i0) (list '+ n i1)
      (= n d)  (list s i1 i1)
               (let g (apply i* ((of common factor) n d))
                 (list s (car:i/ n g) (car:i/ d g)))))

(def factor (x (o d i2))
  (if (i< x d)
      nil
      (let (q r) (i/ x d)
        (if (= r i0)
            (cons d (factor q d))
            (factor x (i+ d i1))))))

(def common (xs ys)
  (if (in nil xs ys)
      nil
      (let (a b) (split (is (car xs)) ys)
        (if b
            (cons (car xs)
                  (common (cdr xs) (append a (cdr b))))
            (common (cdr xs) ys)))))

(set buildnum (of litnum simplify))

(def recip (x)
  (with (r (numr x)
         i (numi x))
    (let d (sr+ (sr* r r) (sr* i i))
      (buildnum (sr/ r d)
                (sr/ (srinv i) d)))))

(def + ns
  (foldr (fn (x y)
           (apply buildnum ((of c+ cddr) x y)))
         0
         ns))

(def - ns
  (if (no ns)       0
      (no (cdr ns)) (inv (car ns))
                    (+ (car ns) (inv (apply + (cdr ns))))))

(def * ns
  (foldr (fn (x y)
           (apply buildnum ((of c* cddr) x y)))
         1
         ns))

(def / ns
  (if (no ns)
      1
      (* (car ns) (recip (apply * (cdr ns))))))

(def inc (n) (+ n 1))

(def dec (n) (- n 1))

(def pos (x ys (o f =))
  (if (no ys)        nil
      (f (car ys) x) 1
                     (aif (pos x (cdr ys) f) (+ it 1))))

(def len (xs)
  (if (no xs) 0 (inc:len:cdr xs)))

(def charn (c)
  (dec:pos c chars caris))

(def < args
  (pairwise bin< args))

(def > args
  (apply < (rev args)))

(def list< (x y)
  (if (no x) y
      (no y) nil
             (or (< (car x) (car y))
                 (and (= (car x) (car y))
                      (< (cdr x) (cdr y))))))

(def bin< args
  (aif (all no args)                    nil
       (find [all (car _) args] comfns) (apply (cdr it) args)
                                        (err 'incomparable)))

(set comfns nil)

(def com (f g)
  (set comfns (put f g comfns)))

(com real (of sr< numr))

(com char (of < charn))

(com string list<)

(com symbol (of list< nom))

(def int (n)
  (and (real n) (= (srden:numr n) i1)))

(def whole (n)
  (and (int n) (~< n 0)))

(def pint (n)
  (and (int n) (> n 0)))

(def yc (f)
  ([_ _] [f (fn a (apply (_ _) a))]))

(mac rfn (name . rest)
  `(yc (fn (,name) (fn ,@rest))))

(mac afn args
  `(rfn self ,@args))

(def wait (f)
  ((afn (v) (if v v (self (f))))
   (f)))

(def runs (f xs (o fon (and xs (f (car xs)))))
  (if (no xs)
      nil
      (let (as bs) (split (if fon ~f f) xs)
        (cons as (runs f bs (no fon))))))

(def whitec (c)
  (in c \sp \lf \tab \cr))

(def tokens (xs (o break whitec))
  (let f (if (function break) break (is break))
    (keep ~f:car (runs f xs))))

(def dups (xs (o f =))
  (if (no xs)                   nil
      (mem (car xs) (cdr xs) f) (cons (car xs)
                                      (dups (rem (car xs) (cdr xs) f) f))
                                (dups (cdr xs) f)))

(set simple (cor atom number))

(mac do1 args
  (letu v
    `(let ,v ,(car args)
       ,@(cdr args)
       ,v)))

(def gets (v kvs (o f =))
  (find [f (cdr _) v] kvs))

(def consif (x y)
  (if x (cons x y) y))

(mac check (x f (o alt))
  (letu v
    `(let ,v ,x
       (if (,f ,v) ,v ,alt))))

(mac withs (parms . body)
  (if (no parms)
      `(do ,@body)
      `(let ,(car parms) ,(cadr parms)
         (withs ,(cddr parms) ,@body))))

(mac bind (var expr . body)
  `(dyn ,var ,expr (do ,@body)))

(mac atomic body
  `(bind lock t ,@body))

(def tail (f xs)
  (if (no xs) nil
      (f xs)  xs
              (tail f (cdr xs))))

(set dock rev:cdr:rev)

(def lastcdr (xs)
  (if (no (cdr xs))
      xs
      (lastcdr (cdr xs))))

(set last car:lastcdr)

(def newq ()
  (list nil))

(def enq (x q)
  (atomic (xar q (snoc (car q) x)))
  q)

(def deq (q)
  (atomic (do1 (car (car q))
               (xar q (cdr (car q))))))

(mac set args
  (cons 'do
        (map (fn ((p (o e t)))
               (letu v
                 `(atomic (let ,v ,e
                            (let (cell loc) (where ,p t)
                              ((case loc a xar d xdr) cell ,v))))))
             (hug args))))

(mac zap (op place . args)
  (letu (vo vc vl va)
    `(atomic (with (,vo       ,op
                    (,vc ,vl) (where ,place)
                    ,va       (list ,@args))
               (case ,vl
                 a (xar ,vc (apply ,vo (car ,vc) ,va))
                 d (xdr ,vc (apply ,vo (cdr ,vc) ,va))
                   (err 'bad-place))))))

(mac ++ (place (o n 1))
  `(zap + ,place ,n))

(mac -- (place (o n 1))
  `(zap - ,place ,n))

(mac push (x place)
  (letu v
    `(let ,v ,x
       (zap [cons ,v _] ,place))))

(mac pull (x place . rest)
  (letu v
    `(let ,v ,x
       (zap [rem ,v _ ,@rest] ,place))))

(set cbuf '((nil)))

(def open args
  (let s (apply ops args)
    (push (list s) cbuf)
    s))

(def close (s)
  (pull s cbuf caris)
  (cls s))

(def peek ((o s ins))
  (if ((cor no stream) s)
      (let c (wait (fn ()
                     (atomic (let p (get s cbuf)
                               (or (cdr p)
                                   (aif (bitc s) (xdr p it) nil))))))
        (if (= c 'eof) nil c))
      (car (car s))))

(def rdc ((o s ins))
  (if ((cor no stream) s)
      (let c (wait (fn ()
                     (atomic (let p (get s cbuf)
                               (aif (cdr p)
                                    (do (xdr p nil) it)
                                    (bitc s))))))
        (if (= c 'eof) nil c))
      (deq s)))

(set bbuf nil)

(def bitc ((o s ins))
  (let bits (get s bbuf)
    (aif (gets (rev (cdr bits)) chars)
         (do (pull s bbuf caris)
             (car it))
         (let b (rdb s)
           (if (in b nil 'eof)
               b
               (do (if bits
                       (push b (cdr bits))
                       (push (list s b) bbuf))
                   (bitc s)))))))

(def digit (c (o base i10))
  (mem c (udrop (udrop base i16) "fedcba9876543210")))

(set breakc (cor no whitec (is \;) [get _ syntax]))

(def signc (c)
  (in c \+ \-))

(def intrac (c)
  (in c \. \!))

(set source (cor no stream (cand pair string:car)))

(def read ((o s|source ins) (o (t base [<= 2 _ 16]) 10) (o eof))
  (car (rdex s (srnum:numr base) eof)))

(def saferead ((o s ins) (o alt) (o base 10))
  (onerr alt (read s base alt)))

(def rdex ((o s ins) (o base i10) (o eof) (o share))
  (eatwhite s)
  (let c (rdc s)
    (aif (no c)         (list eof share)
         (get c syntax) ((cdr it) s base share)
                        (list (rdword s c base) share))))

(def eatwhite (s)
  (pcase (peek s)
    whitec  (do (rdc s)
                (eatwhite s))
    (is \;) (do (charstil s (is \lf))
                (eatwhite s))))

(def charstil (s f)
  (if ((cor no f) (peek s))
      nil
      (cons (rdc s) (charstil s f))))

(set syntax nil)

(mac syn (c . rest)
  `(set syntax (put ,c (fn ,@rest) syntax)))

(syn \( (s base share)
  (rdlist s \) base share))

(syn \) args
  (err 'unexpected-terminator))

(syn \[ (s base share)
  (let (e newshare) (rdlist s \] base share)
    (list (list 'fn '(_) e) newshare)))

(syn \] args
  (err 'unexpected-terminator))

(def rdlist (s term base share (o acc))
  (eatwhite s)
  (pcase (peek s)
    no        (err 'unterminated-list)
    (is \.)   (do (rdc s) (rddot s term base share acc))
    (is term) (do (rdc s) (list acc share))
              (let (e newshare) (rdex s base nil share)
                (rdlist s term base newshare (snoc acc e)))))

(def rddot (s term base share acc)
  (pcase (peek s)
    no     (err 'unterminated-list)
    breakc (if (no acc)
               (err 'missing-car)
               (let (e newshare) (hard-rdex s base share 'missing-cdr)
                 (if (car (rdlist s term base share))
                     (err 'duplicate-cdr)
                     (list (apply cons (snoc acc e))
                           newshare))))
           (rdlist s term base share (snoc acc (rdword s \. base)))))

(def hard-rdex (s base share msg)
  (let eof (join)
    (let v (rdex s base eof share)
      (if (id (car v) eof) (err msg) v))))

(set namecs '((bel . \bel) (tab . \tab) (lf . \lf) (cr . \cr) (sp . \sp)))

(syn \\ (s base share)
  (list (pcase (peek s)
          no     (err 'escape-without-char)
          breakc (rdc s)
                 (let cs (charstil s breakc)
                   (if (cdr cs)
                       (aif (get (sym cs) namecs)
                            (cdr it)
                            (err 'unknown-named-char))
                       (car cs))))
        share))

(syn \' (s base share)
  (rdwrap s 'quote base share))

(syn \` (s base share)
  (rdwrap s 'bquote base share))

(syn \, (s base share)
  (case (peek s)
    \@ (do (rdc s)
           (rdwrap s 'comma-at base share))
       (rdwrap s 'comma base share)))

(def rdwrap (s token base share)
  (let (e newshare) (hard-rdex s base share 'missing-expression)
    (list (list token e) newshare)))

(syn \" (s base share)
  (list (rddelim s \") share))

(syn \¦ (s base share)
  (list (sym (rddelim s \¦)) share))

(def rddelim (s d (o esc))
  (let c (rdc s)
    (if (no c)   (err 'missing-delimiter)
        esc      (cons c (rddelim s d))
        (= c \\) (rddelim s d t)
        (= c d)  nil
                 (cons c (rddelim s d)))))

(syn \# (s base share)
  (let name (charstil s ~digit)
    (if (= (peek s) \=)
        (do (rdc s)
            (rdtarget s base name (join) share))
        (aif (get name share)
             (list (cdr it) share)
             (err 'unknown-label)))))

(def rdtarget (s base name cell oldshare)
  (withs (share        (cons (cons name cell) oldshare)
          (e newshare) (hard-rdex s base share 'missing-target))
    (if (simple e)
        (err 'bad-target)
        (do (xar cell (car e))
            (xdr cell (cdr e))
            (list cell newshare)))))

(def rdword (s c base)
  (parseword (cons c (charstil s breakc)) base))

(def parseword (cs base)
  (or (parsenum cs base)
      (if (= cs ".")       (err 'unexpected-dot)
          (mem \| cs)      (parset cs base)
          (some intrac cs) (parseslist (runs intrac cs) base)
                           (parsecom cs base))))

(def parsenum (cs base)
  (if (validi cs base)
      (buildnum srzero (parsei cs base))
      (let sign (check (car cs) signc)
        (let (ds es) (split signc (if sign (cdr cs) cs))
          (and (validr ds base)
               (or (no es) (validi es base))
               (buildnum (parsesr (consif sign ds) base)
                         (if (no es) srzero (parsei es base))))))))

(def validi (cs base)
  (and (signc (car cs))
       (= (last cs) \i)
       (let digs (cdr (dock cs))
         (or (no digs) (validr digs base)))))

(def validr (cs base)
  (or (validd cs base)
      (let (n d) (split (is \/) cs)
        (and (validd n base)
             (validd (cdr d) base)))))

(def validd (cs base)
  (and (all (cor [digit _ base] (is \.)) cs)
       (some [digit _ base] cs)
       (~cdr (keep (is \.) cs))))

(def parsei (cs base)
  (if (cddr cs)
      (parsesr (dock cs) base)
      (if (caris cs \+)
          srone
          (srinv srone))))

(def parsesr (cs base)
  (withs (sign  (if (signc (car cs)) (sym (list (car cs))))
          (n d) (split (is \/) (if sign (cdr cs) cs)))
    (simplify (cons (or sign '+)
                    (r/ (parsed n base)
                        (if d
                            (let rd (parsed (cdr d) base)
                              (if (caris rd i0)
                                  (err 'zero-denominator)
                                  rd))
                            (list i1 i1)))))))

(def parsed (cs base)
  (let (i f) (split (is \.) cs)
    (if (cdr f)
        (list (parseint (rev (append i (cdr f))) base)
              (i^ base
                  (apply i+ (map (con i1) (cdr f)))))
        (list (parseint (rev i) base) i1))))

(def parseint (ds base)
  (if ds
      (i+ (charint (car ds))
          (i* base (parseint (cdr ds) base)))
      i0))

(def charint (c)
  (map (con t) (mem c "fedcba987654321")))

(def parset (cs base)
  (if (cdr (keep (is \|) cs))
      (err 'multiple-bars)
      (let vt (tokens cs \|)
        (if (= (len vt) 2)
            (cons t (map [parseword _ base] vt))
            (err 'bad-tspec)))))

(def parseslist (rs base)
  (if (intrac (car (last rs)))
      (err 'final-intrasymbol)
      (map (fn ((cs ds))
             (if (cdr cs)      (err 'double-intrasymbol)
                 (caris cs \!) (list 'quote (parsecom ds base))
                               (parsecom ds base)))
           (hug (if (intrac (car (car rs)))
                    (cons "." "upon" rs)
                    (cons "." rs))))))

(def parsecom (cs base)
  (if (mem \: cs)
      (cons 'compose (map [parseno _ base] (tokens cs \:)))
      (parseno cs base)))

(def parseno (cs base)
  (if (caris cs \~)
      (if (cdr cs)
          (list 'compose 'no (parseno (cdr cs) base))
          'no)
      (or (parsenum cs base) (sym cs))))

(mac bquote (e)
  (let (sub change) (bqex e nil)
    (if change sub (list 'quote e))))

(def bqex (e n)
  (if (no e)   (list nil nil)
      (atom e) (list (list 'quote e) nil)
               (case (car e)
                 bquote   (bqthru e (list n) 'bquote)
                 comma    (if (no n)
                              (list (cadr e) t)
                              (bqthru e (car n) 'comma))
                 comma-at (if (no n)
                              (list (list 'splice (cadr e)) t)
                              (bqthru e (car n) 'comma-at))
                          (bqexpair e n))))

(def bqthru (e n op)
  (let (sub change) (bqex (cadr e) n)
    (if change
        (list (if (caris sub 'splice)
                  `(cons ',op ,(cadr sub))
                  `(list ',op ,sub))
              t)
        (list (list 'quote e) nil))))

(def bqexpair (e n)
  (with ((a achange) (bqex (car e) n)
         (d dchange) (bqex (cdr e) n))
    (if (or achange dchange)
        (list (if (caris d 'splice)
                  (if (caris a 'splice)
                      `(apply append (spa ,(cadr a)) (spd ,(cadr d)))
                      `(apply cons ,a (spd ,(cadr d))))
                  (caris a 'splice)
                  `(append (spa ,(cadr a)) ,d)
                  `(cons ,a ,d))
              t)
        (list (list 'quote e) nil))))

(def spa (x)
  (if (and x (atom x))
      (err 'splice-atom)
      x))

(def spd (x)
  (pcase x
    no   (err 'splice-empty-cdr)
    atom (err 'splice-atom)
    cdr  (err 'splice-multiple-cdrs)
         x))

(mac comma args
  '(err 'comma-outside-backquote))

(mac comma-at args
  '(err 'comma-at-outside-backquote))

(mac splice args
  '(err 'comma-at-outside-list))

(def print (x (o s outs) (o names (namedups x)) (o hist))
  (aif (simple x)        (do (prsimple x s) hist)
       (ustring x names) (prstring x s names hist)
       (get x names id)  (do (prc \# s)
                             (print (cdr it) s)
                             (if (mem x hist id)
                                 hist
                                 (do (prc \= s)
                                     (if (ustring (cdr x) names)
                                         (prstring x s names (cons x hist))
                                         (prpair x s names (cons x hist))))))
                         (prpair x s names hist)))

(def namedups (x (o n 0))
  (map [cons _ (++ n)] (dups (cells x) id)))

(def cells (x (o seen))
  (if (simple x)      seen
      (mem x seen id) (snoc seen x)
                      (cells (cdr x)
                             (cells (car x) (snoc seen x)))))

(def prc (c (o s outs))
  (if (atom s)
      (aif (get c chars)
           (map [wrb _ s] (cdr it))
           (err 'unknown char))
      (enq c s))
  c)

(def ustring (x names)
  (and x (string x) (~tail [get _ names id] x)))

(def prstring (x s names hist)
  (prc \" s)
  (presc x \" s)
  (prc \" s)
  hist)

(def presc (cs esc s)
  (map (fn (c)
         (if (in c esc \\) (prc \\ s))
         (prc c s))
       cs))

(def prsimple (x s)
  (pcase x
    symbol (prsymbol x s)
    char   (do (prc \\ s) (prc x s))
    stream (map [prc _ s] "<stream>")
    number (prnum (numr x) (numi x) s)
           (err 'cannot-print)))

(def prsymbol (x s)
  (let cs (nom x)
    (let odd (~= (saferead (list cs)) x)
      (if odd (prc \¦ s))
      (presc cs \¦ s)
      (if odd (prc \¦ s)))))

(def prnum (r i s)
  (unless (and (= r srzero) (~= i srzero))
    (if (caris r '-) (prc \- s))
    (map [prc _ s] (rrep (cdr r))))
  (unless (= i srzero)
    (print (car i) s)
    (unless (apply = (cdr i))
      (map [prc _ s] (rrep (cdr i))))
    (prc \i s)))

(def rrep ((n d) (o base i10))
  (append (irep n base)
          (if (= d i1) nil (cons \/ (irep d base)))))

(def irep (x base)
  (if (i< x base)
      (list (intchar x))
      (let (q r) (i/ x base)
        (snoc (irep q base) (intchar r)))))

(def intchar (x)
  (car (udrop x "0123456789abcdef")))

(def prpair (x s names hist)
  (prc \( s)
  (do1 (prelts x s names hist)
       (prc \) s)))

(def prelts ((x . rest) s names hist)
  (let newhist (print x s names hist)
    (if (or (and rest (simple rest))
            (ustring rest names)
            (get rest names id))
        (do (map [prc _ s] " . ")
            (print rest s names newhist))
        (if rest
            (do (prc \sp s)
                (prelts rest s names newhist))
            newhist))))

(def prn args
  (map [do (print _) (prc \sp)] args)
  (prc \lf)
  (last args))

(def pr args
  (map prnice args))

(def prnice (x (o s outs))
  (pcase x
    char   (prc x s)
    string (map [prc _ s] x)
           (print x s nil))
  x)

(def drop (n|whole xs)
  (if (= n 0)
      xs
      (drop (- n 1) (cdr xs))))

(def nth (n|pint xs|pair)
  (if (= n 1)
      (car xs)
      (nth (- n 1) (cdr xs))))

(vir num (f args)
  `(nth ,f ,@args))

(def nchar (n)
  (car ((+ n 1) chars)))

(def first (n|whole xs)
  (if (or (= n 0) (no xs))
      nil
      (cons (car xs)
            (first (- n 1) (cdr xs)))))

(mac catch body
  (letu v
    `(ccc (fn (,v) (bind throw ,v ,@body)))))

(def cut (xs (o start 1) (o end (len xs)))
  (first (- (+ end 1 (if (< end 0) (len xs) 0))
            start)
         (drop (- start 1) xs)))

(mac whenlet (var expr . body)
  `(iflet ,var ,expr (do ,@body)))

(mac awhen args
  `(whenlet it ,@args))

(mac each (var expr . body)
  `(map (fn (,var) ,@body) ,expr))

(def flip (f)
  (fn args (apply f (rev args))))

(def part (f . args)
  (fn rest
    (apply f (append args rest))))

(def trap (f . args)
  (flip (apply part (flip f) (rev args))))

(def only (f)
  (fn args
    (if (car args) (apply f args))))

(def >= args
  (pairwise ~bin< args))

(def <= args
  (apply >= (rev args)))

(def floor (x|real)
  (let (s n d) (numr x)
    (let (f m) (i/ n d)
      (litnum (list s
                    (i+ f (if (or (= s '+) (= m i0))
                              i0
                              i1))
                    i1)))))

(set ceil -:floor:-)

(def mod (x y)
  (* (- (/ x y) (floor (/ x y)))
     y))

(mac whilet (var expr . body)
  (letu (vf vp)
    `((rfn ,vf (,vp)
        (whenlet ,var ,vp ,@body (,vf ,expr)))
      ,expr)))

(mac loop (var init update test . body)
  (letu v
    `((rfn ,v (,var)
        (when ,test ,@body (,v ,update)))
      ,init)))

(mac while (expr . body)
  (letu v
    `(loop ,v ,expr ,expr ,v ,@body)))

(mac til (var expr test . body)
  `(loop ,var ,expr ,expr (no ,test)
     ,@body))

(mac for (var init max . body)
  (letu (vi vm)
    `(with (,vi ,init
            ,vm ,max)
       (loop ,var ,vi (+ ,var 1) (<= ,var ,vm)
         ,@body))))

(mac repeat (n . body)
  `(for ,(uvar) 1 ,n ,@body))

(mac poll (expr f)
  (letu (vr ve vf)
    `((rfn ,vr (,ve ,vf)
        (if (,vf ,ve) ,ve (,vr ,expr ,vf)))
      ,expr
      ,f)))

(mac accum (var . body)
  (letu v
    `(withs (,v   nil
             ,var [push _ ,v])
       ,@body
       (rev ,v))))

(mac nof (n expr)
  (letu v
    `(accum ,v (repeat ,n (,v ,expr)))))

(mac drain (expr (o f 'no))
  (letu v
    `(accum ,v
       (poll ,expr (cor ,f (compose no ,v))))))

(def ^w (x y|whole)
  (apply * (nof y x)))

(def clog2 (n)
  (if (<= n 2) 1 (inc:clog2 (/ n 2))))

(def randlen (n)
  (read (list (nof n (if (coin) \0 \1)))
        2))

(def rand (n|pint)
  (poll (randlen (clog2 n)) [< _ n]))

(mac wipe args
  `(set ,@(fuse [list _ nil] args)))

(mac pop (place)
  `(let (cell loc) (where ,place)
     (let xs ((case loc a car d cdr) cell)
       ((case loc a xar d xdr) cell (cdr xs))
       (car xs))))

(mac clean (f place)
  (letu v
    `(let ,v (compose no ,f)
       (zap [keep ,v _] ,place))))

(mac swap places
  (let vs (map [nof 3 (uvar)] places)
    `(atomic (withs ,(fuse (fn (place (cell loc val))
                             (list (list cell loc)
                                   `(where ,place)
                                   val
                                   `((case ,loc a car d cdr) ,cell)))
                           places
                           vs)
               ,@(map (fn ((cellx locx valx) (celly locy valy))
                        `((case ,locx a xar d xdr) ,cellx ,valy))
                      vs
                      (snoc (cdr vs) (car vs)))))))

(def adjoin (x ys (o f =))
  (if (mem x ys f) ys (cons x ys)))

(mac pushnew (x place (o f '=))
  (letu v
    `(let ,v ,x
       (zap [adjoin ,v _ ,f] ,place))))

(def dedup (xs (o f =))
  (rev (foldl (trap adjoin f) nil xs)))

(def insert (f x ys)
  (if (no ys)        (list x)
      (f x (car ys)) (cons x ys)
                     (cons (car ys) (insert f x (cdr ys)))))

(def sort (f xs)
  (foldr (part insert f) nil (rev xs)))

(set best car:sort)

(def max args
  (best > args))

(def min args
  (best < args))

(def even (n)
  (int (/ n 2)))

(set odd (cand int ~even))

(def round (n)
  (let r (fn (n)
           (withs (f (floor n)
                   d (- n f))
             (if (or (> d 1/2) (and (= d 1/2) (odd f)))
                 (ceil n)
                 f)))
    (if (< n 0) (-:r:- n) (r n))))

(mac withfile (var name dir . body)
  `(let ,var (open ,name ,dir)
     (after (do ,@body) (close ,var))))

(mac from (name . body)
  (letu v
    `(withfile ,v ,name 'in
       (bind ins ,v ,@body))))

(mac to (name . body)
  (letu v
    `(withfile ,v ,name 'out
       (bind outs ,v ,@body))))

(def readall ((o s ins) (o base 10))
  (let eof (join)
    (drain (read s base eof) [id _ eof])))

(def load (name)
  (let eof (join)
    (withfile s name 'in
      (til e (read s 10 eof) (id e eof)
        (bel e)))))

(mac record body
  (letu v
    `(let ,v (newq)
       (bind outs ,v ,@body)
       (car ,v))))

(def prs args
  (record (apply pr args)))

(def array (dims (o default))
  (if (no dims)
      default
      `(lit arr ,@(nof (car dims)
                       (array (cdr dims) default)))))

(vir arr (f args)
  `(aref ,f ,@args))

(def aref (a|isa!arr n . ns)
  (if (no ns)
      (n (cddr a))
      (apply aref (n (cddr a)) ns)))

(def table ((o kvs))
  `(lit tab ,@kvs))

(vir tab (f args)
  `(tabref ,f ,@args))

(def tabref (tab key (o default))
  (aif (get key (cddr tab))
       (cdr it)
       default))

(loc isa!tab (f args a s r m)
  (let e `(list (tabloc ,f ,@(map [list 'quote _] args)) 'd)
    (mev (cons (list e a) (cdr s)) r m)))

(def tabloc (tab key)
  (or (get key (cddr tab))
      (let kv (cons key nil)
        (push kv (cddr tab))
        kv)))

(def tabrem (tab key (o f =))
  (clean [caris _ key f] (cddr tab)))

(set templates (table))

(mac tem (name . fields)
  `(set (templates ',name)
        (list ,@(map (fn ((k v)) `(cons ',k (fn () ,v)))
                     (hug fields)))))

(mac make (name . args)
  `(inst ',name
         (list ,@(map (fn ((k v)) `(cons ',k ,v))
                      (hug args)))))

(def inst (name kvs)
  (aif templates.name
       (table (map (fn ((k . f))
                     (cons k
                           (aif (get k kvs) (cdr it) (f))))
                   it))
       (err 'no-template)))

(def readas (name (o s ins))
  (withs (eof (join)
          v   (read s 10 eof))
    (if (id v eof)  nil
        (isa!tab v) (inst name (cddr v))
                    (err 'inst-nontable))))
