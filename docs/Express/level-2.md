---
sidebar_position: 1
---

# Level 2

```bash
npm install --global yarn
yarn
```

## .env .env.example

```bash
NODE_ENV=development
PORT=8000

# Authentication
JWT_SECRET=visiarch1@
JWT_REFRESH_SECRET=visiarch1@
JWT_ALGORITHM=HS256
JWT_ISSUER=api.visiarch.com
JWT_EXPIRATION=3600
JWT_REFRESH_EXPIRATION=86400

# Database
DATABASE_CLIENT="mysql"
DATABASE_HOST=127.0.0.1
DATABASE_PORT=3306
DATABASE_USER=root
DATABASE_PASSWORD=
DATABASE_DATABASE=gbs-research
```

## routes/user.js

```jsx
import { Router } from "express";

const routes = Router();

import * as auth from "../controllers/auth";
import jwtMiddleware from "../middlewares/jwt";

routes.post("/signup", auth.register);
routes.post("/login", auth.login);
routes.get("/me", jwtMiddleware, auth.me);

export default routes;
```

## controllers/auth.js

```jsx
import validate from "../libs/form";
import * as auth from "../services/auth";

export const register = async (req, res) => {
  const rules = {
    first_name: "required",
    last_name: "required",
    email: "required|email",
    password: "required|min:8",
  };
  const errors = await validate(req.body, rules);

  if (errors) {
    return res.json({
      status: false,
      message: errors,
    });
  }

  try {
    const repo = await auth.register(req.body);
    return res.json({
      status: true,
      data: repo,
    });
  } catch (error) {
    return res.json({
      status: false,
      message: error.toString(),
    });
  }
};

export const login = async (req, res) => {
  const rules = {
    email: "required|email",
    password: "required|min:8",
  };
  const errors = await validate(req.body, rules);

  if (errors) {
    return res.json({
      status: false,
      message: errors,
    });
  }

  try {
    const repo = await auth.login(req.body);
    return res.json({
      status: true,
      data: repo,
    });
  } catch (error) {
    return res.json({
      status: false,
      message: error.toString(),
    });
  }
};

export const me = async (req, res) => {
  try {
    const repo = await auth.me(req.auth);
    return res.json({
      status: true,
      data: repo,
    });
  } catch (error) {
    return res.json({
      status: false,
      message: error.toString(),
    });
  }
};
```

## middlewares/jwt.js

```jsx
import config from "../config";
var { expressjwt: jwt } = require("express-jwt");

const jwtMiddleware = jwt({
  secret: config.jwt.SECRET,
  algorithms: [config.jwt.ALGORITHM],
  issuer: config.jwt.ISSUER,
});

export default jwtMiddleware;
```

```bash
yarn add express-jwt
```

## libs/form.js

```jsx
import { validator } from "indicative";

export const validateAll = async (data, rules) => {
  return await validator
    .validateAll(data, rules)
    .then(() => null)
    .catch((err) => err);
};

export default validateAll;
```

## services/auth.js

```jsx
export const me = async (jwtPayload) => {
  const auth = await Auth.query().findOne({
    id: jwtPayload.id,
  });

  if (!auth) {
    throw Error("Error getting data.");
  }

  return auth;
};
```

```bash
yarn add jsonwebtoken
yarn add bcrypt
yarn add nanoid
yarn add knex
yarn add objection
yarn add objection-visibility
yarn add mysql
yarn add indicative
```

```bash title="knex command"
knex init
```

```jsx title="knexfile.js"
// Update with your config settings.

/**
 * @type { Object.<string, import("knex").Knex.Config> }
 */
module.exports = {
  development: {
    client: "mysql",
    connection: {
      database: "gbs-research",
      user: "root",
      password: "",
    },
  },

  staging: {
    client: "mysql",
    connection: {
      database: "gbs-research-staging",
      user: "gbs-research",
      password: "gbssecret",
    },
    pool: {
      min: 2,
      max: 10,
    },
    migrations: {
      tableName: "knex_migrations",
    },
  },

  production: {
    client: "mysql",
    connection: {
      database: "visiarch-api",
      user: "visiarch",
      password: "password",
    },
    pool: {
      min: 2,
      max: 10,
    },
    migrations: {
      tableName: "knex_migrations",
    },
  },
};
```

