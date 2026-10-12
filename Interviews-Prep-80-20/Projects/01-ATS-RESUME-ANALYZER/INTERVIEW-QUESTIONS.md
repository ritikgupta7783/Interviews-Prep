# ATS Resume Analyzer & Interview Assistant — 80/20 Interview Questions

**Resume framing:** Full-stack AI app using React, Node.js, Express, MongoDB, Google Gemini AI and PDF processing. Resume claims resume/JD analysis, match score, skill gaps, interview questions, prep roadmap, authentication, report history and PDF generation.

**Source inspected:** `Backend/src/app.js`, `Backend/src/routes/auth.routes.js`, `Backend/src/routes/interview.routes.js`, `Backend/src/controllers/auth.controller.js`, `Backend/src/controllers/interview.controller.js`, `Backend/src/services/ai.service.js`, `Backend/src/middlewares/auth.middleware.js`, `Backend/src/middlewares/file.middleware.js`, `Backend/src/models/{user.model.js,interviewReport.model.js,blacklist.model.js}`, `Frontend/src/features/interview/services/interview.api.js`.

## P0 — Must answer confidently

### 1. Explain the project in 60 seconds.
**Answer cue:** “It is a full-stack interview-preparation app. A signed-in user supplies a job description, a self-description, and a resume PDF. The frontend sends multipart form data to an Express API. The backend extracts text from the PDF, sends the text and job context to Gemini through a service, validates/structures the expected response using a Zod schema, and stores a report in MongoDB. The user can reopen report history and generate a resume PDF.” Mention only the parts you personally built.

**Follow-up:** What happens if Gemini returns malformed output?  
**Answer cue:** The service asks for structured output/schema, but still must handle API errors and invalid responses. Explain the actual catch/validation behavior from `ai.service.js`; do not claim perfect guarantees.

### 2. Draw the architecture and trace one request.
**Answer cue:** React UI → Axios service (`interview.api.js`) → Express route `/api/interview/` → `authUser` middleware → Multer memory upload → controller → PDF text extraction → `ai.service.js`/Gemini → `interviewReportModel.create()` → JSON response → React report UI. Auth routes use `/api/auth`.

### 3. Why use a separate AI service instead of calling Gemini inside the controller?
**Answer cue:** Separation of concerns: controller coordinates HTTP and user context; service owns prompt/model integration and output structure. It is easier to test, change models, and handle provider-specific behavior in one place.

### 4. How does resume PDF upload work?
**Answer cue:** Frontend builds `FormData` with `jobDescription`, `selfDescription`, and `resume`; Axios posts multipart data. `upload.single("resume")` uses Multer memory storage with a 3 MB limit. Controller converts `req.file.buffer` to a `Uint8Array`, extracts text using `pdf-parse`, and passes text onward. Discuss memory limits, scanned PDFs/OCR, malformed files, and extraction failures.

### 5. How is the AI result represented and persisted?
**Answer cue:** `ai.service.js` defines a Zod schema for `matchScore`, technical/behavioral questions, skill gaps and preparation plan. `interviewReport.model.js` stores report fields and the owning user. The controller persists extracted resume text, self-description, job description and generated fields. Do not call the score a scientifically calibrated ATS score without evaluation evidence.

### 6. How do you prevent one user from reading another user's report?
**Answer cue:** `authUser` reads the `token` cookie, checks the blacklist, verifies JWT, and attaches decoded claims to `req.user`. The report-by-ID query includes both `_id: interviewId` and `user: req.user.id`; the history query filters by user. This ownership filter is as important as authentication.

### 7. Explain authentication, cookies, JWT and logout in this code.
**Answer cue:** Registration hashes the password with `bcrypt.hash(password, 10)`, signs a JWT with `JWT_SECRET`, and sets an HttpOnly, Secure, SameSite=None cookie with a one-day lifetime. Middleware verifies the token. Logout clears the cookie and records the token in a blacklist. Be prepared to explain cookie flags, expiration, secret management, and why storing raw tokens in a blacklist has trade-offs.

### 8. What is your most important security improvement?
**Answer cue:** Start with verified risks, not generic claims. Enforce strict upload validation (MIME/type plus PDF signature, size and parsing limits), ensure all report queries are owner-scoped, validate request bodies, rate-limit expensive AI endpoints, avoid leaking provider errors, protect environment secrets, and consider a hashed-token or session strategy for logout. Check current code before saying any of these are already implemented.

### 9. What are the key failure paths?
**Answer cue:** Missing PDF returns 400; AI returning no report returns 500; controller catches generation errors and sends an error response. Also consider invalid PDF, empty extracted text, Gemini quota/timeouts, malformed AI output, MongoDB failure, expired token, oversized upload and PDF rendering failure. Explain logs and tests for each.

### 10. What would you measure to prove this app works well?
**Answer cue:** End-to-end success rate; PDF extraction success; Gemini latency/error rate/cost; schema-valid response rate; report persistence success; user-visible failure rate; and quality of recommendations judged against a labeled evaluation set. Do not invent values.

## P1 — Strong follow-up questions

