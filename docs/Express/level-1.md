---
sidebar_position: 1
---

# Level 1

## Preliminary

```bash
mkdir folder
cd folder
npm init -y
npm install express
.gitignore
```

```bash title=".gitignore"
# npm
node_modules
package-lock.json
*.log
*.gz

# Coveralls
.nyc_output
coverage

# Benchmarking
benchmarks/graphs

# ignore additional files using core.excludesFile
# https://git-scm.com/docs/gitignore
```

```bash
npm i backpack-core
npm i cors
npm i dotenv
```

```json
  "scripts": {
    "dev": "backpack",
    "build": "backpack build"
  },
```

```text
mkdir src
```

- index.js
- app.js
- config/index.js
- config/app.js
- routes/server.js

## index.js

```jsx
import "dotenv/config";
import config from "./config";
import app from "./app";

app.listen(config.app.PORT, () => {
  console.info(`App listening at http://localhost:${config.app.PORT}`);
});
```

## app.js

```jsx
import express from "express";
import bodyParser from "body-parser";
import cors from "cors";
import serverRoutes from "./routes/server";

const app = express();
app.use(cors());
app.use(bodyParser.json());
app.use(serverRoutes);

export default app;
```

## config/index.js

```jsx
import app from "./app";

export default {
  app,
};
```

## config/app.js

```jsx
const env = process.env;

export default {
  ENV: env.NODE_ENV,
  PORT: +env.PORT,
};
```

## routes/server.js

```jsx
import { Router } from "express";

const routes = Router();

routes.get("/", (_, res) => {
  return res.json({ ping: "pong!" });
});

export default routes;
```

## Create .env

```bash title="environment"
NODE_ENV=development
PORT=8000
```

### run dev

```bash
npm run dev
```
