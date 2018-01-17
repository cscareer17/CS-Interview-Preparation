# Regex CheatSheet

Practise with regex101 for one hour.
 
*: match between zero and one  
\w: characters and numbers  
\d: any digit  
\s: whitespace  
.: any single character  
\S\W\D: any non whitespace, character, digit  

seq.*seq: match as much character as possible in between   
seq*?seq: match as few character as possible in between  
match{3}: match a sequence three times  
match{3,}: match a sequence three times or more  
(?:) = (): non capturing group VS capturing group  
*

word(?=whatever): positive lookahead (must contain)  
word(?!whatever): negative lookahead (must not contain)  
(?<=whatever)word: positive lookbehind  
(?< !whatever)word: negative lookbehind  

### Flags
* (?i): case insensitive. 
* (?c): case sensitive

More [here](https://www.regular-expressions.info/modifiers.html)
