# Contributing to X-Algo Preview Resource

Thank you for your interest in contributing! This project is open source and contributions of all kinds are welcome.

---

## Ground Rule: Accuracy First

The most important rule in this project is **never introduce claims that aren't in the [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm) repo**.

Before submitting any change that adds or modifies factual claims about the algorithm:
1. Find the exact line or section in the `xai-org/x-algorithm` README that supports it
2. Include that reference in your PR description
3. If it's a simulator estimate (not a repo fact), label it clearly with `*` and a disclaimer

---

## Types of Contributions

### 🐛 Accuracy Fixes (highest priority)
Found a claim that isn't in the current repo? Please open an issue or PR.

- Label the issue: `accuracy`
- Include: the claim text, where it appears in the UI, and what the repo actually says
- If the claim is from the old `twitter/the-algorithm` (2023) repo, note that

### 📚 New Learn Lessons
The Learn section currently covers 9 lessons. New lessons are welcome if they cover documented aspects of `xai-org/x-algorithm` not yet included.

Lesson structure (in `index.html`, `lessons` array):
```javascript
{
  id: 'L10',
  title: 'Your Lesson Title',
  icon: '🔧',
  xp: 25,   // 15–35 XP depending on complexity
  text: `Lesson content here. Must be verifiable against xai-org/x-algorithm.`,
  quiz: {
    q: 'Quiz question?',
    options: ['Option A', 'Option B', 'Option C', 'Option D'],
    correct: 1,   // 0-indexed
    feedback: 'Explanation of the correct answer, citing the repo.'
  }
}
```

### 🧪 New Strategy Lab Scenarios
New what-if scenarios are welcome. Each scenario needs:
- A `data-scenario` key in the HTML card
- A matching entry in the `scenarios` JS object
- `before`/`after` scores (clearly labeled as simulator estimates)
- 3–5 `insights` grounded in repo facts

### 🎨 UI / UX Improvements
- Mobile responsiveness improvements
- Accessibility (ARIA labels, keyboard navigation, contrast)
- Performance improvements
- Additional visual clarity for the simulator disclaimers

### 📖 Documentation
- Improving this file or `ACCURACY.md`
- Adding examples or clarifications to `docs/ARCHITECTURE.md`

---

## How to Contribute

1. **Fork** the repo
2. **Clone** your fork: `git clone https://github.com/YOUR_USERNAME/X-ALGO-PREVIEW-RESOURCE.git`
3. **Make your changes** in `index.html` (or docs files)
4. **Test** by opening `index.html` in a browser — no build step needed
5. **Submit a Pull Request** with a clear description of what changed and why

---

## Project Structure

Everything lives in a single HTML file by design — no framework, no build step, no dependencies:

```
index.html
├── <style>          CSS variables, layout, components
├── <nav>            Navigation bar
├── #page-calc       Score Calculator page
├── #page-decoder    Algo Decoder page
├── #page-lab        Strategy Lab page
├── #page-learn      Learn page
└── <script>         All JS — navigation, calculator, scenarios, lessons, XP system
```

When editing `index.html`:
- The `lessons` array is around line 1713
- The `scenarios` object is around line 1590
- The score computation is the `computeScore()` function
- The pipeline stages are in `#page-decoder`

---

## Commit Message Format

```
fix: remove fabricated 30x weight claim from calculator
feat: add L10 lesson on Side Effects in CandidatePipeline
docs: update ACCURACY.md with new verified claims
ui: improve mobile layout for score panel
```

---

## Questions?

Open an issue with the label `question`. Response time may vary — this is a volunteer project.
