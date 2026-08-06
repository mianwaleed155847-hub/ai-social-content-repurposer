# AI Social Media Content Generator

A multi-agent system that takes a single piece of source content (a blog post, article, or notes) and repurposes it into platform-native posts for LinkedIn, Twitter/X, and Instagram — each with its own tone, structure, length, and hashtags, generated in parallel by dedicated AI agents.

Built with CrewAI, powered by Groq, and deployed as an interactive Streamlit app.

---

## Why this project exists

Most "repurpose this content for social media" tools just paraphrase the same post three times with different emojis. This project is built around one core problem: making each platform's output genuinely distinct, not a copy-paste with light editing.

To do that, it goes beyond prompting. Every generated post passes through a validation layer that checks and auto-corrects:

- Twitter/X — each tweet is checked against a strict word-count limit
- LinkedIn — verifies the post ends with an engagement-driving question
- Instagram — hashtags are compared against LinkedIn's, and any duplicates are automatically swapped out from a fallback pool
- All platforms — strips AI-generated preambles ("Here's the rewritten post:") and stray quotation-mark wrapping before the text ever reaches the user

## How it works

Source Content
      |
      v
+-------------+--------------+----------------+
|  LinkedIn   |  Twitter/X   |   Instagram    |
|   Writer    |Thread Writer |Caption Writer  |
|  (agent)    |  (agent)     |   (agent)      |
+------+------+-------+------+--------+-------+
       |              |               |
       v              v               v
 Professional    Punchy thread   Casual, emoji
  150-250 word   (<=35 words/    -rich caption
    post          tweet)          (8-15 tags)
       |              |               |
       +------+-------+-------+-------+
              v               v
       Post-processing / validation layer
   (preamble & quote stripping, word-count check,
    question-ending check, hashtag de-duplication)
              |
              v
       Clean, copy-paste-ready output
        (Streamlit UI, 3 text panels)

Three specialized CrewAI agents run against the same source content independently (context=[] on every task, so no agent's output leaks into another's), each governed by a task description that encodes platform-specific rules:

| Platform | Agent focus | Key constraints |
|---|---|---|
| LinkedIn | Thought-leadership tone | 150-250 words, must end in a question, 3-5 professional hashtags |
| Twitter/X | Punchy, conversational thread | <=35 words per tweet, hashtags spread across the thread (not stacked on one tweet) |
| Instagram | Casual, emoji-driven caption | Short paragraphs, 8-15 hashtags, explicitly required to differ in tone and hashtags from LinkedIn/Twitter |

## Tech stack

- CrewAI — multi-agent orchestration
- Groq (openai/gpt-oss-120b) — LLM inference
- Streamlit — web UI
- pyngrok — public tunnel for running the app out of Google Colab
- Python re — post-generation validation and hashtag de-duplication (no extra ML dependency for this step — deterministic, guaranteed rules instead of hoping the LLM follows every instruction)

## Running it

This project is built to run in Google Colab:

1. Open content_generator_updated.ipynb in Colab
2. Add your Groq API key as a Colab secret named GROQ_API_KEY
3. Add an ngrok auth token as a Colab secret named NGROK_AUTHTOKEN (get one free at https://dashboard.ngrok.com/get-started/your-authtoken)
4. Run all cells top to bottom
5. The last cell prints a public ngrok URL — open it to use the app

### Running locally instead of Colab

pip install -r requirements.txt
export GROQ_API_KEY="your-key-here"
streamlit run app.py

## What the validation layer catches

LLMs are inconsistent at following soft instructions like "keep each tweet under 35 words" or "use hashtags different from the other platforms." Rather than relying purely on prompt engineering, this project treats those as hard constraints enforced in code after generation:

- Preamble stripping — removes meta-text like "Here's the rewritten thread:" that models sometimes prepend
- Quote-wrap removal — strips accidental wrapping of the whole post in quotation marks
- Twitter word-count report — flags any tweet over the 35-word limit
- LinkedIn question-ending check — flags posts that don't close with a question
- Instagram hashtag auto-fix — detects any hashtag duplicated from the LinkedIn post and swaps it for a fresh one from a curated fallback pool, guaranteeing the two platforms never ship identical tags

These results are surfaced in a "Validation details" panel in the app so it's transparent what was checked and what (if anything) was corrected.

## Project structure

.
├── content_generator_updated.ipynb   # Full build notebook (setup -> agents -> app.py generation -> deployment)
├── app.py                            # Generated Streamlit app (written out by the notebook)
├── requirements.txt
└── README.md

## Notes / limitations

- Uses Groq's openai/gpt-oss-120b — a fast, cost-efficient model. Larger/more expensive models would likely need the validation layer less, but the deterministic checks are kept regardless since they're cheap and remove any doubt.
- ngrok's free tier gives a new random URL each session unless you're on a paid plan with a reserved domain.
- This is a portfolio/learning project, not a production content pipeline — there's no rate-limiting, auth, or persistence layer.

## Author

Built by Waleed Ahmad as part of an ongoing LLM & Generative AI learning roadmap, alongside a separate multi-agent Data Analyst System (CrewAI + Streamlit).
