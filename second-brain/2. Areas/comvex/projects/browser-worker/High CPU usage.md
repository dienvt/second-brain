```
Root causes

  1. A brand-new Chrome process is launched for every job (biggest cause)

  Every job — GetSubmissions, GetSubmissionsPolling, StoreCredentials, CaptureScreenshot — calls client.NewContext(), which does:

  // infra/browser/chrome/client.go:93
  allocCtx, allocCancel := chromedp.NewExecAllocator(context.Background(), allocatorOptions...)

  NewExecAllocator spawns a fresh Chromium binary from scratch each time, then tears it down (defer allocCancel()). There is no browser pooling or reuse anywhere. Cold-starting Chromium is one of the most CPU-expensive things you can do — it forks a browser process + renderer + GPU + utility processes, JIT-compiles,
  and re-initializes everything. Since GetSubmissionsPolling runs on a schedule across many portals/accounts, you get a continuous stream of Chrome cold-starts → sustained high CPU.

  The comment at service.go:120 says a fresh context is needed per job "to avoid data leakage between accounts" — but that only requires a fresh tab/context (which FetchPageForParallelGet already does via chromedp.NewContext(ctx)), not a fresh browser process.

  2. chromium-swiftshader = software rendering on the CPU

  RUN apk add chromium && apk add chromium-swiftshader

  The container has no GPU, so Chrome falls back to SwiftShader, which emulates all GPU work (rasterization, compositing, WebGL) on the CPU. This is a classic headless-Chrome-in-container CPU sink, and it's made worse by chromedp.FullScreenshot(&screenshot, 90) in HandleScreenshot — which now runs on every fetch
  failure and re-renders/encodes the whole page in software.

  3. Missing container-tuning flags

  The allocator only adds ignore-certificate-errors, lang, timezone, and UA. It's missing the flags that cut container CPU/memory: --no-sandbox, --disable-software-rasterizer, --disable-extensions, --disable-background-*, --disable-features=.... Nothing disables the SwiftShader rasterizer.

  4. Long fixed sleeps keep heavy processes alive

  chromedp.Sleep(15*time.Second) (athome login), 6s, 5s, 2s sleeps everywhere. These don't burn CPU directly, but they hold a full Chrome process (with SwiftShader rendering loops) open far longer than needed, widening the CPU window — and if jobs run concurrently, multiple Chromes overlap.

  Recommended fixes (highest impact first)

  1. Reuse one browser, new tab per job. Create the ExecAllocator + browser context once at startup (in chrome.Client), then per job call chromedp.NewContext(browserCtx) to get an isolated tab. This eliminates ~all the cold-start CPU while preserving per-account isolation. ~90% of the win.
  2. Drop SwiftShader if you don't actually need to render pixels for most jobs — most scrapers here just need HTML (OuterHTML). Add --disable-software-rasterizer and reserve full rendering only for the screenshot path.
  3. Add the container flags (--no-sandbox, --disable-dev-shm-usage if not already, --disable-extensions, --disable-background-networking, etc.).
  4. Cap job concurrency so you never run more than N browsers/tabs at once.
  5. Replace fixed Sleeps with WaitVisible/WaitReady conditions where possible.

  Want me to implement #1 (shared browser + per-job tab) and #2/#3 (flags)? That's the combination that will move the CPU needle the most. I'd do it behind the existing chrome.Client so the service layer barely changes.
```

### Solution
change base image:
https://hub.docker.com/r/chromedp/headless-shell/
references: https://stackoverflow.com/questions/78965677/chromedp-and-high-cpu-usage-golang