<!--[meta]
section: guides
title: New Schema Cheatsheet
[meta]-->

## One to Many (one-sided)

### List config

```javascript
keystone.createList('User', { fields: { name: { type: Text } } });

keystone.createList('Post', {
  fields: {
    title: { type: Text },
    content: { type: Text },
    author: { type: Relationship, ref: 'User', many: false },
  },
});
```

### Migration Strategy

#### PostgreSQL

- No changes are required for these relationships.

## Many to Many (one-sided)

### List config

```javascript
keystone.createList('User', { fields: { name: { type: Text } } });

keystone.createList('Post', {
  fields: {
    title: { type: Text },
    content: { type: Text },
    authors: { type: Relationship, ref: 'User', many: true },
  },
});
```

### Migration Strategy
#### PostgreSQL

- Rename `Post_authors` to `Post_authors_many`.
- Rename `Post_id` to `Post_left_id` and `User_id` to `User_right_id`.

## One to Many (two-sided)

### List config

```javascript
keystone.createList('User', {
  fields: {
    name: { type: Text },
    posts: { type: Relationship, ref: 'Post.author', many: true },
  },
});

keystone.createList('Post', {
  fields: {
    title: { type: Text },
    content: { type: Text },
    author: { type: Relationship, ref: 'User.posts', many: false },
  },
});
```

### Migration Strategy
#### PostgreSQL

- Drop the `User_posts` table.

## Many to Many (two-sided)

### List config

```javascript
keystone.createList('User', {
  fields: {
    name: { type: Text },
    posts: { type: Relationship, ref: 'Post.authors', many: true },
  },
});

keystone.createList('Post', {
  fields: {
    title: { type: Text },
    content: { type: Text },
    authors: { type: Relationship, ref: 'User.posts', many: true },
  },
});
```

### Migration Strategy
#### PostgreSQL

- Drop the `Post_authors` table.
- Rename `User_posts` to `User_posts_Post_authors`.
- Rename `User_id` to `User_left_id` and `Post_id` to `Post_right_id`.

## One to One (two-sided)

### List config

```javascript
keystone.createList('User', {
  fields: {
    name: { type: Text },
    post: { type: Relationship, ref: 'Post.author', many: false },
  },
});

keystone.createList('Post', {
  fields: {
    title: { type: Text },
    content: { type: Text },
    author: { type: Relationship, ref: 'User.post', many: false },
  },
});
```

### Migration Strategy
#### PostgreSQL

One to one relationships in the `before` state had a foreign key column on each table.
In the `after` state, only one of these is stored.
Because of the symmetry of the one to one relationship, Keystone makes an arbitrary decision about which column to use.

- Identify the foreign key column which is no longer required, and delete it.
- In our example above we would delete the `Post.author` column.
