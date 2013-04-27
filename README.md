vs-grammar
==========

Tool for generating parsers in accordance with given specifications


Installation
------------

Like many other node modules, vs-grammar can be easily installed by typing `npm install -g vs-grammar` or `npm install vs-grammar` in your terminal. Global install `-g` enables easy access to the useful shell scripts that come with the module.


Syntax
------

Grammar rules are described using BNF-like notation:

```
<           > = /\s+/

<NAME>        = /<[0-9A-Za-z]+>/
<REGEX>       = /\/(?:\\\/|[^\/])*?\//
<STRING>      = /'(?:\\'|[^'])*?'|"(?:\\"|[^"])*?"/
<SPECIAL>     = /[<>()|=]/
<QUANTIFIER>  = /[?*+]|\{(?:\d*,)?\d*\}/

<grammar>     = ( <skip> | <rule> ) +
<rule>        = <NAME> '=' <expression>
<expression>  = <alternative> ( '|' <alternative> ) *
<alternative> = <element> +
<element>     = ( <group> | <NAME> | <REGEX> | <STRING> ) <QUANTIFIER> *
<group>       = '(' <expression> ')'

<skip>        = '<' '>' '=' <REGEX>
```

Skip is a special rule used for ignoring meaningless elements that exist solely for human readability.

Let's use the following grammar description for illustration:

```
<        > = /\s+/

<regular>  = /[a-z]+/
<capital>  = /[A-Z][a-z]*/

<sentence> = <capital> ( <regular> + ) ? '.'
```


Creation
--------

There are three ways of creating grammar with vs-grammar module.


1. Run a shell script to generate javascript code (for global install):

   ```
   vs-grammar path/to/grammar/description path/to/generated/grammar.js
   ```

   Include the generated file in your code:

   ```javascript
   var grammar = require('./path/to/generated/grammar');
   ```
  
2. Generate grammar from a file:

   ```javascript
   var grammar = require('vs-grammar').generate('path/to/grammar/description');
   ```

3. Generate grammar from an explicit description:

   ```javascript
   var grammar = require('vs-grammar').generate([
     "<       >  = /\\s+/",
     "<regular>  = /[a-z]+/",
     "<capital>  = /[A-Z][a-z]*/",
     "<sentence> = <capital> ( <regular> + ) ? '.'"
   ]);
   ```


Parsing
-------

Simply ask created grammar to parse text with a specific rule:

```javascript
var result = grammar.parse.sentence('This is a valid sentence.');
```

Parsed result is a nested/tree structure created by mapping rule elements:

| Rule Element                         | Parsed Representation                                   |
| ------------------------------------ | ------------------------------------------------------- |
| `'string'`                           | `'matched string'`                                      |
| `/regexp/`                           | `'matched string'`                                      |
| `( group )`                          | `{ type: '', matched: [ <matched element> ] }`          |
| `<rule>`                             | `{ type: 'rule name', matched: [ <matched element> ] }` |
| `<element> {m,n}` where `m <= n > 1` | `[ <matched element> ]`                                 |
| `<element> ?`                        | `<matched element>` or `null`                           |

```
{
  type: 'sentence',
  matches: [
    {
      type: 'capital',
      matches: [ 'This' ]
    },
    {
      type: '',
      matches: [
        [
          {
            type: 'regular',
            matches: [ 'is' ]
          },
          {
            type: 'regular',
            matches: [ 'a' ]
          },
          {
            type: 'regular',
            matches: [ 'valid' ]
          },
          {
            type: 'regular',
            matches: [ 'sentence' ]
          }
        ]
      ]
    },
    '.'
  ]
}
```


Mapping
-------

Parsed result as a nested/tree structure is still a bit raw... Mapping allows to turn this generic representation into instances of the right classes by providing corresponding constructors or wrappers:

```javascript
var sentence = result.map({ sentence: Sentence, regular: String, capital: String });
```

The process of mapping traverses through the result tree and applies lists of matches to the corresponding constructors or wrappers. By default, arrays are used for the unmapped rules.


Example
-------

Here is everything brought together in one place:

```javascript
var Sentence = function Sentence ( matches ) {
  if ( !(this instanceof Sentence) ) return new Sentence(matches);

  this.words = [ matches[0] ].concat(matches[1][0]);
}

Sentence.prototype.count = function count ( ) {
  return this.words.length;
}

Sentence.prototype.toString = function toString ( ) {
  return this.words.join(' ') + '.';
}

var grammar = require('vs-grammar').generate([
  "<       >  = /\\s+/",
  "<regular>  = /[a-z]+/",
  "<capital>  = /[A-Z][a-z]*/",
  "<sentence> = <capital> ( <regular> + ) ? '.'"
]);

var result = grammar.parse.sentence('This is a valid sentence.');

var sentence = result.map({ sentence: Sentence, regular: String, capital: String });

console.log('"' + sentence + '"' + ' contains ' + sentence.count() + ' words');
```

Output produced by the code:

```
"This is a valid sentence." contains 5 words
```


License
=======

# MIT #
