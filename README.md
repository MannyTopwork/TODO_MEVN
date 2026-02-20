# todos app | fullstack mevn
Basic fullstack todolist app built using the MEVN stack.                         |


### Prerequisites

-   Rename server/config/examplekeys.js to server/config/keys.js

-   Modify the MongoDB connection string in server/config/keys.js to your own

-   Have NodeJS installed

## API Routes

-   GET /api/todos (retrieves todolist from DB and returns as an array)

-   GET /api/todo/:id (retrieves single todo task from DB and returns as object)

-   POST /api/todo (adds a todo task to the todolist in DB)

-   PUT /api/todo/:id (updates todo task in DB)

-   DELETE /api/todo/:id (deletes todo task in DB)

## Version

nodejs: 16.20.2
mongodb: 5.0.2

## License

[MIT](http://opensource.org/licenses/MIT)
Copyright (c) 2026-present Manny
