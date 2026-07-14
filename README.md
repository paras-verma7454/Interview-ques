# Full Stack Interview Prep

## Table of Contents

- [1. HTTP Methods](#1-http-methods)
- [2. Context API vs Redux](#2-context-api-vs-redux)
- [3. DOM & Virtual DOM](#3-dom--virtual-dom)
- [4. Closures in JS](#4-closures-in-js)
- [5. Event Loop](#5-event-loop)
- [6. Promises & Combinators](#6-promises--combinators)
- [7. Sync vs Async Functions](#7-sync-vs-async-functions)
- [8. Deep Copy vs Shallow Copy](#8-deep-copy-vs-shallow-copy)
- [9. setTimeout & setInterval](#9-settimeout--setinterval)
- [10. React Hooks](#10-react-hooks)
- [11. Authentication vs Authorization](#11-authentication-vs-authorization)
- [12. React.js vs Next.js](#12-reactjs-vs-nextjs)
- [13. Single-Page Application (SPA)](#13-single-page-application-spa)
- [14. Middleware](#14-middleware)
- [15. CSS Flexbox](#15-css-flexbox)
- [16. WebSockets](#16-websockets)
- [17. Boilerplate: Simple GET Route](#17-boilerplate-simple-get-route)
- [18. MVC Architecture](#18-mvc-architecture)
- [19. SQL vs NoSQL Databases](#19-sql-vs-nosql-databases)
- [20. Idempotency](#20-idempotency)

------------------------------------------------------------------------

# 1. HTTP Methods

HTTP methods define what action you want to perform on a resource. The
core ones are:

GET --- Retrieve data. No request body. Idempotent and safe. Example:
fetching a user profile.

POST --- Create a new resource. Has a request body. Not idempotent ---
calling it twice creates two resources. Example: submitting a form.

PUT --- Replace an entire resource. Idempotent --- sending the same PUT
twice gives the same result. Example: updating a full user object.

PATCH --- Partially update a resource. Only send the fields you want to
change. Example: updating just a user's email.

DELETE --- Remove a resource. Idempotent. Example: deleting a post.

In an interview, I'd also mention OPTIONS (used in CORS preflight
requests to check what methods a server allows) and HEAD (like GET but
returns only headers, useful to check if a resource exists).

QUERY --- Like GET, but with a body. Safe and idempotent, but instead of
cramming filters into a URL, you send a structured JSON payload. Solves
the problem of complex search queries hitting URL length limits. Before
this, developers used POST for reads (breaking caching) or shoehorned
everything into query strings. Example: a search endpoint with 20 filter
fields, pagination, and sorting sent as a JSON body.

```js
    QUERY /api/search HTTP/1.1
    Content-Type: application/json

    {
      "filters": { "status": ["active"], "createdAt": { "from": "2025-01-01" } },
      "pagination": { "page": 1, "size": 50 },
      "sort": [{ "field": "createdAt", "direction": "desc" }]
    }
```

Use cases: search APIs with many filter fields, reporting endpoints,
analytics queries --- anywhere a simple query string isn't enough but
you still want read semantics.

> Follow-up tip: They often ask about safe vs idempotent. Safe = no side
> effects (GET, HEAD, QUERY). Idempotent = same result no matter how
> many times you call it (GET, PUT, DELETE, QUERY). POST is neither safe
> nor idempotent. Mentioning QUERY shows you're up to date with where
> the industry is heading.

# 2. Context API vs Redux

React's Context API is a built-in mechanism to share state across the
component tree without prop-drilling. Redux is a full state management
library with a stricter, more opinionated pattern.

Context API works by wrapping components in a Provider , and any child
can consume the value using useContext . It's great for low-frequency
updates like theme, auth state, or locale.

```js
    const ThemeContext = createContext();
    function App() {
      return (
        <ThemeContext.Provider value="dark">
          <Child />
        </ThemeContext.Provider>
      );
    }
    const theme = useContext(ThemeContext);
```

Redux introduces a global store, actions, and reducers. Every state
change flows through a predictable pipeline: Action → Reducer → New
State. It's better for complex, high-frequency, or cross-feature state.

The key differences: Context re-renders all consumers when value changes
(performance concern at scale). Redux uses selectors so components only
re-render when their specific slice changes. Redux also has devtools for
time-travel debugging.

> Rule of thumb: Use Context for simple global state (auth, theme). Use
> Redux (or Zustand) when you have complex state logic, many updates per
> second, or multiple unrelated features sharing state.

# 3. DOM & Virtual DOM

DOM (Document Object Model) is the browser's tree representation of your
HTML. Every HTML tag becomes a node. JavaScript can read and manipulate
it --- but direct DOM manipulation is expensive because the browser has
to recalculate layout and repaint the screen.

Virtual DOM is React's solution. It's a lightweight in-memory copy of
the real DOM. When state changes, React builds a new virtual DOM tree,
diffs it against the previous one (reconciliation), and only applies the
minimal set of real DOM changes needed.

The diffing algorithm (called Fiber in React 16+) works by comparing
nodes at the same level. If a node's type changes, React tears down the
whole subtree. If it's the same type, it updates only the changed
attributes --- that's why key props matter in lists: they help React
identify which items changed.

> Analogy that lands in interviews: "Think of it like editing a Word
> document. You don't reprint the whole page every time you fix a typo
> --- you just update the changed characters. Virtual DOM does that for
> your UI."

# 4. Closures in JS

A closure is when a function "remembers" variables from its outer scope
even after that outer function has returned. It's one of the most
fundamental concepts in JavaScript.

```js

    function makeCounter() {
      let count = 0; // outer variable
      return function() {
        count++;     // inner function closes over count
        return count;
      };
    }
    const counter = makeCounter();
    counter(); // 1
    counter(); // 2 — count persists!

```

Here, count should be gone after makeCounter() returns --- but it isn't,
because the returned function holds a reference to it via the closure.

Closures are everywhere in JS: React's useState setter captures state,
event listeners capture DOM references, and module patterns use closures
to make variables private.

A common interview gotcha: the classic for loop with var problem --- all
callbacks share the same i because var is function-scoped. The fix is to
use let (block-scoped, creates a new binding per iteration) or wrap in
an IIFE.

> They'll almost certainly give you a code snippet to spot the closure
> bug. The answer is almost always: use let instead of var, or use a
> factory function.

# 5. Event Loop

JavaScript is single-threaded --- it can only do one thing at a time.
The event loop is what makes async operations possible without blocking
the main thread.

The execution model has three key parts:

Call Stack --- where synchronous code runs, LIFO (last in, first out).
When a function is called, it's pushed. When it returns, it's popped.

Web APIs / Task Queue --- browser-provided async mechanisms (setTimeout,
fetch, DOM events). When async work completes, the callback is pushed to
the task queue (also called the macrotask queue).

Microtask Queue --- Promise .then() callbacks, queueMicrotask() . This
runs before the task queue after every task.

```js

    console.log("1");
    setTimeout(() => console.log("2"), 0);
    Promise.resolve().then(() => console.log("3"));
    console.log("4");
    // Output: 1, 4, 3, 2

```

Why? "1" and "4" are synchronous. Then the event loop checks the
microtask queue first --- Promise resolves → "3". Then the macrotask
queue --- setTimeout → "2".

> The interview-winning detail: Microtasks (Promises) always drain
> completely before the next macrotask (setTimeout) runs. This is why a
> Promise with 0 delay still beats a setTimeout with 0 delay.

# 6. Promises & Combinators

A Promise represents a value that will be available in the future. It
has three states: pending, fulfilled, or rejected. You chain it with
.then() for success and .catch() for errors.

The four key combinators:

Promise.all(arr) --- Waits for ALL promises to resolve. If any one
rejects, it immediately rejects. Use when you need all results and a
single failure should abort everything. Example: fetching user +
orders + preferences in parallel.

Promise.allSettled(arr) --- Waits for ALL promises, regardless of
outcome. Returns an array of {status, value/reason} objects. Use when
you want every result even if some fail.

Promise.race(arr) --- Settles as soon as the first promise settles
(either way). Use for timeouts: race a fetch against a setTimeout that
rejects after 5 seconds.

Promise.any(arr) --- Resolves as soon as the first promise fulfills.
Only rejects if all fail. Use for redundant requests --- send to
multiple servers, use whoever responds first.

> Memory trick: all = everyone must succeed. allSettled = just wait for
> everyone. race = whoever finishes first wins. any = first success
> wins.

# 7. Sync vs Async Functions

Synchronous code runs line by line, blocking execution until each
operation completes. If one operation takes 5 seconds, nothing else runs
during those 5 seconds.

Asynchronous code allows the program to start an operation and move on,
coming back to handle the result later. This is critical for I/O-heavy
operations like network requests or file reads.

In JavaScript, async is handled through callbacks → Promises →
async/await (syntactic sugar over Promises).

```js

    // Async/Await
    async function getUser(id) {
      try {
        const res = await fetch(\`/api/users/\${id}\`);
        const user = await res.json();
        return user;
      } catch (err) {
        console.error(err);
      }
    }

```

async functions always return a Promise. await pauses execution inside
the async function only --- the rest of the app keeps running. Under the
hood, it's the same as .then() chaining.

> Common mistake to avoid: putting await in a loop ( for...await runs
> sequentially). If the iterations are independent, use
> Promise.all(arr.map(fn)) to run them in parallel --- much faster.

# 8. Deep Copy vs Shallow Copy

This is about how JavaScript handles copying objects and arrays ---
which are reference types.

Shallow copy creates a new object, but nested objects/arrays still share
the same reference. Changing a nested property in the copy affects the
original.

```js

    const obj = { a: 1, nested: { b: 2 } };
    const shallow = { ...obj };
    shallow.nested.b = 99; // also changes obj.nested.b!

```

Deep copy creates a completely independent clone --- every level is a
new reference.

```js

    // Modern approach (structured clone)
    const deep = structuredClone(obj);

    // Old approach (has limitations — loses functions, Dates)
    const deep2 = JSON.parse(JSON.stringify(obj));

```

Shallow copy methods: spread operator ( ...obj ), Object.assign() ,
Array.slice() . These only go one level deep.

Deep copy methods: structuredClone() (native, modern, recommended),
JSON.parse(JSON.stringify()) (simple but breaks on functions, undefined,
Dates, circular refs), or lodash's \_.cloneDeep() .

> In React, this matters a lot. State updates need new references to
> trigger re-renders, which is why spreading nested state wrong is a
> common bug.

# 9. setTimeout & setInterval

Both are Web API timer functions, but they work differently.

setTimeout(fn, delay) --- Executes a function once after at least delay
milliseconds. The "at least" is key: if the call stack is busy, it'll
run later. Returns a timer ID you can pass to clearTimeout(id) to cancel
it.

setInterval(fn, delay) --- Executes a function repeatedly every delay
ms. Returns a timer ID for clearInterval(id) .

```js

    // Debounce pattern using setTimeout
    function debounce(fn, delay) {
      let timer;
      return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn(...args), delay);
      };
    }

```

A key difference: setInterval fires at a fixed interval regardless of
how long the callback takes. If the callback takes longer than the
interval, you can get callbacks queuing up. The safer pattern for
repeated async work is a recursive setTimeout : run the task, then
schedule the next one only after it completes.

> In React, always clear timers in a useEffect cleanup function.
> Forgetting this causes memory leaks and state updates on unmounted
> components (the infamous "can't update state on unmounted component"
> warning).

# 10. React Hooks

Hooks let functional components use state and lifecycle features that
were previously only in class components.

useState --- Local component state. Returns \[value, setter\]. Calling
the setter schedules a re-render.

useEffect --- Runs side effects after render. The dependency array
controls when it re-runs. Return a cleanup function for
subscriptions/timers. Empty array = run once on mount.

useMemo --- Memoizes a computed value. Only recomputes when dependencies
change. Use for expensive calculations --- not every computation needs
it.

```js

    const sorted = useMemo(() => items.sort(...), [items]);
```

useCallback --- Memoizes a function reference. Important when passing
callbacks to child components wrapped in React.memo --- prevents
unnecessary re-renders.

useParams (React Router) --- Reads URL params like /user/:id . Returns
an object: const { id } = useParams() .

Custom Hooks --- Any function starting with use that calls other hooks.
They're the way to extract and reuse stateful logic.

```js

    function useFetch(url) {
      const [data, setData] = useState(null);
      useEffect(() => {
        fetch(url).then(r => r.json()).then(setData);
      }, [url]);
      return data;
    }

```

> Rules of hooks: only call at the top level (not inside
> conditionals/loops) and only call from React functions. React relies
> on call order being stable between renders.

# 11. Authentication vs Authorization

These are frequently confused. The simplest way I frame it:

Authentication = "Who are you?" --- Verifying identity. This is the
login step: checking that the username/password matches, or validating a
JWT token, or verifying an OAuth token from Google.

Authorization = "What are you allowed to do?" --- Checking permissions.
After identity is confirmed, the system decides if this user can access
a specific resource or perform an action.

Example: A user logs into a dashboard (authentication). They're a
viewer, not an admin, so they can read reports but can't delete users
(authorization).

In practice: Authentication typically produces a token (JWT, session
cookie). Authorization uses claims/roles inside that token. A middleware
intercepts requests, validates the token (authentication), then checks
if the user's role permits the route (authorization).

```js

    // Express example
    app.get('/admin', authenticate, authorize('admin'), handler);

```

> Common follow-up: JWT structure --- header.payload.signature. The
> payload contains claims like userId and role. The signature is
> verified server-side with a secret. The server never stores JWTs ---
> it just verifies them on each request.

# 12. React.js vs Next.js

React is a UI library --- it handles rendering components, managing
state, and the virtual DOM. It does nothing about routing, data fetching
strategy, or how your app is served. You're responsible for wiring
everything up.

Next.js is a React framework --- it builds on top of React and makes
opinionated decisions about those things for you.

Key things Next.js adds:

Rendering strategies --- SSR (Server-Side Rendering: HTML generated per
request), SSG (Static Site Generation: HTML built at deploy time), ISR
(Incremental Static Regeneration: cached but periodically rebuilt).
React alone is CSR (Client-Side Rendering) --- the browser does all the
work.

File-based routing --- Creating app/about/page.tsx automatically creates
the /about route. No react-router needed.

API routes --- You can write backend endpoints inside the same Next.js
project.

SEO --- SSR/SSG means the HTML is fully rendered when search engine
crawlers hit it. Pure React CSR returns an empty shell, which hurts SEO.

> When would you use React without Next.js? Dashboards behind a login
> (SEO doesn't matter), apps deployed to a CDN as static files, or when
> you need full control over your build setup.

# 13. Single-Page Application (SPA)

A Single-Page Application loads one HTML file once from the server.
After that, all navigation, rendering, and data fetching happens in the
browser via JavaScript --- no full page reloads.

When you click a link, the JS router intercepts it, updates the URL
using the History API, and swaps out the components on the page ---
giving the illusion of navigation without a server round-trip.

Advantages
:   Fast, fluid UX after initial load. Rich interactivity. Clear
    separation of frontend/backend (API-driven). Mobile-app-like feel.

Disadvantages
:   Larger initial JS bundle (slow first load). SEO challenges ---
    crawlers see an empty HTML shell. State management complexity grows.
    Browser back/forward button behavior needs explicit handling.

Examples: Gmail, Google Maps, Twitter --- page never fully reloads, just
content changes.

React by default creates SPAs. Next.js is often used to solve the SEO
problem by adding SSR/SSG on top.

> Contrast with MPA (Multi-Page Application): every route is a full
> server-rendered HTML page reload. SPAs trade a heavier initial load
> for a smoother subsequent experience.

# 14. Middleware

Middleware is code that runs between receiving a request and sending a
response. It's a function in the request-response pipeline that can
inspect, modify, or terminate the cycle.

In Express.js, every middleware function receives (req, res, next) .
Calling next() passes control to the next middleware in the chain. Not
calling it halts the pipeline. In the example, the auth middleware
checks for a token in the Authorization header --- if missing, it
terminates the cycle by returning a 401. If present, it logs and calls
next() , allowing the request to reach the /orders route handler.

```js

    function auth(req, res, next) {
      const token = req.headers.authorization;
      if (!token) {
        return res.status(401).json({ message: "Unauthorized" });
      }
      console.log("User authenticated");
      next();
    }

    app.get('/orders', auth, (req, res) => {
      res.json({ orders: ["Order1", "Order2"] });
    });

``` 
This pattern is the foundation of Express auth --- the middleware acts
as a gatekeeper. Any route that needs protection simply adds the auth
function as a second argument. Middleware can also be applied globally
via app.use(auth) to protect all routes.

Common uses: logging, authentication, authorization, request body
parsing, rate limiting, CORS headers, error handling.

In Next.js, middleware.ts runs at the edge before any route handler ---
great for auth checks and redirects without hitting your server.

> Error-handling middleware in Express takes four arguments (err, req,
> res, next). Express detects the arity (4 params) and routes unhandled
> errors to it automatically. Always define it last.

# 15. CSS Flexbox

Flexbox is a one-dimensional layout model --- it lays items out either
in a row or column. You apply it to the container ( display: flex ) and
it controls how children (flex items) are sized and positioned.

Key container properties:

flex-direction --- row (default) or column. Sets the main axis.

justify-content --- aligns items along the main axis. Values:
flex-start, center, flex-end, space-between (gaps between items),
space-around, space-evenly.

align-items --- aligns items along the cross axis (perpendicular to
main). Values: stretch (default), center, flex-start, flex-end,
baseline.

flex-wrap --- whether items wrap to the next line (wrap) or stay on one
line (nowrap).

gap --- spacing between items. Much cleaner than using margins.

```css

    /* Center something perfectly */
    .container {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

```
Key item properties: flex: 1 (shorthand for grow, shrink, basis ---
makes item take up remaining space). align-self overrides align-items
for one item.

> Grid vs Flexbox: Flexbox is 1D (one axis at a time). CSS Grid is 2D
> (rows and columns simultaneously). Use Flexbox for nav bars, button
> groups, card rows. Use Grid for full-page layouts.

# 16. WebSockets

WebSocket is a protocol that provides a persistent, full-duplex
(two-way) communication channel over a single TCP connection. Unlike
HTTP, which is request-response, a WebSocket connection stays open and
both client and server can send messages at any time.

The lifecycle: client sends an HTTP "upgrade" request → server upgrades
the connection to WebSocket → both parties can now send frames anytime →
either side closes the connection.

```js

    // Client
    const ws = new WebSocket('wss://api.example.com/chat');
    ws.onopen = () => ws.send('Hello!');
    ws.onmessage = (event) => console.log(event.data);

    // Server (Node.js, using 'ws' library)
    wss.on('connection', (ws) => {
      ws.on('message', (msg) => {
        wss.clients.forEach(client => client.send(msg)); // broadcast
      });
    });

```

Use cases: real-time chat, live dashboards, multiplayer games,
collaborative editing (like Google Docs), live notifications, stock
tickers.

Compared to polling (client repeatedly asking "anything new?") and SSE
(Server-Sent Events --- server pushes to client, but client can't send),
WebSockets are the only true bidirectional option.

> SSE is simpler and sufficient for one-way push (notifications, live
> feeds). WebSockets shine when you need the client to also send data in
> real-time (chat, gaming). In production, Socket.io adds reconnection,
> rooms, and fallbacks.

# 17. Boilerplate: Simple GET Route

Express.js (Node.js):

```js

    const express = require('express');
    const app = express();
    app.use(express.json()); // parse JSON bodies

    app.get('/api/users/:id', async (req, res) => {
      try {
        const { id } = req.params;
        const user = await db.findUserById(id);
        if (!user) return res.status(404).json({ error: 'Not found' });
        res.status(200).json({ data: user });
      } catch (err) {
        res.status(500).json({ error: 'Internal server error' });
      }
    });

    app.listen(3000, () => console.log('Server on port 3000'));
```

FastAPI (Python):

```py
    from fastapi import FastAPI, HTTPException

    app = FastAPI()

    @app.get("/users/{user_id}")
    async def get_user(user_id: int):
        user = await db.find_user(user_id)
        if not user:
            raise HTTPException(status_code=404, detail="Not found")
        return {"data": user}

```
> Always: use try/catch, return proper HTTP status codes (200 OK, 201
> Created, 400 Bad Request, 401 Unauthorized, 404 Not Found, 500 Server
> Error), and validate/sanitize input. Never trust req.params or
> req.body directly in production.

# 18. MVC Architecture

MVC (Model-View-Controller) is a design pattern that separates an
application into three interconnected layers:

Model --- The data layer. Represents the business data and logic. Talks
to the database. Defines schemas, queries, and validations. In a Node
app, this would be your Mongoose models or Prisma schema.

View --- The presentation layer. What the user sees. In traditional web
apps, this is HTML templates (EJS, Handlebars). In modern apps, it's
your React components.

Controller --- The glue. Handles incoming requests, calls the Model to
get/update data, and sends the response (or tells the View what to
render). Contains your route handler logic.

```js

    // Controller
    async function getUser(req, res) {
      const user = await UserModel.findById(req.params.id); // calls Model
      res.json(user); // sends to View (client)
    }

    // Model
    const UserModel = {
      findById: (id) => db.query('SELECT * FROM users WHERE id = $1', [id])
    };

```

The benefit is separation of concerns --- you can change your database
(Model) without touching the UI (View), or redesign the UI without
touching business logic.

> In REST APIs with React on the frontend, the "View" is essentially the
> React app. The Express layer is just Model + Controller (sometimes
> called MC or just "backend API"). That's fine --- MVC is a pattern,
> not a rigid rule.

# 19. SQL vs NoSQL Databases

The choice comes down to your data structure, scale requirements, and
consistency needs.

SQL (Relational) --- Data is organized in tables with rows and columns.
Schema is defined upfront. Relationships enforced with foreign keys.
ACID-compliant (Atomicity, Consistency, Isolation, Durability). Queried
with SQL. Examples: PostgreSQL, MySQL.

Best for: structured, relational data (users, orders, products),
financial systems where consistency is critical, complex queries with
JOINs.

NoSQL --- Flexible schemas. Documents, key-value pairs, graphs, or wide
columns. Scales horizontally more easily. Eventual consistency (in most
cases). Examples: MongoDB (documents), Redis (key-value), Cassandra
(wide column), Neo4j (graph).

Best for: unstructured or rapidly changing data, huge write throughput,
caching (Redis), real-time analytics.

```sql
    -- SQL: strict schema
    CREATE TABLE users (id SERIAL PRIMARY KEY, email TEXT NOT NULL);

```

```js

    // MongoDB: flexible document
    { _id: ObjectId, email: "a@b.com", preferences: { theme: "dark" } }
```

A common real-world pattern: PostgreSQL for primary application data +
Redis for caching/sessions.

> Don't frame it as "which is better" --- frame it as tradeoffs. SQL =
> stronger consistency, complex queries. NoSQL = flexibility, horizontal
> scale. Many production systems use both.

# 20. Idempotency

An operation is idempotent if calling it multiple times produces the
same result as calling it once. The key word is "result" --- not
necessarily "side effect count."

In HTTP: GET, PUT, DELETE are idempotent. POST is not. Sending the same
PUT request 5 times should leave the resource in the same state as
sending it once. Sending the same POST 5 times creates 5 resources.

Why does this matter? Networks are unreliable. Clients retry requests on
timeout. If your payment endpoint isn't idempotent, a retry could charge
the user twice.

The standard solution is an idempotency key : the client generates a
unique ID for each logical operation and sends it as a header (
Idempotency-Key: uuid ). The server stores the result of the first
request. If the same key comes again, it returns the cached result
without re-executing.

```js

    // Server-side idempotency check
    app.post('/payments', async (req, res) => {
      const key = req.headers['idempotency-key'];
      const cached = await redis.get(key);
      if (cached) return res.json(JSON.parse(cached));

      const result = await processPayment(req.body);
      await redis.setex(key, 86400, JSON.stringify(result));
      res.json(result);
    });

```
> Stripe uses this pattern for every payment API call. It's a must-know
> for any fintech or payment system interview. They'll respect you for
> bringing it up proactively.
