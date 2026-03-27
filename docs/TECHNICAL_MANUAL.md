# 💻 Technical Architecture Manual

This document is intended for developers or advanced users who wish to maintain or scale the application.

---

## 🏗️ 1. File Structure & Responsibilities

- **`nia-language-adventure-index.html`**: The beating heart of the app. It handles the dashboard, progress bars, video player modal, and search logic.
- **`quiz-system.js`**: The functional engine. It is a class-based system that handles all quiz logic (rendering, scoring, validation, storage).
- **`quiz-data.js`**: The content database. It contains the large objects for all 8 quizzes (`S1_QUIZ`, `M1_QUIZ`, etc.).
- **`s1-guide.html` (to `m4-guide.html`)**: The individual content pages. They host the interactive lessons and provide the container for the `QuizSystem` class to render.

---

## ⚙️ 2. The Quiz Engine (`QuizSystem`)

The application uses a class-based architecture to manage quizzes.

### Key Methods
- `constructor(quizId, questionsData)`: Initializes the quiz and loads any saved progress from `localStorage`.
- `validateAnswer(question, userAnswer)`: Uses multiple logic paths (selection, direct string match, fuzzy match) to determine correctness.
- `calculateSimilarity(str1, str2)`: A custom implementation of the Dice's Coefficient to allow for 80% similarity as a "Correct" answer, helping Nia if she makes a minor spelling mistake.
- `saveToStorage()` / `loadFromStorage()`: Ensures Nia can close her computer and resume the exact same question later.

---

## 📋 3. Data Schema: Question Object

Every question in `quiz-data.js` must follow this structure:

```javascript
{
  id: "UNIQ-ID",
  type: "mcq" | "matching" | "fill-blank" | "arrange" | "true-false",
  question: "The question text",
  options: ["Option A", "Option B"], // For MCQ/TF
  correctAnswer: 0, // Index for MCQ, or string for fill-blank, or object for matching
  explanation: "Pedagogical feedback displayed after answering",
  pairs: [{left: "A", right: "B"}] // ONLY for 'matching' type
  items: ["A", "B", "C"] // ONLY for 'arrange' type
}
```

---

## 💾 4. Persistence Layer (LocalStorage)

The application relies heavily on `localStorage` for state management.

| Key | Value Type | Purpose |
| :--- | :--- | :--- |
| `nia_done` | Array (Strings) | Stores IDs of completed videos/quizzes. |
| `nia_yt` | Object (Maps) | Stores custom YouTube links provided by the user. |
| `quiz_results` | Object (Maps) | Stores pass/fail status and percentages for all 8 quizzes. |
| `quiz_{ID}` | Object | Stores the temporary state (current question, answers) of an active quiz. |

---

## 🔄 5. Communication Bridge

When a quiz is completed in a guide file (`_blank` tab), it needs to tell the main dashboard to update its progress. 
1. The Guide file writes the completion ID to `nia_done` in `localStorage`.
2. The Dashboard (`index.html`) listens for the `pageshow` or `focus` event and re-calculates statistics using `updateStats()`.
3. This ensures that when Nia goes back to the dashboard, she immediately sees her new "Done" badge.

---

## 🚀 6. Future Scaling

To add a **Phase 5 (Advanced Literature)**:
1. Update the `VIDEOS` array in `index.html` with new objects marked `phase: 5`.
2. Create `s5-guide.html` and `m5-guide.html` using the existing templates.
3. Add `S5_QUIZ` and `M5_QUIZ` to `quiz-data.js`.
4. The dashboard statistics will automatically update to include the new items in the global total.
