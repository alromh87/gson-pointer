# Json Pointer

This is a json-pointer implementation following [RFC 6901](https://tools.ietf.org/html/rfc6901).
As the _error handling_ is not further specified, this implementation will return `undefined` for any invalid
pointer/missing data, making it very handy to check uncertain data, i.e.

```js
if (pointer.get({}, "/path/to/nested/item") !== undefined) {
  // value is set, do something
}

// instead of
if (path && path.to && path.to.nested && path.to.nested.item) {
  // value is set, do something
}
```

**install with** `npm i gson-pointer --save`


Besides the standard `get` function, this library offers additional functions to work with json-pointer

| method								| description
| ------------------------------------- | -------------------------------------------------------------
| get (data, pointer) -> [mixed]  		| returns the value at given pointer
| set (data, pointer, value) -> data  	| sets the value at the given path
| delete (data, pointer) -> data  		| removes a property from data
| join (pointer*) -> pointer  			| joins multiple pointers to a single one
| split (pointer) -> [array]  			| returns a json-pointer as an array


## Examples

### pointer.get(data, pointer)

```js
const gp = require("gson-pointer");

const data = {
	parent: {
		child: {
			title: "title of child"
		}
	}
}

const titleOfChild = gp.get(data, "/parent/child/title"); // output: "title of child"
console.log(gp.get(data, "/parent/missing/path")); // output: undefined
```


### pointer.set(data, pointer, value)

set values on an object

```js
const gp = require("gson-pointer");

var data = {
	parent: {
		children: [
			{
				title: "title of child"
			}
		]
	}
};

pointer.set(data, "#/parent/children/1", {title: "second child"});
console.log(data.parent.children.length); // output: 2
```

and may create data on its path

```js
const gp = require("gson-pointer");
const data = gp.set({}, "/list/[]/value", 42);
console.log(data); // output: { list: [ { value: 42 } ] }
```


### pointer.delete(data, pointer)

delete properties or array items

```js
const gp = require("gson-pointer");
const data = gp.delete({ parent: { arrayOrObject: [ 0, 1 ] }}, "#/parent/arrayOrObject/1");
console.log(data.parent.arrayOrObject); // output: [0]
```


### pointer.split(pointer)

returns a json-pointer as a list of properties

```js
const gp = require("gson-pointer");
const list = gp.split("#/parent/arrayOrObject/1");
console.log(list); // output: ["parent", "arrayOrObject", "1"]
```


### pointer.join(pointer, pointer, ...)

joins all arguments to a valid json pointer

```js
const gp = require("gson-pointer");
const key = "my key";
console.log(gp.join("root", key, "/to/target")); // output: "/root/my key/to/target"
```

and joins relative pointers as expected

```js
const gp = require("gson-pointer");
console.log(gp.join("/path/to/value", "../object")); // output: "/path/to/object"
```

in order to join an array received from split, you can use `join.list()` to retrieve a valid pointer

```js
const gp = require("gson-pointer");
const list = gp.split("/my/path/to/child");
list.pop();
console.log(gp.join.list(list)); // output: "/my/path/to"
```


## Fragment identifier

All methods support a leading uri fragment identifier (#), which will ensure that property-values are uri decoded
when resolving the path within data and also ensure that any pointer is returned uri encoded with a leading `#`. e.g.

```js
const gp = require("gson-pointer");

// join
const list = gp.split("/my value/to/child");
list.pop();
console.log(gp.join.list(list)); // output: "/my%20value/to"

// get
const value = gp.get({ "my value": true }, "#/my%20value");
console.log(value); // output: true
```


