# CONDITIONAL: AI/LLM Integration

Only load if AI/LLM dependencies detected (openai, @anthropic-ai/sdk, ai, langchain, llamaindex, @google/generative-ai, cohere-ai, replicate).

## API Key Exposure

| Pattern | Issue |
|---------|-------|
| `NEXT_PUBLIC_OPENAI` | OpenAI key in client bundle |
| `NEXT_PUBLIC_ANTHROPIC` | Anthropic key in client bundle |
| `openai.*api_key.*=.*['"]sk-` | Hardcoded OpenAI key |
| `new OpenAI\(\{` without `apiKey: process.env` | Possible hardcoded key in constructor |

## Prompt Injection via User Input

AI passes user input directly to LLM without sanitization.

| Pattern | Issue |
|---------|-------|
| `messages.*role.*user.*content.*req\.(body\|query\|params)` | Raw user input in LLM message |
| `prompt.*\$\{.*req\.` | User input interpolated into prompt template |
| `prompt.*\+.*req\.` | User input concatenated into prompt |
| `generateText\(.*\$\{` | Vercel AI SDK with interpolated user input |

If AI features exist: check that user input is never directly interpolated into system prompts. User input should go in the `user` message role, not concatenated into system/instruction text.

## Missing Rate Limiting on AI Endpoints

AI endpoints are expensive. Search for rate limiting on routes that call LLM APIs. If no rate limiting on `/api/ai`, `/api/chat`, `/api/generate`, or similar: HIGH (attacker can run up your API bill).
