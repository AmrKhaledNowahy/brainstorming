I’m AI Assistant.

Below are curated interview-style questions and model answers for a Senior Angular developer role. They cover basic topics you requested (data passing, decorators, pipes, directives) and progress to intermediate and advanced topics (change detection, zones, RxJS, DI, modules, AOT/Ivy, performance, router, testing, security, state management). Use these to practice explanations and whiteboard-style answers.

Basic (core) — fundamentals you should be fluent in
1) How do you pass data from parent to child and child to parent in Angular?
   Answer:
- Parent → Child: Use an @Input() property on the child.
- Child → Parent: Use an @Output() EventEmitter on the child and emit events the parent subscribes to in template bindings.
- Example:

```typescript
// TypeScript
// parent.component.html
// <app-child [item]="parentItem" (selected)="onChildSelected($event)"></app-child>

// child.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `<div (click)="select()">{{ item?.name }}</div>`
})
export class ChildComponent {
  @Input() item!: { id: number; name: string };
  @Output() selected = new EventEmitter<{ id: number }>();

  select() {
    this.selected.emit({ id: this.item.id });
  }
}
```


Notes:
- Use setter on @Input to react to changes.
- For complex interactions, consider a shared service (singleton) with Observables.

2) What are Angular decorators and give examples?
   Answer:
- Decorators are metadata annotations (functions) that modify classes, properties, parameters, or methods at design time. They tell Angular how to process and instantiate things.
- Common decorators:
    - @Component — marks a class as an Angular component and provides template, styles, change detection, etc.
    - @Directive — creates a directive.
    - @NgModule — declares module metadata.
    - @Injectable — marks a class as available to DI.
    - @Input, @Output — for component property bindings.
    - @HostListener, @HostBinding — interact with host element events and properties.
- Example: @Injectable used with providedIn to register a service.

3) What are Pipes, and how do you make a custom pipe? When would you use pure vs impure?
   Answer:
- Pipes transform data in templates (e.g., date, currency).
- Custom pipe example:

```typescript
// TypeScript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 20): string {
    if (!value) return '';
    return value.length > limit ? value.substr(0, limit) + '…' : value;
  }
}
```


- Pure vs impure:
    - Pure (default): executed only when input reference changes — best for performance.
    - Impure (pure: false): runs on every change detection cycle; needed when pipe depends on mutable inputs or non-pure data (e.g., array contents mutated).

4) What's the difference between structural and attribute directives? Give examples.
   Answer:
- Structural directives change DOM layout by adding/removing elements (they have a * syntax). Example: *ngIf, *ngFor, *ngSwitch.
- Attribute directives change the appearance or behavior of an existing element. Example: [ngClass], [ngStyle], custom directive that sets focus.
- Example of a simple attribute directive:

```typescript
// TypeScript
import { Directive, ElementRef, Renderer2, HostListener } from '@angular/core';

@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  @HostListener('mouseenter') onEnter() {
    this.renderer.setStyle(this.el.nativeElement, 'background-color', 'yellow');
  }
  @HostListener('mouseleave') onLeave() {
    this.renderer.removeStyle(this.el.nativeElement, 'background-color');
  }
}
```


Intermediate — architecture, DI, change detection
5) Explain Angular’s Dependency Injection hierarchy and provider scopes.
   Answer:
- Providers can be registered at:
    - root injector (providedIn: 'root' or providers in AppModule) → singleton application-wide.
    - NgModule providers → scoped to the module injector.
    - Component providers → new instance per component subtree.
- Hierarchical injectors allow child injectors to override parent providers.
- Use providedIn: 'root' for app singletons; component providers for isolated instances (e.g., component-local service state).

6) Describe Angular change detection. How do OnPush and Default strategies differ?
   Answer:
- Angular change detection processes component trees to update templates when data changes. It runs when:
    - events, XHR/promises, setTimeout, user interactions, manual triggers (ChangeDetectorRef.markForCheck/detectChanges).
- Default strategy: checks every component on each CD cycle.
- OnPush: component only checked when:
    - input reference changes
    - an event originated inside the component
    - observable bound via async pipe emits
    - manual markForCheck/detectChanges usage
- OnPush is recommended for performance; requires immutable patterns or manual change detection for mutated data.

7) How to manually trigger change detection or skip it for performance?
   Answer:
- Use ChangeDetectorRef:
    - detectChanges() — runs change detection on current component subtree.
    - markForCheck() — marks OnPush components for checking on next cycle.
    - detach() — detach component from change detection; reattach() to resume.
- Use NgZone.runOutsideAngular() to run heavy non-Angular work without triggering CD; when updates are needed, use NgZone.run() or ChangeDetectorRef to update.

Advanced — deep knowledge expected from senior
8) What are Angular Zones (NgZone)? How do they relate to change detection?
   Answer:
