# λ-Calculus Interpreter [![License MIT][badge-license]](LICENSE.txt)
A simple lambda calculus interpreter

## 2 Modes:
#### 1) Give lambda expressions:
1. λ : \\
2. e.g. "\\x.\\y.x"
```
> reduceNF (myparse "(\\x.\\y.zxy)w")
["(\\x.\\y.zxy)w", "\\b.zwb", "zw"]

> reduceNF (myparse "(\\x.\\y.zxy)(wy)")
["(\\x.\\y.zxy)(wy)", "\\b.z(wy)b", "z(wy)"]

> reduceNF (myparse "(\\n.\\f.\\x.nf(fx))(\\f.\\x.fx)")
["(\\n.\\f.\\x.nf(fx))(\\f.\\x.fx)", "\\b.\\c.(\\f.\\x.fx)b(bc)", "\\b.\\c.(\\c.bc)(bc)", "\\b.\\c.b(bc)"]
```
#### 2) Give fixed terms (true, false, chuch 2, etc):
1. true       : chTrue
2. false      : chFalse
3. ifthenelse : chCond
4. church num : church
5. succ       : chSucc
6. plus       : chPlus
7. mult       : chMult
8. exp        : chExp
9. iszero     : chIsZero
10. pair      : chPair
11. fst       : chFst
12. snd       : chSnd
13. and       : chAnd
14. or        : chOr
```
> prettyprint (chSucc (church 2))
"\\f.\\x.f(f(fx))"

> prettyprint (chIsZero (church 0))
"\\x.\\y.x"

> prettyprint (chPlus (chSucc (church 2)) (church 3))
"\\f.\\x.f(f(f(f(f(fx)))))"

> prettyprint (chMult (church 2) (church 3))
"\\f.\\b.f(f(f(f(f(fb)))))"

> prettyprint (chExp (church 2) (church 3))
"\\b.\\c.b(b(b(b(b(b(b(bc)))))))"

> prettyprint (chNot chFalse)
"\\x.\\y.x"

> prettyprint (chSnd (chPair (church 2) (church 3)))
"\\f.\\x.f(f(fx))"

> prettyprint (chOr chFalse  chTrue)
"\\x.\\y.x"
```

2015-2016


[badge-license]: https://img.shields.io/badge/license-MIT-green.svg?style=flat-square

### 3) Extra feature (Fitur tambahan)
by
Ardanto Finkan Septa - 1706039736
Sebagai penyelesaian tugas 5 Pemfung 2020 CSUI
- mengubah bentuk numerik ke bentuk church numeralnya
```
> "2"
Before parsed (\f.\x.f(fx))
Normal form of 2: 
\f.\x.f(fx)
In Integer : 2

> "3"
Before parsed (\f.\x.f(f(fx)))
Normal form of 3:
\f.\x.f(f(fx))
In Integer : 3
```

- mengubah bentuk church numeral ke bentuk numerik
```
> "\\f.\\x.f(f(fx))" 
Before parsed \f.\x.f(f(fx))
Normal form of \f.\x.f(f(fx)):
\f.\x.f(f(fx))
In Integer : 3
```

- operator tambah
```
> "2+2"
Before parsed (\f.\x.f(fx))(\w.\y.\x.y(wyx))(\f.\x.f(fx))
Normal form of 2+2: 
\b.\c.b(b(b(bc)))
In Integer : 4
```

- operator kali
```
> "2*3"
Before parsed \f.((\f.\x.f(fx)))(((\f.\x.f(f(fx))))f)
Normal form of 2*3: 
\f.\b.f(f(f(f(f(fb)))))
In Integer : 6
```

- gabungan
```
> "(\\f.\\x.f(fx))*3" 
Before parsed \f.((\f.\x.f(fx)))(((\f.\x.f(f(fx))))f)
Normal form of (\f.\x.f(fx))*3: 
\f.\b.f(f(f(f(f(fb)))))
In Integer : 6
```


