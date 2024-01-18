## getOwner
why this exists? 

Internally, computations (effects, memos, etc.) create owners which are children of their owner, all the way up to the root owner created by createRoot or render

when a createEffect's dependencies change, the effect calls all descendant onCleanup callbacks before running the effect function again

Components are not computations, so do not create an owner node, but they are typically rendered from a createEffect which does, so the result is similar: when a component gets unmounted, all descendant onCleanup callbacks get called. 

## runWithOwner
Having a (correct) owner is important for two reasons:

Computations without an owner cannot be cleaned up. For example, if you call createEffect without an owner (e.g., in the global scope), the effect will continue running forever, instead of being disposed when its owner gets disposed.
useContext obtains context by walking up the owner tree to find the nearest ancestor providing the desired context. So without an owner you cannot look up any provided context (and with the wrong owner, you might obtain the wrong context).

Manually setting the owner is especially helpful when doing reactivity outside of any owner scope. In particular, asynchronous computation (via either async functions or callbacks like setTimeout) lose the automatically set owner, so remembering the original owner via getOwner and restoring it via runWithOwner is necessary in these cases.

## createEffect
The first execution of the effect function is not immediate; it's scheduled to run after the current rendering phase. Note that the first run of the effect still runs before the browser renders the DOM to the screen 

## createRenderEffect
differs in when Solid schedules the first execution of the effect function. While createEffect waits for the current rendering phase to be complete, createRenderEffect immediately calls the function.

 Indeed, Solid uses createRenderEffect to implement the rendering phase itself, including setting of refs.

Reactive updates to render effects are identical to effects: they queue up in response to a reactive change (e.g., a single signal update, or a batch of changes, or collective changes during an entire render phase) and run in a single batch afterward (together with effects).

## createComputed
Like createRenderEffect, createComputed calls its function for the first time immediately. But they differ in how updates are performed. While createComputed generally updates immediately, createRenderEffect updates queue to run (along with createEffects) after the current render phase.

## createContext, useContext
Context provides a form of dependency injection in Solid. It is used to save from needing to pass data as props through intermediate components.




## lazy, createResource
lazy uses createResource internally to manage its dynamic imports

## Suspense, SuspenseList

[SuspenseList example](https://www.solidjs.com/tutorial/async_suspense_list?solved)

## useTransition, startTransition
Used to batch async updates in a transaction deferring commit until all async processes are complete. This is tied into Suspense and only tracks resources read under Suspense boundaries.
[async transitions](https://www.solidjs.com/tutorial/async_transitions)

Suspense allows us to show fallback content when data is loading. This is great for initial loading, but on subsequent navigation it is often worse UX to fallback to the skeleton state.

We can avoid going back to the fallback state by leveraging useTransition. It provides a wrapper and a pending indicator. The wrapper puts all downstream updates in a transaction that doesn't commit until all async events complete.

- It continues to show the current branch while rendering the next off-screen
- Resource reads under existing boundaries add it to the transition
- However, any new nested Suspense components will show "fallback" if they have not completed loading before coming into view.

[example](https://www.solidjs.com/tutorial/async_transitions?solved)
