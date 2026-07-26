# ⏰ HOUR 1 — The AI Landscape: Big Picture for Backend Developers

> **Your current state:** You write REST APIs, manage databases, handle authentication, deploy microservices.  
> **Your target state:** You build AI-powered applications that understand, generate, and act.  
> **This hour:** We map the terrain so you know WHERE you are and WHERE you're going.

---

## 🧭 Hour Structure

```
25 min → Concepts (this section)
25 min → Hands-on: Build your AI landscape explorer
10 min → Interview Drill
```

---

## 1.1 Why Should a Backend Developer Care About AI?

Think about your last backend project. You probably built:
- A REST API that fetches user data from PostgreSQL
- A microservice that processes payments via Stripe
- A notification service that sends emails/SMS

**Every single one of these is about to change.**

In 2024-2025, backend engineers are being asked to build:
- APIs that don't just fetch data — they **understand** user queries in natural language
- Services that don't just process — they **generate** reports, emails, code
- Systems that don't just route — they **decide** which tool to call, when to search, when to ask

**Analogy:** Remember when "backend dev" meant just CRUD apps, then suddenly you had to know Docker, K8s, message queues, and caching? AI is that same shift — but bigger. The backend is becoming the **brain**, not just the **waiter**.

> **Backend Dev → AI Dev = Waiter who takes orders → Chef who understands taste, creates dishes, and improves the menu.**

---

## 1.2 The Five Layers of AI (Your New Mental Model)

Your notes show a nested diagram. Here's what it actually means:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ARTIFICIAL INTELLIGENCE (AI)                                  │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │ "Simulates Human Intelligence"                          │    │
│   │                                                         │    │
│   │   MACHINE LEARNING (ML)                                 │    │
│   │   ┌─────────────────────────────────────────────────┐    │    │
│   │   │ "Learns from Data"                                │    │    │
│   │   │                                                   │    │    │
│   │   │   DEEP LEARNING (DL)                              │    │    │
│   │   │   ┌─────────────────────────────────────────────┐  │    │    │
│   │   │   │ "Uses Neural Networks (Many Layers)"        │  │    │    │
│   │   │   │                                             │  │    │    │
│   │   │   │   GENERATIVE AI (GenAI)                     │  │    │    │
│   │   │   │   ┌─────────────────────────────────────┐  │  │    │    │
│   │   │   │   │ "Creates New Content"               │  │  │    │    │
│   │   │   │   │                                     │  │  │    │    │
│   │   │   │   │   AGENTIC AI                        │  │  │    │    │
│   │   │   │   │   "Plans, Acts & Achieves Goals"    │  │  │    │    │
│   │   │   │   │   "Autonomously"                    │  │  │    │    │
│   │   │   │   └─────────────────────────────────────┘  │  │    │    │
│   │   │   └─────────────────────────────────────────────┘  │    │    │
│   │   └─────────────────────────────────────────────────────┘    │    │
│   └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘

Broader Concept ──────────────────────────────────────────────────────→
More Specific & Advanced ───────────────────────────────────────────→
```

**The rule:** As you go down, systems become **more powerful, capable, and independent** — but also **more complex to build and control**.

---

### ⭐ Layer 1: Artificial Intelligence (AI)

**Definition:** The ability of machines to simulate human intelligence and perform tasks that normally require human intelligence.

**Backend Analogy:** AI is like the entire "software engineering" field. Saying "I do AI" is like saying "I do software" — it's the umbrella. Just like software includes web dev, mobile, backend, frontend... AI includes ML, DL, GenAI, and more.

**Real Examples:**
- Chatbots that understand customer complaints
- Self-driving cars that "see" the road
- Chess engines that beat grandmasters
- Your spam filter that learned what "spam" means

**As a backend dev, think of it as:** The entire ecosystem you're about to enter.

---

### ⭐ Layer 2: Machine Learning (ML)

**Definition:** A subset of AI where machines learn patterns from data and improve automatically without being explicitly programmed.

**Backend Analogy:** This is like the difference between:
- **Hard-coded rules:** `if email.contains("free money") → spam` (you write every rule)
- **ML:** You show the system 10,000 labeled emails (spam/not spam). It **learns** the patterns itself. It might discover that "congratulations you've won" is spammy even though you never told it.

**The key shift:**
```
Traditional Backend:    IF this → THEN that    (you code the logic)
ML Backend:            SHOW examples → SYSTEM finds the logic
```

**Real Examples:**
- Netflix recommendations ("Because you watched X")
- Fraud detection in banking (learns YOUR spending patterns)
- Email spam detection
- Price prediction on Uber (surge pricing)

**Backend Dev Connection:** You already use ML indirectly — that "smart" search on your e-commerce site? That's probably ML ranking results, not just SQL `LIKE` queries.

---

### ⭐ Layer 3: Deep Learning (DL)

**Definition:** A subset of ML that uses neural networks with many layers to learn complex patterns from large amounts of data.

**Backend Analogy:** 

Think of a traditional ML model as a **single SQL query** — it looks at data in one way and gives an answer.

Deep Learning is like a **microservices architecture with 100+ layers**:
- Layer 1 detects edges in an image
- Layer 10 detects shapes
- Layer 50 detects faces
- Layer 100 recognizes "this is a cat"

Each layer passes its output to the next, building increasingly complex understanding. The "deep" in deep learning = **many layers**.

```
Input Image
    ↓