##### Perubahan yang dilakukan
Di interpreter ini, sebuah input akan di parse dan dijadikan sebuah Term. Sebelum melakukan parse, input tersebut dilakukan sebuah operasi "preParse" dimana pada tahap ini dilakukan perubahan string input. Jika dia adalah operator "+" maka ubah ke bentuk (\\w.\\y.\\x.y(wyx)), jika dia adalah elemen dari 0-9, maka ubah ke bentuk churchnya. Berikut adalah kode untuk mengubah bentuk numerik ke church numeralnya
```
church :: Integer -> Term
church 0 = myparse "\\f.\\x.x"
church 1 = myparse "\\f.\\x.fx"
church n = myparse (res!!size) where
    res = reduceNF (myparse ("\\f.\\x.f(("++(prettyprint (church (n-1)))++")fx)"))
    size = ((length res)-1)
```

Pada looping utama, sebelum melakukan parse dengan fungsi myParse, pertama dilakukan dulu preParse untuk mengubah bntuk numerik dan operatornya.
```
loopPrinter = do 
    putStr "> "
    inputStr <- readLn 
    let beforeParsed = preParse inputStr
    putStrLn("Before parsed " ++ beforeParsed)
    let parsedString = myparse beforeParsed  
    putStrLn ("Normal form of " ++ inputStr ++ ": ")
    putStrLn ((reduceNF parsedString)!!((length (reduceNF parsedString))-1))
    putStrLn ("In Integer : " ++ show ( churchToInt ((reduceNF parsedString)!!((length (reduceNF parsedString))-1))))
    loopPrinter
```

##### Kode yang ditambahkan
Untuk melakukan PreParsing, dilakukan folding pada string untuk mengecek apakah string tersebut adalah operator atau bentuk numerik. Jika iya, maka string tersebut akan diganti dengan bentuk churchnya. Disini ada yang spesial dengan * karena interpreter bawaan tidak bisa menanganinya jadi saya menggunanakan fungsi mult untuk menngalikan. Tambahan fungsi charFound untuk mengecek apakah ada * di fungsi, sertia fungsi split untuk membagi.
```
preParse :: [Char] -> [Char]
preParse str = if charFound '*' str then preParse (mult (splitted!!0) (splitted!!1)) else foldr convertToLambda [] str
    where
        splitted = split str

convertToLambda :: Char -> [Char] -> [Char]
convertToLambda chr acc | chr `elem` ['0'..'9'] = ['('] ++ prettyprint (church (read [chr] :: Integer)) ++ [')'] ++ acc
    | chr == '+' = "(\\w.\\y.\\x.y(wyx))" ++ acc
    | otherwise = [chr] ++ acc

mult :: String -> String -> String
mult str1 str2 = "\\f.(" ++ (str1) ++ ")((" ++ (str2) ++")f)"

charFound :: Char -> String -> Bool
charFound _ "" = False
charFound c (x:xs)
    | c == x = True
    | otherwise = charFound c xs

split :: String -> [String]
split [] = [""]
split (c:cs) | c == '*'  = "" : rest
             | otherwise = (c : head rest) : tail rest
    where rest = split cs
```

Untuk mengubah bentuk church ke bentuk numerik, dapat digunakan fungsi berikut, dimana churchTranslator akan mengeluarkan nilai Integer dari sebuah Term. Jika tidak bisa, churchTranslator akan mengeluarkan nothing, dimana nanti akan diubah ke -1 di churchToInt
```
churchToInt :: String -> Integer
churchToInt str = fromMaybe (-1) (churchTranslator (myparse str))

churchTranslator :: Term -> Maybe Integer
churchTranslator (Abstraction s (Abstraction z apps)) = go apps
    where
        go (Var x) | x == z = Just 0
        go (Application (Var f) e) | f == s = (+ 1) <$> go e
        go _ = Nothing
churchTranslator _ = Nothing
```

Batasan Program
- Hanya boleh satu operator saja (tidak bisa menangani contoh 2*2+3)
- Interpreternya cukup aneh dikarenakan fungsi \\f.\\x.fx jadi direduksi ke \f.f di hasil akhirnya. Jadi disini nilai 1 di hasil akhir tidak bisa terbaca karena berbentuk \f.f