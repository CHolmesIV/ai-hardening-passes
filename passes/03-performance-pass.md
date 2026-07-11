# The Performance Pass

AI-written apps are slow in boring, predictable ways. Not exotic algorithm problems: default problems. The model optimizes for "works in the demo," and in the demo there are nine rows and one user. Here are the five to check today. All of them are promptable fixes.

## 1. Unpaginated tables

The table has 5,000 rows, so the API sends 5,000 rows, so the browser renders 5,000 rows. Every time. Send a page. Twenty to fifty rows, a total count, and a way to get more. Server-side, not "fetch everything then slice in JavaScript."

**Work order:** "Find every list endpoint that returns unbounded rows. Convert each to server-side pagination with a total count. Show me the query LIMIT and the new response shape."

## 2. Every read hits the database

Same query, same result, full round trip, every request. See the [scale pass](02-scale-pass.md) for the three-layer fix. The point here: this is a performance problem long before it is a scale problem. Your one user feels it too.

## 3. Heavy work on the request thread

Report generation, file parsing, calling an LLM, sending email: if it happens inside the request handler, the user stares at a spinner and the server can not take the next request. Anything slower than a couple hundred milliseconds belongs in a background job with a status the UI can poll.

**Work order:** "List everything in a request handler that does network calls or heavy computation. Move the slow ones to a background worker and return a job id. Show me the queue and the status endpoint."

## 4. Nobody measured the actual page load

Largest Contentful Paint is the number that decides whether your app feels fast. Giant unoptimized images, render-blocking scripts, fonts loading late: all default AI output.

**Work order:** "Run Lighthouse against the main page. Report LCP and the top three contributors. Fix image sizing, defer non-critical scripts, and show me the before and after scores."

## 5. Event handlers firing constantly

Scroll and resize handlers that do real work on every pixel of movement will make the whole screen stutter. Throttle them, debounce them, or better, replace them with IntersectionObserver and let the browser do the watching.

**Work order:** "Find every scroll, resize, and input handler. For each, tell me how often it fires and what it does. Throttle or replace anything doing layout reads or state updates per event."

## The habit that beats all five

Ask for the audit before you ask for the feature. One prompt: "Before we add anything new, act as a performance engineer and profile what we have. Slowest endpoints, heaviest queries, biggest render costs. Numbers, then a ranked fix list." The model is good at this. It just never volunteers.