[Layer 1: Edge Detection]     → "I see lines"
    ↓
[Layer 10: Shape Detection]  → "I see circles and triangles"
    ↓
[Layer 50: Feature Detection] → "I see ears, whiskers, fur"
    ↓
[Layer 100: Classification]   → "This is a cat (99.8% confidence)"
    ↓
Output: "cat"
```

**Real Examples:**
- Image recognition (Google Photos finding your dog)
- Speech recognition (Siri understanding your accent)
- Language translation (Google Translate)
- Medical image analysis (detecting tumors in X-rays)

**Backend Dev Connection:** You don't need to build neural networks from scratch (just like you don't build Linux from scratch). You USE them via APIs (OpenAI, HuggingFace) or frameworks (PyTorch, TensorFlow).

---

### ⭐ Layer 4: Generative AI (GenAI)

**Definition:** A type of AI that can generate new content (text, image, code, audio, video, etc.) based on what it has learned.

**Backend Analogy:** 

This is the biggest shift. Traditional software is **retrieval-based**:
```
User asks: "What are your business hours?"
Backend: SELECT hours FROM business_info WHERE id=1
Backend: Returns stored data
```

GenAI is **generation-based**:
```
User asks: "Write a professional email apologizing for a delayed shipment"
GenAI: Generates a NEVER-BEFORE-SEEN email that fits the request
```

It's not fetching from a database. It's **creating** something new, like a human writer would.

**Analogy:** Traditional backend = librarian (finds existing books). GenAI = author (writes new books).

**Real Examples:**
- ChatGPT writing essays, code, poems
- DALL·E generating images from text descriptions
- GitHub Copilot completing your code
- Midjourney creating art
- ElevenLabs cloning voices

**Backend Dev Connection:** This is where YOUR new job lives. You're not replacing your backend — you're adding a **generative layer** on top:
```
User Request
    ↓
[Your Backend API]  ←── You already build this
    ↓
[GenAI Layer]       ←── This is what you're learning (prompt → generate)
    ↓
[Your Backend Logic] ←── Post-process, store, validate
    ↓
Response to User
```

---

### ⭐ Layer 5: Agentic AI

**Definition:** AI systems that can think, plan, take actions using tools, and achieve goals with minimal human intervention.

**Backend Analogy:**

This is where AI becomes a **senior backend engineer** instead of just a function:

```
Traditional API Endpoint:
  GET /weather?city=Delhi
  → Calls weather API
  → Returns JSON
  (Fixed pipeline — no decisions)

Agentic AI:
  User: "Plan my weekend trip to Goa"
  → AI THINKS: "I need flights, hotels, weather, activities"
  → AI PLANS: Step 1: Search flights. Step 2: Check weather. Step 3: Find hotels.
  → AI ACTS: Calls flight API, weather API, hotel API
  → AI OBSERVES: "Flights are expensive, weather looks rainy"
  → AI DECIDES: "Suggest alternate dates"
  → Returns complete itinerary
  (Dynamic pipeline — makes decisions)
