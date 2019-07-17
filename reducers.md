# Reducers

## Storing data on Maps

### Reducers

Imagine we need to fetch an external resource, that will later be stored in a reducer like so:

```tsx
// component
const products = await fetch('some-products-url');
dispatch({
  type: 'PRODUCTS_FETCHED',
  payload: products
});

// reducer
// initialState = []
function productsReducer(state = initialState, action) {
  switch ( action.payload ) {
    case 'PRODUCTS_FETCHED':
      return [
        ...state,
        ...action.payload
      ]
  }
}
```

Now, there are many problems with this approach, so let's imagine the following case:

What happens if somehow our data fetching strategy has a bug, where we might store the same product twice in our reducer? A good way to solve this problem would be using an object instead of an array:

```tsx
// initialState = {};
case 'PRODUCTS_FETCHED':
	return action.payload
    .reduce((productList, product) => ({ ...productList, [product.id]: product }), state);
```

So now we have a object with the id as the keys, and the entity as the values. But there are still many things to improve...

A better approach would be to make use of ES6 [Maps](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Map), like so:

```tsx
// initialState = new Map();
return new Map(
    action.payload
      .reduce(
        (productList, product) => productList.set(product.id, product),
        state
      )
  );
```

With this approach, we get several benefits:
1.  We have the unique identifier
2.  We avoid duplication by overwriting the previous result
3.  We still only loop over the new elements
4.  We only create a new instance at the end of the operation, (and not in every iteration) so we don't lose performance over immutability
5.  We now access a native `.size` attribute, to quickly figure how many elements we have stored
6.  We can access native `.has`, `get` and `.delete` helpful methods to manage our data
7.  We can easily achieve immutability taking advantage of the following:
    1.  Every `Map` operation returns the modified Map
    2.  We can create a Map by passing another Map as a parameter

We could create a simple helper method to reuse this strategy on many reducers:

```tsx
export const createMapFromArray = (array, initialMap = new Map(), identifier = 'id') => {
  const newMap = new Map(initialMap);
  for (item of array) {
    newMap.set(item[identifier], item);
  }
  return newMap;
}
```

The `for` loop is the most performant way to loop over an array ([here](https://github.com/dg92/Performance-analysis-es6)'s the proof), so again we are gaining even more on the performance side.

> **Note**: This transformation can also be done in the action creator. It is up to you. Just be consistent about it. Sometimes even a consistent bad practice is better than inconsistent good practices.

### Moving on to React Components

So, we made huge improvements on the data store side. But now we need to render components based on that stucture. And we dont have the `Array.prototype.map`, so what do we do?

Well, ES6 got us covered again! We can do this:

```tsx
function SomeComponent({ products }) {
  return (
    <ul>
    {Array.from(
      products,
      [id, product] => <Product key={id} product={product} />
    )}
    </ul>
  );
}
```

[Array.from](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/from) can receive a Map as a first paramenter, and the mapper function as second parameter! So not only do we get a clean and easy way to render our products, but we also get the unique keys aswell!