### Architecture and APIs
11. What are the main endpoints?
- `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/logout`, `GET /api/auth/get-me`.
- `POST /api/interview/` (authenticated, multipart `resume`), `GET /api/interview/`, `GET /api/interview/report/:interviewId`, and `POST /api/interview/resume/pdf/:interviewReportId`.
- Note: the route comment for PDF says `/resume/pdf`, while the actual route includes `:interviewReportId`; cite the actual route.

12. Why is `withCredentials: true` used in the Axios instance?
**Answer cue:** It allows the browser to include cookies on cross-origin requests. It must pair with a specific allowed CORS origin and `credentials: true`; wildcard origin cannot be used with credentialed CORS.

13. Why use `multipart/form-data` rather than JSON for resume upload?
**Answer cue:** File bytes and text fields can be submitted together. JSON is suitable for structured text but does not natively represent raw file uploads in the same way.

14. Why use Multer memory storage? What is the trade-off?
**Answer cue:** It exposes uploaded bytes as `req.file.buffer`, avoiding temporary disk writes. Memory use scales with concurrent upload size; strict limits and concurrency controls matter. Disk/object storage may be preferable for larger files.

15. Why use MongoDB/Mongoose for reports?
**Answer cue:** Reports have nested arrays of questions, skill gaps and roadmap tasks, which fit document storage. Mongoose schemas add validation and model helpers. Alternatives include PostgreSQL/JSONB; justify based on query patterns and relational needs rather than claiming NoSQL is always faster.

16. How would you handle long-running Gemini calls?
**Answer cue:** Timeouts, bounded retries for transient errors, user-visible pending state, request IDs, rate limits, and possibly background jobs if latency makes synchronous requests unreliable. A job queue is a proposal unless implemented.

17. What is prompt injection in this app?
**Answer cue:** Resume and job-description text are untrusted input that may contain instructions aimed at the model. Delimit user content, instruct the model to treat it as data, constrain output with schemas, avoid exposing secrets/tools, and test adversarial documents.

18. How would you evaluate the match score?
**Answer cue:** Define what “match” means, create human-reviewed resume/JD pairs, compare generated scores and skill gaps to rubric-based labels, evaluate consistency and bias, calibrate scores, and avoid presenting an LLM score as objective truth.

19. What happens when a user opens a report ID belonging to another user?
**Answer cue:** The query includes both report ID and authenticated user ID, so it should return not found rather than disclose the record. Verify every read/download route follows the same ownership rule.

20. Why use a token blacklist? What are its limitations?
**Answer cue:** It allows a token to be invalidated before its natural expiry, but requires a database lookup per authenticated request and storage/cleanup policy. Token expiry and session-based alternatives have different trade-offs.

21. What should be indexed in MongoDB?
**Answer cue:** User email/username uniqueness and report owner plus creation date are likely useful query patterns. Verify existing indexes in schemas/database before claiming them; add compound indexes only after checking query patterns and explain plans.

22. How would you test this app?
**Answer cue:** Unit-test schema parsing and PDF handling; integration-test routes with mocked Gemini; test auth and ownership isolation; test oversized/invalid PDFs and malformed model output; test MongoDB persistence; add a small end-to-end flow. The source archive does not establish comprehensive test coverage.

23. How would you avoid duplicate reports if a user double-clicks Generate?
**Answer cue:** Disable duplicate submission in the UI as a basic measure; stronger correctness uses request IDs/idempotency keys and a unique constraint or generation state. Avoid claiming this currently exists without checking.

24. How would you make PDF export reliable?
**Answer cue:** Validate report ownership, render with a controlled template, handle browser/Puppeteer errors and timeouts, avoid injecting unsafe HTML, and test fonts/long text. PDF generation is implemented in the service/controller path; inspect it before describing exact template behavior.

25. What is the difference between validation in Zod and validation in Mongoose?
**Answer cue:** Zod validates the AI/service payload shape before or during processing; Mongoose validates persistence schema constraints. They protect different boundaries and neither replaces HTTP input validation.

## P2 — Code-level cross-questions

26. Why does `authUser` call `next()` only after `jwt.verify()`?
27. What happens if `JWT_SECRET` is missing or rotated?
28. Why is `bcrypt` used instead of reversible encryption?
29. What does `unique: true` in a Mongoose schema guarantee, and what database index is actually needed?
30. Why filter history with `.sort({ createdAt: -1 })` and omit resume text from list results?
31. What is the risk of logging `err.message` directly to the client?
32. How should an empty or image-only PDF be handled?
33. Why is an AI schema not the same as correctness of AI content?
34. What if Gemini times out after generating a result but before the server receives it?
35. What is the difference between a 400, 401, 403, 404, 413, 429 and 500 in this application?
36. Why should logout normally be a POST/DELETE-style state-changing request rather than a GET?
37. How would you protect the app against AI endpoint abuse and cost spikes?
38. What sensitive personal data is stored, for how long, and how would deletion work?
39. How would you version prompts and compare output quality across versions?
40. Which exact part did you implement, and what evidence can you point to in the code/history?

## Final readiness test

Without notes, draw the request flow and answer questions 2, 4, 6, 7, 8 and 18. Then open the referenced files and verify each statement. If you cannot explain a code path from source, mark it as a gap instead of memorizing a guessed answer.
