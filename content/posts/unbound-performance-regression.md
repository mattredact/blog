+++
title = 'Unbound 1.23.1 and the RFC 8767 performance regression'
date = 2025-09-10T10:00:00+00:00
draft = false
tags = ['dns', 'unbound', 'performance', 'infrastructure']
categories = ['tech']
+++

Upgraded to Unbound 1.23.1 and immediately noticed cache hit rates plummeting from ~85% to 45%. The issue stems from RFC 8767's implementation, specifically the `serve-expired-client-timeout` default changing from 0 to 1800ms.

The behavior is counterintuitive: when a query comes in for expired cache data, instead of serving the stale record immediately, Unbound now attempts a fresh upstream lookup if it can complete within 1.8 seconds. Only if the upstream query exceeds this timeout does it fall back to serving the expired data.

This fundamentally breaks local caching. Expired entries that could be served instantly now trigger unnecessary upstream queries, destroying cache efficiency. The RFC's intent—improving data freshness—comes at the cost of the primary benefit of running a local resolver: speed.

For self-hosted scenarios where you control the resolver and understand the trade-offs, this default is misguided. While I understand the security implications of serving stale data, setting the default to 1800ms feels rough, it cuts effective home user cache performance in half at minimum. Setting `serve-expired-client-timeout: 0` restores sanity, but it's concerning that the upstream project chose this as the new default without considering the performance implications for existing deployments.

After running with the timeout set back to 0 for one day, cache efficiency recovered to 83.1%:

![Unbound stats showing 83.1% cache hit rate](/images/unbound-stats.png)

Much better.

