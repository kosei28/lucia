---
order: 11
layout: "@layouts/DocumentLayout.astro"
title: "Handle errors"
---

Errors handled by throwing [`LuciaError`](/reference/types/lucia-types#luciaerror) inside Lucia. A list of error messages are provided in [Errors](/reference/types/errors) reference and the API reference lists errors thrown by each method.

Using a try-catch block, the error message can be read with like so:

```ts
import { LuciaError } from "lucia-sveltekit";
import { auth } from "$lib/server/lucia";

try {
    await auth.createUser()
} (e) catch {
    if (e instanceof LuciaError) {
        const message = e.message
    }
}
```

However, due to the nature of Lucia and how it uses database adapters, it cannot catch every single database errors and it does not expect the adapters to do so. This means, errors that are expected (known errors - listed below) are caught and thrown using `LuciaError`, while unexpected errors, including ones related to user attributes, are handled by re-throwing the database error. This means errors like user attributes violating foreign or unique constraints and connection errors, must be handled by the user. 

For example, you may have a `username` unique column inside the `user` table. If you were using Prisma and you tried to create a user with an existing username using `createUser()`, it will throw a Prisma error and not `LuciaError`. This can be handled outside of Lucia or by providing an error handler to your database adapter.

```ts
try {
     await auth.createUser()
} catch (e) {
    if (e instanceof Prisma.PrismaKnownClientError) {
        // check error code
    }
}
```

## Known errors

Known errors for databases related actions are:

-   Duplicate provider id on user creation and update (`AUTH_DUPLICATE_PROVIDER_ID`)
-   Invalid user id on session creation and update (`AUTH_INVALID_USER_ID`)
-   Duplicate session id on session creation and renewal (`AUTH_DUPLICATE_SESSION_ID`)