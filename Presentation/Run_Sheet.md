# Spinlog: Presentation Run Sheet

**Team:** Marcus Rowe, Owen Mueller  **Advisor:** Seokki Lee

## 1. Slide run order

Start times are cumulative from 0:00. Seconds match the deck's "Outline and Timing" backup slide.

| **Slide** | **Title**                         | **Speaker** | **Planned start** | **Sec** | **Hand-off line**                                |
|-----------|-----------------------------------|-------------|-------------------|---------|--------------------------------------------------|
| 1         | Spinlog: Vinyl Collection Tracker | Marcus      | 0:00              | 10      | "Owen will introduce the team."                  |
| 2         | Team and Advisor                  | Owen        | 0:10              | 10      | "Now let’s get into what it is."                 |
| 3         | Project Abstract                  | Owen        | 0:20              | 65      | "Now, who is this for?"                          |
| 4         | User Stories                      | Owen        | 1:25              | 25      | "Marcus will show how the system fits together." |
| 5         | System Design                     | Marcus      | 1:50              | 65      | "That design has to respect four constraints."   |
| 6         | Major Constraints                 | Marcus      | 2:55              | 20      | "So how far along are we?"                       |
| 7         | Progress Review                   | Marcus      | 3:15              | 20      | "Owen will cover what we finish this term."      |
| 8         | Expected by End of Term           | Owen        | 3:35              | 25      | "After that comes the build, and who does what." |
| 9         | Division of Work                  | Owen        | 4:00              | 25      | "Marcus will show what we plan to demo at Expo." |
| 10        | Expected Expo Demo                | Marcus      | 4:25              | 35      | Closing: "Any questions?"                        |
|           | **End**                           |             | **5:00**          | **300** |                                                  |

Speaking time: Marcus 150 s (slides 1, 5, 6, 7, 10). Owen 150 s (slides 2, 3, 4, 8, 9).

## 2. Speaking roles

| **Role**                    | **Person**              | **What it covers**                                                         |
|-----------------------------|-------------------------|----------------------------------------------------------------------------|
| Opening and closing speaker | Marcus                  | Slide 1 and slide 10, including the final thank-you                        |
| Design and status speaker   | Marcus                  | Slides 5, 6, 7                                                             |
| Team and product speaker    | Owen                    | Slides 2, 3, 4                                                             |
| Plan speaker                | Owen                    | Slides 8 and 9                                                             |
| Slide advancer              | Whoever is speaking     | Advances their own slides                                                  |
| Recorder                    | Marcus                  | Starts and stops the recording, checks audio first                         |
| Q&A                         | By topic, see section 3 | Answers questions in the owner's area; the other member adds only if asked |

## 3. Anticipated questions

| **#** | **Question**                                           | **Owner** | **Short answer**                                                                                                                                                                         |
|--------|--------------------------------------------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1      | Why NFC tags instead of detecting the song from audio? | Marcus    | NFC gives us a hardware component to build now and add to later. Audio detection is the planned third stage and is out of scope for this release.                                        |
| 2      | Why one backend instead of microservices?              | Marcus    | We are new to hosting, so one deployable FastAPI backend is simpler. We will revisit it if it becomes a problem.                                                                         |
| 3      | How secure is user data?                               | Marcus    | Data is encrypted in transit with TLS 1.2 or higher in production. Only hashes of refresh and pairing tokens are stored, and reuse of a refresh token revokes all of that user's tokens. |
| 4      | What is the biggest technical risk?                    | Marcus    | The NFC module misreading tags                                                                                                                                                           |
| 5      | What is finished and what is not?                      | Marcus    | The design, requirements, test plan, and ethics statement are done, no application code is written yet                                                                                   |
| 6      | How is collection value calculated?                    | Owen      | Each day's total uses every release's newest price snapshot on or before that day                                                                                                        |
| 7      | Will the reader and tags really cost under $20?       | Owen      | That is our target. We have not priced the final parts yet and will check it with the prototype.                                                                                         |
| 8      | Why a 1,000 record limit?                              | Owen      | It is the minimum we set so the app suits real collections.                                                                                                                              |
| 9      | Is the work split evenly?                              | Owen      | The plan is 194 hours: Marcus 99 (51%) and Owen 95 (49%). Tasks are paired so either of us can take over.                                                                                |

## 4. Rehearsal log

| **Slide** | **Title**                         | **Speaker** | **Actual time (s)** |
|-----------|-----------------------------------|-------------|---------------------|
| 1         | Spinlog: Vinyl Collection Tracker | Marcus      | 20                  |
| 2         | Team and Advisor                  | Owen        | 23                  |
| 3         | Project Abstract                  | Owen        | 50                  |
| 4         | User Stories                      | Owen        | 40                  |
| 5         | System Design                     | Marcus      | 58                  |
| 6         | Major Constraints                 | Marcus      | 33                  |
| 7         | Progress Review                   | Marcus      | 22                  |
| 8         | Expected by End of Term           | Owen        | 17                  |
| 9         | Division of Work                  | Owen        | 18                  |
| 10        | Expected Expo Demo                | Marcus      | 35                  |
|           | **Total**                         |             | 316                 |