```

**Analogy:** A traditional API is a vending machine — you press a button, you get a soda. Agentic AI is a personal assistant — you say "I'm thirsty," and they decide whether to get you water, juice, or coffee based on context.

**Real Examples:**
- AutoGPT (autonomous task completion)
- Devin (AI software engineer)
- Customer service bots that can refund, escalate, AND apologize
- Research assistants that search, summarize, and write reports

**Backend Dev Connection:** This is the ENDGAME. You're building systems that:
1. Receive a goal (not just a request)
2. Break it into steps (like microservices orchestration)
3. Call the right tools/APIs (like service mesh)
4. Observe results and adapt (like circuit breakers + retries)
5. Achieve the goal autonomously

---

## 1.3 Quick Comparison Table (Memorize This)

| Concept | Focus | Example | Used For | Backend Parallel |
|---------|-------|---------|----------|------------------|
| **AI** | Simulate human intelligence | Chatbots, Self-driving cars | General problem solving | "Software Engineering" |
| **ML** | Learn from data | Spam detection, recommendations | Pattern learning & prediction | Learning from logs/metrics |
| **DL** | Learn using deep neural networks | Image recognition, speech recognition | Complex data (images, audio) | Multi-layer microservices |
| **GenAI** | Generate new content | ChatGPT, DALL·E, GitHub Copilot | Content creation, automation | Dynamic code/content generation |
| **Agentic AI** | Act autonomously to achieve goals | AutoGPT, AI Agents with tools | Task automation, decision making | Self-healing, auto-scaling systems |

---

## 1.4 How Everything Connects (The Backend Dev View)

As a backend developer entering AI, here's how your existing skills map:

```
YOUR CURRENT SKILLS                    YOUR NEW AI SKILLS
─────────────────────                  ─────────────────────
REST APIs              ────────→       LLM APIs (OpenAI, Claude)
PostgreSQL/MySQL       ────────→       Vector Databases (Pinecone, FAISS)
Redis Cache            ────────→       Semantic Cache (embedding-based)
Microservices          ────────→       AI Agents (orchestrated tools)
Message Queues         ────────→       LLM Chains (LangChain)
Load Balancers         ────────→       Router LLMs (route to right tool)
Docker/K8s             ────────→       Model Serving (vLLM, TGI)
Unit Tests             ────────→       LLM Evaluation (RAGAS, TruLens)
```

**The beautiful truth:** You're not starting from zero. You're **upgrading** your backend superpowers.

---

## 1.5 🧪 Hands-On (25 min): Build Your AI Landscape Explorer

Let's write Python code that demonstrates the difference between these layers. No API keys needed — we'll use a simple text-based simulation.

### Step 1: The "Traditional Backend" Approach (Hard-coded Rules)

```python
# ═══════════════════════════════════════════════════════════════
# HOUR 1 — PART A: Traditional Backend (Hard-coded Rules)
# ═══════════════════════════════════════════════════════════════

class TraditionalSupportBot:
    """
    This is how we USED to build chatbots.
    Every response is hard-coded. Zero learning.
    Like a giant switch-case statement.
    """

    def __init__(self):
        self.rules = {
            "refund": "Our refund policy allows returns within 30 days.",
            "hours": "We are open 9 AM to 6 PM, Monday to Friday.",
            "shipping": "Shipping takes 3-5 business days.",
            "password": "Click 'Forgot Password' on the login page.",
        }

    def respond(self, user_query: str) -> str:
        """Keyword matching — the old way."""
        user_query = user_query.lower()

        for keyword, response in self.rules.items():
            if keyword in user_query:
                return response

        return "Sorry, I don't understand. Please contact support@company.com"

# Test it
bot = TraditionalSupportBot()

test_queries = [
    "How do I get a refund?",
    "What are your business hours?",
    "I want my money back",           # ❌ FAILS — "refund" not in query!
    "When do you close?",              # ❌ FAILS — "hours" not in query!
    "Tell me about returns",           # ❌ FAILS — no exact match!
]

print("═" * 60)
print("TRADITIONAL BACKEND BOT (Hard-coded Rules)")
print("═" * 60)
for q in test_queries:
    print(f"\n🙋 User: {q}")
    print(f"🤖 Bot:  {bot.respond(q)}")
