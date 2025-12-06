# PUT endpoint

## `PUT /api/snippets/:id`

Let's start with a simplified version of the `PUT /api/snippets/:id` route. First we add the PUT route to `api/snippets.js`:

```js
// ...

// PUT /api/snippets/:id
router.put("/:id", async (request, response) => {
  // TODO
});

// ...
```

To be able to update a row in the `snippets` table, we need to have an existing snippet. Create a snippet first and note what the snippet ID is.

The PUT request we want to make will look something like this:

```text
PUT /api/snippets/1
{
    "title": "Updated snippet title",
    "contents": "#hello world",
    "is_private": true
}
```

### Exercise: Implement the PUT endpoint

When we now make a request like:

```text
PUT /api/snippets/1
{
    "title": "Updated snippet title",
    "contents": "#hello world",
    "is_private": true
}
```

you should update the existing row in the `snippets` table with the data from the request body, where the snippet ID matches the `:id` parameter in the URL.

When updating a snippet we also need to specify a `user_id`. For now, you can just pass in the `user_id` in the request body (alongside the other snippet data).
