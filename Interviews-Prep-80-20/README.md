# Interview Prep — 80/20 Project Question Bank

This is a focused, high-yield interview prep pack based on the supplied resume and the accessible project source archives/notebook. It deliberately omits low-value documentation boilerplate and focuses on questions most likely to trigger technical follow-ups.

## Recommended order

1. **ATS Resume Analyzer & Interview Assistant** — full-stack, AI integration, auth, PDF parsing, MongoDB.
2. **Kafka Order Tracker** — Kafka fundamentals, event-driven design, partitions, consumers, correctness.
3. **InsightStudy** — strongest resume ownership signal: founder/frontend developer, 100+ users; full-stack flows, AI/OCR, queue.
4. **EcoMotion** — first-prize hackathon project, ML data pipeline and evidence-based discussion.
5. **NeuroBreath** — frontend fundamentals, state/timing, UX, accessibility.

## How to use this pack

- First answer every **P0** question aloud without notes.
- For each answer, be ready for the listed follow-up.
- Use the code references to verify the implementation rather than memorizing generic claims.
- Only claim personal ownership for the work you actually did. InsightStudy and EcoMotion are group projects according to the resume.
- Separate **implemented**, **inferred**, and **proposed** behavior. Do not invent accuracy, production scale, tests, or architecture.
- The short answer cues are a preparation scaffold, not a script to recite verbatim.

## Evidence boundaries

- Resume: `RitikKumarGupta_.pdf`.
- ATS, Kafka, InsightStudy, NeuroBreath: supplied ZIP archives.
- EcoMotion: public notebook/repository [Tahmid019/Hyperspectral_dataAnalysis](https://github.com/Tahmid019/Hyperspectral_dataAnalysis) and [live demo](https://ecomotionhack.vercel.app/).
- The GitHub target repository was found to be empty at audit time, and the connected GitHub integration rejected writes. These files are therefore prepared locally for upload; they have **not** been committed to the remote repository.

## 30-minute revision loop

1. 5 min — project pitch and architecture diagram from memory.
2. 10 min — P0 questions and follow-ups.
3. 10 min — trace one critical code path using the source.
4. 5 min — state one limitation honestly and propose one justified improvement.

## Resume-level questions (prepare these for every interview)

1. Walk me through your resume in 60 seconds.
2. Which project best represents your strongest engineering ability, and why?
3. Which feature did you personally implement end to end?
4. What is the hardest bug you personally diagnosed? What evidence led to the root cause?
5. What did you learn from a deployed application that a classroom project would not teach you?
6. Your resume says InsightStudy has 100+ registered users. What exactly counts as a registered user, and how did you verify that number?
7. You list React, Node, MongoDB, PostgreSQL, FastAPI, and Go. Which have you used hands-on, and in which specific project?
8. You mention 550+ LeetCode problems. Pick one hard problem and explain the algorithm, complexity, and an alternative.
9. For group projects, how do you distinguish your contribution from the team's contribution?
10. Which project would you redesign first, and what is the highest-impact change?

## Cross-project interview traps

- **Metrics:** State the dataset, split, metric, model, and evaluation setup whenever you give a number.
- **AI output:** Explain that model output is probabilistic and needs validation; do not call an LLM score an objective ATS score unless you have a calibrated evaluation.
- **Security:** Explain actual auth and data-access checks; do not say “secure because JWT is used.”
- **Deployment:** A live frontend does not prove every notebook/model or backend subsystem is connected to it.
- **Production claims:** Use “deployed demo” or “project deployment” unless you have evidence for production-grade reliability.