```

**Run this and observe:** The bot fails on "I want my money back" because it doesn't know "money back" = "refund." It has ZERO understanding of meaning. This is why keyword-based chatbots feel dumb.

---

### Step 2: The "ML" Approach (Pattern Learning)

```python
# ═══════════════════════════════════════════════════════════════
# HOUR 1 — PART B: ML Approach (Learning from Examples)
# ═══════════════════════════════════════════════════════════════

class MLSupportBot:
    """
    This simulates what ML does: learn from labeled examples.
    Instead of rules, we have TRAINING DATA.
    """

    def __init__(self):
        # Training data: (example_query, intent_label)
        self.training_data = [
            ("How do I get a refund?", "refund"),
            ("I want my money back", "refund"),
            ("Can I return this item?", "refund"),
            ("What are your business hours?", "hours"),
            ("When do you open?", "hours"),
            ("What time do you close?", "hours"),
            ("How long does shipping take?", "shipping"),
            ("When will my package arrive?", "shipping"),
            ("I forgot my password", "password"),
            ("How do I reset my password?", "password"),
        ]

        # Learn word frequencies per intent (Naive Bayes idea)
        self.intent_words = {"refund": {}, "hours": {}, "shipping": {}, "password": {}}
        self.intent_counts = {"refund": 0, "hours": 0, "shipping": 0, "password": 0}

        self._train()

    def _train(self):
        """Learn patterns from training data."""
        for query, intent in self.training_data:
            self.intent_counts[intent] += 1
            words = query.lower().split()
            for word in words:
                if word not in ["i", "my", "do", "a", "the", "how", "can", "this", "will"]:
                    self.intent_words[intent][word] = self.intent_words[intent].get(word, 0) + 1

    def respond(self, user_query: str) -> str:
        """Predict intent based on learned patterns."""
        words = user_query.lower().split()
        scores = {intent: 0 for intent in self.intent_words}

        for word in words:
            for intent, word_counts in self.intent_words.items():
                scores[intent] += word_counts.get(word, 0)

        # Normalize by intent frequency
        for intent in scores:
            scores[intent] /= max(self.intent_counts[intent], 1)

        best_intent = max(scores, key=scores.get)

        responses = {
            "refund": "Our refund policy allows returns within 30 days.",
            "hours": "We are open 9 AM to 6 PM, Monday to Friday.",
            "shipping": "Shipping takes 3-5 business days.",
            "password": "Click 'Forgot Password' on the login page.",
        }

        return f"[Predicted intent: {best_intent}] {responses[best_intent]}"

# Test it
ml_bot = MLSupportBot()

print("\n" + "═" * 60)
print("ML BOT (Learns from Examples)")
print("═" * 60)
for q in test_queries:
    print(f"\n🙋 User: {q}")
    print(f"🤖 Bot:  {ml_bot.respond(q)}")
```

**Run this and observe:** Now "I want my money back" is correctly classified as "refund" because the bot LEARNED that "money" and "back" appear in refund examples. It's not perfect, but it's learning!

**Analogy:** Traditional = writing 1000 if-else statements. ML = showing 1000 examples and letting the system figure out the rules.

---

### Step 3: The "GenAI" Approach (Understanding & Generating)

```python
# ═══════════════════════════════════════════════════════════════
# HOUR 1 — PART C: GenAI Approach (Simulated)
# ═══════════════════════════════════════════════════════════════

