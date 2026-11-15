# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 changed my results the most. Bare was 16 / 4 / 4 (correct / wrong / no answer, out of 24), and with layer 1 it was 18 / 6 / 0. It’s the only layer that changed the number of correct answers. The bare loop stopped at the first tool error, so it gave no answer on all four alias questions. With layer 1 the model saw “Unknown region” and could try again. q23 and q24 became correct, but q21 and q22 became wrong, so wrong went from 4 to 6. So layer 1 let the agent answer more, but two of the new answers were wrong where before it just didn’t answer. Layer 2 changed nothing (still 18 / 6 / 0, same 62 model calls and 48708 tokens), because no question needed more than 5 steps. Layer 3 kept 18 correct, wrong went from 6 to 4 and no answer from 0 to 2. It didn’t help the agent get more right, it just made it say “I don’t know” in two cases where it used to be wrong.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

I picked q18: by how many percentage points did CPI in Jambyl fall between 2022 and 2025. With layer 3 the answer is UNVERIFIED, so it counts as no answer. In the trace the first step is fine: get_series returns 16.1 for 2022 and 10.6 for 2025. The problem is the second step. The model calls calculate with (16.1 - 10.6)/100 and gets 0.055, but percentage points are just the difference, 5.5. Then it writes ANSWER: 0.55, which isn’t even what calculate returned. Layer 3 sees that 0.55 isn’t in any tool output and sends it back. The model repeats the same calculate call, the repeated call ends tool use, it writes 0.55 again and the harness returns UNVERIFIED. My fix would be a check for pp questions: the calculate expression has to be a plain difference of two values from get_series, and if it isn’t, the harness tells the model. That’s one more model call and some tokens, and if the rule is too strict it could break other questions.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one. 

I’d ship the fixed workflow. On the 15 fixed-shape questions it got 13/15 correct with 0 wrong. The bare agent got 9/15 correct with 2 wrong. The workflow used 15 model calls, 3852 tokens and 16.8 seconds, and the agent used 26 model calls, 19835 tokens and 46.8 seconds. That’s about 5 times fewer tokens and about 3 times faster. The monthly ranking always has the same shape (one indicator, one year, highest and lowest), and that’s exactly what the workflow does. It didn’t miss a single rank question. The agent got q08 and q11 wrong, and both were “lowest” questions. Also, when the workflow fails it gives no answer instead of a wrong one, which matters for something we publish. I’d use the agent when the question shape isn’t known in advance, like a change between two years in percentage points. The workflow can’t even try that.

---

I wrote this file myself, without any AI tool.
