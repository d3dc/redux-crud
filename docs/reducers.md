# Reducers

## `.List.reducersFor` and `.Map.reducersFor`

There are `reducersFor` for each type of store:

```js
var reduxCrud = require('redux-crud');
var reducers = reduxCrud.List.reducersFor('users');

// or

var reducers = reduxCrud.Map.reducersFor('users');
```

`reducersFor` creates a reducer function for the given resource. Redux CRUD assumes that all records will have a unique key, e.g. `id`. 

It generates the following reducers:

- `fetchSuccess`
- `createStart`
- `createSuccess`
- `createError`
- `updateStart`
- `updateSuccess`
- `updateError`
- `deleteStart`
- `deleteSuccess`
- `deleteError`

*Note: There are no `fetchStart` and `fetchError` reducers.*

```js
var reduxCrud = require('redux-crud');
var reducers = reduxCrud.List.reducersFor('users');

// reducers =>

function (state, action) {
  switch (action.type) {
    case 'USERS_FETCH_SUCCESS':
      ...
    case 'USERS_CREATE_START':
      ...
    case 'USERS_CREATE_SUCCESS':
      ...
  }
}
```

`reducersFor` takes an optional config object as second argument:

```js
reduxCrud.reducersFor('users', {key: '_id'});
```

__config.key__

Key to be used for merging records. Default: 'id'.

## What each reducer does

### `fetchSuccess`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  records: users,
  type:   'USERS_FETCH_SUCCESS',
}
```

Takes one record or an array of records and adds them to the current state. Uses the given `key` or `id` by default to merge.

### `createStart`

Listens for an action like:

```js
{
  type:   'USERS_CREATE_START',
  record: user,
}
```

Adds the record optimistically to the collection. The record must have a client generated key e.g. `id`, otherwise the reducer will throw an error. This key is necessary for matching records on `createSuccess` and `createError`. This client generated key is just temporary, is not expected that you will use this key when saving your data in the backend, it is just there so records can be matched.

__This action is optional, dispatch this only if you want optimistic creation.__ [Read more about this](#about-optimistic-changes).

For generating keys see [cuid](https://github.com/ericelliott/cuid).

Also adds `busy` and `pendingCreate` to the record so you can display proper indicators in your UI.

### `createSuccess`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_CREATE_SUCCESS',
  record: user,
  cid:    clientGeneratedId
}
```

Takes one record and adds it to the current state. Uses the given `key` (`id` by default) to merge. 

The `cid` attribute is optional but it should be used when dispatching `createStart`. This `cid` will be used for matching the record and replacing it with the saved one.

### `createError`

Listens for an action like:

```js
{
  type: 'USERS_CREATE_ERROR',
  record: user,
}
```

This reducer removes the record from the collection. The record key is used for matching the records. So if a record was added optimistically using `createStart` then the keys must match.

### `updateStart`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:  'USERS_UPDATE_START',
  record: user
}
```

Takes one record and merges it to the current state. Uses the given `key` or `id` by default to merge.

It also add these two properties to the record:
- `busy`
- `pendingUpdate`

You can use this to display relevant information in the UI e.g. a spinner.

### `updateSuccess`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_UPDATE_SUCCESS',
  record: user
}
```

Takes one record and merges it to the current state. Uses the given `key` or `id` by default to merge.

### `updateError`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_UPDATE_ERROR',
  record: user,
  error:  error
}
```

This reducer will remove `busy` from the given record. It will not rollback the record to their previous state as we don't want users to lose their changes. The record will keep the `pendingUpdate` attribute set to true.

## `deleteStart`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_DELETE_START',
  record: user
}
```

Marks the given record as `deleted` and `busy`. This reducer doesn't actually remove it. In your UI you can filter out records with `deleted` to hide them.

## `deleteSuccess`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_DELETE_SUCCESS',
  record: user
}
```

This reducer removes the given record from the store.

## `deleteError`

Listens for an action like this (generated by `actionCreatorsFor`):

```js
{
  type:   'USERS_DELETE_ERROR',
  record: user,
  error:  error
}
```

Removes `deleted` and `busy` from the given record.