- Angular uses Zone.js to monkey-patch async APIs (setTimeout, promises, DOM events) to know when to run change detection.
- NgZone provides APIs:
    - run(fn) — execute with change detection enabled.
    - runOutsideAngular(fn) — execute without triggering change detection.
- Use runOutsideAngular for expensive loops/listeners and run to re-enter Angular zone to update UI.

9) Explain RxJS best practices in Angular (subscriptions, error handling, operators).
   Answer:
- Unsubscribe from Observables to avoid memory leaks:
    - Use async pipe in templates (auto unsubscribes).
    - Use takeUntil with a destroy subject in components.
    - Use first(), take(1) for single-value streams.
- Prefer higher-order mapping operators (switchMap vs mergeMap vs concatMap) depending on desired concurrency:
    - switchMap cancels previous inner observable → good for typeahead/search.
    - mergeMap runs all → good for parallel.
    - concatMap queues inner observables sequentially.
- Use shareReplay(1) carefully when multicasting HTTP responses.
- Handle errors with catchError and return user-friendly values or rethrow.
- Avoid nested subscriptions; use pipeable operators.

10) How does Angular’s router handle lazy loading and preloading? What are the performance trade-offs?
    Answer:
- Lazy loading via loadChildren in route config loads module on demand — improves initial bundle size.
- Preloading strategies:
    - PreloadAllModules — loads lazily configured modules after app stabilizes to speed navigations.
    - Custom preloading — selective preloading based on conditions.
- Trade-offs: lazy loading reduces initial payload but adds navigation latency. Preloading mitigates navigation latency at cost of additional network transfer after initial load.

11) What is AOT compilation vs JIT? Benefits of AOT and Ivy?
    Answer:
- JIT compiles templates in the browser at runtime.
- AOT compiles templates at build time to JS, producing smaller bundles and faster startup.
- Ivy (renderer since Angular 9) benefits:
    - smaller bundles, faster builds, improved tree-shaking, better debugging, incremental compilation.
    - Locality of generated code: components compiled independently → faster ngcc and faster builds.

12) How do you structure large Angular applications (modules, folders, shared vs core, feature modules)?
    Answer:
- Use feature modules to encapsulate features (with routing where applicable).
- CoreModule: singleton services and global guards/interceptors (imported only by AppModule).
- SharedModule: shared components, directives, pipes (exported, imported by feature modules).
- Lazy-load feature modules for large app.
- Use domain-driven module boundaries, keep components small and single-responsibility.
- Use explicit public APIs (index.ts) for modules/libraries.

13) How to optimize performance (bundles, runtime, change detection)?
    Answer:
- Bundle-level:
    - Use lazy loading, route-based splitting, and differential loading if needed.
    - Enable build optimizer, AOT, and production mode (ng build --prod).
    - Use gzip/Brotli compression on server.
- Runtime:
    - Use OnPush, immutable data, and avoid expensive computations in templates.
    - Virtual scrolling for long lists (cdk-virtual-scroll-viewport).
    - Use trackBy in ngFor.
    - Debounce inputs and throttle events.
- Network:
    - Cache responses, use service workers (Angular PWA) for offline and caching.
- Profiling:
    - Use Chrome DevTools, Augury, or Angular profiler to find bottlenecks.

14) Describe security best practices in Angular apps (XSS, CSRF, safe handling).
    Answer:
- XSS: Angular templates sanitize values by default. Avoid bypassSecurityTrust… unless absolutely necessary and audited.
- Use HttpClient which provides XSRF protection integration with server cookies and headers (HttpClientXsrfModule).
- Validate/sanitize user inputs server-side; never trust client-side only.
- Content Security Policy headers, secure cookies (HttpOnly, Secure, SameSite), and proper CORS config on server.
- Avoid storing sensitive tokens in localStorage if possible; prefer secure cookies or short-lived tokens + refresh on server.

15) How do you test Angular components and services? What to mock and when?
    Answer:
- Unit tests:
    - Use TestBed to configure component/service with mocked dependencies.
    - Use shallow tests with NO_ERRORS_SCHEMA or MockComponents for isolated component tests.
    - Mock HttpClient with HttpTestingController for HTTP interactions.
    - Mock services when unit-testing a component to control behavior deterministically.
- Integration tests:
    - Use real providers but mocked APIs (in-memory backend or HttpTestingController).
- E2E tests: Cypress or Protractor (though Protractor deprecated) for user flows.
- Focus on small, fast unit tests; integration tests for cross-cutting behaviors.

Deep/advanced questions — be prepared to whiteboard
16) Explain change detection tree walk cost and strategies to minimize re-rendering.
    Answer:
