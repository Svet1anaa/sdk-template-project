\# Module 1 — System Map



\## a. System Diagram



```mermaid

graph TD

&#x20;   Browser\[Browser]

&#x20;   Frontend\["Frontend<br/>(Vanilla JS + Vite, port 5173)"]

&#x20;   Products\["products-service<br/>(PHP/Slim, port 8082)"]

&#x20;   Users\["users-service<br/>(Python/FastAPI, port 8000)"]

&#x20;   Orders\["orders-service<br/>(Java/Spring Boot, port 8083)"]

&#x20;   DB\[(PostgreSQL<br/>ecommerce\_db)]

&#x20;   Migration\["migration-runner<br/>(Node.js + Prisma)"]



&#x20;   Browser --> Frontend

&#x20;   Frontend --> Products

&#x20;   Frontend --> Users

&#x20;   Frontend --> Orders

&#x20;   Products --> DB

&#x20;   Users --> DB

&#x20;   Orders --> DB

&#x20;   Migration --> DB

```



\## b. Request trace: GET /products



1\. frontend/src/components/products.js — renderProducts() calls

&#x20;  fetchData(`${apiUrl}/products`)

2\. frontend/src/api/api.js — fetchData() performs fetch(url)

3\. apiUrl comes from frontend/src/main.js — config.productsApiUrl,

&#x20;  read from VITE\_PRODUCTS\_API\_URL, defaults to http://localhost:8082

4\. products-service/public/index.php — route $app->get('/products', ...)

&#x20;  receives the request

5\. products-service/public/index.php — $pdo->query("SELECT \* FROM \\"Product\\"")

&#x20;  runs against the "Product" table in the ecommerce\_db database

6\. frontend/src/components/products.js — renderProducts() sets

&#x20;  container.innerHTML to render the returned JSON as cards



\## c. Environment gotchas



\- Docker Compose failed with "no port specified: :<empty>" on first run —

&#x20; caused by missing environment variables (.env file was not copied from

&#x20; .env.example before the first build).

\- Build failed with "input/output error" during image builds — caused by

&#x20; drive C being almost full (only 1.9 GB free). Fixed by freeing space

&#x20; and eventually moving Docker Desktop's installation and WSL2 data disk

&#x20; to drive D.

\- Docker Desktop showed "There was a problem with WSL" after a failed

&#x20; build — resolved by reinstalling Docker Desktop.



\## d. Documentation inaccuracy



ARCHITECTURE.md states that Products Service "Manages product catalog (CRUD operations)" and that Users Service "Handles user registration and authentication". In reality, products-service/public/index.php only implements a single GET /products route with a read-only SELECT query.

There is no POST, PUT, DELETE, registration or authentication logic in

the code. The documentation describes an intended future state, not the current implementation.


## e. Request trace: GET /users

1. frontend/src/components/users.js — renderUsers() calls
   fetchData(`${apiUrl}/users`)
2. frontend/src/api/api.js — fetchData() performs fetch(url)
3. apiUrl comes from frontend/src/main.js — config.usersApiUrl,
   read from VITE_USERS_API_URL, defaults to http://localhost:8000
4. users-service/main.py — route @app.get("/users") receives the request
5. users-service/main.py — conn.fetch(f'SELECT {USER_COLUMNS} FROM "User"')
   runs against the "User" table in ecommerce_db. Note: the password column
   is deliberately excluded from USER_COLUMNS, so it never reaches the API
   response even though the route uses no authentication.
6. frontend/src/components/users.js — renderUsers() sets container.innerHTML
   to render the returned JSON as a list

