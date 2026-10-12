# InsightStudy — 80/20 Interview Questions

**Resume framing:** Founder & Frontend Developer in a group project; deployed ed-tech platform with 100+ registered users; React frontend and REST API integration for authentication, user management and course delivery. The source archive also contains backend routes/services for a Gemini chatbot, OCR image questions, MongoDB, Redis/BullMQ and a delayed unverified-user deletion worker. Verify which of these features are live and user-facing before claiming them.

**Source inspected:** `backend/src/index.js`, `route/auth.route.js`, `route/chatbot.route.js`, `controller/chatbot.controllers.js`, `services/ai.service.js`, `services/ocr.services.js`, `middleware/auth.middleware.js`, `queues/userqueues.js`, `workers/workers.js`, and package manifest.

## P0 — Must answer confidently

### 1. What is InsightStudy and what did you personally contribute?
**Answer cue:** Explain the learning problem and user journey, then state your real personal scope: resume says founder and frontend developer in a group project, with React UI and REST API integration. Separate your work from teammates' backend/database work. Use a specific page/feature you can demonstrate rather than claiming sole ownership of the whole stack.

### 2. What does “100+ registered users” mean?
**Answer cue:** Explain the source of the count (database count, analytics, or admin data) and date if you know it. If you cannot verify the exact method, say the figure is from the project/resume and avoid implying 100+ active users or retention.

### 3. Draw the architecture and main user flow.
**Answer cue:** React/Vite frontend ↔ Express backend routes ↔ MongoDB for user/account data; auth middleware reads JWT cookie. The backend also contains Gemini chat service and OCR via Tesseract.js. Redis/BullMQ code schedules deletion of unverified users. Do not assume every module is on the main learning flow or production deployment without verifying.

### 4. How does authentication work in the backend?
**Answer cue:** `protectroute` reads `req.cookies.jwt`, verifies it with `JWT_SECRET`, loads the user from MongoDB using the decoded user ID, excludes the password, attaches `req.user`, and calls `next()`. Mention the actual cookie flags and signup/login/logout implementations after reading `auth.controller.js`.

### 5. How does the chatbot handle a text question?
**Answer cue:** `POST /api/v1/bot/chat` maps to `chat` controller. It checks that `message` exists, selects a supplied or default session ID, calls `generateAIResponse`, and returns reply/session ID. The AI service constructs an NCERT-oriented prompt and calls the configured Gemini model.

### 6. How does image question answering work?
**Answer cue:** `POST /api/v1/bot/upload` uses Multer field `image`; controller checks for a file, runs `performOCR`, then sends extracted text to the Gemini-backed service and returns a reply. OCR is a separate stage from language-model generation. OCR errors and low-quality images need explicit handling.

### 7. What is OCR, and what are its limitations?
**Answer cue:** Optical Character Recognition converts image pixels containing text into machine-readable text. Accuracy depends on resolution, handwriting, rotation, contrast, language and layout. The code uses a singleton Tesseract worker for English; OCR text can be noisy and should not be treated as ground truth.

### 8. What is the risk of in-memory `sessionHistories`?
**Answer cue:** It is a JavaScript `Map`, so conversation history exists only in the process memory. It disappears on restart, is not shared across multiple server instances, can grow without bounds, and can mix users if session IDs are guessable/shared. A persistent store with ownership, TTL, limits and isolation would be a future improvement.

### 9. Why use BullMQ and Redis for delayed user deletion?
**Answer cue:** `scheduleUserDeletion` enqueues a `deleteUser` job with a five-minute delay. A BullMQ `Worker` processes the job and deletes a user only if found and `!user.isVerified`. This moves delayed work out of the request path. Explain what happens if Redis is unavailable and whether retry/retention policies are configured.

### 10. What is the most important security concern you found?
**Answer cue:** The supplied archive includes Redis connection credentials directly in `backend/src/config/redis.js`. Do not repeat the value. If it is a real active credential, rotate/revoke it immediately, remove it from source history where possible, and load secrets from environment/secret storage. Also review session isolation, upload limits, auth route protection and error handling. Never publish the credential in this study repo.

## P1 — Strong follow-up questions

11. Why is React a suitable choice for the frontend?
12. How does the frontend call REST APIs and represent loading/error states? Cite the exact component/API service you worked on.
13. What is the difference between authentication and authorization?
14. Why use HttpOnly cookies instead of storing JWT in localStorage?
15. Why does CORS use `credentials: true`, and why must the origin be explicit?
16. What does `User.findById(decoded.userId).select('-password')` do?
17. Why should a missing/invalid token return 401 rather than a generic 500?
18. What happens if Gemini is slow, rate-limited or unavailable?
19. How would you limit prompt injection and prevent the chatbot from following hostile instructions embedded in OCR text?
20. What is the trade-off between keeping chat history in memory and storing it in MongoDB/Redis?
21. How would you add per-user quotas and rate limiting to an expensive AI endpoint?
22. How would you validate image MIME type, size, dimensions and file contents?
23. Why is deleting an unverified account after a delay better than deleting immediately after signup?
24. What happens if the user verifies at the same time the delayed deletion job runs?
25. How would you make delayed deletion safe to retry and idempotent?
26. What should happen to OCR-uploaded files after processing? Does the code delete them on every path?
27. How would you evaluate the chatbot's educational answer quality?
28. How would you distinguish a registered user from a monthly active user?
29. How would you debug “frontend request works locally but fails after deployment”?
30. How would you handle email OTP expiration, reuse and brute-force attempts? Answer from actual controller code, not assumptions.

## P2 — Code-level cross-questions

31. Why does the Express app load `dotenv.config()` before using environment variables?
32. What is the purpose of the global `errorHandler` middleware?
33. Why does `performOCR` initialize one worker instead of creating one for every image?
34. What can go wrong if the worker is never terminated on shutdown?
35. Why should user-provided `sessionId` be validated and scoped to the authenticated user?
36. What happens if two requests update the same session history at the same time?
37. What is a Redis queue job versus a normal function call?
38. What is the difference between a queue producer and worker?
39. What if Redis credentials expire or TLS certificate validation fails?
40. How would you monitor failed jobs and retry them safely?
41. Why is it dangerous to commit API keys, database passwords or certificates?
42. Which exact frontend components did you build, and what design/API integration decision can you defend?
43. Which backend functions were written by teammates, and how did you coordinate the API contract?
44. What user feedback changed a feature or interface?
45. What would you improve first: security, AI quality, reliability, or UX—and why?

## Honest ownership checklist

Prepare a short “I owned / I collaborated on / another teammate owned” list. The resume identifies this as a group project and your role as founder & frontend developer; do not claim authorship of every backend subsystem unless accurate.