class GenAISupportBot:
    """
    This SIMULATES what GenAI does.
    In reality, this would call GPT-4/Claude. Here we simulate
    the BEHAVIOR to understand the difference.

    Key difference: It UNDERSTANDS meaning and GENERATES responses.
    """

    def __init__(self):
        # GenAI has "read" the internet during training
        # It has learned patterns, not rules
        self.knowledge_base = """
        Our company policies:
        - Refunds: Allowed within 30 days with original receipt.
          Process: Log in → Orders → Request Return → Print label.
        - Hours: Mon-Fri 9AM-6PM IST. Closed weekends.
        - Shipping: Standard 3-5 days. Express 1-2 days (extra charge).
        - Password: Use 'Forgot Password' link. Reset link expires in 1 hour.
        """

    def respond(self, user_query: str) -> str:
        """
        SIMULATED GenAI behavior:
        1. Understand the query (not just keyword match)
        2. Retrieve relevant knowledge (RAG-like)
        3. Generate a natural, contextual response
        """
        query_lower = user_query.lower()

        # Simulated "understanding" (in reality, LLM does this)
        if any(word in query_lower for word in ["refund", "money back", "return", "money"]):
            return self._generate_refund_response(user_query)
        elif any(word in query_lower for word in ["hours", "open", "close", "time"]):
            return self._generate_hours_response(user_query)
        elif any(word in query_lower for word in ["shipping", "package", "delivery", "arrive"]):
            return self._generate_shipping_response(user_query)
        elif any(word in query_lower for word in ["password", "login", "forgot"]):
            return self._generate_password_response(user_query)
        else:
            return self._generate_fallback_response(user_query)

    def _generate_refund_response(self, query):
        # GenAI generates NATURAL, CONTEXTUAL responses
        if "how" in query.lower():
            return "You can request a refund by logging into your account, going to Orders, and clicking 'Request Return.' You'll need your original receipt, and the item must be returned within 30 days. Would you like me to walk you through it step by step?"
        elif "long" in query.lower() or "time" in query.lower():
            return "Refunds are processed within 5-7 business days after we receive your returned item. You'll get an email confirmation once it's initiated."
        else:
            return "Our refund policy allows returns within 30 days of purchase with the original receipt. Is there a specific order you'd like to return?"

    def _generate_hours_response(self, query):
        if "weekend" in query.lower():
            return "We're closed on weekends, but we're open Monday through Friday from 9 AM to 6 PM IST. You can always reach us via email at support@company.com outside these hours!"
        return "We're open Monday to Friday, 9 AM to 6 PM IST. How can I help you today?"

    def _generate_shipping_response(self, query):
        return "Standard shipping takes 3-5 business days. We also offer express delivery (1-2 days) for an additional charge. Would you like to upgrade your shipping?"

    def _generate_password_response(self, query):
        return "No worries! Click 'Forgot Password' on the login page, enter your email, and we'll send you a reset link. The link expires in 1 hour for security. Let me know if you don't receive it!"

    def _generate_fallback_response(self, query):
        return f"I understand you're asking about '{query}'. Let me connect you with a human agent who can help you better. In the meantime, is there anything else I can assist with?"

# Test it
genai_bot = GenAISupportBot()

extended_queries = [
    "How do I get a refund?",
    "I want my money back",
    "When do you close?",
    "What are your weekend hours?",
    "My package hasn't arrived yet",
    "I forgot my password help",
    "Can you write a poem about our product?",  # GenAI can do this!
]

print("\n" + "═" * 60)
print("GENAI BOT (Understands Meaning + Generates Responses)")
print("═" * 60)
for q in extended_queries:
    print(f"\n🙋 User: {q}")
    print(f"🤖 Bot:  {genai_bot.respond(q)}")
```

**Run this and observe:** Notice how the GenAI bot:
1. Understands "money back" = "refund" (no hard-coded rule needed)
2. Adapts its answer based on context ("how" vs "how long")
3. Can handle questions it was never explicitly programmed for
4. Sounds NATURAL, not robotic

**Analogy:** Traditional bot = a phone menu ("Press 1 for refunds"). GenAI bot = a human customer service rep who actually listens and thinks.

---

### Step 4: Compare All Three Side-by-Side

```python
# ═══════════════════════════════════════════════════════════════
# HOUR 1 — PART D: Side-by-Side Comparison
# ═══════════════════════════════════════════════════════════════

print("\n" + "═" * 80)
print("SIDE-BY-SIDE COMPARISON")
print("═" * 80)

comparison_queries = [
    "I want my money back",
    "What are your weekend hours?",
    "My package is late",
]

for q in comparison_queries:
    print(f"\n{'─' * 80}")
    print(f"🙋 USER: "{q}"")
    print(f"{'─' * 80}")
    print(f"🔧 TRADITIONAL: {bot.respond(q)}")
    print(f"🎓 ML:          {ml_bot.respond(q)}")
    print(f"🧠 GENAI:       {genai_bot.respond(q)}")

