# SQL CRUD

An assignment to design relational database tables with particular applications in mind.

<!-- The contents of this file will be deleted and replaced with the content described in the [instructions](./instructions.md) -->

## Part 1: Restaurant finder
### Table structure
```sql
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY, 

    name TEXT, -- name of the restaurant, chain restaurant may have same name
    category TEXT, -- food genere served by the restaurant
    price_tier TEXT, -- cheap, medium, expensive
    neighborhood TEXT, -- neighborhood of restaurant

    /* Set opening hours */
    open_start TEXT DEFAULT '11:00',
    open_end TEXT DEFAULT '22:00',

    average_rating REAL, -- average rating between 1 - 5
    good_for_kids INTEGER, -- good 1, not good 0

    customer_review TEXT, -- long text consists of customer rewiews

    /* Store time when record created */
    created DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Practice data
The following code imports data from [restaurants.csv](./data/restaurants.csv) to table `restaurants`.
```sql
/* Create `temp_table` */
CREATE TABLE temp_table (
    name TEXT NOT NULL, -- name of the restaurant, chain restaurant may have same name
    category TEXT, -- food genere served by the restaurant
    price_tier TEXT, -- cheap, medium, expensive
    neighborhood TEXT, -- neighborhood of restaurant

    /* Set opening hours in UTC time */
    open_start TEXT DEFAULT '11:00:00.000',
    open_end TEXT DEFAULT '22:00:00.000',

    average_rating REAL, -- average rating between 1 - 5
    good_for_kids INTEGER, -- good 1, not good 0

    customer_review TEXT -- long text consists of customer rewiews
);

/* Setup mode */
.mode csv
.header on

/* Import data to `temp_table` */
.import data/restaurants.csv temp_table --skip 1

/* Copy data to `restaurants` */
INSERT INTO restaurants (name, category, price_tier, neighborhood, open_start, open_end, average_rating, good_for_kids, customer_review) 
    SELECT * FROM temp_table;

/* Delete `temp_table` */
DROP TABLE temp_table;
```

### Queries
1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).
```sql
SELECT id, name FROM restaurants
WHERE price_tier = "Cheap" AND neighborhood = "Brooklyn";
```

2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
```sql
SELECT id, name, average_rating FROM restaurants
WHERE category = "Italian" AND average_rating >= 3 
ORDER BY average_rating DESC;
```

3. Find all restaurants that are open now.
```sql
SELECT r.id, r.name, cur_time
FROM restaurants r LEFT JOIN (SELECT STRFTIME('%H:%M', 'now') AS cur_time)
WHERE cur_time >= r.open_start AND cur_time < r.open_end;
```

4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).
```sql
/* "new_review" represent place for 
actual new review */
UPDATE restaurants 
SET customer_review = customer_review || CHAR(10) || "new_review" -- CHAR(10) or x'0a' for new line
WHERE id = 43;
```

5. Delete all restaurants that are not good for kids.
```sql
DELETE FROM restaurants WHERE good_for_kids != 1;
```

6. Find the number of restaurants in each NYC neighborhood.
```sql
SELECT neighborhood, COUNT(id) FROM restaurants 
GROUP BY neighborhood;
```

## Part 2: Social media app
### Tables structure
#### Users:
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,

    email TEXT NOT NULL, -- user email
    password INTEGER KEY NOT NULL, -- login password encrypted
    username TEXT NOT NULL, -- i.e. handle

    /* Store time when record created */
    created DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### Posts - Messages & Stories:
```sql
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,

    /* Content metadata */
    sender_id TEXT NOT NULL, -- sender id
    receiver_id INTEGER, -- receiver id, 0 means stories
    post_time TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP, -- time posted
    is_visible INTEGER NOT NULL, -- is visible to by users

    content TEXT, -- content of messages or stories
    
    /* Store time when record created */
    created DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Practice data
The following code imports data from [users.csv](./data/users.csv) to table `users`.
```sql
/* Setup mode */
.mode csv
.header on

/* Create temporary table `temp_users` */
CREATE TABLE temp_users (
    email TEXT NOT NULL, -- user email
    password TEXT NOT NULL, -- login password encrypted
    username TEXT NOT NULL -- i.e. handle
);

/* Import data to `temp_users` */
.import ./data/users.csv temp_users --skip 1

/* Copy data to `users` */
INSERT INTO users (email, password, username)
    SELECT * FROM temp_users;

/* Delete `temp_users` */
DROP TABLE temp_users;
```

The following code imports data from [posts.csv](./data/posts.csv) to table `posts`.

```sql
/* Create temporary table `temp_post` */
CREATE TABLE temp_posts (
    /* Content metadata */
    sender_id TEXT NOT NULL, -- sender id
    receiver_id INTEGER, -- receiver id, 0 means stories
    post_time TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP, -- time posted
    is_visible INTEGER NOT NULL, -- is visible to users, 1 visible, 0 hidden

    content TEXT -- content of messages or stories
);

/* Import data to `temp_posts` */
.import ./data/posts.csv temp_posts --skip 1

/* Copy data to `posts` */
INSERT INTO posts (sender_id, receiver_id, post_time, is_visible, content)
    SELECT * FROM temp_posts;

/* Delete `temp_posts` */
DROP TABLE temp_posts;
```

### Queries
1. Register a new User.
```sql
/* "user@email.com", "password", "username" represent the 
actual user info to be added */
INSERT INTO users (email, password, username)
VALUES ("user@email.com", "password", "username");
```

2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
```sql
/* "message" represents the 
actual message info to be added */
INSERT INTO posts (sender_id, receiver_id, post_time, is_visible, content)
VALUES (2, 4, CURRENT_TIMESTAMP, 1, "message");
```

3. Create a new Story by a particular User (pick any User for example).
```sql
/* "story" represents the 
actual story info to be added */
INSERT INTO posts (sender_id, receiver_id, post_time, is_visible, content)
VALUES (5, 0, CURRENT_TIMESTAMP, 1, "story");
```

4. Show the 10 most recent visible Messages and Stories, in order of recency.
```sql
SELECT id, post_time, content FROM posts
ORDER BY post_time DESC
LIMIT 10;
```

5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```sql
SELECT id, post_time, content FROM posts
WHERE sender_id = 2 AND receiver_id = 3
ORDER BY post_time DESC
LIMIT 10;
```

6. Make all Stories that are more than 24 hours old invisible.
```sql
UPDATE posts SET is_visible = 0
WHERE ROUND((JULIANDAY('now') - JULIANDAY(post_time)) * 24) > 24;
```

7. Show all invisible Messages and Stories, in order of recency.
```sql
SELECT id, post_time, content FROM posts
WHERE is_visible = 0
ORDER BY post_time DESC;
```

8. Show the number of posts by each User.
```sql
SELECT users.id, username, COUNT(posts.sender_id) as number_of_posts 
FROM users LEFT JOIN posts ON users.id = posts.sender_id 
GROUP BY users.id;
```

9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```sql
SELECT users.id, users.username, users.email, posts.content 
FROM posts INNER JOIN users ON posts.sender_id = users.id WHERE ROUND((JULIANDAY('now') - JULIANDAY(posts.post_time)) * 24) <= 24;
```

10. Show the email addresses of all Users who have not posted anything yet.
```sql
SELECT users.id, users.email FROM users LEFT JOIN posts ON users.id = posts.sender_id 
WHERE posts.id IS NULL;
```