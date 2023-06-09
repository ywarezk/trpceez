# @trpceez

`@trpceez` is a framework for creating, micro service oriented, class based, trpc api.

## What we believe

Here is a list of our main belief system, that will help us understand what we want to include in this library.

### 1. class based approach

we use class based approach and we use decorators to signal what is the task of each class.  
This approach is heavily inspired by nestjs, angular, type-graphql, etc.

So if we want to create a trpc router, it will be done with:

```typescript
import { Router } from '@trpceez/routers';

@Router()
export class HelloRouter {...}
```

If we want to create a trpc query it will be done with:

```typescript
import { Router, Query, Input, Output } from '@trpceez/routers';
import { z } from 'zod'

@Router()
export class HelloRouter {
	@Query()
	@Input(z.object({ ... }))
	@Output(z.object({ ... }))
	async hello({input, ctx}) {... }
}
```

### 2. Incremental adoption

We do not force the entire trpc api to be created using `@trpceez/*`.  
If you already have a trpc api, and you simply want to create your next trpc router with a class based approach, then no need to change all your routers.  
Simply create your class based router using `@trpceez` approach:

```typescript
import { Router } from '@trpceez/routers';

@Router()
export class HelloRouter {...}
```

you can then grab your `HelloRouter` and transform it to a regular `trpc` router and connect it to your app:

```typescript
import { HelloRouter } from './hello.router';
import { toTrpcRouter } from '@trpceez/routers';
import { t } from './trpc'; // @see https://trpc.io/docs/server/routers

export const helloTrpcRouter = toTrpcRouter(t, HelloRouter);
```

You can then take your router and use regular trpc to join with the rest of your routers.

```typescript
import { router } from './trpc';
import { helloTrpcRouter } from './hello-trpc.router'; 
import {fooRouter, barRouter} from './my-other-routers';

export const appRouter = router({
    hello: helloTrpcRouter,
    foo: fooRouter,
    bar: barRouter
});
```

### 3. Micro service approach

We need to be ready to create not a single trpc api but many.  
Software projects can become huge, with many developers, many websites, where each website might have many sections.  
Therfor there should not be a single server with a single trpc api, rather we need to split our backends as well, creating many trpc api's which will be stitched for the client in a single gateway.

```typescript
import { Application } from '@trpceez/application';
import { HelloRouter, FooRouter, BarRouter } from './my-routers';

@Application({
	routers: [
		HelloRouter, FooRouter, BarRouter
	]	
})
export class GreetingApplication {}
```

Our end result should not be a single `/trpc` url, rather we need to strive for:

```
/trpc/greeting/hello.sayHello
/trpc/common/someRouter.commonTask
```

divide our trpc to sections.

### 4. client should not care about the micro service approach

Client should not care about the micro service architecture in the backend.  
He should have a typescript validated gateway for connecting to each on of the backends

```typescript
import { Federation } from '@trpceez/federation';
import { NerdeezApplication, AcademeezApplication, CommonApplication} from './my-applications'

@Federation({
	applications: [
		NerdeezApplication, AcademeezApplication, CommonApplication
	]
})
export class Api {}
```

From here we can construct our entire trpc gateway:

```typescript
import type { Api } from './my-federation';
import type { inferFederation } from '@trpceez/federation';

export const trpc = createTRPCReact<inferFederation<Api>>();

```

the client can now using this single `trpc` to communicate with each one of the microservices, with full ide autocompletion as well as full typescript check:

```typescript
const { data } = trpc.nerdeez.hello.greetings.useQuery();
```

### 5. Bugs should be thrown as much as possible to typescript

This goal is inherited from trpc goals, use typescript as best as we can to move the bugs, including the most repeated bugs that derive from client server communication, move these bugs to typescript compilation instead of runtime.

## Class and Decorators