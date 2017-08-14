# Pg Walkthrough Notes

## Getting Started
```sh
git clone https://github.com/foundersandcoders/pg-walkthrough.git
npm i
```

## Step 1 – Navigating the initial files
1. First open src/handler.js. Here we see the `/static` endpoint sends back to our server a file called `static.js`. If we open this file, we see that it contains a data object of two superheroes.
2. run `npm start` and navigate to `http://localhost:3000/static`. Here we see our hardcoded data from `static.js`.

## Step 2 – Setting up the database
1. If we want to serve the data from our database instead, we then need to set up our database.
2. Inside the folder `/database`, create a file called `db_build.sql`. Inside this folder, we're going to set up the structure of our database, as well as inserting some initial data into it. In this file:
  - start the file with `BEGIN;` - This defines the start of a transaction: `a unit/sequence of work that is performed against a database` that can be rolled back or committed.
  - then add `DROP TABLE IF EXISTS superheroes cascade;`. This line gets drops our old database every time this file is run - i.e. every time we build our database.
    > This should never be used in production other than for initialisation, since you only want to delete/reset your test database with mock data

    - Cascade will delete tables with relations to `superheroes` too
  - Then we outline the structure of our table. This sets out all the columns we want in our table.
    - Use `DEFAULT 100` to set a default weight (instead of NULL)
  - We then initialise our table with some data, using `INSERT INTO` and specifying which rows we want to insert data into.
  - we then `COMMIT;` our database - to confirm and execute the entire transaction.

## Step 3 – Connecting and building the database
Pg is a non-blocking PostgreSQL client for node.js that lets you access SQL values as JavaScript data values. Translates data types appropriately to/from JS data types.

1. Our database is now outlined, but we need a way to connect it. Inside the `/database` folder, create a file called `db_connection.js`.
  - For this we require the modules `pg` and `env2`, so `npm install --save pg env2` and require them at the top of this file.
  - You'll notice that this file requires a `config.env`. We'll set this up later.
    - This is a *gitignored* file with environment variables which are accessed with `process.env.NAME_HERE` (can be set in bash with `DB_URL='127...' npm start`)
  - `{ Pool }` is syntactic sugar ([destructuring assignment](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)) that lets you use `Pool` instead of `pg.Pool` or `Pool = pg.Pool`
  - [`Connection pooling`](https://en.wikipedia.org/wiki/Connection_pool) (Pool object lets you interact with a connection pool) is a cache of database connections that are kept open and reused when future requests are required, minimising the resource impact of opening/closing connections constantly for write/read heavy apps. Minimises latency. `new Pool(options)` creates a new pool object from the Pool class.
    - More connections uses more memory and too many can crash database server. Always return connections to the pool, or the pool will deplete.
  - [`pg options`](https://node-postgres.com/features/connecting#programmatic) - pass options object with object connection keys/values or `connectionString` - URL.
  - [Node `url` module](https://nodejs.org/api/url.html#url_url_strings_and_url_objects) is used to split URL strings into an object.
2. We also need a way of building our database. In the `/database` folder, create a file called `db_build.js`.
 - This file builds the database by requiring in our sql file that sets out our schema and using this to query the connection file we just created.

## Step 4 – Building the database
1. Now we have all the correct files, let's now get this database up and running.
2. in the terminal run `psql` (mac) or `sudo -u postgres psql` (linux).
3. create the database by typing `CREATE DATABASE superheroes;`
4. Create a user specifically for the database with a password by typing `CREATE USER [the new username] WITH PASSWORD '[the password of the database]'`; (the password needs to be in single-quotes, otherwise you get an error) + (for security: could clear command history and use a password manager - and don't expose port/database to outside world).
  - This is done so you can connect to the database with a username and password that only has access to the database for the specific application (security: don't use superusers).
5. Change ownership of the database to the new user by typing `GRANT ALL PRIVILEGES ON DATABASE [name of the database] TO [the new username];`;
6. Add a config.env file and add the database's url in this format: `DB_URL = postgres://[username]:[password]@localhost:5432/[database]`
7. Now we build the tables we set out in db_build.sql by running our `db_build.js` file: `node database/db_build.js`.
8. Connect to the database by typing `psql postgres://[username]:[password]@localhost:5432/[database]` and test if everything worked by typing `SELECT * FROM superheroes;`. You should see the data we entered in `db_build.sql` appear.

## Step 5 – connecting to our database from the server
1. Let's first write a file that gets our information from the database. In `/src` create a file called `dynamic.js`.
2. Require in `db_connection.js` at the top of this file. We use this to connect to and query our database.
3. Write a function that connects to the database and uses a SQL query to select all the data from it and returns it to our server. Export this so we can require it in our handler.
4. This function should take in a cb, that we use the error first callback method with to handle our errors.
5. Back in `handler.js`, we require in the file we just wrote, and write an endpoint for `/dynamic`, which uses the data we get back from the database.
6. Navigate to `http://localhost:3000/dynamic` to check it's worked.