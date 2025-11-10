# Assignment

Once again, you will deliver 2 pull requests:

- A pull request for the **Warmup** - in your regular hyf-homework repository
- A pull request for the additional **meal sharing endpoints** - in the meal-sharing repository

In both repositories, create a `nodejs-week2` branch from `main` to work on the homework (`git checkout -b nodejs-week2` ).

TODO - We should add a task to practice postman too. Maybe add all their final endpoints to their collection.

## Warmup

For the warmup you will be handed a Contacts API with a single endpoint:

- `GET /api/contacts`

This endpoint accepts a query parameter `sort`. Here's how you can use it:

- `GET /api/contacts?sort=first_name%20ASC`
  - Sorts contacts by first name, ascending
- `GET /api/contacts?sort=last_name%20DESC`
  - Sorts contacts by last name, descending

But this `sort` query parameter has been introduced with a SQL injection vulnerability and the goal is to demonstrate the issue and then fix and remove the vulnerability.

### Setup

TODO - Review assignment to work with sqlite.

Go to `nodejs/week2` in your `hyf-homework` repo:

```shell
npm init -y
npm i express mysql2 knex
npm i --save-dev nodemon
npm set-script dev "nodemon app.js"
```

Make sure you have `"type": "module"` in your `package.json`.

You should also ensure that the `node_modules/` folder is ignored by Git:

```shell
echo node_modules/ >> .gitignore
```

Create a database/schema called `hyf_node_week2_warmup` with a `contacts` table:

```sql
CREATE TABLE `contacts` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `first_name` varchar(255) NOT NULL,
  `last_name` varchar(255) NOT NULL,
  `email` varchar(255) DEFAULT NULL,
  `phone` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Sample data
insert into contacts (id, first_name, last_name, email, phone) values (1, 'Selig', 'Matussov', 'smatussov0@pinterest.com', '176-630-4577');
insert into contacts (id, first_name, last_name, email, phone) values (2, 'Kenny', 'Yerrington', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (3, 'Emilie', 'Gaitskell', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (4, 'Jordon', 'Tokell', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (5, 'Sallyann', 'Persse', 'spersse4@webnode.com', '219-157-2368');
insert into contacts (id, first_name, last_name, email, phone) values (6, 'Berri', 'Bulter', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (7, 'Lanni', 'Ivanilov', 'livanilov6@fda.gov', null);
insert into contacts (id, first_name, last_name, email, phone) values (8, 'Dagny', 'Milnthorpe', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (9, 'Annadiane', 'Bansal', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (10, 'Tawsha', 'Hackley', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (11, 'Rubetta', 'Ozelton', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (12, 'Charles', 'Boughey', 'cbougheyb@senate.gov', '605-358-5664');
insert into contacts (id, first_name, last_name, email, phone) values (13, 'Shantee', 'Robbe', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (14, 'Gleda', 'Peat', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (15, 'Arlinda', 'Ethersey', 'aetherseye@biglobe.ne.jp', '916-139-1300');
insert into contacts (id, first_name, last_name, email, phone) values (16, 'Armando', 'Meachem', 'ameachemf@oaic.gov.au', '631-442-5339');
insert into contacts (id, first_name, last_name, email, phone) values (17, 'Codi', 'Redhouse', null, '401-953-6897');
insert into contacts (id, first_name, last_name, email, phone) values (18, 'Ann', 'Buncombe', 'abuncombeh@ow.ly', '210-338-0748');
insert into contacts (id, first_name, last_name, email, phone) values (19, 'Louis', 'Matzkaitis', 'lmatzkaitisi@ebay.com', '583-996-6979');
insert into contacts (id, first_name, last_name, email, phone) values (20, 'Jessey', 'Pala', null, null);
insert into contacts (id, first_name, last_name, email, phone) values (21, 'Archy', 'Scipsey', 'ascipseyk@ask.com', '420-983-2426');
insert into contacts (id, first_name, last_name, email, phone) values (22, 'Benoit', 'Mould', 'bmouldl@bing.com', '271-217-9218');
insert into contacts (id, first_name, last_name, email, phone) values (23, 'Sherm', 'Girardey', 'sgirardeym@guardian.co.uk', '916-999-2957');
insert into contacts (id, first_name, last_name, email, phone) values (24, 'Raquel', 'Mudge', 'rmudgen@slate.com', '789-830-7473');
insert into contacts (id, first_name, last_name, email, phone) values (25, 'Tabor', 'Reavey', null, null);
```

Create `app.js`:

```js
import knex from "knex";
const knexInstance = knex({
  client: "mysql2",
  connection: {
    host: process.env.DB_HOST || "127.0.0.1",
    port: process.env.DB_PORT || 3306,
    user: process.env.DB_USER || "root",
    password: process.env.DB_PASSWORD || "my-secret-pw",
    database: process.env.DB_NAME || "hyf_node_week2_warmup",
    multipleStatements: true,
  },
});

import express from "express";
const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

const apiRouter = express.Router();
app.use("/api", apiRouter);

const contactsAPIRouter = express.Router();
apiRouter.use("/contacts", contactsAPIRouter);

contactsAPIRouter.get("/", async (req, res) => {
  let query = knexInstance.select("*").from("contacts");

  if ("sort" in req.query) {
    const orderBy = req.query.sort.toString();
    if (orderBy.length > 0) {
      query = query.orderByRaw(orderBy);
    }
  }

  console.log("SQL", query.toSQL().sql);

  try {
    const data = await query;
    res.json({ data });
  } catch (e) {
    console.error(e);
    res.status(500).json({ error: "Internal server error" });
  }
});

app.listen(port, () => {
  console.log(`Listening on port ${port}`);
});
```

As mentioned above, the `sort` query parameter has been introduced with a SQL injection vulnerability.

First, you should demonstrate the SQL injection and that it for instance is possible to drop/delete the `contacts` table with the `sort` query parameter.
You can for instance demonstrate this with a screen recording and include it in the PR description.

After having demonstrated the SQL injection vulnerability, the goal is then to fix the issue by updating `app.js`.

**Hint:** the `multipleStatements: true` part in the configuration indicates how you can use the vulnerability. The configuration should not be changed though, the SQL injection should be fixed by making changes in the `/api/contacts` route.
