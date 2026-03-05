# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
- List at least two concrete bugs you noticed at the start  
  
When I first ran the game, the hints weren't behaving correctly — the hint would show up after the first guess, but once I unchecked and rechecked the "Show hint" checkbox, it would disappear and never come back without submitting a new guess. The New Game button also failed to actually reset the game; clicking it would show a "New game started" message but the game state (status, score, history) carried over from the previous round, leaving the player stuck. Two concrete bugs I noticed right away were: (1) the show hint checkbox not toggling the hint on and off reliably, and (2) the New Game button not resetting the status field, which meant the game-over guard immediately blocked play again.
---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).

I used Claude (via claude.ai) as my AI pair programmer throughout this project. I would identify the specific function or behavior that was broken, describe what was wrong about it in plain language, and ask Claude to fix it — for example, my first prompt pointed directly at check_guess and noted that the higher/lower logic looked reversed, which Claude correctly identified and fixed by swapping the hint messages so "Too High" maps to "Go LOWER" and "Too Low" maps to "Go HIGHER." One example of an AI suggestion that was correct: Claude's explanation of why the hint checkbox wasn't working — it correctly diagnosed that st.warning(message) was only called inside the if submit: block, meaning toggling the checkbox triggered a rerun with no hint to display; storing the hint in st.session_state.last_hint and rendering it outside the submit block fixed this, which I verified by checking and unchecking the box after guessing. One area where I had to stay sharp: the attempts counter fix required careful reasoning about where to initialize and increment attempts — Claude's first pass was logically sound, but I needed to trace through the math myself to confirm the display formula and the >= vs > limit check were both correct.
---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
- Describe at least one test you ran (manual or using pytest)  
  and what it showed you about your code.
- Did AI help you design or understand any tests? How?

I decided a bug was really fixed by manually playing through the game after each change — submitting guesses, watching the attempts counter decrement, toggling the hint checkbox, and clicking New Game to verify a fresh round started cleanly with the correct state. For the attempts counter specifically, I ran through a full Easy game (6 attempts) and watched the counter go from 6 down to 0, confirming it decremented by 1 each guess and hit "Out of attempts" exactly when it should. I didn't write pytest tests, but manually tracing through edge cases — like submitting 0, -5, or 101 after the parse_guess fix — let me confirm invalid inputs were rejected with the right error message. Claude helped me understand what to test by explaining which variables were involved in each bug, which pointed me toward exactly what to watch during manual testing.
---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
- What change did you make that finally gave the game a stable secret number?

In the original app, the secret number kept changing because st.session_state.secret = random.randint(low, high) was called at the top level of the script without checking if a secret already existed — since Streamlit reruns the entire script from top to bottom on every interaction, a new secret was generated on every button click or widget change. A Streamlit "rerun" is basically the whole Python file re-executing from scratch every time the user does anything; session_state is a dictionary that survives those reruns, so anything you want to remember (like the secret number or the score) has to be stored there. The fix was wrapping the initialization in if key not in st.session_state, so the secret is only generated once at the start of a new game and preserved across all subsequent reruns.
---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.

One habit I want to reuse is isolating the broken function first before touching anything else — pointing AI at a specific, small piece of code (like check_guess) produced a clean, verifiable fix, whereas vague prompts about "the game not working" would likely have produced noisier suggestions. Next time I work with AI on a coding task, I'd manually trace through the logic of any fix before accepting it, rather than just running it and seeing if it "seems" to work — the attempts counter bug in particular looked correct at a glance but required careful math to fully validate. This project changed how I think about AI-generated code: it's a strong first draft and a fast explainer, but it can carry subtle logic errors that only show up when you think through edge cases yourself, so treating it as a collaborator to verify rather than an authority to trust blindly is the right mindset.
