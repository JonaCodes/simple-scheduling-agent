# A Humble Agent, in Parts

In this project you will build a simple scheduling agent. But we will build it in parts

## Part 1: The Memory Loop
For starters, let's see we can talk back and forth with an LLM.
We'll be using the command line for our interface - you already have a function ready for you, exposed from `cli-io.js`:

```js
import { getUserInput } from './cli-io.js';

const userInput = await getUserInput('Say something: ');
console.log("You said:", userInput);
```

Your task is to write a simple infinite loop in `agent.js` that gets the user input, sends it to the LLM, prints the LLM's response, and repeats.

Feel free to use the `askGeminiWithMessages` function from `llmService.js` to send the user input to the LLM.

Example interaction:
```
You: Is the sky blue?
Gemini: I can't see if the sky is blue, so I don't know.
You: Usually? 
Gemini: Yes, usually the sky is blue during the day if there aren't clouds blocking the sun.
```

Notice the LLM retained the context, so when you say "usually?" it knows what you're referring to.

## Part 2: The Tools
For simplicity, we will **not** be using `zod` or a proper schema for our tools.
Instead we will create simple functions and tell the LLM about them in our system prompt.

Here are your tools:
```js
const checkAvailability = (date) => {
    console.log("Checking availability for date:", date);
    return { available: true }
}

const scheduleAppointment = (date) => {
    console.log("Scheduling appointment for date:", date);
    return { success: true }
}

const deleteAppointment = (date) => {
    console.log("Deleting appointment for date:", date);
    return { success: true }
}
```

Now modify your code so that when you send "check availability for tomorrow at 10AM" to the LLM, it tells you to run the `checkAvailability` function, and then *you run it*

Example interaction:
```
You: Hi
Gemini: Hello
You: check availability for tomorrow at 10AM
Checking availability for date: 1991-04-03 10:00:00 // this is a tool call!
```

### Notes
- Your simple agent should either answer normally, or trigger a tool call
- Notice that in the example interaction above, the last log is from the function call itself, **not** the LLM
- For now, simply worry about the answerUser vs. runTool flow - **no need to handle to whole flow**

## Part 3: Finalizing the Agent
Finally, modify your agent loop so that it automatically runs as many times as it needs to complete the user's request.

### Considerations:
- When do you let the user input text?
- How do you exit the loop?
- What and when do you store in the agent's memory?

Example interaction:
```
You: schedule an appointment for tomorrow at 6am
Checking availability for date: April 3rd, 1991
Scheduling appointment for date: April 3rd, 1991
Gemini: Your appointment for April 3rd, 1991 has been scheduled.

You: actually, move it to 9:45 in the evening
Deleting appointment for date: April 3rd, 1991
Checking availability for date: April 3rd, 1991
Scheduling appointment for date: April 3rd, 1991
Gemini: Your appointment has been moved to April 3rd, 1991 at 9:45 PM. I'm finished.
```

If you want to be fancy, you can use [chalk](https://www.npmjs.com/package/chalk) to add colors
<img width="616" height="218" alt="image" src="https://github.com/user-attachments/assets/84a3919f-6440-4a95-87d3-f3e72e0ca580" />

## Gemini API

Use `gemini-3.5-flash-lite` for generation. Leave thinking settings unset to use
the model's default minimal thinking; this does not guarantee zero thinking tokens.

If you run out of Flash-Lite usage, you can also try `gemini-3.8-flash` with the
same Gemini API key, but it can be slower. Availability and quotas depend on your
account; switching models may not bypass shared limits. Check your current
[Gemini API limits](https://ai.google.dev/gemini-api/docs/rate-limits).

### Other API providers

You can also try these providers. Use the provider's own API key and adapt the
request wrapper to its endpoint or SDK.

| Provider | Free offering |
| --- | --- |
| [**Mistral**](https://docs.mistral.ai/getting-started/quickstarts/studio/activate-and-generate-api-key) | Free API mode, **no credit card required**. Usage quotas are shown in the account dashboard. |
| [**Groq**](https://console.groq.com/docs/rate-limits) | For `qwen/qwen3.8-27b`: **30 requests/minute, 1,000/day, 8,000 tokens/minute**, subject to account limits. |
| [**OpenRouter**](https://openrouter.ai/pricing) | Free models, **50 requests/day** without purchasing credits. |

Free-tier limits can change; check the linked provider pages and your account dashboard.

The supplied `askGeminiWithMessages` helper treats the first `role: 'model'`
memory entry as the system instruction. Later model turns remain conversation history.
