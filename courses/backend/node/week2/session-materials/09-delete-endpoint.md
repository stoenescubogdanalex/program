# DELETE endpoint

## `DELETE /api/snippets/:id`

Let's start with a simplified version of the `DELETE /api/snippets/:id` route. First we add the DELETE route to `api/snippets.js`:

```js
// ...

// DELETE /api/snippets/:id
router.delete("/:id", async (request, response) => {
  // TODO
});

// ...
```

To be able to delete a row from the `snippets` table, we need to have an existing snippet. Create a snippet first and note what the snippet ID is.

The DELETE request we want to make will look something like this:

```text
DELETE /api/snippets/1
```

### Exercise: Implement the DELETE endpoint

When we now make a request like:

```text
DELETE /api/snippets/1
```

you should delete the row from the `snippets` table where the snippet ID matches the `:id` parameter in the URL.