print("\n" + "═" * 80)
print("KEY TAKEAWAY")
print("═" * 80)
print("""
┌─────────────────────────────────────────────────────────────────────┐
│  Traditional  →  ML  →  DL  →  GenAI  →  Agentic AI                 │
│                                                                     │
│  Rules        →  Patterns →  Complex  →  Generate  →  Decide      │
│  (dumb)         (smarter)   patterns    (creative)    + Act       │
│                                                                     │
│  As a backend dev, you're moving from COLUMN 1 to COLUMN 5.       │
│  You still build APIs — but now they THINK, GENERATE, and ACT.      │
└─────────────────────────────────────────────────────────────────────┘
""")
```

---

## 1.6 🎤 Interview Drill (10 min)

These questions WILL come up in AI developer interviews. Practice answering them out loud.

### Q1: "Explain the difference between AI, ML, DL, and GenAI."

**Your answer (30 seconds):**
> "AI is the umbrella — any machine simulating human intelligence. ML is a subset where machines learn patterns from data instead of following hard-coded rules. DL goes deeper — it uses neural networks with many layers to learn complex patterns from massive data, like images or speech. GenAI is the newest wave — it doesn't just classify or predict, it CREATES new content like text, images, or code. As a backend developer, I see this as moving from retrieval-based systems to generation-based systems."

### Q2: "Why does a backend developer need to understand GenAI?"

**Your answer:**
> "Because the nature of backend systems is changing. We used to build APIs that fetch and transform data. Now we're building APIs that understand natural language, generate content, and make decisions. A modern backend might need to: embed user queries into vector spaces, retrieve from vector databases, orchestrate LLM calls via LangChain, and cache semantic responses. These are backend problems — just with AI components."

### Q3: "What's the difference between a traditional chatbot and a GenAI chatbot?"

**Your answer:**
> "Traditional chatbots use keyword matching or intent classification — they recognize patterns and return pre-written responses. GenAI chatbots use large language models that understand context, generate novel responses, and can handle questions they were never explicitly trained on. It's the difference between a phone menu and a human conversation."

### Q4: "Where does Agentic AI fit in?"

**Your answer:**
> "Agentic AI is the evolution from 'answer questions' to 'achieve goals.' An AI agent doesn't just respond — it plans steps, uses tools like APIs and databases, observes results, and loops until the goal is achieved. For a backend developer, this is like building a microservice that doesn't just handle one request, but orchestrates an entire workflow autonomously."

### Q5: "Give me a real-world example where you'd use each layer."

**Your answer:**
> "Sure:
> - **AI:** A smart home system that adjusts temperature, lighting, and security.
> - **ML:** A fraud detection system that learns normal vs. suspicious transaction patterns.
> - **DL:** A medical imaging system that detects tumors in X-rays using deep neural networks.
> - **GenAI:** A content generation tool that writes marketing copy or code based on prompts.
> - **Agentic AI:** A travel planning bot that searches flights, checks weather, books hotels, and handles cancellations — all autonomously."

---

## 📚 Hour 1 Cheat Sheet

```
┌────────────────────────────────────────────────────────────────┐
│  AI LANDSCAPE — THE 5 LAYERS                                   │
├────────────────────────────────────────────────────────────────┤
│  AI          → Umbrella. Any smart machine behavior.           │
│  ML          → Learns patterns from data.                    │
│  DL          → Many-layered neural networks. Complex patterns. │
│  GenAI       → CREATES new content. The creative layer.      │
│  Agentic AI  → Plans, acts, uses tools. The autonomous layer.  │
├────────────────────────────────────────────────────────────────┤
│  BACKEND DEV MAPPING:                                          │
│  REST API    → LLM API                                         │
│  SQL DB      → Vector DB                                       │
│  Redis       → Semantic Cache                                  │
│  Microservices → AI Agents                                     │
│  API Gateway → Router LLM                                      │
│  Docker/K8s  → Model Serving                                   │
└────────────────────────────────────────────────────────────────┘
```

---

## 🎯 What to Do Before Hour 2

1. **Run the code above** in a Jupyter notebook or Python file.
2. **Add 3 more queries** to each bot and observe the differences.
3. **Write down:** Which layer excites you most? (RAG? Agents? GenAI?)
4. **Read ahead:** Skim the PDF pages for Hour 2 (How LLMs Work).

---

> 🟨 **Remember:** You're not learning a new career. You're **evolving** your backend career. Every concept in AI has a backend parallel you already understand.

---

*Next: Hour 2 — How LLMs Work: Tokens, Transformers, and Next-Token Prediction*
