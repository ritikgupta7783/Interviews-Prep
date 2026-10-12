# NeuroBreath Wellness Platform — 80/20 Interview Questions

**Resume framing:** HTML, CSS and JavaScript wellness app for guided breathing/relaxation with interactive animations, session tracking, responsive UI and accessibility/performance claims.

**Source inspected:** `index.html`, `script.js`, `style.css` from the supplied archive. This is a small static frontend project, so focus on browser behavior, state, timing, responsive design, accessibility and maintainability—not backend/system-design topics that do not exist.

## P0 — Must answer confidently

### 1. Explain the project in 30–60 seconds.
**Answer cue:** “NeuroBreath is a browser-based wellness interface that guides users through breathing sessions using animated visual cues. HTML structures the experience, CSS styles and animates it, and JavaScript controls user interactions and session timing.” Explain the actual implemented session tracking from `script.js`, not features you intended but did not finish.

### 2. How does a breathing session work from click to finish?
**Answer cue:** Trace the exact button/event handler in `script.js`, the state/timer changes, DOM updates and end-of-session UI. Know what begins/stops the timer, how the current phase changes, and what happens if the user pauses/restarts or leaves the page.

### 3. How do HTML, CSS and JavaScript divide responsibilities?
**Answer cue:** HTML is semantic structure; CSS handles layout, responsive styling and visual transitions; JS owns interaction/state/timing. Keep content, presentation and behavior separate so each is easier to test and change.

### 4. How do you keep the animation synchronized with the breathing instructions?
**Answer cue:** Explain the actual timing source and phase transitions. Avoid relying on repeated `setInterval` ticks as a precise clock; timers can drift/background tabs can throttle them. A robust approach computes elapsed time against timestamps and derives the current phase.

### 5. What happens if a user starts a second session while one is running?
**Answer cue:** Check whether old intervals/timeouts are cleared before starting another. If not, multiple timers may update the same UI. Use a single timer handle, explicit session state, and cleanup on stop/reset/unload.

### 6. How did you make it responsive?
**Answer cue:** Point to actual CSS media queries, flexible dimensions, viewport behavior and touch target sizes. Test at mobile widths, tablet and desktop. Do not claim tested device coverage unless you actually tested it.

### 7. What does accessibility mean for this interface?
**Answer cue:** Keyboard-operable controls, visible focus, semantic buttons/labels, adequate contrast, readable instructions, reduced-motion support, and not relying only on color/animation to convey phase. Distinguish current support from improvements.

### 8. What data does session tracking store?
**Answer cue:** Identify whether the current code tracks elapsed time, completed sessions or counts only in memory/local storage. Do not imply server-side accounts or durable cross-device history if none exists.

### 9. How would you test the app?
**Answer cue:** Test start/pause/resume/reset/end; ensure one timer only; verify phase sequence/durations; test background-tab behavior; keyboard navigation; reduced-motion preference; and responsive layouts. Manual tests are not the same as automated tests.

### 10. What would you improve first?
**Answer cue:** Prioritize correctness of the session state machine and timer cleanup, then accessibility/reduced motion, mobile usability, and clear session summaries. Add persistence only if it helps users and privacy expectations are clear.

## P1 — Strong follow-up questions

11. What is the DOM and how does JavaScript update it?
12. What is event bubbling, and when might event delegation help?
13. What is the difference between `setTimeout` and `setInterval`?
14. Why do browser timers drift, and how can elapsed-time calculations reduce drift?
15. How would you model phases such as inhale, hold, exhale and rest as a state machine?
16. What should happen if the browser tab becomes hidden?
17. How would you support `prefers-reduced-motion`?
18. What are semantic HTML elements and why use a real `<button>`?
19. How do CSS Flexbox and Grid differ, and which layout is used in the project?
20. How do you avoid layout shifts when the animation or text changes?
21. How would you persist session history locally? What are the privacy and data-loss trade-offs?
22. How would you validate that a session completed rather than being abandoned?
23. How would you keep the UI responsive if more features or analytics were added?
24. What does “performance optimized” mean, and how would you measure it?
25. How would you check keyboard navigation and screen-reader behavior?
26. What is the difference between visual animation and the actual breathing timer state?
27. How would you organize a small app if `script.js` became difficult to maintain?
28. What did you personally build, and which feature would you demonstrate live?

## P2 — Code-level cross-questions

29. Which function is the entry point for interaction?
30. Which variables represent current phase, time remaining and session status?
31. What happens if a DOM selector returns `null`?
32. What cleanup is required when resetting or leaving a session?
33. What edge cases arise when a duration is zero, negative or not a number?
34. How would you prevent double-clicks from starting duplicate timers?
35. Why should the visual cue not be the only source of breathing instructions?
36. How would you test phase transitions deterministically?
37. What is the difference between storing data in a JS variable and `localStorage`?
38. How would you avoid claiming a medical benefit from a general wellness tool?

## Final readiness test

Open `index.html`, `style.css` and `script.js`; trace the start-to-finish path aloud. Be able to explain one timer-related edge case and one accessibility improvement without claiming either is already implemented if it is not.
