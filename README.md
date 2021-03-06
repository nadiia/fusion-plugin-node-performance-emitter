# node-performance-emitter

[![Build status](https://badge.buildkite.com/cd218d02957b19e9397231aea7fe019ea61f5e50225d7a47a8.svg?branch=master)](https://buildkite.com/uberopensource/fusion-plugin-node-performance-emitter)

A tool for collecting, aggregating, and emitting Node.js performance stats.

Collects stats on the event loop lag, garbage collection events, and memory statistics.  Note that this plugin is server-side only and will throw an error if used browser-side.

---

### Example

```js
// main.js
import App from 'fusion-react';
import Root from './components/root';
import UniversalEvents from 'fusion-plugin-universal-events';
import Analytics from './plugins/analytics';

import NodePerformanceEmitter from 'fusion-plugin-node-performance-emitter';

export default function() {
  const app = new App(<Root />);
  const EventEmitter = app.plugin(UniversalEvents);

  if(__NODE__) {
    const PerfStats = app.plugin(NodePerformanceEmitter, {EventEmitter});
    app.plugin(AnalyticsPlugin, {PerfStats});
  }

  return app;
}

// plugins/analytics.js
export default function({PerfStats}) => (ctx, next) => {
  const perf = PerfStats.of(ctx); // Get an instance of the NodePerformanceEmitter
  perf.start(); // Start collecting stats

  /* do stuff! */

  perf.stop(); // Stop collecting stats
  return next();
}
```

---

### API

```js
const PerfStats = app.plugin(NodePerformanceEmitter, {EventEmitter, config})
```

- `EventEmitter` - Required. An event emitter plugin to emit stats events to
- `config: Object` - Optional
  - `eventLoopLagInterval: number` - Optional. Stats emission frequency for event loop lag stats. Defaults to 10,000 (ms)
  - `memoryInterval: number` - Optional. Stats emission frequency for memory stats. Defaults to 10,000 (ms)
  - `socketInterval: number` - Optional. Stats emission frequency for socket stats. Defaults to 10,000 (ms)

The following events are emitted through the `EventEmitter`:

- `node-performance-emitter:gauge:event_loop_lag`
- `node-performance-emitter:gauge:rss`- process.memoryUsage().rss
- `node-performance-emitter:gauge:externalMemory` - process.memoryUsage().external
- `node-performance-emitter:gauge:heapTotal` - process.memoryUsage().heapTotal
- `node-performance-emitter:gauge:heapUsed` - process.memoryUsage().heapUsed
- `node-performance-emitter:timing:gc` - time spent doing garbage collection
- `node-performance-emitter:gauge:globalAgentSockets` - http.globalAgent.sockets
- `node-performance-emitter:gauge:globalAgentRequests`- http.globalAgent.requests
- `node-performance-emitter:gauge:globalAgentFreeSockets`- http.globalAgent.freeSockets


