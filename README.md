### Hi there 👋

<!--
**jasongriller/jasongriller** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->

  The Bottom Line

  The Chat feature is basically a plain chatbot with light RAG — it finds some docs and talks. The Generator is a quality-engineered RAG pipeline with validation gates,     
  reranking, structured prompts, and post-processing. The Chat should be leveraging the same retrieval intelligence and prompt engineering that makes the Generator output   
  good.

  Want me to plan out how to bring the Chat up to parity? The main upgrades would be:
  1. Multi-query retrieval with LLM reranking (Gate 2)
  2. Richer system prompts with domain context and quality rules
  3. Configurable thresholds from global settings
  4. Over-fetch + filter pattern
  5. Optional structured output mode when generating work items in chat




  Problem 1: Manual doc selection explicitly excludes wiki (Chat.tsx:99)
  const kbDocs = docs.filter(doc => doc.source !== 'wiki');
  Users literally cannot select wiki docs as context documents. This is the same in Home.tsx (line 130) — both UIs filter them out.

  Problem 2: Semantic search CAN find wiki docs, but barely
  The Chat's performSemanticSearch (line 502) queries the full Bedrock KB with no filterConfiguration — so wiki docs can appear in results if they score > 0.3. The backend  
  even correctly labels them as sourceType: 'wiki' (line 523).

  But:
  - Only 10 results are fetched, top 5 returned
  - No reranking — just raw vector similarity
  - Single query using the raw user message (often vague/conversational)
  - The 0.3 threshold is low but with only 10 slots, wiki docs compete with curated docs

  So wiki content can theoretically surface through semantic search, but it's a thin pipe — and it can never be intentionally loaded as reference context.

  The Generator has the same Home.tsx filter (line 130), but compensates with:
  - Over-fetch 2x (min 20 results) → more wiki docs can make it through
  - LLM-based Gate 2 reranking → relevant wiki chunks survive
  - Dynamic multi-query searches → domain keywords trigger targeted wiki hits
  - The Agent Chat generates 4-8 specialized queries which can catch wiki content that a single search would miss

  Bottom line

  The Chat technically can surface wiki content via semantic search, but the odds are low because of the small fetch window and single-shot query. And users can never       
  manually select wiki docs as context. The Generator overcomes this with its multi-query, over-fetch, and reranking pipeline.
