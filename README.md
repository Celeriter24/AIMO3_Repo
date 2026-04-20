# AIMO3_Repo
# This is my base repo for my AIMO3 Submission

Files:
original.py: The original .py script I used to write and debug the code on my local machine.
base_for_linux.py: A claude code refactored version of original.py
base_for_linux.ipynb: A claude code refactored ipynb version of original.py

## Features: 

Solver: 
Backend: Qwen3.5-27b fp8 with speculative_tokens = 3 for fast and ressource-efficient LLM Inference for Mathematical Reasoning
Prompts: Input Prompts (System prompts, User prompts, Tools Preference Prompts)
Tools: Local Python Sandbox Environment as Jupyter Kernel, Lean4 Proofing with Lokal Lean compiler according to a simplified Goedel-v2-profer implementation (spawns multiple attempts and profer)

### Solver Features:
1. Weighted Entropy Calculation:
The entropy was calculated not only based on the prop of the top20 Logprob-Token, but also weighted on the semantic differentiation of the chosen token and the top Logprob-Tokens (calculation took quite long so it was reduced to top2 instead of top20)

2. Wait-Interception:
If `wait` or `Wait` is one of the top logprob-Tokens, the generation stops and a prompt `Wait, I'm not sure. I should think about this again, maybe some tests/experiments or examples with Python, will help me.` is injected.

3. Final Guessing:
If the solver timeout is reached, the solver has the chance to generate a final answer based on the previous messages.

4. Timebudget:
Each problem gets a specific time budget. Simplyfied it is 'max(min(remaining_time/remaining_problems,900),300)'. So usually 900, but reduced if the average time per upcoming problem is lower than 900 but at least 300.

5. Answer selection/comparing answers from the 6 Solver attempts:
   - Majority vote (3 or more same answers) or
   - Filtered all attempts by not None Answers: Sum over all individual attempts with same answer: Weighting the mean weighted entropy by guessing (yes/no) + Weighing python calc time--> Score per answer, pick the highed scored answer


## Highfly Workflow:

Solver is given a AIMO3 Problem 
Spawning 6 concurrent attempts to solve the problem
Solver LLM is corresponding to the Input prompts in a loop with tool access.
The attempt ends when Solver LLM prints a `int` in \\boxed{} or tme budget is consumed or majority vote (3 of 6 have the same answer) stops all running attempt

## Data used:
Local Python Env wheel data: `/kaggle/input/notebooks/shelterw/aimo-3-vllm-v16`
`
Solver LLM Backend data: `/kaggle/input/models/shelterw/qwen3.5/transformers/qwen3.5-27b-fp8/1`

For fast LeanRepl: `/kaggle/input/datasets/frederikbeats/leanrepl` 

For LeanCompiler:  `/kaggle/input/datasets/frederikbeats/lean-mathlib-toolchain`

For translation NL-->Lean Statement: `/kaggle/input/models/frederikbeats/goedel-formalizer-v2-8b/transformers/default/1`

For LeanProfGeneration:`/kaggle/input/models/frederikbeats/goedel-prover-v2-8b/transformers/default/1`

### Hardware/Setup/Kaggle Constrains:
No internet connection (env is loaded from local source, lean compiler is installed from local source)
H100 GPU with 80 GB VRAM
Around 200 GB RAM 
Some Intel Xeon: 26 CPU Kernel 
5h of GPU compute time
One submission per day (has to be saved/run on a testset before submission)

# Learnings from the completion

1. None of the about 60.000 submissions of around 4000 Teams made it, to solve more than 45 problems within the ressources. It seems that it is not possible to solve these problem dataset in 5 hours on a H100 with OSS models. (If closed source LLMs are able to solve these problems is also not sure/validated.)
2. The reference difficulty discussion (https://www.kaggle.com/competitions/ai-mathematical-olympiad-progress-prize-3/discussion/679559) from the organisators ![Comparison](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F23065446%2F384e1fee641e643720c2a13672a36cfe%2F1000054923.png?generation=1772442630228730&alt=media) shows that it is theoretically possible to solve the problemset but with a lot of uncertainties:
   - We don't know the models A or B nor their quantisation, most likely A is a ~30B model and B is a ~120B model
   - We don't know the hardware setup of their tests
   - We don't know anything about the time budget for these calculations per problem. 
   - The metric pass@n doesn't include the need to select exactly one answer per problem. So these results are not comparable to "Run n attempts and you have the correct final answers out of these n attempts" as the competition requires it
   - The smaller model A is not sufficient to solve more than 40 problems.
   - The bigger model B is able to find a correct solution for 40+ problems at 5, 20 or 100 attempts, but the selection of the correct answer is not included in pass@n tests.
3. The usage of 2 additional LLMs for the Lean Proofing as vllm Server per H100GPU seems to hindering the solving process more than helping it, since the proofing uses a lot of computation and VRAM! 
   - The Goedel-v2-profer generates on the first proof attempts around 20.000 Tokens which takes already 200sec (100 tokens/sec).
   - Because of the 2 additional LLMs I needed to reserve 38% of the 80GB VRAM of the H100, which could be used for a bigger model or increasing the context of the Solver LLM  
4. Some teams seems to have send each day a new (probably automated) submission with probably only different seeds/prompts--> Randomly and prompting doesn't solve the dataset
5. Kaggle's infrastructure for this kind of competition was not suitable for extensive testing





