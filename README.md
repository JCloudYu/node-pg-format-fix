node-pg-format
==============

This is a minor fix to the archived repo which contians a little bug when the global absolute path is rewritten.

Node.js implementation of [PostgreSQL format()](http://www.postgresql.org/docs/9.3/static/functions-string.html#FUNCTIONS-STRING-FORMAT) to safely create dynamic SQL queries. SQL identifiers and literals are escaped to help prevent SQL injection. The behavior is equivalent to [PostgreSQL format()](http://www.postgresql.org/docs/9.3/static/functions-string.html#FUNCTIONS-STRING-FORMAT). This module also supports Node buffers, arrays, and objects which is explained [below](#arrobject).

## Install

    npm install pg-format

## Example
```js
var format = require('pg-format');
var sql = format('SELECT * FROM %I WHERE my_col = %L %s', 'my_table', 34, 'LIMIT 10');
console.log(sql); // SELECT * FROM my_table WHERE my_col = '34' LIMIT 10
```

## API

### format(fmt, ...)
Returns a formatted string based on ```fmt``` which has a style similar to the C function ```sprintf()```.
* ```%%``` outputs a literal ```%``` character.
* ```%I``` outputs an escaped SQL identifier.
* ```%L``` outputs an escaped SQL literal.
* ```%s``` outputs a simple string.

#### Argument position
You can define where an argument is positioned using ```n$``` where ```n``` is the argument index starting at 1.
```js
var format = require('pg-format');
var sql = format('SELECT %1$L, %1$L, %L', 34, 'test');
console.log(sql); // SELECT '34', '34', 'test'
```

### format.config(cfg)
Changes the global configuration. You can change which letters are used to denote identifiers, literals, and strings in the formatted string. This is useful when the formatted string contains a PL/pgSQL function which calls [PostgreSQL format()](http://www.postgresql.org/docs/9.3/static/functions-string.html#FUNCTIONS-STRING-FORMAT) itself.
```js
var format = require('pg-format');
format.config({
    pattern: {
        ident: 'V',
        literal: 'C',
        string: 't'
    }
});
format.config(); // reset to default
```

### format.ident(input)
Returns the input as an escaped SQL identifier string. ```undefined```, ```null```, and objects will throw an error.

### format.literal(input)
Returns the input as an escaped SQL literal string. ```undefined``` and ```null``` will return ```'NULL'```;

### format.string(input)
Returns the input as a simple string. ```undefined``` and ```null``` will return an empty string. If an array element is ```undefined``` or ```null```, it will be removed from the output string.

### format.withArray(fmt, array)
Same as ```format(fmt, ...)``` except parameters are provided in an array rather than as function arguments. This is useful when dynamically creating a SQL query and the number of parameters is unknown or variable.

## <a name="buffer"></a> Node Buffers
Node buffers can be used for literals (```%L```) and strings (```%s```), and will be converted to [PostgreSQL bytea hex format](http://www.postgresql.org/docs/9.3/static/datatype-binary.html).

## <a name="arrobject"></a> Arrays and Objects
For arrays, each element is escaped when appropriate and concatenated to a comma-delimited string. Nested arrays are turned into grouped lists (for bulk inserts), e.g. [['a', 'b'], ['c', 'd']] turns into ('a', 'b'), ('c', 'd'). Nested array expansion can be used for literals (```%L```) and strings (```%s```), but not identifiers (```%I```).  
For objects, ```JSON.stringify()``` is called and the resulting string is escaped if appropriate. Objects can be used for literals (```%L```) and strings (```%s```), but not identifiers (```%I```). See the example below.

```js
var format = require('pg-format');

var myArray = [ 1, 2, 3 ];
var myObject = { a: 1, b: 2 };
var myNestedArray = [['a', 1], ['b', 2]];

var sql = format('SELECT * FROM t WHERE c1 IN (%L) AND c2 = %L', myArray, myObject);
console.log(sql); // SELECT * FROM t WHERE c1 IN ('1','2','3') AND c2 = '{"a":1,"b":2}'

sql = format('INSERT INTO t (name, age) VALUES %L', myNestedArray); 
console.log(sql); // INSERT INTO t (name, age) VALUES ('a', '1'), ('b', '2')
```

## Testing

```
npm install
npm test
```