## migration

```jsx
exports.up = function (knex) {
  return knex.schema
    .createTable("authentications", function (table) {
      table.string("id", 64).notNullable().primary().index();

      table.string("email", 150).index();
      table.string("password", 150).notNullable();
      table.string("secret_key", 64);
      table.string("reset_email_token", 64);
      table.timestamp("email_verified_at").nullable();
      table.timestamp("deleted_at").nullable();
      table.timestamps(true, true);
    })
    .createTable("tokens", function (table) {
      table.string("id", 64).notNullable().primary().index();
      table.string("auth_id", 64).notNullable();
      table
        .foreign("auth_id")
        .references("id")
        .inTable("authentications")
        .onDelete("CASCADE");
      table.text("refresh_token");
      table.datetime("expired_at");
      table.timestamps(true, true);
      table.datetime("deleted_at");
    });
};

exports.down = function (knex) {
  return knex.schema
    .dropTableIfExists("authentications")
    .dropTableIfExists("tokens");
};
```

## config

```jsx title="index.js"
import app from "./app";
import jwt from "./jwt";
import db from "./db";
export default {
  app,
  db,
  jwt,
};
```

```jsx title="app.js"
const env = process.env;

export default {
  ENV: env.NODE_ENV,
  PORT: +env.PORT,
};
```

```jsx title="db.js"
const env = process.env;

export default {
  CLIENT: env.DATABASE_CLIENT,
  HOST: env.DATABASE_HOST,
  PORT: env.DATABASE_PORT,
  USER: env.DATABASE_USER,
  PASSWORD: env.DATABASE_PASSWORD,
  DATABASE: env.DATABASE_DATABASE,
};
```

```jsx title="jwt.js"
const env = process.env;

export default {
  SECRET: env.JWT_SECRET,
  REFRESH_SECRET: env.JWT_REFRESH_SECRET,
  ALGORITHM: env.JWT_ALGORITHM,
  ISSUER: env.JWT_ISSUER,
  EXPIRATION: env.JWT_EXPIRATION,
  REFRESH_EXPIRATION: env.JWT_REFRESH_EXPIRATION,
};
```

## models

### models/auth.js

```jsx
import { BaseModel } from "../database";

export class Token extends BaseModel {
  static tableName = "tokens";
}

class Authentication extends BaseModel {
  static tableName = "authentications";

  static get hidden() {
    return ["password", "secret_key", "reset_email_token"];
  }

  static relationMappings = () => ({
    token: {
      relation: BaseModel.HasOneRelation,
      modelClass: Token,
      join: {
        from: "authentications.id",
        to: "tokens.auth_id",
      },
    },
  });
}

export default Authentication;
```

## database

### database.js

```jsx
import knex from "knex";
import { Model } from "objection";
import visibility from "objection-visibility";
import config from "./config";

export const conn = knex({
  client: config.db.CLIENT,
  connection: {
    host: config.db.HOST,
    port: config.db.PORT,
    user: config.db.USER,
    password: config.db.PASSWORD,
    database: config.db.DATABASE,
  },
});

export class BaseModel extends visibility(Model) {
  static get modifiers() {
    return {
      notDeleted(builder) {
        builder.whereNull("deleted_at");
      },
    };
  }
}

BaseModel.knex(conn);
```

## Update

## app.js

```jsx
import express from "express";
import bodyParser from "body-parser";
import cors from "cors";
import serverRoutes from "./routes/server";
import userRoutes from "./routes/user";

const app = express();
app.use(cors());
app.use(bodyParser.json());
app.use(serverRoutes);
app.use(userRoutes);

export default app;
```