- In Default strategy, a CD cycle walks the whole component tree checking bindings — costly for large trees.
- Use OnPush to reduce checks to components with changed inputs; detach rarely-changing subtrees.
- Use immutable data to ensure reference changes trigger OnPush.
- Use trackBy in ngFor to avoid DOM re-create on list updates.
- Move expensive computations out of template into memoized pipes or precomputed properties.

17) How do you handle global error handling and logging?
    Answer:
- Use ErrorHandler class (provide custom implementation) to catch uncaught exceptions.
- Use HTTP interceptors to catch HTTP errors and centralize retry logic or mapping.
- Integrate logging services that send errors to monitoring backends (Sentry, LogRocket, Datadog).
- Avoid exposing sensitive information in error payloads shown to users.

18) Describe how to build a library in Angular and publish it.
    Answer:
- Use Angular CLI ng generate library <name>.
- Keep well-defined public API (public-api.ts).
- Build library with ng build <lib>.
- Use ng-packagr for packaging; publish the generated package to npm with versioning.
- Ensure peerDependencies for Angular versions and include TypeScript types.

19) How would you implement server-side rendering (SSR) and why?
    Answer:
- Use Angular Universal to render pages on server.
- Benefits: improved SEO, faster first meaningful paint, better perceived performance.
- Considerations: avoid direct DOM usage (use PLATFORM_ID & isPlatformBrowser/isPlatformServer), handle transfer state to avoid duplicate HTTP calls (TransferState API), manage secrets carefully.

Practical/Scenario questions — deeper thinking
20) You have a large list rendered by ngFor and items frequently update. Rendering is janky. How do you approach fixing it?
    Answer:
- Use trackBy to avoid recreation of DOM nodes.
- Use OnPush and ensure updates change object references.
- Virtual scroll (cdk-virtual-scroll-viewport) to render only visible items.
- Move heavy computations out of template; memoize using pure pipes or caching.
- Debounce frequent updates and batch changes.

21) How to share state across many components and pages in a large app?
    Answer:
- Options:
    - Service with BehaviorSubject/ReplaySubject (simple).
    - Use state management library (NgRx, Akita, NgXs) for complex apps: provides immutability, actions, selectors, devtools, time-travel debugging.
    - Component-local service providers for feature-local state.
- Choose based on team familiarity and complexity. For simple requirements, Rx-based services are often adequate.

22) Explain how HTTP interceptors work and give practical uses.
    Answer:
- Interceptors implement HttpInterceptor; they intercept outgoing requests and incoming responses.
- Use cases:
    - Add Authorization headers or refresh tokens.
    - Centralized error handling and retry logic.
    - Request/response logging.
    - Mocking/responses for testing.
- Interceptors chain: order matters as provided in module.

Quick knowledge checks (short answers you should give concisely)
23) What is ViewChild vs ContentChild?
    Answer:
- @ViewChild selects an element/component from the component's own template.
- @ContentChild selects projected content (ng-content) passed from parent.
- Use afterViewInit / afterContentInit lifecycle hooks respectively to access them.

24) When is ngOnInit called vs constructor?
    Answer:
- constructor runs when class is instantiated (no bindings resolved).
- ngOnInit runs after inputs are bound — safe to access @Input values there.

25) When to use ngZone.runOutsideAngular?
    Answer:
- For heavy non-UI async tasks or continuous events (e.g., canvas animation, third-party libs) to avoid frequent change detection. Re-enter with run() when UI update is required.

Behavioral / senior-level design questions
26) How would you onboard a junior team member to an Angular codebase?
    Answer:
- Walk through app architecture (modules, key services, state flow).
- Pair-program on a small task: bugfix or new component.
- Share coding standards, lint rules, testing approach, CI/CD pipeline.
- Provide small checklist for PRs and review guidance.

27) How do you approach incremental large refactors in production applications?
    Answer:
- Break work into small, testable, deployable steps.
- Use feature flags to toggle new behavior.
- Add unit/integration tests around behavior before changing.
- Keep backward compatibility where possible; migrate gradually.

Resources to master and demonstrate in interviews
- Angular documentation (official), RxJS docs, Angular style guide, NgRx docs, performance profiling tools, Zone.js basics.
- Demonstrate sample projects: feature module with lazy loading, shared component library, using OnPush, virtual scroll, and unit tests.

Practice tips
- Be ready to walk through a small code sample live: create a component, explain change detection implications, show how you'd add OnPush, and how you'd test it.
- Explain trade-offs and mention maintainability and team skillset when proposing libraries/architectural choices.
- Use concrete examples and metrics when possible (bundle size reductions, startup improvements).

If you want, I can:
- Generate 30-50 targeted interview questions with short model answers organized by junior/intermediate/senior.
- Create a one-hour mock interview script with follow-ups and ideal answers.
- Provide code snippets for specific topics (e.g., OnPush + immutable state example, custom RxJS operator, HTTP interceptor + token refresh flow).

Which of those would you like next?